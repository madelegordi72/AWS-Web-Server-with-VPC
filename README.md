# Create a VPC Then Launch Web Server

This project builds a custom Amazon VPC with a public and a private subnet, an internet gateway, and a NAT gateway. It then creates a security group that allows HTTP and launches an EC2 web server into the public subnet, where a startup script installs Apache, PHP, and a web application.

**Skill:** Web Server Launch inside Virtual Private Cloud
**Course:** IT 442 Cloud Infrastructure, George Mason University (Lab 3: Build Your VPC and Launch a Web Server)

## Architecture

Internet → Internet gateway (`lab-igw`) → Public subnet `lab-subnet-public1-us-east-1a` (10.0.0.0/24) → Web Server 1 (EC2, `t2.micro`, Amazon Linux 2023)

Private subnet `lab-subnet-private1-us-east-1a` (10.0.1.0/24) routes outbound traffic through the NAT gateway `lab-nat-public1-us-east-1a`.

| Component | Value |
|---|---|
| VPC | `lab-vpc`, 10.0.0.0/16 |
| Region | N. Virginia (us-east-1) |
| Availability Zone | us-east-1a |
| Public route table | `lab-rtb-public`: 10.0.0.0/16 → local, 0.0.0.0/0 → internet gateway |
| Private route table | `lab-rtb-private1-us-east-1a`: 10.0.0.0/16 → local, 0.0.0.0/0 → NAT gateway |

## Steps

### Task 1: Create the VPC
1. Opened the VPC console in N. Virginia (us-east-1).
2. Chose **Create VPC** and selected **VPC and more**.
3. Configured the settings:
   - Name tag auto-generation: `lab`
   - IPv4 CIDR block: 10.0.0.0/16
   - Availability Zones: 1
   - Public subnet CIDR (us-east-1a): 10.0.0.0/24
   - Private subnet CIDR (us-east-1a): 10.0.1.0/24
   - NAT gateways: In 1 AZ
   - VPC endpoints: None
   - DNS hostnames and DNS resolution: enabled
4. Confirmed the preview, then chose **Create VPC** and waited for all resources to finish provisioning, including the NAT gateway.

### Task 2: Create a VPC security group
1. Created the security group `Web Security Group` with the description "Enable HTTP access" in `lab-vpc`.
2. Added one inbound rule: **HTTP**, source **Anywhere-IPv4**, description "Permit web requests."

### Task 3: Launch the web server instance
1. Launched an EC2 instance named **Web Server 1**.
2. AMI: Amazon Linux 2023. Instance type: `t2.micro`.
3. Network settings:
   - VPC: `lab-vpc`
   - Subnet: `lab-subnet-public1-us-east-1a` (public, not private)
   - Auto-assign public IP: Enable
   - Security group: existing `Web Security Group`
4. Kept the default storage (8 GiB gp3 root volume).
5. Added this user data script under Advanced details:

```bash
#!/bin/bash
# Install Apache Web Server and PHP
dnf install -y httpd wget php mariadb105-server
# Download Lab files
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CURTF-100-ACCLFO-2/2-lab2-vpc/s3/lab-app.zip
unzip lab-app.zip -d /var/www/html/
# Turn on web server
chkconfig httpd on
service httpd start
```

6. Launched the instance without a key pair, then waited for **2/2 checks passed**.
7. Opened the instance's public IPv4 DNS name in a browser. The page showed the AWS logo and instance metadata, including the instance ID, the Availability Zone (us-east-1a), and current CPU load.

### Task 4: Verification
- **VPC:** `lab-vpc` appears in the Your VPCs list, available.
- **Elastic IP:** One Elastic IP is allocated, which the NAT gateway uses.
- **Security group:** `Web Security Group` has one inbound HTTP rule from anywhere (IPv4).
- **Instance:** Web Server 1 is running in us-east-1a with a public IPv4 address.
- **Web page:** Loaded over HTTP from the instance's public IP and showed the instance metadata.

## Notes

- The security group allows HTTP from anywhere (`0.0.0.0/0`). This is fine for a lab but would be restricted in production.
- Instance IDs, IP addresses, DNS names, VPC and subnet IDs, and the account number have been redacted.

## Cleanup

Deleted the resources in this order to avoid charges:
1. Terminated the EC2 instance
2. Deleted the NAT gateway
3. Detached and deleted the internet gateway
4. Released the Elastic IP
5. Deleted the public and private subnets
6. Deleted both route tables
7. Deleted the VPC
