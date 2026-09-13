# IKB42603 Lab 6: Object Storage and Data Lifecycle

## Course Information

| Item | Details |
| --- | --- |
| Course | IKB42603 Cloud Computing Security Essentials |
| Lab | 6 |
| Name | Muhammad Aiman |
| No. ID | 52215124380 |

## Report Purpose

This report documents the step-by-step environment setup and lab execution for **IKB42603 Lab 6: Object Storage and Data Lifecycle**. The lab uses an AWS-compatible local cloud environment to configure S3 object storage, IAM access controls, encryption, object versioning, lifecycle rules, and cleanup.

The supporting screenshots are stored in the folder:

`Lab 6 Evidences/`

## Lab Environment

| Component | Configuration |
| --- | --- |
| Operating environment | Ubuntu terminal |
| Cloud emulator | LocalStack |
| AWS CLI endpoint variable | `EP='--endpoint-url=http://localhost:4566'` |
| AWS region | `us-east-1` |
| S3 bucket | `miit-patient-records-17619` |
| Main evidence folder | `Lab 6 Evidences/ALL_Original_Screenshots` |
| Task evidence folders | `Lab 6 Evidences/Task_1` to `Lab 6 Evidences/Task_8` |

## Step 1: Prepare LocalStack and AWS CLI Endpoint

1. Open the Ubuntu terminal.
2. Define the LocalStack endpoint variable:

   ```bash
   export EP='--endpoint-url=http://localhost:4566'
   ```

3. Define the S3 bucket name:

   ```bash
   export BUCKET=miit-patient-records-17619
   ```

4. Check running containers:

   ```bash
   docker ps
   ```

5. Test AWS CLI access to LocalStack:

   ```bash
   aws $EP sts get-caller-identity
   ```

6. During the setup, an endpoint connection error was observed when LocalStack was not reachable at `http://localhost:4566/`. The environment was then corrected before continuing the S3 tasks.

**Evidence:**

- `Lab 6 Evidences/Other_Setup_Troubleshooting`
- `Lab 6 Evidences/ALL_Original_Screenshots`

## Step 2: Create the S3 Bucket

1. Confirm the bucket variable:

   ```bash
   echo $BUCKET
   ```

2. Create the S3 bucket:

   ```bash
   aws $EP s3api create-bucket --bucket $BUCKET
   ```

3. Confirm that the bucket was created successfully. The command returned the bucket location and bucket ARN:

   ```text
   Location: /miit-patient-records-17619
   BucketArn: arn:aws:s3:::miit-patient-records-17619
   ```

**Evidence:**

- `Lab 6 Evidences/Task_1`
- `Lab 6 Evidences/Task_3`

## Step 3: Upload and Tag Lab Objects

1. Upload a public notice object and apply the `public` classification tag:

   ```bash
   aws $EP s3api put-object \
     --bucket $BUCKET \
     --key public/notice.txt \
     --body public-notice.txt \
     --tagging 'classification=public'
   ```

2. Upload an internal roster object and apply the `internal` classification tag:

   ```bash
   aws $EP s3api put-object \
     --bucket $BUCKET \
     --key internal/roster.txt \
     --body internal-roster.txt \
     --tagging 'classification=internal'
   ```

3. Upload a confidential record object and apply the `confidential` classification tag:

   ```bash
   aws $EP s3api put-object \
     --bucket $BUCKET \
     --key confidential/record.txt \
     --body confidential-record.txt \
     --tagging 'classification=confidential'
   ```

4. List the objects in the bucket:

   ```bash
   aws $EP s3api list-objects-v2 \
     --bucket $BUCKET \
     --query 'Contents[].{Key:Key,Size:Size}' \
     --output table
   ```

5. Confirm that the uploaded objects were present:

   | Object Key | Classification |
   | --- | --- |
   | `public/notice.txt` | `public` |
   | `internal/roster.txt` | `internal` |
   | `confidential/record.txt` | `confidential` |

6. Verify object tagging for the confidential record:

   ```bash
   aws $EP s3api get-object-tagging \
     --bucket $BUCKET \
     --key confidential/record.txt
   ```

7. The confidential object returned the tag:

   ```text
   classification=confidential
   ```

**Evidence:**

- `Lab 6 Evidences/Task_1`
- `Lab 6 Evidences/Task_2`

## Step 4: Configure IAM Access for Data Analyst

1. Create or verify an IAM user named `DataAnalyst`.
2. List existing access keys for the user:

   ```bash
   aws $EP iam list-access-keys \
     --user-name DataAnalyst \
     --query 'AccessKeyMetadata[].AccessKeyId' \
     --output text
   ```

3. Delete old access keys if required:

   ```bash
   for key in $(aws $EP iam list-access-keys \
     --user-name DataAnalyst \
     --query 'AccessKeyMetadata[].AccessKeyId' \
     --output text); do
     aws $EP iam delete-access-key \
       --user-name DataAnalyst \
       --access-key-id "$key"
   done
   ```

4. Create a new access key for the user:

   ```bash
   read -r ANALYST_KEY_ID ANALYST_SECRET <<(
     aws $EP iam create-access-key \
       --user-name DataAnalyst \
       --query 'AccessKey.[AccessKeyId,SecretAccessKey]' \
       --output text
   )
   ```

5. Configure the local AWS CLI profile for the analyst:

   ```bash
   aws configure --profile analyst set aws_access_key_id "$ANALYST_KEY_ID"
   aws configure --profile analyst set aws_secret_access_key "$ANALYST_SECRET"
   aws configure --profile analyst set region us-east-1
   ```

**Evidence:**

- `Lab 6 Evidences/Task_4`

## Step 5: Apply Bucket Access Policy

1. Create a bucket policy that allows account-level read access only to objects under the `internal/` prefix.
2. Apply the policy to the bucket:

   ```bash
   aws $EP s3api put-bucket-policy \
     --bucket $BUCKET \
     --policy file://bucket-policy.json
   ```

3. Retrieve and review the bucket policy:

   ```bash
   aws $EP s3api get-bucket-policy \
     --bucket $BUCKET \
     --query Policy \
     --output text
   ```

4. The retrieved policy contained the statement:

   ```json
   {
     "Sid": "AccountReadInternalOnly",
     "Effect": "Allow",
     "Principal": {
       "AWS": "arn:aws:iam::000000000000:root"
     },
     "Action": "s3:GetObject",
     "Resource": "arn:aws:s3:::miit-patient-records-17619/internal/*"
   }
   ```

5. This policy demonstrates prefix-based access control for object storage.

**Evidence:**

- `Lab 6 Evidences/Task_4`

## Step 6: Configure KMS Encryption

1. Create or use a KMS key for server-side encryption.
2. Confirm that the KMS key is enabled:

   ```bash
   aws $EP kms describe-key \
     --key-id $KEY_ID \
     --query 'KeyMetadata.[KeyId,KeyState,Enabled]' \
     --output text
   ```

3. The KMS key used in the lab was:

   ```text
   637db677-6f3f-46d0-9609-636ff5829102
   ```

4. Create the bucket encryption configuration:

   ```bash
   cat > encryption.json <<JSON
   {
     "Rules": [{
       "ApplyServerSideEncryptionByDefault": {
         "SSEAlgorithm": "aws:kms",
         "KMSMasterKeyID": "$KEY_ID"
       },
       "BucketKeyEnabled": true
     }]
   }
   JSON
   ```

5. Apply the encryption configuration:

   ```bash
   aws $EP s3api put-bucket-encryption \
     --bucket $BUCKET \
     --server-side-encryption-configuration file://encryption.json
   ```

6. Upload a new confidential object version after encryption was enabled:

   ```bash
   aws $EP s3api put-object \
     --bucket $BUCKET \
     --key confidential/record-v2.txt \
     --body confidential-record.txt
   ```

7. Verify encryption metadata on the object:

   ```bash
   aws $EP s3api head-object \
     --bucket $BUCKET \
     --key confidential/record-v2.txt \
     --query '[ServerSideEncryption,SSEKMSKeyId,BucketKeyEnabled]' \
     --output text
   ```

8. The object metadata confirmed:

   ```text
   aws:kms
   arn:aws:kms:us-east-1:000000000000:key/637db677-6f3f-46d0-9609-636ff5829102
   True
   ```

**Evidence:**

- `Lab 6 Evidences/Task_5`
- `Lab 6 Evidences/Task_8`

## Step 7: Enforce Secure Transport Policy

1. Remove the previous bucket policy before applying the secure transport policy:

   ```bash
   aws $EP s3api delete-bucket-policy \
     --bucket $BUCKET
   ```

2. Create a secure transport policy:

   ```bash
   cat > secure-transport.json <<JSON
   {
     "Version": "2012-10-17",
     "Statement": [{
       "Sid": "DenyUnencryptedTransport",
       "Effect": "Deny",
       "Principal": "*",
       "Action": "s3:*",
       "Resource": [
         "arn:aws:s3:::miit-patient-records-17619",
         "arn:aws:s3:::miit-patient-records-17619/*"
       ],
       "Condition": {
         "Bool": {
           "aws:SecureTransport": "false"
         }
       }
     }]
   }
   JSON
   ```

3. Apply the policy:

   ```bash
   aws $EP s3api put-bucket-policy \
     --bucket $BUCKET \
     --policy file://secure-transport.json
   ```

4. The policy is designed to deny S3 access when requests are not made through secure transport.
5. In this LocalStack run, the HTTP secure transport denial behavior was not fully enforced, as noted in the evidence README. The policy file and related command evidence were retained.

**Evidence:**

- `Lab 6 Evidences/Task_6`
- `Lab 6 Evidences/README.txt`

## Step 8: Enable Bucket Versioning

1. Delete the temporary secure transport bucket policy:

   ```bash
   aws $EP s3api delete-bucket-policy \
     --bucket $BUCKET
   ```

2. Enable bucket versioning:

   ```bash
   aws $EP s3api put-bucket-versioning \
     --bucket $BUCKET \
     --versioning-configuration Status=Enabled
   ```

3. Verify the versioning status:

   ```bash
   aws $EP s3api get-bucket-versioning \
     --bucket $BUCKET
   ```

4. The output confirmed:

   ```json
   {
     "Status": "Enabled"
   }
   ```

5. List object versions for the confidential record:

   ```bash
   aws $EP s3api list-object-versions \
     --bucket $BUCKET \
     --prefix confidential/record.txt \
     --query 'Versions[].{VersionId:VersionId,IsLatest:IsLatest,Size:Size}' \
     --output table
   ```

6. The output showed multiple versions, including the latest version and previous versions.

**Evidence:**

- `Lab 6 Evidences/Task_7`

## Step 9: Configure Object Lifecycle Rules

1. Create a lifecycle configuration file named `lifecycle.json`.
2. Include rules for confidential record retention and incomplete multipart upload cleanup.
3. Apply the lifecycle configuration:

   ```bash
   aws $EP s3api put-bucket-lifecycle-configuration \
     --bucket $BUCKET \
     --lifecycle-configuration file://lifecycle.json
   ```

4. Verify the lifecycle configuration:

   ```bash
   aws $EP s3api get-bucket-lifecycle-configuration \
     --bucket $BUCKET \
     --query 'Rules[].{ID:ID,Status:Status}' \
     --output table
   ```

5. The command output confirmed that the following lifecycle rules were enabled:

   | Rule ID | Status |
   | --- | --- |
   | `RetireConfidentialRecords` | `Enabled` |
   | `AbortIncompleteUploads` | `Enabled` |

**Evidence:**

- `Lab 6 Evidences/Task_8`

## Step 10: Final Validation

The lab environment was validated using AWS CLI commands against LocalStack. The following outcomes were confirmed:

| Validation Area | Result |
| --- | --- |
| Bucket creation | Successful |
| Object upload | Successful |
| Object classification tags | Verified |
| IAM user/profile setup | Completed |
| Bucket policy | Applied and retrieved |
| KMS encryption | Enabled and verified |
| Secure transport policy | Policy created; LocalStack enforcement limitation observed |
| Versioning | Enabled and verified |
| Lifecycle rules | Enabled and verified |

## Evidence Summary

| Folder | Purpose |
| --- | --- |
| `Lab 6 Evidences/Task_1` | Object upload, listing, and tag verification |
| `Lab 6 Evidences/Task_2` | Additional object tagging and upload evidence |
| `Lab 6 Evidences/Task_3` | Bucket creation and initial object upload |
| `Lab 6 Evidences/Task_4` | IAM profile and bucket policy configuration |
| `Lab 6 Evidences/Task_5` | KMS encryption configuration and encrypted object validation |
| `Lab 6 Evidences/Task_6` | Secure transport policy setup |
| `Lab 6 Evidences/Task_7` | Versioning configuration and object version validation |
| `Lab 6 Evidences/Task_8` | KMS key verification and lifecycle configuration |
| `Lab 6 Evidences/Other_Setup_Troubleshooting` | Environment setup, errors, and troubleshooting |
| `Lab 6 Evidences/ALL_Original_Screenshots` | Complete screenshot collection |

## Embedded Evidence Images

The following images are embedded directly in this Markdown report so the evidence can be viewed without opening the folders separately.

### Environment Setup and Troubleshooting

![LocalStack endpoint setup and initial connection issue](<Lab 6 Evidences/Other_Setup_Troubleshooting/02_bf4c6db6-0646-4a59-bffb-244daadd5650.png>)

![Setup and troubleshooting evidence](<Lab 6 Evidences/Other_Setup_Troubleshooting/45_ec146757-4352-48ab-8853-4db547d6e836.png>)

### Task 1 Evidence: Object Classification and Tagging

![Uploaded objects and confidential object tag](<Lab 6 Evidences/Task_1/03_37855738-b1f2-41e9-9c67-faa8b88121e7.png>)

![Task 1 supporting evidence](<Lab 6 Evidences/Task_1/84_c2bc6445-a758-4d24-82c5-437d5284fe4c.png>)

### Task 2 Evidence: Object Uploads

![Internal and confidential object uploads](<Lab 6 Evidences/Task_2/37_3c8157ff-6e24-4b59-b3a6-c5bf5ca58a20.png>)

![Task 2 supporting evidence](<Lab 6 Evidences/Task_2/29_29df169c-e605-4d54-88a5-8ea98fddc735.png>)

### Task 3 Evidence: Bucket Creation

![Bucket creation and public object upload](<Lab 6 Evidences/Task_3/41_9b3bd765-8398-42bd-b6d2-2306df681b96.png>)

![Task 3 supporting evidence](<Lab 6 Evidences/Task_3/12_894c273e-a214-4aed-82fe-7f62f8232f27.png>)

### Task 4 Evidence: IAM and Bucket Policy

![DataAnalyst access key and profile setup](<Lab 6 Evidences/Task_4/33_ea7832b3-d340-4449-94a1-db1af274d28d.png>)

![Bucket policy allowing internal object access](<Lab 6 Evidences/Task_4/46_7a7219a7-dc2a-4f5d-b4b9-b0b1d473f5f3.png>)

### Task 5 Evidence: KMS Encryption

![Bucket encryption JSON configuration](<Lab 6 Evidences/Task_5/14_74423bc5-0b52-4275-86b4-42d3c8d5bc8c.png>)

![Encrypted object validation](<Lab 6 Evidences/Task_5/23_0564ff60-f050-447a-b990-94910ed06efc.png>)

### Task 6 Evidence: Secure Transport Policy

![Secure transport bucket policy](<Lab 6 Evidences/Task_6/67_b0d988f4-47a9-4d6c-98e4-e9b64f3a24cf.png>)

![Task 6 supporting evidence](<Lab 6 Evidences/Task_6/61_05c545b4-50be-4eb9-b09f-a449c606e306.png>)

### Task 7 Evidence: Versioning and Remanence

![Bucket versioning enabled](<Lab 6 Evidences/Task_7/19_e1e0d727-2f5d-48a9-b2dd-fe9ecc64f16b.png>)

![Object version list](<Lab 6 Evidences/Task_7/30_7586e115-7848-4e66-8567-b947d9483ca3.png>)

### Task 8 Evidence: Lifecycle and Cryptographic Erasure

![KMS key enabled](<Lab 6 Evidences/Task_8/11_72512b73-9066-4e05-93d0-3bc532883e7e.png>)

![Lifecycle rules enabled](<Lab 6 Evidences/Task_8/32_b64a76c4-37a3-4d86-bd87-6921db653017.png>)

## Guideline Questions and Answers

### Task 1: Classification Table

| Classification | S3 Key or Prefix | Control Implemented |
| --- | --- | --- |
| Public | `public/notice.txt` | Public classification tag and later public access blocking |
| Internal | `internal/roster.txt` | Bucket policy scoped to the `internal/` prefix |
| Confidential | `confidential/record.txt` | Confidential tag, explicit deny policy, KMS encryption, versioning, lifecycle retention, and cryptographic erasure support |

### Task 2: Public Bucket Exposure

**Question:** Which single word in the JSON policy caused the exposure?

**Answer:** The exposure was caused by the wildcard principal `"*"`. It allowed anonymous access because the bucket policy granted `s3:GetObject` to everyone instead of limiting access to a specific AWS account, IAM user, or role.

### Task 3: Block Public Access

**Question:** Which Block Public Access flag would have rejected the public bucket policy on real AWS?

**Answer:** `BlockPublicPolicy` would reject a public bucket policy that grants access to everyone. The full guardrail is stronger when all four Block Public Access settings are enabled: `BlockPublicAcls`, `IgnorePublicAcls`, `BlockPublicPolicy`, and `RestrictPublicBuckets`.

**Question:** Why is a preventative guardrail stronger than a detective control?

**Answer:** A preventative guardrail blocks the risky configuration before exposure happens. A detective control only alerts after the bucket is already public, which means data may already have been accessed.

### Task 4: Identity Policy vs Resource Policy

**Question:** What is the difference between an identity-based policy and a resource-based policy?

**Answer:** An identity-based policy is attached to an IAM user, group, or role and defines what that identity can do. A resource-based policy is attached directly to a resource, such as an S3 bucket, and defines who can access that resource.

**Question:** In the analyst test, which policy decided each request?

**Answer:** The analyst IAM policy allowed broad S3 read actions, but the bucket policy explicitly denied access to the confidential prefix. The internal object request was allowed because both policy layers permitted it. The confidential object request was denied because the resource-based bucket policy contained an explicit deny, and explicit deny overrides allow.

### Task 5: Default SSE-KMS Encryption

**Question:** Does default SSE-KMS encryption protect the confidential record from the analyst in Task 4?

**Answer:** No. Server-side encryption protects data at rest, but it does not replace authorization. If an analyst is authorized to call `s3:GetObject` and the system permits KMS decrypt access, the object can still be read. Access control must still be enforced with IAM and bucket policies.

**Question:** What does server-side encryption defend against?

**Answer:** Server-side encryption protects stored object data if the underlying storage media, snapshots, or backups are exposed. It reduces the risk of raw storage compromise, but it does not stop an authorized API caller from reading the object.

### Task 6: Presigned URL and Secure Transport

**Question:** What do `X-Amz-Expires` and `X-Amz-Signature` mean in a presigned URL?

**Answer:** `X-Amz-Expires` defines how long the URL remains valid. `X-Amz-Signature` proves that the URL was signed by credentials allowed to perform the requested action. Anyone holding the URL before expiry can use it.

**Question:** Why must a condition key be evaluated against the environment where it runs?

**Answer:** A policy condition such as `aws:SecureTransport` depends on how requests reach the service. In real AWS, HTTPS requests normally evaluate as secure transport. In this LocalStack lab, the endpoint used plain HTTP at `http://localhost:4566`, so the same condition can behave differently or lock out ordinary local commands.

### Task 7: Versioning and Data Remanence

**Question:** Why is deleting the current object not enough for an erasure request?

**Answer:** With versioning enabled, `delete-object` creates a delete marker instead of removing all historical object versions. Older versions can still contain the original confidential data. A compliant erasure process must remove every object version and delete marker.

**Question:** What mechanisms make deletion provable?

**Answer:** Lifecycle expiration rules provide automated retention enforcement, while explicit deletion of all versions removes historical copies. Cryptographic erasure provides stronger assurance by disabling or deleting the encryption key so encrypted object data becomes unrecoverable.

### Task 8: Lifecycle and Cryptographic Erasure

**Question:** Why does cryptographic erasure provide stronger assurance than overwriting?

**Answer:** In cloud storage, users do not control the physical disks, replicas, snapshots, or backups. Destroying or disabling the encryption key can make all encrypted copies unreadable, even if ciphertext remains on storage media.

**Question:** Which commands would be useful as compliance evidence?

**Answer:** Useful evidence commands include:

```bash
aws $EP s3api get-public-access-block --bucket $BUCKET
aws $EP s3api get-bucket-encryption --bucket $BUCKET
aws $EP s3api get-bucket-versioning --bucket $BUCKET
aws $EP s3api get-bucket-lifecycle-configuration --bucket $BUCKET
aws $EP kms describe-key --key-id $KEY_ID
```

These commands prove public access controls, encryption, versioning, lifecycle policy, and KMS key state.

## Final Verification Command Block

The guideline also asks for a final verification posture. The following command block can be run to collect final evidence:

```bash
echo "=== IKB42603 Lab 6 Verification: $BUCKET ==="
aws $EP s3api get-public-access-block \
  --bucket $BUCKET \
  --query "PublicAccessBlockConfiguration"
aws $EP s3api get-bucket-encryption \
  --bucket $BUCKET \
  --query "ServerSideEncryptionConfiguration.Rules[0].ApplyServerSideEncryptionByDefault.[SSEAlgorithm,KMSMasterKeyID]"
aws $EP s3api get-bucket-versioning \
  --bucket $BUCKET \
  --query "Status"
aws $EP s3api get-bucket-lifecycle-configuration \
  --bucket $BUCKET \
  --query "Rules[].{ID:ID,Status:Status}"
aws $EP kms describe-key \
  --key-id $KEY_ID \
  --query "KeyMetadata.[KeyId,KeyState,Enabled]"
```

## Conclusion

Lab 6 successfully demonstrated secure object storage management using an AWS-compatible LocalStack environment. The lab covered bucket creation, object classification, IAM access configuration, bucket policy enforcement, KMS-based server-side encryption, secure transport policy design, versioning, lifecycle configuration, and evidence-based validation.
