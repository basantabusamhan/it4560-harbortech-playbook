# Week 5: Scaling, Load Balancing, and DNS

## HarborTech Ticket Summary

TKT-2026-0005 is about preparing Riverside Goods for a seasonal traffic increase. The application currently depends on one EC2 instance and one public endpoint. The main concerns are limited capacity and the risk of the single server becoming unavailable.

## Client Impact

The expected traffic increase could cause high CPU usage, slower response times, or application outages. The single EC2 instance is also a single point of failure. If it fails, customers may lose access to the application during the promotion.

## Provided Ticket Evidence

The ticket provided the following evidence:

* Previous promotion traffic reached 92% CPU utilization.
* Upcoming promotion traffic is expected to double.
* The proposed Auto Scaling design is minimum 2, desired 2, and maximum 6.
* The proposed Application Load Balancer has two healthy test targets.
* A secondary recovery endpoint is available.
* Route 53 failover has not been confirmed.

This is provided ticket evidence and was not personally verified in my AWS environment.

## AWS Commands Used

### Auto Scaling

```bash
aws autoscaling describe-auto-scaling-groups \
  --query 'AutoScalingGroups[].{Name:AutoScalingGroupName,Min:MinSize,Desired:DesiredCapacity,Max:MaxSize}' \
  --output table
```

### Target Groups

```bash
aws elbv2 describe-target-groups \
  --query 'TargetGroups[].{Name:TargetGroupName,TargetGroupArn:TargetGroupArn,Protocol:Protocol,Port:Port}' \
  --output table
```

### Route 53

```bash
aws route53 list-hosted-zones \
  --query 'HostedZones[].{Name:Name,Id:Id,Private:Config.PrivateZone}' \
  --output table
```

## AWS Evidence Collected

The Auto Scaling command returned no Auto Scaling groups.

The target group command returned no target groups. Because no target group was available, target health could not be verified.

The Route 53 command returned no hosted zones.

These results represent what I personally observed in my assigned AWS environment.

## Virtualization Connection

Cloud virtualization allows EC2 instances to be added or removed as demand changes. Auto Scaling can provide additional compute capacity, while a load balancer can distribute traffic across multiple instances. This reduces dependence on one server and makes the application more flexible.

## Operational Analysis

The ticket evidence supports additional capacity because CPU previously reached 92% and traffic is expected to double. The proposed 2/2/6 Auto Scaling design could provide more capacity during the promotion. However, my AWS investigation returned no Auto Scaling groups, so I could not verify that configuration.

The ticket also reports two healthy test targets, but my AWS investigation returned no target groups. Therefore, I could not personally verify target health. A load balancer would be needed to distribute traffic across healthy instances.

The ticket states that a secondary recovery endpoint exists, but Route 53 failover has not been confirmed. My AWS investigation returned no hosted zones, so I could not verify DNS failover. The remaining unknowns include the actual Auto Scaling configuration, target health, DNS records, and failover health checks.

## Recommendation

HarborTech should prepare for the expected traffic increase by evaluating the proposed 2/2/6 Auto Scaling design. A load balancer should distribute traffic across healthy instances to reduce the risk of relying on one server. Health checks should be verified to make sure unhealthy targets are removed from normal traffic.

Route 53 failover should not be considered confirmed until the hosted zone, DNS records, health checks, and primary and secondary endpoints are verified. After implementation, HarborTech should monitor CPU usage, instance count, request volume, response time, target health, errors, and endpoint availability. These controls should improve availability while allowing resources to scale based on actual demand.

## Escalation Notes

Production changes to Auto Scaling, the load balancer, target groups, or Route 53 require approval from the appropriate HarborTech team. As an intern, I can document the findings and make recommendations, but I should not make unapproved production changes.

## Lessons Learned

Week 5 taught me that cloud decisions should be based on actual evidence. High CPU can support the need for more capacity, but adding instances does not automatically distribute traffic. Load balancers and health checks help manage traffic and unhealthy resources. I also learned that having a backup endpoint does not prove that DNS failover is configured. AWS results showing no resources are also important evidence and should be documented accurately.

## Professional Vocabulary

**Elasticity:** The ability to add or remove cloud resources as demand changes.

**Scalability:** The ability of a system to handle more workload by increasing its resources.

**Load Balancer:** A service that distributes client traffic across available servers.

**Target Group:** A group of servers that can receive traffic from a load balancer.

**Health Check:** A test that determines whether a server or application is healthy.

**Auto Scaling Group:** A group of EC2 instances that can automatically adjust based on demand.

**Launch Template:** A configuration used to create new EC2 instances with consistent settings.

**Desired Capacity:** The number of instances an Auto Scaling group normally tries to maintain.

**Route 53:** AWS's DNS service used to route users to applications and support failover.

**Failover:** Moving traffic from a primary resource to a backup resource when the primary becomes unavailable.
