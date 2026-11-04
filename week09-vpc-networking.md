# Week 9: VPC Networking and Troubleshooting

## HarborTech Ticket Summary

HarborTech ticket TKT-2026-0009 involves an EC2 web application that is running but cannot be reached by internet users. The web service is listening on TCP port 80, but the network evidence does not show a complete path between the internet and the application.

The required traffic flow is an internet client connecting to the EC2 web server through TCP port 80.

## Client Impact

Incorrect routing, subnet configuration, or traffic filtering can prevent users from accessing an application. This can interrupt business operations, delay employees, and affect customer access to services. Even when an application is running correctly, users cannot reach it if the network path is incomplete.

## AWS Services Involved

- **Amazon VPC:** Provides an isolated virtual network for AWS resources.
- **Subnets:** Divide a VPC into smaller network sections.
- **Route Tables:** Determine where network traffic is sent.
- **Internet Gateway:** Provides a connection between a VPC and the public internet.
- **NAT Gateway:** Allows private subnet resources to initiate outbound internet connections.
- **Security Groups:** Control inbound and outbound traffic for resources.
- **Network ACLs:** Filter traffic entering and leaving subnets.
- **CIDR Blocks:** Define IP address ranges for networks.
- **Public and Private Subnets:** Separate resources based on their intended network access.
- **VPC Flow Logs:** Record information about network traffic to help troubleshoot connectivity problems.

## Virtualization Connection

VPC networking uses virtualization to create software-defined network boundaries, subnets, routes, gateways, and security controls. These features perform many of the same functions as physical networking equipment. Instead of relying on physical routers and firewalls, AWS allows administrators to configure these network components through software. However, traffic still needs a valid route, correct addressing, and permission through security controls.

## Evidence Reviewed

The following information came from the simulated evidence packet provided in the assignment. These values were not collected from my AWS Learner Lab.

- VPC ID: vpc-0a91
- VPC CIDR: 10.42.0.0/16
- Subnet ID: subnet-0a91
- EC2 public IPv4 address: 198.51.100.24
- EC2 instance status: Running
- Web service: Listening on TCP port 80
- Associated route table: rtb-0a91
- Route shown: 10.42.0.0/16 pointing to local
- Internet Gateway: igw-0a91 attached to the VPC
- Security Group: sg-0a91
- Inbound security group rule: TCP port 80 from 0.0.0.0/0 allowed
- Network ACL: Inbound TCP 80 allowed
- Network ACL outbound rule: TCP ports 1024-65535 allowed
- Required application port: TCP 80
- Full routing and security rules: Not provided
- VPC Flow Logs: Not provided
- Connectivity test results: Not provided

## Operational Analysis

The required network path is from an internet client to the EC2 web server at public IPv4 address 198.51.100.24 using TCP port 80.

The instance is running, and the web service is listening. The VPC also has an attached Internet Gateway, and the instance has a public IPv4 address.

However, the associated route table only shows the local VPC route for 10.42.0.0/16. It does not show a default route of 0.0.0.0/0 pointing to the Internet Gateway.

The first unsupported assumption is that the subnet is public simply because the instance has a public IP address and the VPC has an attached Internet Gateway. A public subnet also needs a route to the Internet Gateway.

The security group allows inbound TCP port 80, and the selected network ACL rules allow the required inbound and outbound traffic. These findings suggest that the security controls support the required connection. However, the complete rules are not provided, so filtering cannot be completely ruled out.

The most likely blocking layer is routing because the evidence does not show a default route to the Internet Gateway.

## Recommendation

I recommend checking the complete route table associated with the subnet to confirm whether the default internet route is missing. If the instance is intended to host a publicly accessible web application, the route table should include 0.0.0.0/0 pointing to igw-0a91.

Any route correction should be reviewed and approved by the network administrator before being made.

After an approved correction, connectivity should be tested from an external client by accessing the server through its public IPv4 address using HTTP. The administrator should confirm that the application responds successfully. If the connection still fails, the complete security group and network ACL rules should be reviewed. VPC Flow Logs could also help identify rejected traffic.

The remaining uncertainty is that the evidence packet does not include every security rule, the complete route configuration, or actual connectivity test results. Therefore, the missing internet route is the most likely cause based on the available evidence, but it has not been confirmed through a live test.

## Escalation Notes

Production changes involving route tables, Internet Gateways, NAT Gateways, security groups, network ACLs, or network architecture require approval from the appropriate administrator. As an intern, I can review the evidence, identify possible problems, and recommend corrections. I should not make production network changes without approval.

No unrelated network settings should be changed during this investigation.

## Lessons Learned

This assignment showed me that having a public IP address and an attached Internet Gateway does not automatically mean an EC2 instance is reachable from the internet. The route table must also provide a valid path. I also learned that security groups and network ACLs control traffic at different levels, so both need to be reviewed when troubleshooting. The best approach is to follow the expected traffic path, identify where the evidence stops supporting connectivity, and recommend only the changes needed to address the problem.

## Professional Vocabulary

- **VPC:** A logically isolated virtual network in AWS where cloud resources communicate.
- **Subnet:** A smaller network section created within a VPC.
- **Route Table:** A collection of rules that determines where network traffic goes.
- **Internet Gateway:** An AWS component that allows communication between a VPC and the internet when properly configured.
- **NAT Gateway:** A service that allows resources in a private subnet to connect to the internet without accepting unsolicited inbound connections.
- **Security Group:** A stateful virtual firewall that controls allowed traffic for associated resources.
- **Network ACL:** A stateless traffic filter that controls inbound and outbound traffic at the subnet level.
- **CIDR:** A way to represent an IP address range and its network size.
- **Public Subnet:** A subnet whose route table includes a route to an Internet Gateway.
- **Private Subnet:** A subnet without a direct route to an Internet Gateway. It can use a NAT Gateway when outbound internet access is needed.
