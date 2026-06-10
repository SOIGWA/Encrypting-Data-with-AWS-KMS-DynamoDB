
#  Encrypting Data with AWS KMS & DynamoDB
> **Platform:** NextWork / AWS  
> **Topic:** Data Encryption, Key Management, Access Control   
> **Estimated Time:** 45 minutes

---

## 📋 Table of Contents

1. [Overview](#overview)
2. [Case Scenario](#case-scenario)
3. [Key Services](#key-services)
4. [Lab Objectives](#lab-objectives)
5. [Prerequisites](#prerequisites)
6. [Lab Walkthrough](#lab-walkthrough)
   - [Step 1 – Create a KMS Encryption Key](#step-1--create-a-kms-encryption-key)
   - [Step 2 – Create and Encrypt a DynamoDB Table](#step-2--create-and-encrypt-a-dynamodb-table)
   - [Step 3 – Add Data to the Table](#step-3--add-data-to-the-table)
   - [Step 4 – Create a Test IAM User](#step-4--create-a-test-iam-user)
   - [Step 5 – Validate KMS Encryption](#step-5--validate-kms-encryption)
7. [Cleanup](#cleanup)
8. [Key Concepts](#key-concepts)
9. [Summary](#summary)

---

## Overview

Data breaches cost companies an average of **$4.5 million** to resolve. This lab explores how to protect sensitive data stored in a cloud database using encryption — ensuring that even if unauthorized users access the storage layer, they cannot read the data without the correct key and permissions.

Using **AWS Key Management Service (KMS)** and **Amazon DynamoDB**, this project demonstrates the full lifecycle of encryption: creating a customer-managed key, attaching it to a database, testing access as both an authorized and unauthorized user, and observing how KMS enforces access boundaries in real time.

---

## Case Scenario

A DynamoDB table needs to be protected so that only authorized users can read its contents. The goal is to:

1. Create a **customer-managed encryption key** in AWS KMS
2. Attach that key to a DynamoDB table to encrypt all stored data
3. Confirm that authorized users can read the data transparently
4. Confirm that unauthorized users — even those with full DynamoDB access — are blocked at the decryption layer

---

## Key Services

| Service | Role in This Lab |
|---------|-----------------|
| **AWS KMS** | Creates and manages the customer-managed encryption key |
| **Amazon DynamoDB** | The database being encrypted with the KMS key |
| **AWS IAM** | Controls which users have permission to use the encryption key |

---

## Lab Objectives

- [x] Create a symmetric customer-managed KMS key with an alias
- [x] Assign key administrators and key users
- [x] Create a DynamoDB table encrypted with the KMS key
- [x] Add data to the encrypted table and confirm authorized access
- [x] Create a test IAM user with DynamoDB access but no KMS key permissions
- [x] Log in as the test user and observe the access denied error
- [x] *(Secret Mission)* Grant the test user decryption access and verify

---

## Prerequisites

- An AWS account (IAM Admin user access)
- Basic familiarity with the AWS Management Console
- No prior encryption experience required

---

## Lab Walkthrough

---

### Step 1 – Create a KMS Encryption Key

The first step is creating a **customer-managed key (CMK)** in AWS KMS. This key will be the engine behind all encryption and decryption operations on the DynamoDB table.

#### Navigate to KMS

1. Log in to the AWS Management Console as your **IAM Admin user**
2. Search for and open **Key Management Service (KMS)**
3. In the left navigation pane, select **Customer managed keys**
4. Click **Create key**

#### Configure the Key

**Key type:** Select **Symmetric**

> 💡 **Symmetric vs. Asymmetric:** Symmetric encryption uses a single key to both encrypt and decrypt data. It is faster and more efficient for large datasets — ideal for a database like DynamoDB. Asymmetric encryption uses a key pair (public + private) and is typically used for secure data exchange between multiple parties.

**Key usage:** Keep **Encrypt and decrypt**

> 💡 **Key usage** defines what cryptographic operations the key is designed for. Selecting Encrypt and decrypt means the key can lock data into ciphertext and restore it to readable plaintext. The key's internal format is built around this usage type and cannot be changed later.

Click **Next**.

#### Name and Describe the Key

- **Alias:** `nextwork-kms-key`
- **Description:**
```
KMS key that will encrypt a DynamoDB database. Created for NextWork's Encryption with AWS KMS project.
```

Click **Next**.

#### Assign Key Administrators and Users

- **Key administrator:** Select your IAM Admin user
- **Key user:** Select your IAM Admin user

> 💡 **Key administrator** controls the key's lifecycle and policies — who can access it, modify it, or schedule its deletion. **Key user** has permission to use the key in cryptographic operations (encrypting and decrypting data) but cannot manage the key's configuration.

Click **Next**, review the auto-generated **Key policy**, then click **Finish**.
<img width="1919" height="510" alt="image" src="https://github.com/user-attachments/assets/7f224d1b-6d05-4c34-9056-8089c4eae5b3" />
---

### Step 2 – Create and Encrypt a DynamoDB Table

With the encryption key ready, the next step is creating a DynamoDB table and attaching the key to encrypt all data stored in it.

#### Create the Table

1. Search for and open **DynamoDB** in the AWS console
2. Click **Create table**
3. Configure the table:
   - **Table name:** `nextwork-kms-table`
   - **Partition key:** `id` (String)
4. Under **Table settings**, keep **Default settings**
5. Click **Create table**

Wait for the table status to change to **Active**.

> 💡 **DynamoDB** is a fully managed NoSQL database service built for fast, flexible data storage. The **partition key** (`id`) is how DynamoDB organizes and distributes data internally across servers for efficient access.

#### Attach the KMS Key for Encryption

1. Select your table `nextwork-kms-table`
2. Go to the **Additional settings** tab
3. Scroll to the **Encryption** section
4. Click **Manage encryption**
5. Select **Stored in your account, and owned and managed by you** (customer managed key)
6. From the dropdown, select `nextwork-kms-key`

> 💡 **DynamoDB Encryption Options:**
> - *Owned by Amazon DynamoDB* — AWS fully manages the key; you have no visibility or control
> - *AWS managed key* — KMS manages the key on your behalf; you can view but not configure it
> - *Customer managed key (CMK)* — You create and manage the key in KMS, giving full control over access policies and lifecycle

<img width="1919" height="535" alt="image" src="https://github.com/user-attachments/assets/2ce949d9-7955-472a-a8e9-fb665fe11f35" />

Click **Save changes**.
---

### Step 3 – Add Data to the Table

With the table encrypted, add an item to test that authorized users can read data transparently.

#### Create an Item

1. In your table's page, click **Actions → Create item**
2. Next to the `id` attribute, enter `1`
3. Click **Create item**

#### Verify the Data is Readable

1. In the left sidebar, select **Explore items**
2. Select `nextwork-kms-table`
3. Refresh the page — you should see the item with `id = 1`

> 💡 **Why can you see the data?** Even though the data is encrypted at rest, DynamoDB performs **transparent data encryption** — it automatically decrypts the data using the KMS key on behalf of any authorized user. The decryption happens invisibly in the background, so authorized users interact with their data normally without any extra steps.

<img width="1862" height="615" alt="image" src="https://github.com/user-attachments/assets/60d1d833-e702-4074-8180-f2f40e969e1f" />

### Step 4 – Create a Test IAM User

To validate that encryption is enforced, create a test user that has full DynamoDB access but **no permission to use the KMS key**.

#### Create the User

1. Open the **IAM** console
2. Select **Users → Create user**
3. Configure the user:
   - **Username:** `nextwork-kms-user`
   - Check **Provide user access to the AWS Management Console**
   - Uncheck **Users must create a new password at next sign-in**
4. Click **Next**
5. Select **Attach existing policies directly**
6. Add the policy: `AmazonDynamoDBFullAccess`
7. Click **Next → Create user**

> ⚠️ **Important:** On the confirmation page, click **Download .csv file** immediately to save the user's login credentials. This option is not available after leaving the page.

> 💡 **Why no KMS permission?** This user has full access to DynamoDB's API — they can create tables, run queries, and manage items. However, the table's data is encrypted with a KMS key this user has no permission to use. This deliberate gap tests whether KMS enforces the encryption boundary independently of DynamoDB permissions.
<img width="1328" height="581" alt="image" src="https://github.com/user-attachments/assets/21d616c3-908a-4f06-80bf-9fe22a95af1f" />
---

### Step 5 – Validate KMS Encryption

Now test what happens when the unauthorized user tries to access the encrypted table.

#### Log In as the Test User

1. From the IAM user's page, copy the **Console sign-in URL**
2. Open a new **incognito/private browser window**
3. Paste the sign-in URL and log in as `nextwork-kms-user`
4. Make sure you are in the **same AWS Region** as your admin account

> 💡 **Why incognito?** Using an incognito window lets you stay logged into both the admin user and the test user simultaneously, making it easy to switch between perspectives without logging in and out.

#### Attempt to View the Encrypted Data

1. Navigate to **DynamoDB**
2. Go to **Explore items**
3. Select `nextwork-kms-table`

The table data fails to load and an error banner appears:

```
User: arn:aws:iam::<account-id>:user/nextwork-kms-user is not authorized
to perform: kms:Decrypt on resource: arn:aws:kms:<region>:<account-id>:key/<key-id>
```

> 💡 **Why is access denied?** `nextwork-kms-user` has `AmazonDynamoDBFullAccess` — they can see the table exists and interact with the DynamoDB API. But reading the actual item data requires decrypting it, which needs permission to use the KMS key (`kms:Decrypt`). Since this user was never added as a key user, KMS blocks the decryption operation entirely.

> This is the encryption boundary working exactly as intended: **DynamoDB permissions and KMS permissions are enforced independently**. Having one does not grant the other.


---

## Cleanup

> ⚠️ **Always delete resources after completing a lab to avoid unnecessary AWS charges.**

### Delete the DynamoDB Table

1. Go to **DynamoDB → Tables**
2. Select `nextwork-kms-table`
3. Click **Delete**
4. Check **Delete all CloudWatch alarms** ✅
5. Leave **Create an on-demand backup** unchecked ❌
6. Type `confirm` → click **Delete**

### Schedule KMS Key Deletion

1. Go to **KMS → Customer managed keys**
2. Select `nextwork-kms-key`
3. Click **Key actions → Schedule key deletion**
4. Set the **Waiting period** to `7` days (minimum)
5. Check the confirmation checkbox → click **Schedule deletion**

> 💡 **Why a waiting period?** Once a KMS key is deleted, any data encrypted solely by that key becomes permanently unrecoverable. The minimum 7-day waiting period gives you a window to cancel if the deletion was a mistake or if dependent systems still need the key.

### Delete the IAM Test User

1. Go to **IAM → Users**
2. Select `nextwork-kms-user`
3. Click **Delete → Enter the username to confirm → Delete user**
4. Delete the downloaded `.csv` credentials file from your local machine

---

## Key Concepts

### Encryption

Encryption converts data into ciphertext using an algorithm and a key. Without the correct key and permission to use it, the ciphertext is unreadable — appearing as random characters. Only authorized users with decryption permissions can restore the data to its original form.

### Symmetric vs. Asymmetric Encryption

| Type | Keys | Best For |
|------|------|----------|
| **Symmetric** | Single key for encrypt + decrypt | Large datasets, databases, stored data |
| **Asymmetric** | Public key (encrypt) + Private key (decrypt) | Secure data exchange between parties, digital signatures |

### AWS KMS — Customer Managed Keys

Customer managed keys give full control over:
- Who can administer the key (lifecycle management)
- Who can use the key (encrypt/decrypt operations)
- Cross-account access policies
- Key rotation schedules
- Audit logs of every key usage via AWS CloudTrail

### Transparent Data Encryption

DynamoDB automatically decrypts data for authorized users when they query the table. The decryption happens at the service layer — authorized users see plain readable data with no extra steps. Unauthorized users hit the KMS permission boundary and are blocked before the data is ever returned.

### Key Policy vs. IAM Policy

Two layers of permission control access to a KMS key:

| Layer | What it controls |
|-------|-----------------|
| **Key policy** | Attached directly to the KMS key; defines who can administer or use it |
| **IAM policy** | Attached to a user or role; grants permission to call KMS API actions |

Both must allow the action for access to succeed. Having `AmazonDynamoDBFullAccess` in IAM does not bypass the KMS key policy.

### Data States and Encryption

| State | Description | KMS Role |
|-------|-------------|----------|
| **At rest** | Stored in a database, file system, or S3 | KMS manages long-lived keys for stored data |
| **In transit** | Moving over a network (e.g. API calls, emails) | TLS/SSL session keys — not KMS |
| **In use** | Actively being processed in memory | Real-time access — not a KMS use case |

---

## Summary

| Step | Action | Outcome |
|------|--------|---------|
| Create KMS key | Created `nextwork-kms-key` as a symmetric CMK | Encryption key ready with admin and user assignments |
| Create DynamoDB table | Created `nextwork-kms-table` with CMK encryption | All data stored in the table is encrypted at rest |
| Add data | Added item `id = 1` to the encrypted table | Confirmed authorized users see data transparently |
| Create test user | Created `nextwork-kms-user` with DynamoDB access only | User has no KMS key permissions |
| Validate encryption | Logged in as test user, accessed the table | Access denied — `kms:Decrypt` permission missing |
| Secret mission | Added test user to key policy | Test user can now read the encrypted data |

---

##  AWS Services Used

- **AWS Key Management Service (KMS)** — Encryption key creation and management
- **Amazon DynamoDB** — NoSQL database encrypted with the KMS key
- **AWS IAM** — User creation and permission management
- **AWS CloudTrail** — *(Background)* Logs every KMS key usage for auditing

---

*Lab completed as part of the NextWork AWS security project series.*
