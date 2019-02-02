# Encrypting Amazon RDS Resources<a name="Overview.Encryption"></a>

You can encrypt your Amazon RDS DB instances and snapshots at rest by enabling the encryption option for your Amazon RDS DB instances\. Data that is encrypted at rest includes the underlying storage for a DB instances, its automated backups, Read Replicas, and snapshots\.

Amazon RDS encrypted DB instances use the industry standard AES\-256 encryption algorithm to encrypt your data on the server that hosts your Amazon RDS DB instances\. Once your data is encrypted, Amazon RDS handles authentication of access and decryption of your data transparently with a minimal impact on performance\. You don't need to modify your database client applications to use encryption\. 

**Topics**
+ [Overview of Encrypting Amazon RDS Resources](#Overview.Encryption.Overview)
+ [Enabling Amazon RDS Encryption for a DB Instance](#Overview.Encryption.Enabling)
+ [Availability of Amazon RDS Encryption](#Overview.Encryption.Availability)
+ [Managing Amazon RDS Encryption Keys](#Overview.Encryption.Keys)
+ [Limitations of Amazon RDS Encrypted DB Instance](#Overview.Encryption.Limitations)

## Overview of Encrypting Amazon RDS Resources<a name="Overview.Encryption.Overview"></a>

Amazon RDS encrypted DB instances provide an additional layer of data protection by securing your data from unauthorized access to the underlying storage\. You can use Amazon RDS encryption to increase data protection of your applications deployed in the cloud, and to fulfill compliance requirements for data\-at\-rest encryption\.

Amazon RDS also supports encrypting an Oracle or SQL Server DB instance with Transparent Data Encryption \(TDE\)\. TDE can be used with encryption at rest, although using TDE and encryption at rest simultaneously might slightly affect the performance of your database\. You must manage different keys for each encryption method\. For more information on TDE, see [Oracle Transparent Data Encryption](Appendix.Oracle.Options.AdvSecurity.md), [Using AWS CloudHSM Classic to Store Amazon RDS Oracle TDE Keys](Appendix.OracleCloudHSM.md), or [Microsoft SQL Server Transparent Data Encryption Support](Appendix.SQLServer.Options.TDE.md)\.

To manage the keys used for encrypting and decrypting your Amazon RDS resources, you use the [AWS Key Management Service \(AWS KMS\)](https://docs.aws.amazon.com/kms/latest/developerguide/)\. AWS KMS combines secure, highly available hardware and software to provide a key management system scaled for the cloud\. Using AWS KMS, you can create encryption keys and define the policies that control how these keys can be used\. AWS KMS supports CloudTrail, so you can audit key usage to verify that keys are being used appropriately\. Your AWS KMS keys can be used in combination with Amazon RDS and supported AWS services such as Amazon Simple Storage Service \(Amazon S3\), Amazon Elastic Block Store \(Amazon EBS\), and Amazon Redshift\. For a list of services that support AWS KMS, go to [Supported Services](https://docs.aws.amazon.com/kms/latest/developerguide/services.html) in the *AWS Key Management Service Developer Guide*\.

For an Amazon RDS encrypted DB instance, all logs, backups, and snapshots are encrypted\. A Read Replica of an Amazon RDS encrypted instance is also encrypted using the same key as the master instance when both are in the same region\. If the master and Read Replica are in different regions, you encrypt using the encryption key for that region\.

For encrypted and unencrypted Amazon RDS DB instances with cross\-region Read Replicas, data sent between the source and the Read Replicas is encrypted\.

## Enabling Amazon RDS Encryption for a DB Instance<a name="Overview.Encryption.Enabling"></a>

To enable encryption for a new DB instance, choose **Enable encryption** on the Amazon RDS console\. For information on creating a DB instance, see one of the following topics:
+ [Creating a DB Instance Running the MySQL Database Engine](USER_CreateInstance.md)
+ [Creating a DB Instance Running the Oracle Database Engine](USER_CreateOracleInstance.md)
+ [Creating a DB Instance Running the Microsoft SQL Server Database Engine](USER_CreateMicrosoftSQLServerInstance.md)
+ [Creating a DB Instance Running the PostgreSQL Database Engine](USER_CreatePostgreSQLInstance.md)
+ [Creating a DB Instance Running the MariaDB Database Engine](USER_CreateMariaDBInstance.md)

If you use the [create\-db\-instance](https://docs.aws.amazon.com/cli/latest/reference/rds/create-db-instance.html) AWS CLI command to create an encrypted RDS DB instance, set the `--storage-encrypted` parameter to true\. If you use the [CreateDBInstance](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateDBInstance.html) API action, set the `StorageEncrypted` parameter to true\.

When you create an encrypted DB instance, you can also supply the AWS KMS key identifier for your encryption key\. If you don't specify an AWS KMS key identifier, then Amazon RDS uses your default encryption key for your new DB instance\. AWS KMS creates your default encryption key for Amazon RDS for your AWS account\. Your AWS account has a different default encryption key for each AWS Region\.

Once you have created an encrypted DB instance, you cannot change the encryption key for that instance\. Therefore, be sure to determine your encryption key requirements before you create your encrypted DB instance\.

If you use the AWS CLI `create-db-instance` command to create an encrypted RDS DB instance, set the `--kms-key-id` parameter to the Amazon Resource Name \(ARN\) for the AWS KMS encryption key for the DB instance\. If you use the Amazon RDS API `CreateDBInstance` action, set the `KmsKeyId` parameter to the ARN for your AWS KMS key for the DB instance\.

You can use the ARN of a key from another account to encrypt an RDS DB instance\. Or you might create a DB instance with the same AWS account that owns the AWS KMS encryption key used to encrypt that new DB instance\. In this case, the AWS KMS key ID that you pass can be the AWS KMS key alias instead of the key's ARN\.

**Important**  
If Amazon RDS loses access to the encryption key for a DB instance—for example, when RDS access to a key is revoked—then the encrypted DB instance goes into a terminal state\. In this case, you can only restore the DB instance from a backup\. We strongly recommend that you always enable backups for encrypted DB instances to guard against the loss of encrypted data in your databases\.

## Availability of Amazon RDS Encryption<a name="Overview.Encryption.Availability"></a>

Amazon RDS encryption is currently available for all database engines and storage types\. Amazon RDS encryption is not currently available in the China \(Beijing\) region\.

Amazon RDS encryption is available for most DB instance classes\. The following table lists DB instance classes that *do not support* Amazon RDS encryption:


| Instance Type | Instance Class | 
| --- | --- | 
| General Purpose \(M1\) |  db\.m1\.small db\.m1\.medium db\.m1\.large db\.m1\.xlarge  | 
| Memory Optimized \(M2\) |  db\.m2\.xlarge db\.m2\.2xlarge db\.m2\.4xlarge  | 
| Burst Capable \(T2\) |  db\.t2\.micro  db\.t3\.micro| 

**Note**  
Encryption at rest is not available for DB instances running SQL Server Express Edition\.   

## Managing Amazon RDS Encryption Keys<a name="Overview.Encryption.Keys"></a>

You can manage keys used for Amazon RDS encrypted DB instances using the [AWS Key Management Service \(AWS KMS\)](https://docs.aws.amazon.com/kms/latest/developerguide/) in the IAM console\. If you want full control over a key, then you must create a customer\-managed key\.

You can't delete, revoke, or rotate default keys provisioned by AWS KMS\. You can't share a snapshot that has been encrypted using the default AWS KMS encryption key of the AWS account that shared the snapshot\.

You can view audit logs of every action taken with a customer\-managed key by using [AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/)\.

**Important**  
If you disable the key for an encrypted DB instance, you cannot read from or write to that DB instance\. When Amazon RDS encounters a DB instance encrypted by a key that Amazon RDS doesn't have access to, Amazon RDS puts the DB instance into a terminal state\. In this state, the DB instance is no longer available and the current state of the database can't be recovered\. To restore the DB instance, you must re\-enable access to the encryption key for Amazon RDS, and then restore the DB instance from a backup\.

## Limitations of Amazon RDS Encrypted DB Instance<a name="Overview.Encryption.Limitations"></a>

The following limitations exist for Amazon RDS encrypted DB instance:
+ You can only enable encryption for an Amazon RDS DB instance when you create it, not after the DB instance is created\.

  However, because you can encrypt a copy of an unencrypted DB snapshot, you can effectively add encryption to an unencrypted DB instance\. That is, you can create a snapshot of your DB instance, and then create an encrypted copy of that snapshot\. You can then restore a DB instance from the encrypted snapshot, and thus you have an encrypted copy of your original DB instance\. For more information, see [Copying a Snapshot](USER_CopySnapshot.md)\.
+ DB instances that are encrypted can't be modified to disable encryption\.
+ You can't have an encrypted Read Replica of an unencrypted DB instance or an unencrypted Read Replica of an encrypted DB instance\.
+ Encrypted Read Replicas must be encrypted with the same key as the source DB instance\.
+ You can't restore an unencrypted backup or snapshot to an encrypted DB instance\.
+ To copy an encrypted snapshot from one region to another, you must specify the KMS key identifier of the destination region\. This is because KMS encryption keys are specific to the region that they are created in\.

   The source snapshot remains encrypted throughout the copy process\. AWS Key Management Service uses envelope encryption to protect data during the copy process\. For more information about envelope encryption, see [ Envelope Encryption](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#enveloping)\.
