# Troubleshooting log (ALB lab)

Format: **symptom, cause, fix, lesson.**

## 1. ALB returned `504 Gateway Timeout`
- **Symptom:** the ALB was Active, but its DNS name showed a 504.
- **Cause:** `backend-sg` only allowed HTTP from my own IP, so the ALB's requests to the servers were dropped and its health checks failed.
- **Fix:** added an inbound HTTP rule on `backend-sg` whose source is the ALB's security group, `alb-sg`. Both targets then turned Healthy.
- **Lesson:** a 504 from a load balancer usually means it cannot reach its targets. Check the target's security group and the target health first.

## 2. Launch wizard defaults would have opened SSH to the internet
- **Symptom (caught before launching):** the wizard's default was a new security group with SSH (port 22) open to `0.0.0.0/0`.
- **Fix:** selected the existing `backend-sg`.
- **Lesson:** read every line of a launch wizard, and use "Preview code" before launching.

## 3. The cloned server would have had no AWS permissions
- **Symptom (caught before launching):** an AMI does not carry the IAM instance profile.
- **Fix:** attached `ec2-backend-role` under Advanced details.
- **Lesson:** the instance profile is a launch setting, not part of the image.

## 4. Stopped server showed `Unused`, not `Unhealthy`
- **Symptom:** after stopping an instance, the target group listed it as Unused.
- **Cause:** the load balancer does not use a stopped instance at all. Unhealthy means a running target is failing its health check.
- **Lesson:** read the "Health status details" column; it gives the reason.
