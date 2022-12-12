# Using Amazon MWAA with Amazon EKS<a name="mwaa-eks-example"></a>

The following sample demonstrates how to use Amazon Managed Workflows for Apache Airflow \(MWAA\) with Amazon EKS\.

**Topics**
+ [Version](#mwaa-eks-example-version)
+ [Prerequisites](#eksctl-prereqs)
+ [Create a public key for Amazon EC2](#eksctl-create-key)
+ [Create the cluster](#create-cluster-eksctl)
+ [Create a `mwaa` namespace](#eksctl-namespace)
+ [Create a role for the `mwaa` namespace](#eksctl-role)
+ [Create and attach an IAM role for the Amazon EKS cluster](#eksctl-iam-role)
+ [Create the requirements\.txt file](#eksctl-requirements)
+ [Create an identity mapping for Amazon EKS](#eksctl-identity-map)
+ [Create the `kubeconfig`](#eksctl-kube-config)
+ [Create a DAG](#eksctl-create-dag)
+ [Add the DAG and `kube_config.yaml` to the Amazon S3 bucket](#eksctl-dag-bucket)
+ [Enable and trigger the example](#eksctl-trigger-pod)

## Version<a name="mwaa-eks-example-version"></a>
+ The sample code on this page can be used with **Apache Airflow v1** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.
+ You can use the code example on this page with **Apache Airflow v2 and above** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="eksctl-prereqs"></a>

To use the example in this topic, you'll need the following:
+ An [Amazon MWAA environment](get-started.md)\.
+ eksctl\. To learn more, see [Install eksctl](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html#install-eksctl)\.
+ kubectl\. To learn more, see [Install and Set Up kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)\. In some case this is installed with eksctl\.
+ An EC2 key pair in the Region where you create your Amazon MWAA environment\. To learn more, see [Creating or importing a key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#prepare-key-pair)\.

**Note**  
When you use an `eksctl` command, you can include a `--profile` to specify a profile other than the default\.

## Create a public key for Amazon EC2<a name="eksctl-create-key"></a>

Use the following command to create a public key from your private key pair\.

```
ssh-keygen -y -f myprivatekey.pem > mypublickey.pub
```

To learn more, see [Retrieving the public key for your key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#retrieving-the-public-key)\.

## Create the cluster<a name="create-cluster-eksctl"></a>

Use the following command to create the cluster\. If you want a custom name for the cluster or to create it in a different Region, replace the name and Region values\. You must create the cluster in the same Region where you create the Amazon MWAA environment\. Replace the values for the subnets to match the subnets in your Amazon VPC network that you use for Amazon MWAA\. Replace the value for the `ssh-public-key` to match the key you use\. You can use an existing key from Amazon EC2 that is in the same Region, or create a new key in the same Region where you create your Amazon MWAA environment\.

```
eksctl create cluster \
--name mwaa-eks \
--region us-west-2 \
--version 1.18 \
--nodegroup-name linux-nodes \
--nodes 3 \
--nodes-min 1 \
--nodes-max 4 \
--with-oidc \
--ssh-access \
--ssh-public-key MyPublicKey \
--managed \
--vpc-public-subnets "subnet-11111111111111111, subnet-2222222222222222222" \
--vpc-private-subnets "subnet-33333333333333333, subnet-44444444444444444"
```

It takes some time to complete creating the cluster\. Once complete, you can verify that the cluster was created successfully and has the IAM OIDC Provider configured by using the following command:

```
eksctl utils associate-iam-oidc-provider \
--region us-west-2 \
--cluster mwaa-eks \
--approve
```

## Create a `mwaa` namespace<a name="eksctl-namespace"></a>

After confirming that the cluster was successfully created, use the following command to create a namespace for the pods\.

```
kubectl create namespace mwaa
```

## Create a role for the `mwaa` namespace<a name="eksctl-role"></a>

After you create the namespace, create a role and role\-binding for an Amazon MWAA user on EKS that can run pods in a the MWAA namespace\. If you used a different name for the namespace, replace mwaa in `-n mwaa` with the name that you used\.

```
cat << EOF | kubectl apply -f - -n mwaa
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: mwaa-role
rules:
  - apiGroups:
      - ""
      - "apps"
      - "batch"
      - "extensions"
    resources:      
      - "jobs"
      - "pods"
      - "pods/attach"
      - "pods/exec"
      - "pods/log"
      - "pods/portforward"
      - "secrets"
      - "services"
    verbs:
      - "create"
      - "delete"
      - "describe"
      - "get"
      - "list"
      - "patch"
      - "update"
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: mwaa-role-binding
subjects:
- kind: User
  name: mwaa-service
roleRef:
  kind: Role
  name: mwaa-role
  apiGroup: rbac.authorization.k8s.io
EOF
```

Confirm that the new role can access the Amazon EKS cluster by running the following command\. Be sure to use the correct name if you did not use *mwaa*:

```
kubectl get pods -n mwaa --as mwaa-service
```

You should see a message returned that says:

```
No resources found in mwaa namespace.
```

## Create and attach an IAM role for the Amazon EKS cluster<a name="eksctl-iam-role"></a>

You must create an IAM role and then bind it to the Amazon EKS \(k8s\) cluster so that it can be used for authentication through IAM\. The role is used only to log in to the cluster, and does not have any permissions for the console or API calls\.

Create a new role for the Amazon MWAA environment using the steps in [Amazon MWAA execution role](mwaa-create-role.md)\. However, instead of creating and attaching the policies described in that topic, attach the following policy:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "airflow:PublishMetrics",
            "Resource": "arn:aws:airflow:${MWAA_REGION}:${ACCOUNT_NUMBER}:environment/${MWAA_ENV_NAME}"
        },
        {
            "Effect": "Deny",
            "Action": "s3:ListAllMyBuckets",
            "Resource": [
                "arn:aws:s3:::{MWAA_S3_BUCKET}",
                "arn:aws:s3:::{MWAA_S3_BUCKET}/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject*",
                "s3:GetBucket*",
                "s3:List*"
            ],
            "Resource": [
                "arn:aws:s3:::{MWAA_S3_BUCKET}",
                "arn:aws:s3:::{MWAA_S3_BUCKET}/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "logs:CreateLogGroup",
                "logs:PutLogEvents",
                "logs:GetLogEvents",
                "logs:GetLogRecord",
                "logs:GetLogGroupFields",
                "logs:GetQueryResults",
                "logs:DescribeLogGroups"
            ],
            "Resource": [
                "arn:aws:logs:${MWAA_REGION}:${ACCOUNT_NUMBER}:log-group:airflow-${MWAA_ENV_NAME}-*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": "cloudwatch:PutMetricData",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "sqs:ChangeMessageVisibility",
                "sqs:DeleteMessage",
                "sqs:GetQueueAttributes",
                "sqs:GetQueueUrl",
                "sqs:ReceiveMessage",
                "sqs:SendMessage"
            ],
            "Resource": "arn:aws:sqs:${MWAA_REGION}:*:airflow-celery-*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "kms:Decrypt",
                "kms:DescribeKey",
                "kms:GenerateDataKey*",
                "kms:Encrypt"
            ],
            "NotResource": "arn:aws:kms:*:${ACCOUNT_NUMBER}:key/*",
            "Condition": {
                "StringLike": {
                    "kms:ViaService": [
                        "sqs.${MWAA_REGION}.amazonaws.com"
                    ]
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "eks:DescribeCluster"
            ],
            "Resource": "arn:aws:eks:${MWAA_REGION}:${ACCOUNT_NUMBER}:cluster/${EKS_CLUSTER_NAME}"
        }     
    ]
}
```

After you create role, edit your Amazon MWAA environment to use the role you created as the execution role for the environment\. To change the role, edit the environment to use\. You select the execution role under **Permissions**\.

**Known issues:**
+ There is a known issue with role ARNs with subpaths not being able to authenticate with Amazon EKS\. The workaround for this is to create the service role manually rather than using the one created by Amazon MWAA itself\. To learn more, see [Roles with paths do not work when the path is included in their ARN in the aws\-auth configmap](https://github.com/kubernetes-sigs/aws-iam-authenticator/issues/268)
+ If Amazon MWAA service listing is not available in IAM you need to choose an alternate service policy, such as Amazon EC2, and then update the role’s trust policy to match the following:

  ```
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {
          "Service": [
            "airflow-env.amazonaws.com",
            "airflow.amazonaws.com"
          ]
        },
        "Action": "sts:AssumeRole"
      }
    ]
  }
  ```

  To learn more, see [How to use trust policies with IAM roles](https://aws.amazon.com/blogs/security/how-to-use-trust-policies-with-iam-roles/)\.

## Create the requirements\.txt file<a name="eksctl-requirements"></a>

To use the sample code in this section, ensure you've added one of the following database options to your `requirements.txt`\. To learn more, see [Installing Python dependencies](working-dags-dependencies.md)\.

------
#### [ Apache Airflow v2 ]

```
kubernetes
apache-airflow[cncf.kubernetes]==3.0.0
```

------
#### [ Apache Airflow v1 ]

```
awscli
kubernetes==12.0.1
```

------

## Create an identity mapping for Amazon EKS<a name="eksctl-identity-map"></a>

Use the ARN for the role you created in the following command to create an identity mapping for Amazon EKS\. Change the Region *your\-region* to the Region where you created the environment\. Replace the ARN for the role, and finally, replace *mwaa\-execution\-role* with your environment's execution role\.

```
eksctl create iamidentitymapping \
--region your-region \
--cluster mwaa-eks \
--arn arn:aws:iam::111222333444:role/mwaa-execution-role \
--username mwaa-service
```

## Create the `kubeconfig`<a name="eksctl-kube-config"></a>

Use the following command to create the `kubeconfig`:

```
aws eks update-kubeconfig \
--region us-west-2 \
--kubeconfig ./kube_config.yaml \
--name mwaa-eks \
--alias aws
```

If you used a specific profile when you ran `update-kubeconfig` you need to remove the `env:` section added to the kube\_config\.yaml file so that it works correctly with Amazon MWAA\. To do so, delete the following from the file and then save it:

```
env:
- name: AWS_PROFILE
  value: profile_name
```

## Create a DAG<a name="eksctl-create-dag"></a>

Use the following code example to create a Python file, such as `mwaa_pod_example.py` for the DAG\.

------
#### [ Apache Airflow v2 ]

```
"""
Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.
Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
the Software, and to permit persons to whom the Software is furnished to do so.
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
"""
from airflow import DAG
from datetime import datetime
from airflow.providers.cncf.kubernetes.operators.kubernetes_pod import KubernetesPodOperator

default_args = {
   'owner': 'aws',
   'depends_on_past': False,
   'start_date': datetime(2019, 2, 20),
   'provide_context': True
}

dag = DAG(
   'kubernetes_pod_example', default_args=default_args, schedule_interval=None)

#use a kube_config stored in s3 dags folder for now
kube_config_path = '/usr/local/airflow/dags/kube_config.yaml'

podRun = KubernetesPodOperator(
                       namespace="mwaa",
                       image="ubuntu:18.04",
                       cmds=["bash"],
                       arguments=["-c", "ls"],
                       labels={"foo": "bar"},
                       name="mwaa-pod-test",
                       task_id="pod-task",
                       get_logs=True,
                       dag=dag,
                       is_delete_operator_pod=False,
                       config_file=kube_config_path,
                       in_cluster=False,
                       cluster_context='aws'
                       )
```

------
#### [ Apache Airflow v1 ]

```
"""
Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.
Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
the Software, and to permit persons to whom the Software is furnished to do so.
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
"""
from airflow import DAG
from datetime import datetime
from airflow.contrib.operators.kubernetes_pod_operator import KubernetesPodOperator

default_args = {
   'owner': 'aws',
   'depends_on_past': False,
   'start_date': datetime(2019, 2, 20),
   'provide_context': True
}

dag = DAG(
   'kubernetes_pod_example', default_args=default_args, schedule_interval=None)

#use a kube_config stored in s3 dags folder for now
kube_config_path = '/usr/local/airflow/dags/kube_config.yaml'

podRun = KubernetesPodOperator(
                       namespace="mwaa",
                       image="ubuntu:18.04",
                       cmds=["bash"],
                       arguments=["-c", "ls"],
                       labels={"foo": "bar"},
                       name="mwaa-pod-test",
                       task_id="pod-task",
                       get_logs=True,
                       dag=dag,
                       is_delete_operator_pod=False,
                       config_file=kube_config_path,
                       in_cluster=False,
                       cluster_context='aws'
                       )
```

------

## Add the DAG and `kube_config.yaml` to the Amazon S3 bucket<a name="eksctl-dag-bucket"></a>

Put the DAG you created and the `kube_config.yaml` file into the Amazon S3 bucket for the Amazon MWAA environment\. You can put files into your bucket using either the Amazon S3 console or the AWS Command Line Interface\.

## Enable and trigger the example<a name="eksctl-trigger-pod"></a>

In Apache Airflow, enable the example and then trigger it\.

After it runs and completes successfully, use the following command to verify the pod:

```
kubectl get pods -n mwaa
```

You should see output similar to the following:

```
NAME READY STATUS RESTARTS AGE
mwaa-pod-test-aa11bb22cc3344445555666677778888 0/1 Completed 0 2m23s
```

You can then verify the output of the pod with the following command\. Replace the name value with the value returned from the previous command:

```
kubectl logs -n mwaa mwaa-pod-test-aa11bb22cc3344445555666677778888
```