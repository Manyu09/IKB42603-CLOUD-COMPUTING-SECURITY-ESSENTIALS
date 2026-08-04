# Environment Setup Report

## Course Information

**Course:** IKB42603 Cloud Computing Security Essentials  
**Lab:** 1  
**Name:** Muhammad Aiman  
**No. ID:** 52215124380  
**Date:** 4/8/2026  

## Lab Information

**Lab Title:** Lab 1 - Account Security and IAM  
**Guide used:** `IKB42603_Lab1_Account_Security_and_IAM.pdf`  
**Evidence files:** `Evidence 1.png` to `Evidence 8.png`

## 1. Objective

This lab was completed to configure account security and identity access management controls using AWS IAM, and to validate role-based access control in a local Kubernetes environment. The setup demonstrates user creation, policy assignment, access key management, group membership, namespace isolation, service accounts, roles, role bindings, and permission testing.

## 2. Environment and Tools

The following tools and services were used:

- AWS CLI
- AWS IAM
- Kubernetes CLI (`kubectl`)
- kind Kubernetes cluster
- Local Ubuntu terminal environment

The AWS CLI commands used the `$EP` endpoint variable as shown in the evidence screenshots.

## 3. Task 1: Cloud Identity Landscape

| Platform | Identity Service | Main Purpose | Common Security Controls |
| --- | --- | --- | --- |
| AWS | AWS Identity and Access Management (IAM) | Manages AWS users, groups, roles, and policies. | Least privilege policies, MFA, access key rotation, IAM roles, and policy reviews. |
| Microsoft Azure | Microsoft Entra ID | Provides cloud identity, authentication, and access management for Azure and Microsoft services. | Conditional access, MFA, role-based access control, identity protection, and privileged identity management. |
| Google Cloud | Google Cloud IAM | Controls access to Google Cloud projects, resources, and services. | IAM roles, service accounts, organization policies, audit logging, and workload identity federation. |
| Kubernetes | Kubernetes RBAC | Controls access to Kubernetes API resources inside a cluster. | Roles, ClusterRoles, RoleBindings, ClusterRoleBindings, service accounts, and namespace isolation. |
| Okta | Okta Workforce Identity | Provides identity federation and single sign-on for cloud and enterprise applications. | SSO, MFA, lifecycle management, adaptive access policies, and centralized identity governance. |

## 4. Short-Answer Questions

### Question 1: Why is least privilege important in cloud identity management?

Least privilege ensures that users, groups, roles, and service accounts receive only the permissions required to perform their assigned tasks. This reduces the impact of credential misuse, accidental changes, and unauthorized access.

### Question 2: What is the difference between an IAM user and an IAM role?

An IAM user represents a long-term identity, usually for a person or application, and may have credentials such as passwords or access keys. An IAM role is an assumable identity with temporary credentials, making it safer for services, automation, and cross-account access.

### Question 3: Why should access keys be deactivated or rotated?

Access keys provide programmatic access to cloud resources. If they are exposed or no longer needed, they can be misused. Deactivating unused keys and rotating active keys helps reduce credential risk.

### Question 4: How does Kubernetes RBAC enforce access control?

Kubernetes RBAC uses roles to define allowed actions and role bindings to assign those permissions to users, groups, or service accounts. Permissions can be limited to a namespace, which supports environment separation such as `dev` and `prod`.

### Question 5: Why did the `dev-user` service account fail to list pods in the `prod` namespace?

The `dev-user` service account was bound to the `pod-reader` role only in the `dev` namespace. Since no role binding was created for `prod`, the service account had no permission to list pods there.

## 5. AWS IAM Setup

### Step 1: Add the Cloud Admin User to the Admins Group

The IAM user `CloudAdmin_Aiman` was added to the `Admins` group.

```bash
aws $EP iam add-user-to-group \
  --group-name Admins \
  --user-name CloudAdmin_Aiman
```

The group membership was then verified.

```bash
aws $EP iam get-group --group-name Admins
```

The output confirmed that `CloudAdmin_Aiman` is a member of the `Admins` group.

![Evidence 1](Evidence%201.png)

### Step 2: Create the Analyst User

A new IAM user named `Analyst_Aiman` was created.

```bash
aws $EP iam create-user --user-name Analyst_Aiman
```

The output returned the user details, including the IAM user ARN and creation timestamp.

![Evidence 2](Evidence%202.png)

### Step 3: Attach Read-Only S3 Permission to the Analyst User

The AWS managed policy `AmazonS3ReadOnlyAccess` was attached to `Analyst_Aiman`.

```bash
aws $EP iam attach-user-policy \
  --user-name Analyst_Aiman \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

The attached policy was verified using:

```bash
aws $EP iam list-attached-user-policies --user-name Analyst_Aiman
```

The output confirmed that `AmazonS3ReadOnlyAccess` was attached successfully.

![Evidence 2](Evidence%202.png)

### Step 4: Create an Access Key for the Analyst User

An access key was created for `Analyst_Aiman`.

```bash
aws $EP iam create-access-key --user-name Analyst_Aiman
```

The output showed the access key status as `Active`. The access key ID and secret access key are treated as sensitive credentials and are masked in the evidence.

![Evidence 3](Evidence%203.png)

### Step 5: Deactivate the Analyst Access Key

The access key for `Analyst_Aiman` was deactivated to demonstrate credential lifecycle control.

```bash
aws $EP iam update-access-key \
  --user-name Analyst_Aiman \
  --access-key-id <masked-access-key-id> \
  --status Inactive
```

The key status was verified.

```bash
aws $EP iam list-access-keys --user-name Analyst_Aiman
```

The output confirmed that the access key status changed to `Inactive`.

![Evidence 4](Evidence%204.png)

### Step 6: Verify AWS Caller Identity

The guide also requires verification of the active AWS caller identity using:

```bash
aws $EP sts get-caller-identity
```

No separate `sts get-caller-identity` screenshot was available in the submitted evidence files. The current evidence set contains `Evidence 1.png` to `Evidence 8.png` only.

## 6. Kubernetes Environment Setup

### Step 7: Create and Verify the kind Cluster

A local kind Kubernetes cluster was created and the context was set to `kind-ccse-lab1`.

The cluster list was checked using:

```bash
kind get clusters
```

The output showed the available clusters, including:

- `ccse`
- `ccse-lab1`

The cluster status was verified using:

```bash
kubectl cluster-info --context kind-ccse-lab1
kubectl get nodes
```

The output confirmed that the Kubernetes control plane was running locally and the node `ccse-lab1-control-plane` was in `Ready` status.

![Evidence 5](Evidence%205.png)

### Step 8: Create Development and Production Namespaces

Two namespaces were created for environment separation:

```bash
kubectl create namespace dev
kubectl create namespace prod
```

The namespaces were verified with:

```bash
kubectl get namespace
```

The output showed both `dev` and `prod` as `Active`.

![Evidence 6](Evidence%206.png)

## 7. Kubernetes RBAC Configuration

### Step 9: Create a Service Account in the Development Namespace

A service account named `dev-user` was created in the `dev` namespace.

```bash
kubectl create serviceaccount dev-user -n dev
```

The output confirmed:

```text
serviceaccount/dev-user created
```

![Evidence 7](Evidence%207.png)

### Step 10: Create a Pod Reader Role

A role named `pod-reader` was created in the `dev` namespace. The role grants permission to get, list, and watch pods.

```bash
kubectl create role pod-reader -n dev \
  --verb=get,list,watch \
  --resource=pods
```

The output confirmed:

```text
role.rbac.authorization.k8s.io/pod-reader created
```

![Evidence 7](Evidence%207.png)

### Step 11: Bind the Role to the Service Account

The `pod-reader` role was bound to the `dev-user` service account.

```bash
kubectl create rolebinding dev-user-binding -n dev \
  --role=pod-reader \
  --serviceaccount=dev:dev-user
```

The output confirmed:

```text
rolebinding.rbac.authorization.k8s.io/dev-user-binding created
```

![Evidence 7](Evidence%207.png)

### Step 12: Verify the RoleBinding YAML

The role binding configuration can be verified with:

```bash
kubectl get rolebinding dev-user-binding -n dev -o yaml
```

Expected verification output:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-user-binding
  namespace: dev
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-reader
subjects:
- kind: ServiceAccount
  name: dev-user
  namespace: dev
```

This output verifies that the `dev-user` service account in the `dev` namespace is bound to the `pod-reader` role.

### Step 13: Test RBAC Permissions

The service account identity was assigned to a shell variable:

```bash
SA=system:serviceaccount:dev:dev-user
```

The following authorization checks were performed:

```bash
kubectl auth can-i delete pods -n dev --as=$SA
kubectl auth can-i list pods -n dev --as=$SA
kubectl auth can-i list pods -n prod --as=$SA
```

The results were:

| Test | Result | Explanation |
| --- | --- | --- |
| Delete pods in `dev` | `no` | The role does not include the `delete` verb. |
| List pods in `dev` | `yes` | The role allows `get`, `list`, and `watch` for pods in `dev`. |
| List pods in `prod` | `no` | The role binding applies only to the `dev` namespace. |

![Evidence 8](Evidence%208.png)

## 8. Issues Encountered and Corrections

During the Kubernetes setup, an incorrect command was entered:

```bash
kind get clsysters
```

This returned an error because `clsysters` is not a valid kind subcommand. The command was corrected to:

```bash
kind get clusters
```

During service account creation, a multi-line command was interrupted with `Ctrl+C`, then re-entered correctly. An attempted service account command also included role flags, which caused an `unknown flag: --verb` error. The correct approach was to create the service account first, then create the role separately.

## 9. Conclusion

The lab environment was configured successfully. AWS IAM tasks confirmed user creation, admin group membership, policy attachment, and access key deactivation. Kubernetes RBAC tasks confirmed namespace separation and least-privilege access by allowing the `dev-user` service account to list pods only inside the `dev` namespace while denying unauthorized actions and cross-namespace access.
