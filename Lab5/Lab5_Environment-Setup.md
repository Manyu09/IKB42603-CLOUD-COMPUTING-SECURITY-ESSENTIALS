# IKB42603 Lab 5 - Environment Setup & Step-by-Step Walkthrough

## Course Information

| Item | Details |
|---|---|
| Course | IKB42603 Cloud Computing Security Essentials |
| Lab | Lab 5 |
| Name | Muhammad Aiman |
| No. ID | 52215124380 |
| Date | 9/7/2026 |

---

## Introduction

This report documents the hands-on work for:

1. `IKB42603_Lab5_Monitoring_Logging_and_Incident_Detection`
2. `IKB42603_Lab5.1_Management_Plane_and_BCDR`

The walkthrough follows the supplied lab guides and explains each task in order, with the matching evidence screenshots from the `Lab5 Evidences` and `Lab5.1 Evidences` folders.

---

# Part 1: Lab 5 - Monitoring, Logging and Incident Detection

## 1. Environment Setup

### Step 1: Start LocalStack

The lab environment was started using Docker and LocalStack to simulate AWS-compatible services locally.

Evidence:

- `Lab5 Evidences/00_Setup_01_Docker_Containers.png`
- `Lab5 Evidences/00_Setup_02_LocalStack_Running.png`

### Step 2: Verify LocalStack health

LocalStack was checked to ensure the service was running correctly before continuing with the lab tasks.

Evidence:

- `Lab5 Evidences/00_Setup_03_LocalStack_Health.png`

### Step 3: Configure the endpoint

The AWS CLI endpoint was configured to point to LocalStack on port `4566`.

Evidence:

- `Lab5 Evidences/00_Setup_04_Endpoint_Configured.png`

### Step 4: Create the log group and stream

The CloudWatch Logs group `/ccse/app` and the `auth` stream were created for the application log.

Evidence:

- `Lab5 Evidences/00_Setup_05_Log_Group_Created.png`
- `Lab5 Evidences/00_Setup_06_Auth_Stream_Verified.png`

---

## 2. Task 1 - Generate Application Logs

### What was done

A sample authentication log was created containing:

- one successful login from a trusted IP
- four failed login attempts from `203.0.113.9`
- one successful login from the same suspicious IP
- one `EXPORT_DATA` action from that IP

### Question answered

**What does the generated log show?**

It shows a suspicious sequence of repeated login failures followed by a successful login and a large data export, which is the pattern used later for incident detection.

Evidence:

- `Lab5 Evidences/01_Task_1_Generated_Auth_Log.png`

---

## 3. Task 2 - Centralise Logs

### What was done

The application log entries were sent into the central log store and then read back to confirm successful ingestion.

### Question answered

**Were the logs successfully centralised?**

Yes. The log entries were written to the central LocalStack CloudWatch Logs stream and then retrieved successfully.

Evidence:

- `Lab5 Evidences/02_Task_2_Centralised_Log_Readback.png`

---

## 4. Task 3 - Query for Security-Relevant Activity

### What was done

The log file was searched for failed login attempts and the source IP was identified.

### Question answered

**How many failed login attempts occurred, and from which IP?**

There were **4 failed login attempts** from **`203.0.113.9`**.

### Question answered

**What is the difference between a log and an event?**

- A **log** is a recorded history of an action or system activity.
- An **event** is a meaningful occurrence or trigger, often created when something important happens, such as repeated failed logins causing an alert.

Evidence:

- `Lab5 Evidences/03_Task_3_Failed_Login_Query.png`

---

## 5. Task 4 - Tamper-Proof Hash-Chained Logs

### What was done

The authentication log was converted into a hash chain using SHA-256 so any later modification would break the chain.

Then the log was tampered with by changing the data export size from `500MB` to `5MB`, and the hash was recalculated.

### Question answered

**Was tampering detected?**

Yes. The tampered log produced a different final hash, proving that the log content had been altered.

Evidence:

- `Lab5 Evidences/04_Task_4_Original_Hash_Chain.png`
- `Lab5 Evidences/04_Task_4_Tampered_Log.png`
- `Lab5 Evidences/04_Task_4_Hash_Comparison.png`

---

## 6. Task 5 - Detect the Incident by Correlation

### What was done

The log events were correlated using this pattern:

`repeated failed logins -> successful login -> large export`

### Question answered

**What incident does the correlation suggest?**

The correlation suggests a probable **brute-force attack followed by account compromise and data exfiltration**.

Evidence:

- `Lab5 Evidences/05_Task_5_Incident_Correlation_Alert.png`

---

## 7. Task 6 - Incident Response

### What was done

The suspicious IP was contained using `iptables`, then the evidence log was copied, hashed, and integrity-checked.

### Question answered

**What was the containment action?**

The IP address `203.0.113.9` was blocked with a DROP rule.

### Question answered

**Was the evidence integrity verified successfully?**

Yes. The SHA-256 verification completed successfully, confirming the evidence file was unchanged.

Evidence:

- `Lab5 Evidences/06_Task_6_Containment_IPTables.png`
- `Lab5 Evidences/06_Task_6_Evidence_Hash.png`
- `Lab5 Evidences/06_Task_6_Evidence_Integrity_OK.png`

---

# Part 2: Lab 5.1 - Management Plane and BCDR

## 8. Environment Setup

### Step 1: Start LocalStack Pro and verify STS

LocalStack Pro was launched with the required settings, and AWS CLI connectivity was verified using STS.

Evidence:

- `Lab5.1 Evidences/00_Setup_LocalStack_Pro_Health_STS.png`

### Question answered

**Why is LocalStack Pro used here?**

It provides the management-plane request logging needed to reconstruct administrative actions and audit the cloud control plane.

---

## 9. Task A1 - Reconstruct the Management Plane Audit Trail

### What was done

An audit bucket was created, the baseline log count was recorded, administrative actions were performed, and the LocalStack request logs were filtered to reconstruct the management-plane trail.

The security-relevant actions included:

- create bucket
- create user
- attach administrator policy
- delete bucket

### Question answered

**What does the management-plane trail show?**

It shows administrative API activity performed against the cloud control plane, not application-level events.

### Question answered

**Why is this trail important?**

Because management-plane actions can change the cloud environment itself and are critical for security auditing and incident investigation.

Evidence:

- `Lab5.1 Evidences/01_A1_Audit_Bucket_Versioning_Baseline.png`
- `Lab5.1 Evidences/02_A1_Administrative_Actions.png`
- `Lab5.1 Evidences/03_A1_Reconstructed_Management_Trail.png`
- `Lab5.1 Evidences/04_A1_Filtered_Security_Events.png`
- `Lab5.1 Evidences/05_A1_Sealed_Trail_SHA256.png`
- `Lab5.1 Evidences/06_A1_Tamper_Detection_FAILED.png`

---

## 10. Task A2 - Backup and Separate Recovery Destination

### What was done

Two separate buckets were created:

- `miit-primary`
- `miit-dr-backup`

Then 200 records were created in the primary bucket and copied into the DR backup bucket.

### Question answered

**How many records were stored in the primary and DR backup buckets?**

Both buckets contained **200 records**.

### Question answered

**Why is a separate backup better than only versioning?**

Because a separate backup is stored in a different recovery destination and is more resilient if the primary bucket is damaged, deleted, or compromised.

Evidence:

- `Lab5.1 Evidences/07_A2_Primary_200_Records.png`
- `Lab5.1 Evidences/08_A2_DR_Backup_200_Records.png`

---

## 11. Task A3 - Timed Restore Drill

### What was done

The primary bucket contents were deleted to simulate loss, then the data was restored from the DR backup bucket and the recovery time was measured.

### Question answered

**What was the measured RTO?**

The measured RTO shown in the evidence was **1 second**.

### Question answered

**What was the RPO?**

The RPO is effectively **0 seconds in the captured drill**, because the backup copy was taken immediately before the restore test in this lab run.

### Required Table: RTO, RPO, and Extrapolation

| Measure | How you obtain it | Your value |
|---|---|---|
| Measured RTO | The elapsed seconds printed above, for 200 objects | `1 second` |
| Extrapolated RTO | Scale the measurement to 1,000,000 objects and state the assumption | `5,000 seconds` or about `83 minutes 20 seconds`, assuming linear scaling |
| RPO | The time between the last `s3 sync` and the incident, meaning data written in that window is lost | `0 seconds` in the captured drill |

### Question answered

**What is the extrapolated RTO for 1,000,000 records?**

Using the lab's simple linear estimate:

- `200 records -> 1 second`
- `1,000,000 records -> 5,000 seconds`

So the extrapolated RTO is **5,000 seconds**, or about **83 minutes 20 seconds**.

Evidence:

- `Lab5.1 Evidences/09_A3_Primary_Deleted_0_Objects.png`
- `Lab5.1 Evidences/10_A3_Restore_200_RTO_1s.png`

---

## 12. Task A4 - Compare Recovery Paths

### What was done

The lab compared two recovery approaches:

1. versioning inside the primary bucket
2. restoring from a separate backup bucket

The versioning evidence shows delete markers and object versions.

### Question answered

**How does versioning compare with a separate backup?**

- **Versioning** helps recover earlier object versions inside the same bucket.
- **Separate backup** provides a copy in another destination, which is more resilient to bucket loss or wider compromise.

### Question answered

**Which approach is safer for disaster recovery?**

A **separate backup bucket** is safer for true disaster recovery because it survives problems that affect the primary bucket.

### Required Table: Recovery Path Comparison

| Criterion | Versioning (in-place) | Separate backup bucket |
|---|---|---|
| Recovery speed | Fast for restoring earlier object versions inside the same bucket | Depends on backup copy and restore time |
| Survives bucket deletion? | No | Yes, if the backup bucket still exists |
| Survives a compromised admin credential? | Not necessarily | More resilient when protected by a separate trust boundary |
| Survives cryptographic erasure of the KMS key? | No, if the retained objects depend on that key | Depends on the backup's encryption and key architecture |
| Cost profile | Extra storage for retained versions | Extra storage plus backup and restore operations |

Evidence:

- `Lab5.1 Evidences/11_A4_Versioning_Upload_Delete.png`
- `Lab5.1 Evidences/12_A4_Two_Versions_Delete_Marker.png`
- `Lab5.1 Evidences/13_A4_Versioning_vs_Backup_Counts.png`

---

## 13. Results Summary

### Lab 5

- LocalStack environment was set up successfully
- Logs were generated and centralised
- 4 failed login attempts from `203.0.113.9` were identified
- Log tampering was detected through hash-chain mismatch
- Correlation indicated brute-force, compromise, and data exfiltration
- The suspicious IP was contained and evidence integrity was verified

### Lab 5.1

- Management-plane activity was reconstructed and sealed
- Tampering of the management trail was detected
- Primary and DR backup buckets each held 200 records
- The restore drill measured an RTO of 1 second
- The extrapolated RTO for 1,000,000 records is 5,000 seconds
- Versioning and separate backup recovery were compared

---

## Conclusion

This lab demonstrates the importance of cloud monitoring, log centralisation, incident correlation, tamper detection, containment, evidence integrity, management-plane auditing, and disaster recovery planning.

The first lab shows how suspicious activity can be detected from application logs and why hash chaining is useful for integrity protection. The second lab extends the idea to the cloud control plane and shows why a separate backup and a tested restore drill are essential for BCDR.

Overall, the exercise proves that security monitoring is not only about collecting logs, but also about making those logs trustworthy, actionable, and recoverable.

---

## Evidence Index

### Lab 5

1. `00_Setup_01_Docker_Containers.png`
2. `00_Setup_02_LocalStack_Running.png`
3. `00_Setup_03_LocalStack_Health.png`
4. `00_Setup_04_Endpoint_Configured.png`
5. `00_Setup_05_Log_Group_Created.png`
6. `00_Setup_06_Auth_Stream_Verified.png`
7. `01_Task_1_Generated_Auth_Log.png`
8. `02_Task_2_Centralised_Log_Readback.png`
9. `03_Task_3_Failed_Login_Query.png`
10. `04_Task_4_Original_Hash_Chain.png`
11. `04_Task_4_Tampered_Log.png`
12. `04_Task_4_Hash_Comparison.png`
13. `05_Task_5_Incident_Correlation_Alert.png`
14. `06_Task_6_Containment_IPTables.png`
15. `06_Task_6_Evidence_Hash.png`
16. `06_Task_6_Evidence_Integrity_OK.png`

### Lab 5.1

17. `00_Setup_LocalStack_Pro_Health_STS.png`
18. `01_A1_Audit_Bucket_Versioning_Baseline.png`
19. `02_A1_Administrative_Actions.png`
20. `03_A1_Reconstructed_Management_Trail.png`
21. `04_A1_Filtered_Security_Events.png`
22. `05_A1_Sealed_Trail_SHA256.png`
23. `06_A1_Tamper_Detection_FAILED.png`
24. `07_A2_Primary_200_Records.png`
25. `08_A2_DR_Backup_200_Records.png`
26. `09_A3_Primary_Deleted_0_Objects.png`
27. `10_A3_Restore_200_RTO_1s.png`
28. `11_A4_Versioning_Upload_Delete.png`
29. `12_A4_Two_Versions_Delete_Marker.png`
30. `13_A4_Versioning_vs_Backup_Counts.png`
