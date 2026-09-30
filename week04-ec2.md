# Week 4: EC2 Evidence Lab

## HarborTech Ticket Summary

Ticket: TKT-2026-0004

Riverside Goods reported that an EC2 web server was running but could not be reached through HTTP. The goal of this investigation was to build the instance, collect evidence, create a controlled HTTP failure, identify the supported cause, correct only the affected layer, verify the result, test stop/start behavior, and clean up the lab resources.

The investigation showed that the EC2 instance was healthy and Apache was running. The HTTP connection failed because the security group did not have an inbound TCP port 80 rule. Adding the required rule restored HTTP access.

## Client Impact

The web server was running, but users could not reach the website through the instance's public IPv4 address. The HTTP request timed out from CloudShell.

The issue affected external HTTP access to the test web server. The local Apache service was still working inside the instance.

## Environment and Resource Names

* AWS Region: `us-east-1`
* VPC: Default VPC
* VPC CIDR: `172.31.0.0/16`
* Subnet: `subnet-0a0ced78541fe01e3`
* Availability Zone: `us-east-1d`
* Instance ID: `i-036be4a2c206e46a4`
* Instance type: `t3.micro`
* AMI: `ami-0b245cc5f82576748`
* Operating system: Amazon Linux 2023
* Security group: `week4-ec2-032109`
* Security group ID: `sg-01c241b0b5cb3f3cb`
* IAM instance profile: `LabInstanceProfile`
* Instance name tag: `week4-ec2-evidence`

The EC2 user data installed Apache, enabled the service, started it, and created the test web page.

## AWS Documentation Evidence

AWS documentation explains that security groups act as a firewall for EC2 instances and control inbound and outbound traffic at the instance level. New security groups start with an outbound rule and require inbound rules to allow incoming traffic.

AWS documentation also explains that user data can be used to configure an instance during launch or run a configuration script. This supported using user data to install and start Apache during instance creation.

AWS documentation for the Instance Metadata Service explains that IMDSv2 uses session-oriented requests. A token is created with a PUT request and then included in later GET requests for metadata.

Sources used:

* AWS EC2 Security Groups: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-security-group.html
* AWS EC2 Instance Launch Parameters and User Data: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-launch-parameters.html
* AWS EC2 Instance Metadata Service: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html

## CloudShell Command Record

I first checked the AWS identity and region:

```bash
aws sts get-caller-identity
aws configure get region
```

The AWS identity showed the expected Learner Lab assumed role and the region was `us-east-1`.

I checked the EC2 environment and created the lab security group and instance. The instance was launched as a `t3.micro` using Amazon Linux 2023 and the `LabInstanceProfile`.

The initial instance information was:

```text
InstanceId: i-036be4a2c206e46a4
ImageId: ami-0b245cc5f82576748
State: running
PrivateIpAddress: 172.31.2.244
PublicIpAddress: 100.58.237.34
```

I waited for the EC2 status checks:

```bash
aws ec2 wait instance-status-ok \
  --region us-east-1 \
  --instance-ids "$INSTANCE_ID"
```

I tested HTTP before making any changes:

```bash
curl -v --connect-timeout 10 "http://$PUBLIC_IP"
```

My first attempt failed because `PUBLIC_IP` had not been defined:

```text
curl: (3) URL rejected: No host part in the URL
```

I corrected the variable:

```bash
PUBLIC_IP=100.58.237.34
```

I ran the HTTP test again:

```bash
curl -v --connect-timeout 10 "http://$PUBLIC_IP"
```

The request timed out:

```text
curl: (28) Connection timed out after 10001 milliseconds
```

I then inspected the security group:

```bash
aws ec2 describe-security-groups \
  --region us-east-1 \
  --group-ids "$SG_ID" \
  --query 'SecurityGroups[0].{GroupId:GroupId,GroupName:GroupName,Inbound:IpPermissions}' \
  --output json
```

The inbound rules were empty:

```text
GroupId: sg-01c241b0b5cb3f3cb
GroupName: week4-ec2-032109
Inbound: []
```

After identifying the missing HTTP rule, I added TCP port 80:

```bash
aws ec2 authorize-security-group-ingress \
  --region us-east-1 \
  --group-id "$SG_ID" \
  --ip-permissions '[
    {
      "IpProtocol": "tcp",
      "FromPort": 80,
      "ToPort": 80,
      "IpRanges": [
        {
          "CidrIp": "0.0.0.0/0",
          "Description": "Week 4 HTTP evidence test"
        }
      ]
    }
  ]'
```

The command returned:

```text
Return: true
RuleId: sgr-026e570faba023d77
```

I verified the security group again and confirmed that TCP port 80 was now allowed.

I then retested HTTP:

```bash
curl -v --connect-timeout 10 "http://$PUBLIC_IP"
```

The request returned:

```text
HTTP/1.1 200 OK
Server: Apache/2.4.68 (Amazon Linux)
```

The expected page content was returned.

## Baseline Evidence

Evidence A showed the instance ID and AMI ID. This proved which EC2 instance was being tested and which AMI was used. It did not prove that Apache was running or that HTTP was reachable.

Evidence B showed that the instance had a public IPv4 address of `100.58.237.34`. This proved that an external connection could be attempted using a public address. It did not prove that inbound HTTP traffic was allowed.

Evidence C showed that both the EC2 instance and system status checks passed. This proved that the instance and underlying AWS systems were healthy. It did not prove that Apache or TCP port 80 was working.

Evidence D showed that the security group had no inbound rules. This proved that the security group was not allowing inbound traffic through port 80. It did not by itself prove that Apache was stopped.

Evidence E showed that the external HTTP request timed out. This proved that the external HTTP connection was unsuccessful. Combined with the empty inbound security group rules, it supported the security group as the network-layer cause.

## Root-Cause Analysis

The confirmed root cause was the missing inbound TCP port 80 rule in the EC2 security group.

The evidence supported this conclusion because the instance was running, the AWS status checks passed, the instance had a public IPv4 address, and the security group had no inbound rules. The external HTTP request timed out.

I did not treat the timeout alone as proof that Apache was the problem. I later verified Apache from inside the instance and confirmed that it was running and responding locally.

This created a clear evidence chain:

1. EC2 instance was running.
2. AWS status checks passed.
3. Public IPv4 address existed.
4. Security group had no inbound rules.
5. External HTTP request timed out.
6. Apache responded locally inside the instance.
7. Adding TCP port 80 restored external HTTP access.

## Corrective Action

The smallest supported corrective action was to modify the security group instead of rebuilding the instance.

I added an inbound TCP port 80 rule with source `0.0.0.0/0` for the lab HTTP test.

The rule ID was:

```text
sgr-026e570faba023d77
```

No changes were made to the AMI, instance type, operating system, Apache configuration, or instance itself.

## Verification Evidence

After adding the security group rule, I verified the rule through the AWS control plane.

The HTTP test was then successful:

```text
HTTP/1.1 200 OK
Server: Apache/2.4.68 (Amazon Linux)
```

The returned page contained:

```text
Riverside Goods Web Server
Week 4 EC2 Evidence Lab
Apache is running.
```

This provided both control-plane and workload-path evidence. The AWS CLI confirmed the security group configuration, while curl confirmed that the web application was reachable externally.

## IMDSv2 and Guest Evidence

I connected to the EC2 instance through Session Manager:

```bash
aws ssm start-session \
  --target "$INSTANCE_ID" \
  --region us-east-1
```

Inside the instance, I checked Apache:

```bash
systemctl status httpd --no-pager
```

The service showed:

```text
Active: active (running)
```

I tested Apache locally:

```bash
curl http://localhost
```

The expected Riverside Goods web page was returned.

I then used IMDSv2 to retrieve the instance ID:

```bash
TOKEN=$(curl -sS -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

curl -sS -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id
```

The output was:

```text
i-036be4a2c206e46a4
```

The result matched the EC2 instance ID from CloudShell.

CloudShell AWS CLI commands examined the EC2 resource from the AWS control plane. Session Manager and IMDSv2 provided evidence from inside the guest. The local curl test confirmed that Apache was working inside the instance, which helped rule out the Apache service as the original external connectivity problem.

## Stop/Start Lifecycle Test

Before stopping the instance:

```text
Instance ID: i-036be4a2c206e46a4
Public IPv4: 100.58.237.34
State: running
```

The web page returned the expected Riverside Goods content.

I stopped the instance:

```bash
aws ec2 stop-instances \
  --region us-east-1 \
  --instance-ids "$INSTANCE_ID"
```

I waited until it was fully stopped:

```bash
aws ec2 wait instance-stopped \
  --region us-east-1 \
  --instance-ids "$INSTANCE_ID"
```

I started it again:

```bash
aws ec2 start-instances \
  --region us-east-1 \
  --instance-ids "$INSTANCE_ID"
```

I waited until it was running:

```bash
aws ec2 wait instance-running \
  --region us-east-1 \
  --instance-ids "$INSTANCE_ID"
```

After the start, the instance ID was still:

```text
i-036be4a2c206e46a4
```

The public IPv4 address changed from:

```text
100.58.237.34
```

to:

```text
34.232.105.218
```

I tested the new address:

```bash
PUBLIC_IP=34.232.105.218
curl -v --connect-timeout 10 "http://$PUBLIC_IP"
```

The web page returned HTTP 200 OK and the expected Riverside Goods content.

The EBS-backed instance retained the workload and configuration through the stop/start cycle. The instance identity remained the same, while the auto-assigned public IPv4 address changed.

## Cleanup Evidence

After completing the investigation, I terminated the EC2 instance:

```bash
aws ec2 terminate-instances \
  --region us-east-1 \
  --instance-ids "$INSTANCE_ID"
```

The instance entered the `shutting-down` state.

I waited for termination:

```bash
aws ec2 wait instance-terminated \
  --region us-east-1 \
  --instance-ids "$INSTANCE_ID"
```

The wait completed successfully.

I then deleted the security group:

```bash
aws ec2 delete-security-group \
  --region us-east-1 \
  --group-id "$SG_ID"
```

The command returned:

```text
Return: true
```

A second deletion attempt returned:

```text
InvalidGroup.NotFound: The security group 'sg-01c241b0b5cb3f3cb' does not exist
```

This confirmed that the security group had already been removed.

## Escalation and Change-Control Notes

No escalation was required for this disposable lab environment.

In a production environment, I would not add an inbound TCP port 80 rule from `0.0.0.0/0` without authorization. This change could expose a production workload to the public internet and increase its attack surface.

I would seek approval from the system owner and the appropriate security or change-management team. Before making the change, I would want the current security group configuration documented, evidence of the current application state, and a rollback plan.

The rollback would be to remove the new rule and restore the previous approved security group configuration.

Rebuilding the instance was not justified because the evidence showed that the instance was healthy, Apache was running, and the problem was isolated to the security group's inbound configuration. Changing the supported network layer corrected the problem without replacing a functioning workload.

## Lessons Learned

The main lesson from this investigation was to use evidence to identify the affected layer before making a change.

A running EC2 instance and passing status checks do not automatically mean that a web service is reachable. The instance can be healthy while network access is blocked by a security group.

I also learned to separate control-plane evidence from guest-level evidence. AWS CLI commands showed the state of the EC2 resources, while Session Manager and local curl showed what was happening inside the instance.

The failed HTTP test, empty inbound security group, successful local Apache test, and successful HTTP test after the security group change created a clear troubleshooting chain.

The lifecycle test also demonstrated that EBS-backed workload data persisted through a stop and start, while the auto-assigned public IPv4 address changed.

Finally, cleanup is part of professional cloud operations. The EC2 instance and security group were removed after testing to avoid leaving unnecessary resources behind.

## Professional Vocabulary

* **Control plane:** AWS management layer used to inspect and change resources.
* **Workload path:** The path used by a client to reach the application running on the instance.
* **Security group:** A virtual firewall controlling inbound and outbound traffic for an EC2 instance.
* **Ingress:** Incoming network traffic allowed to a resource.
* **Egress:** Outgoing network traffic allowed from a resource.
* **Root cause:** The confirmed underlying reason for the observed failure.
* **Corrective action:** The specific change made to resolve the confirmed problem.
* **Verification:** Evidence collected after a change to confirm the expected result.
* **EBS-backed instance:** An EC2 instance whose root volume is stored on Amazon EBS.
* **Instance identity:** The unique EC2 instance ID that identifies the instance.
* **Auto-assigned public IPv4:** A public IPv4 address assigned to an instance that can change after a stop and start.
* **IMDSv2:** The session-oriented version of the EC2 Instance Metadata Service.
* **Least change:** Making the smallest supported change necessary to correct the issue.
* **Rollback:** Reversing a change and restoring the previous approved configuration.
* **Change control:** The process used to review and authorize changes to production systems.
* **Evidence-based troubleshooting:** Using observable results to identify the affected layer instead of relying on assumptions.
* **Escalation:** Passing an issue to the appropriate team when authorization, expertise, or additional access is required.
