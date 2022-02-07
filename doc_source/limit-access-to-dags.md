# Tutorial: Restricting an Amazon MWAA user's access to a subset of DAGs<a name="limit-access-to-dags"></a>

 Amazon MWAA manages access to your environment by mapping your IAM principals to one or more of Apache Airflow's [default roles](https://airflow.apache.org/docs/apache-airflow/stable/security/access-control.html#default-roles)\. The following tutorial shows how you can restrict individual Amazon MWAA users to only view and interact with a specific DAG or a set of DAGs\. 

**Note**  
 The steps in this tutorial can be completed using federated access, as long as the IAM roles can be assumed\. 

**Topics**
+ [Prerequisites](#limit-access-to-dags-prerequisites)
+ [Step one: Provide Amazon MWAA web server access to your IAM user with the default `Public` Apache Airflow role\.](#limit-access-to-dags-apply-public-access)
+ [Step two: Create a new Apache Airflow custom role](#limit-access-to-dags-create-new-airflow-role)
+ [Step three: Assign the role you created to your Amazon MWAA user](#limit-access-to-dags-assign-role)
+ [Next steps](#limit-access-to-dags-next-up)
+ [Related resources](#limit-access-to-dags-related-resources)

## Prerequisites<a name="limit-access-to-dags-prerequisites"></a>

 To complete the steps in this this tutorial, you'll need the following: 
+  An [Amazon MWAA environment with multiple DAGs](get-started.md) 
+  An IAM principal, `Admin` with [AdministratorAccess](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/AdministratorAccess$jsonEditor) permissions, and an IAM user, `MWAAUser`, as the principal for which you can limit DAG access\. For more information about admin users, see [Administrator job function](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_job-functions.html#jf_administrator) in the *IAM User Guide* 
+  [AWS Command Line Interface version 2](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install) installed\. 

## Step one: Provide Amazon MWAA web server access to your IAM user with the default `Public` Apache Airflow role\.<a name="limit-access-to-dags-apply-public-access"></a>

**To grant permission using the AWS Management Console**

1.  Sign in to your AWS account as an `Admin` and open the [IAM console](https://console.aws.amazon.com/iam/)\. 

1.  In the left navigation pane, choose **Users**, then choose your Amazon MWAA IAM user from the users table\. 

1.  On the user details page, under **Summary**, choose the **Permissions** tab, then choose **Permissions policies** to expand the card and choose **Add permissions**\. 

1.  In the **Grant permissions** section, choose **Attach existing policies directly**, then choose **Create policy** to create and attach your own custom permissions policy\. 

1.  On the **Create policy** page, choose **JSON**, then copy and paste the following JSON permissions policy in the policy editor\. Tha policy grants web server access to the user with the default `Public` Apache Airflow role\. 

   ```
   {
       "Version": "2012-10-17",
       "Statement": [
           {
           "Effect": "Allow",
           "Action": "airflow:CreateWebLoginToken",
           "Resource": [
               "arn:aws:airflow:YOUR_REGION:YOUR_ACCOUNT_ID:role/YOUR_ENVIRONMENT_NAME/Public"
               ]
           }
       ]
    }
   ```

## Step two: Create a new Apache Airflow custom role<a name="limit-access-to-dags-create-new-airflow-role"></a>

**To create a new role using the Apache Airflow UI**

1.  With your administrator IAM user, open the [Amazon MWAA console](https://console.aws.amazon.com/mwaa/home) and launch your environment's Apache Airflow UI\. 

1.  From the navigation pane at the top, hover on **Security** to open the dropdown list, then choose **List Roles** to view the default Apache Airflow roles\. 

1.  From the roles list, select **User**, then at the top of the page choose **Actions** to open the dropdown\. Choose **Copy Role**, and confirm **Ok** 
**Note**  
 Copy the **Ops** or **Viewer** roles to grant more or less access, respectively\. 

1.  Locate the new role you created in the table and choose **Edit record**\. 

1.  On the **Edit Role** page, do the following: 
   +  For **Name**, type a new name for the role in the text field\. For example, **Restricted**\. 
   + For the list of **Perimssions**, remove `can read on DAGs` and `can edit on DAGs`, then add read and write permissions for the set of DAGs you want to provide access to\. For example, for a DAG, `example_dag.py`, add **`can read on DAG:example_dag`** and **`can edit on DAG:example_dag`**\.

    Choose **Save**\. You should now have a new role that limits access to a subset of DAGs available in your Amazon MWAA environment\. You can now assign this role to any existing Apache Airflow users\. 

## Step three: Assign the role you created to your Amazon MWAA user<a name="limit-access-to-dags-assign-role"></a>

**To assign the new role**

1.  Using access credentials for `MWAAUser`, run the following CLI command to retrieve your environment's web server URL\. 

   ```
   $ aws mwaa get-environment --name YOUR_ENVIRONMENT_NAME | jq '.Environment.WebserverUrl'
   ```

   If successful, you'll see the following output:

   ```
   "ab1b2345-678a-90a1-a2aa-34a567a8a901.c13.us-west-2.airflow.amazonaws.com"
   ```

1.  With `MWAAUser` signed in to the AWS Management Console, open a new browser window and access the following URl\. Replace `Webserver-URL` with your information\. 

   ```
   https://<Webserver-URL>/home
   ```

    If successful, you'll see a `Forbidden` error page because `MWAAUser` has not been granted permission to access the Apache Airflow UI yet\. 

1.  With `Admin` signed in to the AWS Management Console, open the Amazon MWAA console again and launch your environment's Apache Airflow UI\. 

1.  From the UI dashboard, expand the **Security** dropdown, and this time choose **List Users**\. 

1.  In the users table, find the new Apache Airflow user and choose **Edit record**\. The user's first name will match your IAM user name in the following pattern: `user/mwaa-user`\. 

1.  On the **Edit User** page, in the **Role** section, add the new custom role you created, then choose **Save**\. 
**Note**  
 The **Last Name** field is required, but a space satisfies the requirement\. 

    The IAM `Public` principal grants the `MWAAUser` permission to access the Apache Airflow UI, while the new role provides the additional permissions needed to see their DAGs\. 

**Important**  
 Any of the 5 default roles \(such as `Admin`\) not authorized by IAM which are added using the Apache Airflow UI will be removed on next user login\. 

## Next steps<a name="limit-access-to-dags-next-up"></a>
+  To learn more about managing access to your Amazon MWAA environment, and to see sample JSON IAM policies you can use for your environment users, see [Accessing an Amazon MWAA environment](access-policies.md) 

## Related resources<a name="limit-access-to-dags-related-resources"></a>
+ [Access Control](https://airflow.apache.org/docs/apache-airflow/stable/security/access-control.html) \(Apache Airflow Documentation\) – Learn more about the default Apache Airflow roles on the Apache Airflow documentation website\.