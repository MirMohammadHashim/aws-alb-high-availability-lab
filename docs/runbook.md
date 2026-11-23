# Runbook: rebuild the ALB lab

Prerequisite: the Credit Book backend is running on one EC2 instance in a public subnet, the RDS database is available, and both are reachable as described in the [main project](https://github.com/MirMohammadHashim/credit-book). Everything below is done in the AWS Console, region `ap-southeast-2`.

> Check current prices before you start. The ALB and extra server bill by the hour while they exist.

## 1. Create the load balancer's firewall
EC2 > Security Groups > Create. Name `alb-sg`, VPC `credit-book-vpc`, inbound **HTTP 80 from My IP**.

## 2. Make an AMI of the working server
EC2 > Instances > select the backend > Actions > Image and templates > Create image. Name `credit-book-backend-ami`. Leave "No reboot" unchecked (the server reboots once; the container restarts by itself). Wait until the AMI is **Available**.

## 3. Create the target group
EC2 > Target Groups > Create. Type **Instances**, name `credit-book-tg`, HTTP 80, VPC `credit-book-vpc`, health check path `/api/health`. Register the first server.

## 4. Launch the second server from the AMI
- Name `credit-book-backend-2`, same instance type as the first.
- No key pair (access is through Session Manager).
- Network: `credit-book-vpc`, the public subnet in the **other** Availability Zone, auto-assign public IP **enabled**.
- Firewall: **select existing** `backend-sg` (do not let the wizard create a new group with SSH open).
- Advanced details > IAM instance profile: `ec2-backend-role` (an AMI does not carry it).
- Check "Preview code" before launching. Wait for **2/2 checks passed**, then open `http://<public-ip>` to confirm it works.

## 5. Register the second server
Target group > Targets > Register targets > tick the new instance > **Include as pending below** > **Register pending targets**.

## 6. Create the ALB
EC2 > Load Balancers > Create > Application Load Balancer. Name `credit-book-alb`, internet-facing, IPv4, VPC `credit-book-vpc`, both Availability Zones with their **public** subnets, security group `alb-sg` only (remove `default`), listener HTTP:80 forwarding to `credit-book-tg`. Wait until **Active**.

## 7. Let the ALB reach the servers
`backend-sg` > Edit inbound rules > add **HTTP 80, source `alb-sg`**. Without this the ALB returns 504.

## 8. Test
- Open `http://<alb-dns-name>` and use the app.
- Target group > Targets: both servers **Healthy** (about 2.5 minutes after registering).
- Stop one server (Instance state > Stop, not Terminate). The target group shows it as **Unused**. Start it again and watch it return to Healthy.

## 9. Teardown (in this order)
1. Delete the ALB.
2. Delete the target group.
3. Terminate the second server.
4. Remove the `alb-sg` rule from `backend-sg`.
5. Delete `alb-sg` (fails if step 4 is not saved).
6. Deregister the AMI and delete its snapshot.
7. Check: only the original server is left, and Load Balancers, Target Groups, AMIs and Snapshots are empty.
