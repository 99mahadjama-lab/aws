# AWS Custom VPC — Cloud Networking From Scratch

## Objective
Build a custom AWS VPC with public and private subnets, configure correct routing for internet access, and deploy EC2 instances across both subnets. As a bonus, deploy a Bastion Host to securely access the private instance.

---

## Knowledge Demonstrated
- VPC architecture and subnet segmentation
- Internet Gateway and NAT Gateway configuration
- Route table design for public and private traffic flows
- Security group rules and least privilege access
- SSH agent forwarding for secure Bastion Host access

---

## Steps

### 1. Create the VPC
Create a custom VPC with CIDR block `10.0.0.0/16`. Then create two subnets — a public subnet (`10.0.16.0/20`) and a private subnet (`10.0.0.0/20`).

<img width="2944" height="396" alt="image" src="https://github.com/user-attachments/assets/19ab1ece-b292-4a98-aa48-7bfa02eb6ff0" />

### 2. Internet Access
Create and attach an Internet Gateway to the VPC. Then create a NAT Gateway inside the public subnet — this requires an Elastic IP, which must be allocated before creation.

> Note: Elastic IPs must be disassociated before they can be released. Deleting the NAT Gateway handles this.

<img width="3354" height="1140" alt="image" src="https://github.com/user-attachments/assets/56fae12f-3f4f-4575-87f3-5a251b6f84bc" />


### 3. Route Tables
Create two route tables:
- **Public**: local route + `0.0.0.0/0` → Internet Gateway
- **Private**: local route + `0.0.0.0/0` → NAT Gateway

Explicitly associate each route table with its respective subnet.

<img width="1296" height="1236" alt="image" src="https://github.com/user-attachments/assets/e0c1a4ae-8534-4f3f-81e7-428a4ddca457" />


### 4. Security Groups
- **Public EC2 SG**: allow SSH (22) and HTTP (80) inbound from your IP only
- **Private EC2 SG**: allow SSH inbound from the public EC2's security group (not an IP address), outbound allow all

> Referencing a security group as a source rather than an IP is more secure and resilient to IP changes.

### 5. Launch EC2 Instances
- **Public EC2**: launch in public subnet, enable auto-assign public IP, assign public SG
- **Private EC2**: launch in private subnet, no public IP, assign private SG

> Always assign a key pair at launch — it cannot be added afterwards.

📸 *Screenshot: Both instances running in EC2 dashboard*

### 6. Bastion Host Access
Use the public EC2 as a Bastion Host to reach the private instance. Add your key to the SSH agent locally and connect using agent forwarding — this avoids copying your private key onto the bastion.

```bash
ssh-add ~/path/to/key.pem
ssh -A ec2-user@<public-ip>
ssh ec2-user@<private-ip>
```

<img width="1244" height="792" alt="image" src="https://github.com/user-attachments/assets/cad8ae5b-8027-4676-a09a-fa264af1b5e1" />


### 7. Verify Connectivity
Confirm outbound internet access from the private instance via the NAT Gateway.

```bash
ping google.com
```

---

## Issues Faced
- Launched an instance without a key pair — had to terminate and relaunch
- Private instance couldn't reach the internet — outbound SG rules were restricted to VPC CIDR only, needed `0.0.0.0/0`
