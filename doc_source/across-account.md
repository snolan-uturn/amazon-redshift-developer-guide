# Sharing data across AWS accounts<a name="across-account"></a>

You can share data for read purposes across AWS accounts\. Sharing data across AWS accounts work similarly to sharing data within an account\. The difference is that there is a two\-way handshake required in sharing data across AWS accounts\. A producer account administrators can either authorize consumer accounts to access datashares or choose not to authorize any access\. To use an authorized datashare, a consumer account administrator can associate the datashare with the entire AWS account, authorize it with specific clusters in the consumer account, or decline the datashare\. For more information about sharing data within an account, see [Sharing data within an account](within-account.md)\.

A datashare can have data consumers that are either cluster namespaces in the same account or different AWS accounts\. You don't need to create separate datashares for sharing within an account and cross\-account sharing\.

For cross\-account data sharing, both the producer and consumer cluster must be encrypted\.

When sharing data with AWS accounts, producer cluster administrators share with the AWS account as an entity\. A consumer cluster administrator can decide which cluster namespaces in the consumer account get access to a datashare\.

**If you are a producer cluster administrator or database owner** – follow these steps:

1. Create datashares in your cluster and add datashare objects to the datashares\. For more detailed steps on how to create datashares and add datashare objects to datashares, see [Sharing data within an account](within-account.md)\. For information about the CREATE DATASHARE and ALTER DATASHARE, see [CREATE DATASHARE](r_CREATE_DATASHARE.md) and [ALTER DATASHARE](r_ALTER_DATASHARE.md)\.

   The following example adds different datashare objects to the datashare SalesShare:

   ```
   -- Add schema to datashare
   ALTER DATASHARE SalesShare ADD SCHEMA PUBLIC;
   
   -- Add table under schema to datashare
   ALTER DATASHARE SalesShare ADD TABLE public.tickit_sales_redshift;
   
   -- Add view to datashare 
   ALTER DATASHARE SalesShare ADD TABLE public.sales_data_summary_view;
   
   -- Add all existing tables and views under schema to datashare (does not include future table)
   ALTER DATASHARE SalesShare ADD ALL TABLES in schema public;
   ```

   You can also use the Amazon Redshift console to create or edit datashares\. For more information, see [Creating datashares](create-datashare-console.md) and [Editing datashares created in your account](edit-datashare-console.md)\.

1. Delegate permissions to operate on the datashare\. For more information, see [GRANT](r_GRANT.md) or [REVOKE](r_REVOKE.md)\.

   The following example grants permissions to dbuser on SalesShare\.

   ```
   GRANT ALTER, SHARE ON DATASHARE SalesShare TO dbuser;
   ```

   Cluster superusers and the owners of the datashare can grant or revoke modification privileges on the datashare to additional users\.

1. Add consumers to or remove consumers from datashares\. The following example adds the AWS account ID to the SalesShare\. For more information, see [GRANT](r_GRANT.md) or [REVOKE](r_REVOKE.md)\.

   ```
   GRANT USAGE ON DATASHARE SalesShare TO ACCOUNT '123456789012';
   ```

   You can only grant privileges to one data consumer in a GRANT statement\.

   Cluster superusers and the owners of datashare objects or users that have SHARE privilege on the datashare can add consumers to or remove consumers from a datashare\. To do so, they use GRANT USAGE or REVOKE USAGE\.

   You can also use the Amazon Redshift console to add or remove data consumers from datashares\. For more information, see [Adding data consumers to datashares](add-data-consumer-console.md) and [Removing data consumers from datashares](remove-data-consumer-console.md)\.

1. \(Optional\) Revoke access to the datashare from AWS accounts if you don't want to share the data with the consumers anymore\.

   ```
   REVOKE USAGE ON DATASHARE SalesShare FROM ACCOUNT '123456789012';
   ```

**If you are a producer account administrator** – follow these steps:

After granting usage to the AWS account, the datashare status is `pending_authorization`\. The producer account administrator should authorize datashares using the Amazon Redshift console and choose the data consumers\.

Sign in to the [https://console\.aws\.amazon\.com/redshift/](https://console.aws.amazon.com/redshift/) and choose which data consumers to authorize to access datashares or to remove authorization from\. Authorized data consumers receive notifications to take actions on datashares\. If you are adding a cluster namespace as a data consumer, you don't have to perform authorization\. After data consumers are authorized, they can access datashare objects and create a consumer database to query the data\. For more information, see [Authorizing or removing authorization from datashares](authorize-datashare-console.md)\.

**If you are a consumer account administrator** – follow these steps:

To associate one or more datashares that are shared from other accounts to your entire AWS account or specific cluster namespaces in your account, use the Amazon Redshift console\. If you aren't interested in the datashare, then you can also choose to decline the datashare\.

Sign in to the [https://console\.aws\.amazon\.com/redshift/](https://console.aws.amazon.com/redshift/) and associate one or more datashares that are shared from other accounts to your entire AWS account  or specific cluster namespaces in your account\. For more information, see [Associating datashares](accept-datashare-console.md)\.

After the AWS account or specific cluster namespaces are associated, the datashares become available for consumption\. You can also change datashare association at any time\. When changing association from individual cluster namespaces to an AWS account, Amazon Redshift overwrites the cluster namespaces with the AWS account information\. When changing association from an AWS account to specific cluster namespaces, Amazon Redshift overwrites the AWS account information with the cluster namespace information\. All cluster namespaces in the account get access to the data\.

**If you are a consumer cluster administrator** – follow these steps:

1. List the datashares made available to you and view the content of datashares\. The content of datashares is available only when the producer cluster administrator has authorized the datashares and the consumer cluster administrator has accepted and associated the datashares\. For more information, see [DESC DATASHARE](r_DESC_DATASHARE.md) and [SHOW DATASHARES](r_SHOW_DATASHARES.md)\.

   The following example displays the information of inbound datashares of a specified producer namespace\. When you run the DESC DATAHSARE as a consumer cluster administrator, you must specify the NAMESPACE option to view inbound datashares\. 

   ```
   SHOW DATASHARES LIKE 'sales%';
   
   share_name | share_owner | source_database | consumer_database | share_type | createdate | is_publicaccessible | share_acl | producer_account |          producer_namespace
   -----------+-------------+-----------------+-------------------+------------+------------+---------------------+-----------+------------------+---------------------------------------
   salesshare |             |                 |                   | INBOUND    |            |        t            |           | 123456789012    | 'dd8772e1-d792-4fa4-996b-1870577efc0d'
   ```

   ```
   DESC DATASHARE SalesShare OF ACCOUNT '123456789012' NAMESPACE 'dd8772e1-d792-4fa4-996b-1870577efc0d';
   
    producer_account |          producer_namespace          | share_type | share_name | object_type |           object_name           | include_new
   ------------------+--------------------------------------+------------+------------+-------------+---------------------------------+---------------
    123456789012     | dd8772e1-d792-4fa4-996b-1870577efc0d | INBOUND    | salesshare | table       | public.tickit_users_redshift    |
    123456789012     | dd8772e1-d792-4fa4-996b-1870577efc0d | INBOUND    | salesshare | table       | public.tickit_venue_redshift    |
    123456789012     | dd8772e1-d792-4fa4-996b-1870577efc0d | INBOUND    | salesshare | table       | public.tickit_category_redshift |
    123456789012     | dd8772e1-d792-4fa4-996b-1870577efc0d | INBOUND    | salesshare | table       | public.tickit_date_redshift     |
    123456789012     | dd8772e1-d792-4fa4-996b-1870577efc0d | INBOUND    | salesshare | table       | public.tickit_event_redshift    |
    123456789012     | dd8772e1-d792-4fa4-996b-1870577efc0d | INBOUND    | salesshare | table       | public.tickit_listing_redshift  |
    123456789012     | dd8772e1-d792-4fa4-996b-1870577efc0d | INBOUND    | salesshare | table       | public.tickit_sales_redshift    |
    123456789012     | dd8772e1-d792-4fa4-996b-1870577efc0d | INBOUND    | salesshare | schema      | public                          |
   (8 rows)
   ```

   Only cluster superusers can do this\. You can also use SVV\_DATASHARES to view the datashares and SVV\_DATASHARE\_OBJECTS to view the objects within the datashare\.

   The following example displays the inbound datashares in a consumer cluster\.

   ```
   SELECT * FROM SVV_DATASHARES WHERE share_name LIKE 'sales%';
   
   share_name | share_owner | source_database | consumer_database | share_type | createdate | is_publicaccessible | share_acl | producer_account |          producer_namespace
   -----------+-------------+-----------------+-------------------+------------+------------+---------------------+-----------+------------------+---------------------------------------
   salesshare |             |                 |                   | INBOUND    |            |        t            |           | 123456789012     | 'dd8772e1-d792-4fa4-996b-1870577efc0d'
   ```

   ```
   SELECT * FROM SVV_DATASHARE_OBJECTS WHERE share_name LIKE 'sales%';
    share_type | share_name | object_type |           object_name           | producer_account |          producer_namespace          | include_new
   ------------+------------+-------------+---------------------------------+------------------+--------------------------------------+-------------
    INBOUND    | salesshare | table       | public.tickit_users_redshift    | 123456789012     | dd8772e1-d792-4fa4-996b-1870577efc0d |
    INBOUND    | salesshare | table       | public.tickit_venue_redshift    | 123456789012     | dd8772e1-d792-4fa4-996b-1870577efc0d |
    INBOUND    | salesshare | table       | public.tickit_category_redshift | 123456789012     | dd8772e1-d792-4fa4-996b-1870577efc0d |
    INBOUND    | salesshare | table       | public.tickit_date_redshift     | 123456789012     | dd8772e1-d792-4fa4-996b-1870577efc0d |
    INBOUND    | salesshare | table       | public.tickit_event_redshift    | 123456789012     | dd8772e1-d792-4fa4-996b-1870577efc0d |
    INBOUND    | salesshare | table       | public.tickit_listing_redshift  | 123456789012     | dd8772e1-d792-4fa4-996b-1870577efc0d |
    INBOUND    | salesshare | table       | public.tickit_sales_redshift    | 123456789012     | dd8772e1-d792-4fa4-996b-1870577efc0d |
    INBOUND    | salesshare | schema      | public                          | 123456789012     | dd8772e1-d792-4fa4-996b-1870577efc0d |
   (8 rows)
   ```

1. Create local databases that reference to the datashares\. For more information, see [CREATE DATABASE](r_CREATE_DATABASE.md)\.

   ```
   CREATE DATABASE Sales_db FROM DATASHARE SalesShare OF ACCOUNT '123456789012' NAMESPACE 'dd8772e1-d792-4fa4-996b-1870577efc0d';
   ```

   You can see databases that you created from the datashare by querying [SVV\_REDSHIFT\_DATABASES](r_SVV_REDSHIFT_DATABASES.md) view\. You can't connect to these databases created from datashares and they are read\-only\. However, you can connect to a local database on your consumer cluster and perform cross\-database query to query the data from the databases created from datashares\. You can't create a datashare on top of database objects created from an existing datashare\. However, you can copy the data into a separate table on the consumer cluster and perform any processing needed and then share the new objects created\.

1. \(Optional\) Create external schemas to refer and assign granular permissions to specific schemas in the consumer database imported on the consumer cluster\. For more information, see [CREATE EXTERNAL SCHEMA](r_CREATE_EXTERNAL_SCHEMA.md)\.

   ```
   CREATE EXTERNAL SCHEMA Sales_schema FROM REDSHIFT DATABASE 'Sales_db' SCHEMA 'public';
   ```

1. Grant permissions on databases and schema references created from the datashares to users groups in the consumer cluster as needed\. For more information, see [GRANT](r_GRANT.md) or [REVOKE](r_REVOKE.md)\.

   ```
   GRANT USAGE ON DATABASE Sales_db TO Bob;
   ```

   ```
   GRANT USAGE ON SCHEMA Sales_schema TO GROUP Analyst_group;
   ```

   As a consumer cluster administrator, you can only assign permissions on the entire database created from the datashare to your users and groups\. In some cases, you need fine\-grained controls on a subset of database objects created from the datashare\. If so, you can create an external schema reference pointing to specific schemas in the datashare as described in the previous step and provide granular permissions at schema level\. You can also create late binding views on top of shared objects and use these to assign granular permissions\. You can also consider having producer clusters create additional datashares for you with the granularity required\. You can create as many schema references to the database created from the datashare\.

1. Query data in the shared objects in the datashares\.

   Users and groups with permissions on consumer databases and schemas on consumer clusters can explore and navigate the metadata of any shared objects\. They can also explore and navigate local objects in a consumer cluster\. To do this, use JDBC or ODBC drivers or SVV\_ALL and SVV\_REDSHIFT views\.

   Producer clusters might have many schemas in the database, tables, and views within each schema\. The users on the consumer side can see only the subset of objects that are made available through the datashare\. These users can't see the entire metadata from the producer cluster\. This approach helps provide granular metadata security control with data sharing\.

   You continue to connect to local cluster databases, but now you can also read from the databases and schemas that are created from the datashare using the three\-part database\.schema\.table notation\. You can perform queries that span across any and all databases that are visible to you\. These can be local databases on the cluster or databases created from the datashares\. Consumer clusters can't connect to the databases created from the datashares\.

   You can access the data using full qualification\. For more information, see [Examples of using a cross\-database query](cross-database_example.md)\.

   ```
   SELECT * FROM sales_db.public.tickit_sales_redshift;
   ```

   You can only use SELECT statements on shared objects\. However, you can create tables in the consumer cluster by querying the data from the shared objects in a different local database\.

   In addition to queries, consumers can create views on shared objects\. Only late binding views or materialized views are supported\. Amazon Redshift doesn't support regular views on shared data\. Views that consumers create can span across multiple local databases or databases created from datashares\. For more information, see [CREATE VIEW](r_CREATE_VIEW.md)\.

   ```
   // Connect to a local cluster database
                  
   // Create a view on shared objects and access it. 
   CREATE VIEW sales_data 
   AS SELECT * 
   FROM sales_db.public.tickit_sales_redshift 
   WITH NO SCHEMA BINDING;
   
   SELECT * FROM sales_data;
   ```