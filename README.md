# Secure Bastion & IAM Infrastructure

This project sets up a secure AWS environment where all application servers run in private subnets with no direct public access. All external SSH access is routed through a dedicated bastion host (jump server) in a public subnet, enforcing least-privilege principles. EC2 instances assume IAM roles for AWS service access (avoiding hardcoded credentials) and use a NAT Gateway for controlled outbound internet connectivity.

## 🚀 Features

- **Private Application Servers:** All app servers run in private subnets without public IPs. Outbound internet access is provided by a NAT Gateway, allowing instances to reach external services while preventing unsolicited inbound connections.  
- **Secure Bastion Host:** A dedicated EC2 bastion host acts as the only entry point to the private network. This jump server provides a single controlled gateway for SSH access, enhancing security by isolating the private instances.  
- **IAM Role-Based Access:** EC2 instances are assigned IAM roles instead of embedding credentials. AWS best practices recommend using IAM roles to provide temporary credentials to EC2, eliminating hardcoded AWS keys.  
- **Least-Privilege Security:** Network and IAM policies follow the principle of least privilege. Security groups, route tables, and IAM policies grant only the minimum permissions required, aligning with AWS security best practices.  
- **Infrastructure as Code:** The network and compute resources are defined using infrastructure-as-code (e.g. Terraform), ensuring the architecture is reproducible, version-controlled, and easy to update.

## 🛠️ Tech Stack

- `Amazon VPC` (custom VPC with public and private subnets)  
- `Amazon EC2` (instances including a bastion host and private app servers)  
- `AWS IAM` (Role-based access management for EC2)  
- `Security Groups` (network ACLs for bastion and app servers)  
- `NAT Gateway` (for outbound internet access from private subnets)  
- `SSH` (secure shell for remote access via the bastion host)  
- `Terraform` (infrastructure-as-code, optional)  
- `AWS CLI` (for deployment and management)

## 📦 Installation

1. **Clone the repository**:  
   ```bash
   git clone https://github.com/<username>/<repo-name>.git
   cd <repo-name>
   ```  
2. **Configure AWS credentials** (if not already set up):  
   ```bash
   aws configure
   ```  
   Enter your AWS Access Key, Secret Key, and desired default region when prompted.  
3. **Initialize and deploy the infrastructure**:  
   ```bash
   terraform init
   terraform plan
   terraform apply -auto-approve
   ```  
   This creates the VPC, subnets, security groups, bastion host, and app server according to the code.  
4. **Verify deployment** (optional):  
   ```bash
   aws ec2 describe-instances \
     --filters "Name=tag:Name,Values=Bastion" \
     --query "Reservations[].Instances[].PublicIpAddress" \
     --output text
   ```  
   Use the output Public IP address to SSH into the bastion host as shown below.

## 🚦 Usage

After deployment, use SSH to connect via the bastion host:

```bash
ssh -i "path/to/your-key.pem" ec2-user@<BASTION_PUBLIC_IP>
ssh -i "path/to/your-key.pem" ec2-user@<APP_SERVER_PRIVATE_IP>
```

Replace `<BASTION_PUBLIC_IP>` with the bastion host’s public IP and `<APP_SERVER_PRIVATE_IP>` with the private IP of the application server. This two-hop SSH ensures the private server is never exposed directly to the internet.

## 🤝 Contributing

Contributions are welcome! Please open an issue to report bugs or suggest improvements. Feel free to fork the repository and submit a pull request with a clear description of your changes and the problem it solves.

**Sources:** AWS documentation on bastion hosts, NAT gateways, and IAM roles were used to guide this architecture design.
