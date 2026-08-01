# IKB42603 Lab 0: Environment Setup Report

## Course Information

- **Course:** IKB42603 Cloud Computing Security Essentials
- **Lab:** Lab 0 - Environment Setup
- **Name:** Muhammad Aiman
- **No. ID:** 52215124380
- **Date:** 30 July 2026

## 1. Objective

This report documents the environment setup steps completed for Lab 0. The setup follows the provided guide, `IKB42603_Lab0_Environment_Setup_Cheatsheet.pdf`, and verifies the required tools for Docker, AWS CLI, kind, kubectl, OpenSSL, oathtool, LocalStack, and a local Kubernetes cluster.

## 2. Working Environment

- Operating system used: Ubuntu 64-bit virtual machine
- User account shown in evidence: `aimanos@aimanos`
- Lab directory evidence files: `Evidence 1.jpeg` to `Evidence 9.jpeg`

## 3. Docker Installation and Verification

### Step 1: Install Docker

The guide requires Docker to be installed using the official Docker installation script:

```bash
curl -fsSL https://get.docker.com | sh
```

During execution, the installer detected that the `docker` command already existed on the system. The installation warning was acknowledged, and Docker was verified afterward.

### Step 2: Check Docker Version

Command used:

```bash
docker --version
```

Result:

```text
Docker version 29.0.2, build dfc4efb
```

Evidence:

![Docker version verification](Evidence%201.jpeg)

### Step 3: Check Docker Service Status

Command used:

```bash
sudo systemctl status docker
```

Result:

- Docker service was loaded and enabled.
- Docker service status was active and running.
- Docker daemon was listening on `/run/docker.sock`.

Evidence:

![Docker service status](Evidence%202.jpeg)

### Step 4: Run Docker Hello World Container

Command used:

```bash
docker run --rm hello-world
```

Result:

```text
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

This confirms that Docker can pull an image from Docker Hub, create a container, run it, and return output to the terminal.

Evidence:

![Docker hello-world output](Evidence%203.jpeg)

## 4. AWS CLI Installation and Verification

### Step 1: Install AWS CLI v2

The guide lists the AWS CLI v2 installation flow:

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
unzip awscliv2.zip
sudo ./aws/install
```

### Step 2: Verify AWS CLI Version

Command used:

```bash
aws --version
```

Result:

```text
aws-cli/2.36.9 Python/3.14.6 Linux/6.17.0-22-generic exe/x86_64.ubuntu.24
```

Evidence:

![AWS CLI version verification](Evidence%203.jpeg)

## 5. kind Installation and Verification

### Step 1: Install kind

The guide lists the kind installation command:

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

### Step 2: Verify kind Version

Command used:

```bash
kind --version
```

Result:

```text
kind version 0.23.0
```

Evidence:

![kind version verification](Evidence%204.jpeg)

## 6. kubectl Installation and Verification

### Step 1: Verify kubectl Client

Command used:

```bash
kubectl version --client
```

Result:

```text
Client Version: v1.36.3
Kustomize Version: v5.8.1
```

Evidence:

![kubectl client version verification](Evidence%205.jpeg)

## 7. OpenSSL and oathtool Verification

### Step 1: Verify OpenSSL

Command used:

```bash
openssl version
```

Result:

```text
OpenSSL 3.0.13 30 Jan 2024 (Library: OpenSSL 3.0.13 30 Jan 2024)
```

### Step 2: Verify oathtool

Command used:

```bash
oathtool --version
```

Result:

```text
oathtool (OATH Toolkit) 2.6.11
```

Evidence:

![OpenSSL and oathtool verification](Evidence%206.jpeg)

## 8. LocalStack Setup and AWS Endpoint Test

### Step 1: Start LocalStack

The guide lists LocalStack startup with Docker:

```bash
docker run -d --name localstack -p 4566:4566 localstack/localstack
```

### Step 2: Check LocalStack Health

Command used:

```bash
curl http://localhost:4566/_localstack/health
```

Result:

- LocalStack returned a health JSON response.
- Multiple AWS-compatible services were shown as `available`.
- LocalStack version shown in the response: `2026.7.0`.

### Step 3: Configure AWS CLI for LocalStack

Commands used:

```bash
aws configure set aws_access_key_id test
aws configure set aws_secret_access_key test
aws configure set region us-east-1
```

### Step 4: Save LocalStack Endpoint Variable

Command used:

```bash
EP='--endpoint-url=http://localhost:4566'
```

### Step 5: Test AWS STS Against LocalStack

Command used:

```bash
aws $EP sts get-caller-identity
```

Result:

```json
{
  "UserId": "000000000000",
  "Account": "000000000000",
  "Arn": "arn:aws:iam::000000000000:root"
}
```

Evidence:

![LocalStack health and AWS CLI configuration](Evidence%207.jpeg)

### Step 6: Open LocalStack Web Console Resource Browser

The additional guide for LocalStack authentication and the web console was used to open the LocalStack console in the browser.

URL shown:

```text
localhost.localstack.cloud:4566
```

Result:

- LocalStack Resource Browser loaded in the browser.
- Region was set to `us-east-1`.
- Account ID was shown as `000000000000`.
- AWS-compatible service categories were visible, including API Gateway, SQS, SNS, EC2, Lambda, CloudFormation, CloudWatch, IAM, S3, DynamoDB, RDS, and others.

Evidence:

![LocalStack web console resource browser](Evidence%209.jpeg)

## 9. Kubernetes Cluster Setup with kind

### Step 1: Create kind Cluster

The guide lists the cluster creation command:

```bash
kind create cluster --name ccse
```

### Step 2: Check Cluster Information

Command used:

```bash
kubectl cluster-info --context kind-ccse
```

Result:

- Kubernetes control plane was running at `https://127.0.0.1:34389`.
- CoreDNS was running through the Kubernetes API proxy.

### Step 3: Check Cluster Nodes

Command used:

```bash
kubectl get nodes
```

Result:

```text
NAME                 STATUS   ROLES           AGE   VERSION
ccse-control-plane   Ready    control-plane   30h   v1.30.0
```

Evidence:

![kind cluster-info and node verification](Evidence%208.jpeg)

## 10. Cleanup Commands from Guide

The guide includes cleanup commands for stopping and deleting resources when the lab session is finished.

Stop and start LocalStack:

```bash
docker stop localstack
docker start localstack
```

Remove LocalStack container completely:

```bash
docker rm localstack
```

Delete the kind cluster:

```bash
kind delete cluster --name ccse
```

Check running containers and clusters:

```bash
docker ps
kind get clusters
```

## 11. Conclusion

The Lab 0 environment setup was completed successfully. Docker is installed and running, Docker container execution was verified, AWS CLI v2 is available, kind and kubectl are installed, OpenSSL and oathtool are available, LocalStack is running locally on port `4566`, AWS CLI can communicate with LocalStack, the LocalStack web console Resource Browser was opened, and a kind Kubernetes cluster named `ccse` is active with a ready control-plane node.
