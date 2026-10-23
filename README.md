# AWS Three-Tier VPC Architecture

## Project Overview

This project demonstrates the setup and validation of a three-tier network architecture on Amazon Web Services (AWS). The architecture separates the Web, Application, and Database tiers using public and private subnets, route tables, an Internet Gateway, a NAT Gateway, and security groups.

The exercise focused on networking, access control, routing, and connectivity testing. No web application or database service was deployed.

## Objectives

- Create a custom VPC in AWS.
- Configure six subnets across two Availability Zones.
- Enable internet access for public subnets through an Internet Gateway.
- Enable outbound internet access for private subnets through a NAT Gateway.
- Configure security groups to control communication between tiers.
- Launch EC2 instances as placeholders for the Bastion, Web, App, and Database tiers.
- Test SSH access, inter-tier connectivity, and outbound internet access.

## Architecture Configuration

| Component | Configuration |
|---|---|
| AWS Region | `us-east-1` |
| VPC Name | `Devopsghost-ThreeTier-VPC` |
| VPC CIDR | `10.0.0.0/16` |
| Internet Gateway | `Devopsghost-ThreeTier-IGW` |
| Public Route Table | `Devopsghost-Public-RT` |
| Private Route Table | `Devopsghost-Private-RT` |
| NAT Gateway | One NAT Gateway |
| EC2 Instance Type | `t3.micro` |
| Operating System | Ubuntu Server |
| Number of EC2 Instances | 4 |

## Subnet Configuration

The VPC contains six subnets distributed across two Availability Zones.

| Tier | Subnet Name | CIDR Block | Availability Zone |
|---|---|---|---|
| Web | `Web-Public-A` | `10.0.1.0/24` | `us-east-1a` |
| Web | `Web-Public-B` | `10.0.2.0/24` | `us-east-1b` |
| App | `App-Private-A` | `10.0.11.0/24` | `us-east-1a` |
| App | `App-Private-B` | `10.0.12.0/24` | `us-east-1b` |
| Database | `DB-Private-A` | `10.0.21.0/24` | `us-east-1a` |
| Database | `DB-Private-B` | `10.0.22.0/24` | `us-east-1b` |

Public IPv4 address assignment was enabled for the two Web subnets. Automatic public IPv4 assignment was disabled for the private App and Database subnets.

## Network Routing

### Public Route Table

The public route table, `Devopsghost-Public-RT`, is associated with both Web subnets.

| Destination | Target |
|---|---|
| `10.0.0.0/16` | Local |
| `0.0.0.0/0` | Internet Gateway |

This configuration allows resources in the public subnets to communicate with the internet, subject to their security-group rules and public IP configuration.

### Private Route Table

The private route table, `Devopsghost-Private-RT`, is associated with all four private subnets.

| Destination | Target |
|---|---|
| `10.0.0.0/16` | Local |
| `0.0.0.0/0` | NAT Gateway |

The NAT Gateway allows instances in the private subnets to initiate outbound internet connections without requiring public IPv4 addresses.

A single NAT Gateway was used to reduce costs for the lab. This introduces an Availability Zone dependency and does not provide the same resilience as deploying a NAT Gateway in each Availability Zone.

## Security Groups

Three security groups were configured to control traffic between the tiers.

### Web-SG
- SSH (TCP 22) restricted to the administrator's IP address.
- HTTP (TCP 80) allowed from the internet.
- ICMP allowed for connectivity testing.

### App-SG
- SSH (TCP 22) permitted according to the configured access rules.
- ICMP allowed for connectivity testing.
- TCP port `8080` allowed from `Web-SG`.

### DB-SG
- SSH (TCP 22) permitted according to the configured access rules.
- ICMP allowed from `App-SG`.
- MySQL (TCP 3306) allowed from `App-SG`.

The database security group does not allow traffic directly from `Web-SG`, enforcing the intended separation between the Web and Database tiers.

## EC2 Instances

Four EC2 instances were used to represent the architecture's components.

| Instance | Role | Subnet |
|---|---|---|
| `Bastion` | Administrative access to private instances | `Web-Public-A` |
| `Web-VM` | Web tier placeholder | `Web-Public-B` |
| `App-VM` | Application tier placeholder | `App-Private-A` |
| `DB-VM` | Database tier placeholder | `DB-Private-B` |

The private instances were accessed through the Bastion using SSH from Git Bash. The SSH `ProxyCommand` method was used to reach private instances without copying the private key onto the Bastion.

## Connectivity Testing

The following tests were completed.

| Test | Expected Behaviour | Result |
|---|---|---|
| Local machine to Bastion using SSH | Connection succeeds | Passed |
| Local machine to App-VM through Bastion | Connection succeeds | Passed |
| Web-VM to App-VM using ping | Connectivity permitted | Passed — 4/4 replies |
| App-VM to DB-VM using ping | Connectivity permitted | Passed |
| Web-VM to DB-VM using ping | Direct ICMP blocked by DB-SG | Passed — 0/4 replies |
| Web-VM outbound HTTPS using `curl` | Internet access available | Passed — HTTP 200 |
| App-VM outbound HTTPS using `curl` | Internet access through NAT | Passed — HTTP 200 |

These tests confirmed the intended connectivity between the Web and Application tiers, between the Application and Database tiers, and the blocking of direct Web-to-Database ICMP traffic.

## Short Report

### What worked?

The VPC and its six subnets were configured successfully across two Availability Zones. The Internet Gateway, NAT Gateway, route tables, and security groups were set up. SSH access through the Bastion worked, the intended inter-tier connectivity tests passed, direct Web-to-Database ping was blocked, and both Web-VM and App-VM successfully accessed the internet.

### What was the hardest part?

The hardest part was configuring SSH access to private instances through the Bastion using Git Bash and verifying that routing and security-group rules permitted the required connections while blocking unauthorized traffic.

### Why do we use separate private subnets for App and DB?

Separate private subnets provide network segmentation between the Application and Database tiers. This allows different security rules and access controls to be applied to each tier, reduces unnecessary direct access to the database, and improves security and manageability. Deploying the tiers across multiple Availability Zones can also support resilience.

## Conclusion

The lab successfully demonstrated the core networking principles of a three-tier AWS architecture, including subnet segmentation, public and private routing, Bastion-based SSH access, security-group enforcement, and outbound internet access through a NAT Gateway.

The architecture uses one NAT Gateway to reduce lab costs, so private-subnet internet egress is not fully resilient to an Availability Zone failure. The EC2 instances served as placeholders; no application or database software was deployed.