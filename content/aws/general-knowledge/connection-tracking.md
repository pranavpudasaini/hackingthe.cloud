---
author: "Nick Frichette"
title: "Connection Tracking"
description: "Abuse security group connection tracking to maintain persistence even when security group rules are changed."
enableEditBtn: true
editBaseURL: "https://github.com/Hacking-the-Cloud/hackingthe.cloud/blob/main/content"
---
Security Groups in AWS have an interesting capability known as [Connection Tracking](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html#security-group-connection-tracking). This allows the security groups to track information about the network traffic and allow/deny that traffic based on the Security Group rules.

There are two kinds of traffic flows; tracked and untracked. For example the AWS documentation mentions a tracked flow as the following, "if you initiate an ICMP ping command to your instance from your home computer, and your inbound security group rules allow ICMP traffic, information about the connection (including the port information) is tracked. Response traffic from the instance for the ping command is not tracked as a new request, but rather as an established connection and is allowed to flow out of the instance, even if your outbound security group rules restrict outbound ICMP traffic".

An interesting side effect of this is that tracked connections are allowed to persist, even after a Security Group rule change. 

Let's take a simple example: There is an EC2 instance that runs a web application. This EC2 instance has a simple Security Group that allows SSH, port 80, and port 443 inbound, and allows all traffic outbound. This EC2 instance is in a public subnet and is internet facing.

![Inbound SG](/images/aws/general-knowledge/connection-tracking/inbound-sg.png)

While performing a penetration test you've gained command execution on this EC2 instance. In doing so, you pop a simple [reverse shell](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet). You work your magic on the box before eventually triggering an alert to our friendly neighborhood defender. They follow their runbooks which may borrow from the official AWS [whitepaper](https://d1.awsstatic.com/whitepapers/aws_security_incident_response.pdf) on incident response. 

As part of the "Isolate" step, the typical goal is to isolate the affected EC2 instance with either a restrictive Security Group or an explicit Deny NACL. The slight problem with this is that NACLs affect the entire subnet, and if you are operating in a space with a ton of EC2 instances the defender is unlikely to want to cause an outage for all of them. As a result, swapping the Security Group is the recommended procedure.

The defender switches the Security Group from the web and ssh one, to one that does not allow anything inbound or outbound.

![Change Security Group](/images/aws/general-knowledge/connection-tracking/change-sg.png)

The beauty of connection tracking is that because you've already established a connection with your shell, it will persist. So long as you ran the shell before the SG change, you can continue scouring the box and looking for other vulnerabilities.

![whoami](/images/aws/general-knowledge/connection-tracking/whoami.png)

To be clear, if the restrictive security group doesn't allow for any outbound rules we won't be able to communicate out (and if you're using a beaconing C2 that will not function).

![No Outbound](/images/aws/general-knowledge/connection-tracking/no-outbound.png)