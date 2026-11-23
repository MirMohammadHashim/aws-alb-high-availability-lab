# Interview notes: ALB lab

Short answers I should be able to give in my own words.

**What is a target group?**
The list of servers an ALB sends traffic to, plus the rule for checking whether each one is healthy.

**Why does an ALB need two Availability Zones?**
An ALB must be placed in subnets in at least two Availability Zones, so one zone failing does not take the load balancer down.

**What did the health check do?**
It called `GET /api/health` on each server. A server needed 5 passes in a row to be Healthy and 2 failures to be Unhealthy. Unhealthy servers stop receiving traffic.

**Why did I get a 504, and how did I fix it?**
`backend-sg` only allowed my IP, so the ALB could not reach the servers. I added a rule allowing HTTP from the ALB's security group.

**Why use a security group as the source instead of an IP range?**
The ALB's IP addresses can change. Referencing its security group stays correct no matter which IPs it uses, and only the ALB (not the internet) can reach the servers on that rule.

**Unused versus Unhealthy?**
Unused: the target is not in use, for example a stopped instance. Unhealthy: the target is running but failing health checks.

**Why did the app work on two servers with no code changes?**
Login uses JWT, so any server can verify a token. Both servers share the same secrets and database, so no sticky sessions were needed.

**Why an AMI for the second server?**
It makes the second server identical to the first and starts in minutes. An AMI does not include the IAM role, so I attached it at launch.

**What would I improve?**
Put the servers in private subnets (needs a NAT Gateway or VPC endpoints), add HTTPS with ACM and a domain, use an Auto Scaling group and launch template instead of a hand-made second server, and rebuild it in Terraform.

**What are the limits of this lab?**
It was a one-time hand-built test, both servers had public IPs, it was HTTP only, and I deleted it afterwards.
