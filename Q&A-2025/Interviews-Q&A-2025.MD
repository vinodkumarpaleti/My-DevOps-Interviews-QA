# My DevOps Engineer Interviews Q&A-2025
---

## Total interview rounds given: 1
## Cleared interview rounds: 0
## No. of pages reviewed: 0

---


---
## Qentelli L1 Q&A Wed Jan 8, 2025 2pm – 2:30pm (GMT+5:30)

### Role: AWS DevOps Engineer

### 1. I have an application hosted in the EC2 and I want to connect this application to the RDS securly ? What is the process?

Ans: To securely connect your EC2 application to RDS:

1. **Ensure Same VPC**: Place EC2 and RDS in the same VPC or establish connectivity (e.g., VPC peering).  
2. **Security Groups**: Update the RDS security group to allow EC2's security group on the DB port (e.g., 3306 for MySQL).  
3. **Private Access**: Ensure RDS is in private subnets and use its private endpoint.  
4. **Enable SSL/TLS**: Use SSL/TLS for secure data transmission.  
5. **IAM Auth (Optional)**: Use IAM roles for authentication instead of hardcoded credentials.  
6. **Test Connection**: Confirm with tools like `telnet` or a database client.  
---
### 2. Suppose we have s3 bucket if the development team wants to upload some files in that, what are the best practices to implement this?

Ans: Here are the best practices for enabling a development team to upload files securely and efficiently to an Amazon S3 bucket:


### 1. **Control Access with IAM Policies**
   - Grant the **principle of least privilege** by providing only the necessary permissions (e.g., `s3:PutObject` for uploads).
   - Use IAM roles instead of users, especially for services like EC2 or Lambda.


### 2. **Bucket Policies**
   - Restrict access to the bucket using bucket policies:
     - Allow access only from specific accounts, roles, or IPs.
     - Example bucket policy:
       ```json
       {
         "Version": "2012-10-17",
         "Statement": [
           {
             "Effect": "Allow",
             "Principal": {"AWS": "arn:aws:iam::ACCOUNT-ID:role/ROLE-NAME"},
             "Action": "s3:PutObject",
             "Resource": "arn:aws:s3:::bucket-name/*"
           }
         ]
       }
       ```


### 3. **Use Pre-Signed URLs for Temporary Access**
   - Generate **pre-signed URLs** to allow developers to upload files temporarily without direct S3 permissions.
   - Benefits:
     - Time-limited and secure.
     - Example with AWS CLI:
       ```bash
       aws s3 presign s3://bucket-name/object-key --expires-in 3600
       ```

### 4. **Enable Encryption**
   - Ensure uploaded files are encrypted at rest:
     - Use **S3-managed keys (SSE-S3)** or **AWS KMS (SSE-KMS)**.
     - Enforce encryption with a bucket policy:
       ```json
       {
         "Effect": "Deny",
         "Principal": "*",
         "Action": "s3:PutObject",
         "Resource": "arn:aws:s3:::bucket-name/*",
         "Condition": {
           "StringNotEquals": {"s3:x-amz-server-side-encryption": "aws:kms"}
         }
       }
       ```

### 5. **Versioning**
   - Enable **S3 bucket versioning** to retain previous versions of files in case of accidental overwrites or deletions.


### 6. **Restrict Public Access**
   - Enable **block public access** at the bucket or account level to prevent accidental public file exposure.



### 7. **Monitoring and Logging**
   - Enable **S3 server access logging** or **AWS CloudTrail** to monitor file uploads and access.
   - Use **Amazon CloudWatch** to track metrics and alerts for unusual activity.


### 8. **Multipart Uploads for Large Files**
   - Use **S3 multipart upload** for files larger than 100 MB to improve upload reliability and performance.



### 9. **Data Transfer Optimization**
   - Use **Amazon S3 Transfer Acceleration** for high-speed uploads from remote locations.



### 10. **Automate and Enforce Policies**
   - Use **S3 Object Lifecycle Policies** to automatically move or delete files based on rules.
   - Validate files after upload using checksum mechanisms.
---
### 3. What is the use of pre-signed URL in s3?
Ans: A **pre-signed URL** in Amazon S3 is a secure, temporary URL generated to provide time-limited access to specific objects in an S3 bucket without requiring the user to have direct S3 permissions.

### **Key Uses of Pre-Signed URLs**
1. **Temporary Access to Files**:  
   - Grant temporary access for downloading or uploading files.
   - Useful when you want to restrict access to specific objects for a limited time.

2. **Avoid Direct Bucket Permissions**:  
   - Developers, clients, or users can interact with S3 objects using the URL without needing AWS credentials or permissions.

3. **Secure File Uploads**:  
   - Allow users to upload files directly to S3 without exposing your AWS credentials or API.

4. **Access Control**:  
   - You can specify expiration times and enforce access conditions (e.g., file size, content type) when creating the URL.

5. **File Distribution**:  
   - Share specific files with customers, users, or systems temporarily without making the entire bucket public.

6. **Integration in Applications**:  
   - Useful in APIs or web applications to let users securely interact with S3 objects.

### Example Workflow
1. **Generate a Pre-Signed URL**:  
   The server generates a pre-signed URL for an object.
   ```bash
   aws s3 presign s3://bucket-name/object-key --expires-in 3600
   ```
2. **Client Uses URL**:  
   - For downloads: The client accesses the URL to download the file.
   - For uploads: The client uses the URL to upload a file directly to S3.

3. **Expiration**:  
   The URL automatically expires after the defined duration.

This approach improves security, reduces server load, and streamlines file handling workflows.

---
### 4. Suppose I have n application is configured on top of EC2 instance, so the application needs to access S3, RDS , secreat manager and parameter store etc. So how you are going to enable access these services to that application?

Ans: To securely enable an application running on an EC2 instance to access services like S3, RDS, Secrets Manager, and Parameter Store, you can use **IAM roles** and **instance profiles**. Here's the process:

### 1. **Create an IAM Role for EC2**
   - **IAM Role**: Assign permissions for accessing S3, RDS, Secrets Manager, and Parameter Store.
   - Attach the required policies:
     - **S3 Access**: Add `AmazonS3ReadOnlyAccess` or a custom policy to limit access to specific buckets.
     - **RDS Access**: Add policies for RDS if using RDS APIs for management tasks (e.g., `rds:DescribeDBInstances`). Direct database connections don't require IAM roles.
     - **Secrets Manager Access**: Add policies like `SecretsManagerReadWrite` or custom policies to fetch secrets.
     - **Parameter Store Access**: Add `AmazonSSMReadOnlyAccess` or custom policies to retrieve parameters.

### 2. **Attach the IAM Role to the EC2 Instance**
   - Create an **instance profile** associated with the IAM role.
   - Attach the instance profile to the EC2 instance:
     ```bash
     aws ec2 associate-iam-instance-profile \
       --instance-id i-xxxxxxx \
       --iam-instance-profile Name=YourInstanceProfileName
     ```

### 3. **Use the AWS SDK/CLI in Your Application**
   - Applications running on the EC2 instance can automatically retrieve temporary credentials from the **instance metadata service (IMDS)**.
   - Example:
     ```python
     import boto3

     # Access S3
     s3 = boto3.client('s3')
     s3.list_buckets()

     # Access Secrets Manager
     secrets = boto3.client('secretsmanager')
     secret_value = secrets.get_secret_value(SecretId='your-secret-name')

     # Access Parameter Store
     ssm = boto3.client('ssm')
     parameter = ssm.get_parameter(Name='your-parameter-name', WithDecryption=True)
     ```

### 4. **Enforce Least Privilege**
   - Use specific resource-based policies and restrict access to the required resources only.
   - Example S3 Bucket Policy:
     ```json
     {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Effect": "Allow",
           "Principal": {"AWS": "arn:aws:iam::account-id:role/role-name"},
           "Action": "s3:GetObject",
           "Resource": "arn:aws:s3:::bucket-name/*"
         }
       ]
     }
     ```

### 5. **Secure and Monitor Access**
   - Use **Secrets Manager** or **Parameter Store** for storing sensitive information (e.g., DB credentials).
   - Enable **CloudTrail** and **AWS Config** for auditing.
   - Use **IAM Access Analyzer** to validate permissions.

### 6. **Best Practices**
   - **IAM Role Rotation**: IAM roles automatically rotate credentials.
   - **Instance Metadata Service v2 (IMDSv2)**: Enforce IMDSv2 for enhanced metadata security.
   - **Encryption**: Use KMS for encrypting data in Secrets Manager, Parameter Store, and S3.
---

### 5. I have 3 different servers and different applications are configured in these servers? So I have certain files are stored somewhere.Now these applications needs to use those files, how will you impliment this?

Ans: To share files between applications running on three different servers, you can implement a centralized and scalable file-sharing solution. Here are some approaches:

### **1. Use Amazon S3 for Centralized File Storage**  
- **Steps**:  
  1. Upload the files to an S3 bucket.  
  2. Grant the servers access to the bucket using IAM roles with appropriate permissions (e.g., `s3:GetObject`).  
  3. Applications download or stream the files as needed using the AWS SDK/CLI.  
  4. Optionally, use **S3 Transfer Acceleration** for faster access from remote locations.

- **Advantages**:  
  - Highly scalable, durable, and cost-effective.
  - Access control via IAM, bucket policies, or pre-signed URLs.

### **2. Use Amazon EFS (Elastic File System)**  
- **Steps**:  
  1. Set up an EFS file system and mount it on all three servers.  
  2. Store the shared files in the mounted EFS directory.  
  3. Applications access the files directly from EFS.  

- **Advantages**:  
  - Fully managed, shared storage that can be accessed concurrently by multiple servers.  
  - Supports POSIX-compliant file operations, ideal for applications requiring shared file systems.

### **3. Use NFS (Network File System) on an EC2 Instance**  
- **Steps**:  
  1. Set up an EC2 instance as an NFS server.  
  2. Store the files on the NFS server.  
  3. Configure all three servers to mount the NFS share.  

- **Advantages**:  
  - Provides centralized storage accessible by all servers.  
  - Suitable for small-scale setups.

### **4. Use Amazon FSx**  
- For Windows-based or Lustre workloads:  
  - Use **Amazon FSx for Windows File Server** or **Amazon FSx for Lustre** for shared file storage.  

### **5. Synchronize Files Locally Using AWS DataSync or rsync**  
- **Steps**:  
  1. Use **AWS DataSync** or **rsync** to copy files periodically from the central storage (e.g., S3 or a designated server) to each server.  
  2. Applications access the files locally.

- **Advantages**:  
  - Reduces latency by having files locally available on each server.


### **6. Use a Distributed File System**  
- Use systems like **GlusterFS** or **CephFS** for shared storage across multiple servers.


### Recommendation  
- For simplicity and scalability, **Amazon S3** is often the best option for most use cases. If the applications require frequent file updates or low-latency access, consider **Amazon EFS**.  

### 6. What is Assume role in AWS?

Ans: In AWS, **AssumeRole** is an action that allows an entity (such as an IAM user, IAM role, or AWS service) to temporarily assume a different IAM role and gain the permissions associated with that role. This enables controlled delegation of access and permission management across accounts or services.

### **How AssumeRole Works**:
1. **Creating a Role**: An IAM role is created with specific permissions. The role also defines which entities (users, groups, or services) are allowed to assume the role via a trust policy.
2. **Assuming the Role**: The authorized entity (e.g., an IAM user or an EC2 instance) requests to assume the role using the `sts:AssumeRole` API call. 
3. **Temporary Security Credentials**: If the request is valid, AWS Security Token Service (STS) generates temporary security credentials (access key, secret key, and session token) for the role. These credentials are used to make requests to AWS resources with the permissions of the assumed role.
4. **Session Duration**: The temporary credentials are valid for a limited duration (from a few minutes to several hours), after which they expire.

### **Common Use Cases for AssumeRole**:
- **Cross-Account Access**: Allow an entity in one AWS account to assume a role in another account to access resources (e.g., an EC2 instance in Account A assumes a role to access resources in Account B).
- **EC2 Instance Role**: EC2 instances assume roles to access AWS services (e.g., S3, DynamoDB) securely without embedding long-term credentials.
- **Delegating Permissions**: Allow users or services to assume a role with elevated permissions for specific tasks, without granting those permissions permanently.

### **Example Scenario**:
- **AssumeRole from One Account to Another**:  
  - **Trust Policy**: In Account B, you create a role with a trust policy that allows an IAM user or role in Account A to assume it.
  - **Permissions Policy**: The role in Account B has a permissions policy that grants access to specific resources (e.g., S3).
  - **Assume the Role**: The IAM user or EC2 instance in Account A uses the `sts:AssumeRole` call to get temporary credentials for the role in Account B.

### **Example of AssumeRole API Call** (using AWS CLI):
```bash
aws sts assume-role --role-arn arn:aws:iam::account-id:role/role-name --role-session-name session-name
```

The result will include temporary credentials (access key, secret key, session token) which can be used for subsequent API calls with the permissions of the assumed role.

### **Security Best Practices**:
- Use **least privilege** when assigning permissions to the role.
- Set a **short duration** for temporary credentials to limit the risk of compromised access.
- Use **IAM policies** and **trust policies** to tightly control who can assume the role and under what conditions.

**AssumeRole** is an essential feature for cross-account access, temporary access management, and security in AWS.

---
### 7. Suppose i have a database which is in the private subnet or there is no internet connectivity for the respective subnet now the developer wants to connect with that perticuler rds how you configure this?

Ans: To allow a developer to connect to an **RDS instance** located in a **private subnet** (without direct internet access), you can set up a secure way for them to access the RDS instance. Below are the key steps to configure this:

### **1. Use a Bastion Host (Jump Box)**

A **Bastion Host** (or Jump Box) is an EC2 instance that acts as a gateway to connect to resources in a private subnet. The developer connects to the Bastion Host first, and from there, they can access the RDS instance.

#### Steps:
1. **Launch an EC2 Bastion Host in a Public Subnet**:
   - Create an EC2 instance in a **public subnet** (with a public IP) in the same VPC as the RDS instance.
   - Ensure the Bastion Host has an **SSH** key pair for secure access.

2. **Security Groups**:
   - The Bastion Host security group should allow SSH (port 22) access from the developer's IP address.
   - The RDS instance's security group should allow inbound access on the required database port (e.g., 3306 for MySQL) from the Bastion Host’s security group.

3. **SSH into the Bastion Host**:
   - The developer connects to the Bastion Host using SSH:
     ```bash
     ssh -i your-key.pem ec2-user@bastion-host-public-ip
     ```

4. **Connect to the RDS Instance from the Bastion Host**:
   - From the Bastion Host, the developer can connect to the RDS instance using database client tools (e.g., `mysql`, `psql`, etc.) as if they were on the same private network.

### **2. Set Up AWS Systems Manager Session Manager (SSM)**

If you don’t want to expose an EC2 instance (Bastion Host) for SSH access, you can use **AWS Systems Manager Session Manager** for secure access without needing SSH access.

#### Steps:
1. **Enable SSM on EC2 Instances**:
   - Ensure the Bastion Host or any EC2 instance in the private subnet has the **SSM agent** installed and the **IAM role** attached with the `AmazonSSMManagedInstanceCore` policy.
   
2. **Enable VPC Endpoints**:
   - Ensure that the **VPC Endpoint** for Systems Manager is configured for the VPC, allowing the EC2 instance to communicate with the SSM service without needing internet access.

3. **Use Session Manager**:
   - The developer can use **Session Manager** to start a secure shell session directly into the Bastion Host or EC2 instance in the private subnet. No need to open SSH ports:
     ```bash
     aws ssm start-session --target instance-id
     ```

4. **Connect to RDS**:
   - After accessing the EC2 instance via SSM, the developer can connect to the RDS instance as if they were on the same network.

### **3. Use VPN or Direct Connect**

If the developer is outside the AWS environment, you can set up a **VPN** connection or **AWS Direct Connect** to securely connect the on-premise network or developer's local machine to the private subnet in AWS.

#### Steps:
1. **Set Up a VPN Gateway**:
   - Set up an **AWS Site-to-Site VPN** or **AWS Client VPN** to securely connect the developer’s machine to the VPC.

2. **Configure Routing**:
   - Ensure that the VPN route tables allow traffic to the private subnet where the RDS instance resides.

3. **Security Groups**:
   - Adjust the RDS security group to allow traffic from the VPN network.

4. **Connect to RDS**:
   - The developer can connect to the RDS instance using standard database clients after establishing the VPN connection.

### **4. Use AWS PrivateLink (For RDS in Different Account)**

If the RDS instance is in a different AWS account and the developer is in a different VPC or subnet, you can use **AWS PrivateLink** to securely connect to the RDS instance.

#### Steps:
1. **Set Up VPC Peering or Transit Gateway**:
   - If the RDS instance is in a different VPC, set up **VPC Peering** or use an **AWS Transit Gateway** to route traffic between the VPCs.

2. **Enable PrivateLink**:
   - Configure **AWS PrivateLink** for the RDS service to allow cross-account access through private IP addresses.

3. **Security Groups**:
   - Modify the security groups to allow communication between the developer's VPC and the RDS instance.

### **5. Use IAM Authentication for RDS (Optional)**

If your RDS instance is configured to use **IAM database authentication**, you can set up **IAM roles** to allow secure access to the database.

#### Steps:
1. **Enable IAM Authentication for RDS**:
   - Enable IAM authentication for the RDS instance by setting the `rds.iam_enabled` option.

2. **Create an IAM Role for the Developer**:
   - Attach the appropriate IAM policy to the developer’s IAM user or role, allowing them to authenticate to the RDS instance.

3. **Access via IAM Credentials**:
   - The developer can then authenticate to the RDS instance using temporary credentials generated by the `sts:assumeRole` or `rds:connect` API call.

### **Summary**:
- Use a **Bastion Host** for SSH access to the private subnet.
- Leverage **AWS Systems Manager Session Manager** for SSH-less access.
- Set up a **VPN** or **AWS Direct Connect** for secure access from on-premise or external environments.
- Use **PrivateLink** for inter-VPC or cross-account access.
- Optionally, use **IAM authentication** for secure database access.

Each approach offers different levels of security, ease of management, and use cases based on your architecture and needs.

---

### 8. There is a loadbalancer is configured and is showing unhealthy, what does it mean and how to troubleshoot this?

Ans: When a **Load Balancer** is showing as **unhealthy**, it means that one or more of the backend **targets** (e.g., EC2 instances, containers, etc.) are failing health checks, and as a result, the Load Balancer is unable to route traffic to them successfully. This can occur in both **Classic Load Balancers (CLB)** and **Application Load Balancers (ALB)** in AWS.

### **Troubleshooting Steps for Unhealthy Load Balancer:**

### **1. Check Health Checks Configuration**
   - **Health Check Path**: Ensure that the health check path is correctly configured. For ALB, this is the URL path where the Load Balancer sends health check requests (e.g., `/health`). 
   - **Protocol & Port**: Make sure the health check is configured to use the correct protocol (HTTP/HTTPS/TCP) and port, which should be the port the backend services are listening to.
   - **Thresholds**: Review the thresholds for the health checks, such as **healthy threshold** (number of successful checks required), **unhealthy threshold**, **timeout**, and **interval**. If the values are too strict, it could cause false negatives.

### **2. Check Target Group Health**
   - **View Target Health**: In the AWS console, navigate to the Load Balancer, select the target group, and check the status of each target (e.g., EC2 instance).
     - If the target is unhealthy, it will show as "unhealthy" in the **Target Health** section.
   - **Verify Application Logs**: Check the application logs on the backend targets (EC2 instances, containers, etc.) to determine if there are any errors or issues preventing the application from responding to health checks.

### **3. Verify Network and Security Settings**
   - **Security Group Rules**: Ensure that the security groups associated with the Load Balancer and backend instances allow traffic on the health check port. For example:
     - Load Balancer security group must allow inbound traffic from the health check IP range (i.e., AWS service IP range).
     - EC2 instances should allow inbound traffic from the Load Balancer on the appropriate port (e.g., HTTP/HTTPS port).
   - **NACLs (Network Access Control Lists)**: Ensure that **Network ACLs** are not blocking health check requests. These can sometimes interfere with the traffic between the Load Balancer and backend instances.
## **4. Check Target Instance Health**
   - **Instance State**: Ensure the backend instances are in a healthy state. If an instance is stopped, terminated, or in any other state other than running, it will not pass health checks.
   - **Check Application Logs**: If the backend service is unhealthy, inspect its logs (e.g., web server logs) to check for crashes, configuration issues, or service failures.
   - **Port Binding**: Ensure that the backend application is running and listening on the expected port (e.g., port 80 for HTTP). Use tools like `netstat` or `ss` to verify the application is bound to the right port.

### **5. Review Load Balancer Logs (ALB/ELB)**
   - **Access Logs**: Enable **Access Logs** for the ALB or NLB (Network Load Balancer) to capture detailed requests and responses that the Load Balancer receives, including health check responses.
     - For **ALB**, enable logging and check for any health check errors in the logs.
   - **Error Responses**: If the Load Balancer is receiving error responses (e.g., 500/502/503 errors), this may indicate a problem with the backend service.

### **6. Check Load Balancer Configuration**
   - **Listener Configuration**: Ensure that the Load Balancer listeners are configured to forward traffic to the correct target group and port.
   - **Routing Rules**: If using an ALB with **path-based** or **host-based routing**, verify that the routing rules are set correctly to route traffic to the correct target group.

### **7. Review Auto Scaling Configuration (If Applicable)**
   - If the Load Balancer is associated with **Auto Scaling**, ensure that Auto Scaling is functioning correctly and scaling the number of instances up or down based on health checks. Sometimes, a misconfigured Auto Scaling group might cause unhealthy targets if new instances are not registered with the Load Balancer.

### **8. Investigate Possible Application-Level Issues**
   - **Check for Timeouts**: If the backend application is taking too long to respond to health checks, you might encounter timeouts. Review the application performance and optimize it.
   - **Database or Dependency Issues**: If the backend application relies on external services (e.g., database, API calls), check whether those dependencies are healthy and responsive.

### **Common Errors & Solutions**:
- **Health Check Failing with 404 or 500 Errors**: Verify the health check endpoint is correct and reachable.
- **Target Group shows "Unhealthy" but Application is Running**: Check if security groups or network ACLs are blocking health check traffic, or verify the instance is listening on the correct port.
- **SSL/TLS Health Check Failures**: Ensure that SSL certificates are correctly configured and the Load Balancer is using the correct protocol.

By systematically reviewing the health check configuration, target health, network settings, and application behavior, you can identify and resolve most Load Balancer issues effectively.

---