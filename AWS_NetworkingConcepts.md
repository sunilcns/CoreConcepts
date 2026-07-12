Bastion Hosts:
A bastion host is a secure “jump server” in a public subnet that lets administrators access private resources in AWS. It’s part of a layered security model but is increasingly replaced by SSM Session Manager for improved security and simplicity.
Internet → Bastion Host (Public Subnet) → Private EC2 Instances (Private Subnet)
A bastion host must be reachable from the internet so administrators can SSH/RDP into it.
To do that, the instance needs:
	• a public IP or Elastic IP
	• a route to the Internet Gateway (IGW)

You SSH from your laptop to the Bastion Host
Example: ssh -i mykey.pem ec2-user@<bastion-public-ip>

From inside the Bastion Host, you SSH into a private EC2
Example: ssh ec2-user@10.0.3.45
Private EC2 has no public IP, so it cannot be reached directly from the internet.


--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

AWS Systems Manager Session Manager is a secure, agent-based way to access your compute resources without exposing them to the internet. It's the modern alternative to bastion hosts and SSH access, providing better security, auditing, and control.
it uses the SSM Agent + IAM permissions to create an encrypted session directly from the AWS console or CLI. SSM agent is nowadays comes with default AMI'S 
Session Manager provides:

✔ Browser-based shell access (via AWS Console)
✔ CLI-based shell access (via AWS CLI + SSM plugin)
✔ Port forwarding (secure tunnels without opening ports)
✔ Secure remote management
✔ Complete audit logging
✔ IAM-controlled access

<img width="877" height="432" alt="image" src="https://github.com/user-attachments/assets/4f7f1e21-cbe4-4e31-ae3f-e02d706e24a9" />



--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Subnets 
• Public subnet: host Load Balancer and Bastion host ( open )
• Private subnet: host App servers and Database ( restricted ) 


--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
An Internet Gateway (IGW) is a VPC-level component that allows your resources (like EC2 instances) to:
✔️ Connect to the Internet
✔️ Receive traffic from the Internet
Think of it as the door between your AWS VPC and the public internet. 

NAT Gateway :
A managed AWS service that enables outbound-only internet access for private subnets, without exposing instances to inbound internet traffic.

Feature	Internet Gateway	NAT Gateway
Purpose	Internet access for public subnets	Internet access for private subnets
Traffic Direction	Inbound + Outbound	Outbound only
Where Used	Public subnet route table	Private subnet route table
Route	0.0.0.0/0 → IGW	0.0.0.0/0 → NAT Gateway
Requires Public IP on EC2?	Yes	No
Security	Exposes instance publicly	Keeps instance private
Billing	Free	Expensive (hourly + data)



--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Component	Purpose
Internet Gateway (IGW)	Allows public subnets to send & receive internet traffic
NAT Gateway	Allows private subnets to access the internet (outbound only)
Bastion Host	Secure jump-server for SSH/RDP into private EC2 instances
Private Subnet	Internal servers (DB, backend apps) not exposed to internet
Public Subnet	Only host things that must be reachable from the internet


--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

VPC Endpoint
A VPC Endpoint is a private connection between your VPC and AWS services without requiring internet, NAT, or VPN. VPC Endpoints allow your private subnet instances to access AWS services. 
“A DynamoDB/S3 VPC Endpoint allows private connectivity from a VPC to DynamoDB/S3 using AWS’s internal network, eliminating the need for internet access, NAT Gateway, or public endpoints, while improving security and reducing cost.”

Gateway Endpoints (FREE)
	• S3
	• DynamoDB
Interface Endpoints (PAID) - ENI Elastic Network Interface
	• Used for 100+ AWS services like
		○ SSM
		○ Secrets Manager
		○ SQS
		○ SNS
		○ CloudWatch
		○ KMS
		○ API Gateway
		○ Lambda

If there is no s3/dynamodb endpoints. Then we have to create NAT gateway for the private subnet which exposes for limited internet access which adds the cost. Instead we can use vpc endpoints s3/dynamodb(free).internet is not required and traffic retains within aws 
Interface Endpoints are VPC Endpoints based on AWS PrivateLink. They cost money because they create ENIs inside your VPC and handle private communication with AWS services, unlike Gateway Endpoints which use route tables and are free.

<img width="506" height="527" alt="image" src="https://github.com/user-attachments/assets/eb2f9a48-ff6a-4557-b5de-e99a45d843c1" />

<img width="812" height="353" alt="image" src="https://github.com/user-attachments/assets/072530b4-a173-4ae0-9b47-bdccc5b77ba7" />

 INTERFACE END points:
 
 <img width="1702" height="517" alt="image" src="https://github.com/user-attachments/assets/0a72b84c-72e4-4d12-8e92-7b3e8aff4257" />

EC2 works without Interface Endpoints.
“Interface Endpoints are NOT needed for every EC2 or Lambda. We create them only when private subnet resources must access AWS services without internet or NAT Gateway.”
You only create Interface Endpoints if your EC2 (especially in a private subnet) needs to access AWS services WITHOUT using a NAT Gateway or Internet Gateway, such as: SSM (Session Manager), CloudWatch Logs, CR (to download images), SQS / SNS

If EC2 needs none of these privately → no endpoint required.
If EC2 is in a public subnet → definitely no endpoint required.

Lambda calling S3, DynamoDB, SQS, SNS — normally works without endpoints. but needed when Lambda is inside a VPC (especially private subnet)

You create Interface VPC Endpoints ONLY IF:
✔️ You have workloads in private subnets
✔️ They need access to AWS services
✔️ You want to avoid public internet + NAT Gateway

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
What is a Site-to-Site VPN
Site-to-Site VPN is a secure, encrypted network connection that connects:
	🏢 Your on-premise (office) network
	↔
	☁️ Your AWS VPC network
	
A Site-to-Site VPN connection consists of the following components:
A virtual private gateway or a transit gateway


Virtual Private Gateway (VGW)
• A virtual private gateway is the Site-to-Site VPN Concentrator on the Amazon side of the Site-to-Site VPN connection. You create a virtual private gateway and attach it to a virtual private cloud (VPC) with resources that must access the Site-to-Site VPN connection.
A Virtual Private Gateway is an AWS component that allows your on-premises network to connect to your VPC using a VPN connection or Direct Connect.  Virtual private gateways do not support IPv6 for Site-to-Site VPN connections. If you need IPv6 support, use a transit gateway or Cloud WAN for your VPN connection.


• 



Transit gateway
• A transit gateway is a transit hub that you can use to interconnect your VPCs and your on-premises networks. For more information, see Amazon VPC Transit Gateways. You can create a Site-to-Site VPN connection as an attachment on a transit gateway. Your Site-to-Site VPN connection on a transit gateway can support IPv4 or IPv6 traffic inside the VPN tunnels (inner IP addresses). Additionally, transit gateways support IPv6 addresses for the outer tunnel IP addresses

<img width="830" height="360" alt="image" src="https://github.com/user-attachments/assets/a570012f-4196-472b-b6c2-82a241c0acd5" />



• IGW exposes your VPC to the Internet
• VGW connects your VPC to your on-prem network privately


1. When to use INTERNET GATEWAY (IGW)

Use IGW when AWS resources need access to the PUBLIC INTERNET.
You need IGW if:
✔️ You want EC2 to be reachable from the internet
	• Hosting a website
	• Public API server
✔️ EC2 needs outbound access to the Internet
	• Download packages, OS updates
	• Call public API endpoints
	• Access GitHub, Docker Hub, etc.

📌 Example Scenario
Your company hosts a public website on EC2/ALB in AWS.
Millions of users across the world must reach it → Use IGW.


Aspect	VGW	TGW
What it is	VPN / Direct Connect gateway for one VPC	Central routing hub for many VPCs and networks
VPCs supported	Single VPC	Multiple VPCs
VPC-to-VPC routing	❌ No	✅ Yes
On-prem connectivity	VPN, Direct Connect	VPN, Direct Connect
IPv4 routing	✅ Yes	✅ Yes
IPv6 routing (VPC / DX)	❌ No	✅ Yes
Routing model	Basic	Advanced (hub-and-spoke)
Scalability	Limited	High
Enterprise use	Limited	Preferred
Cost	Lower	Higher
Operational complexity	Low	Medium

-----------------------------------------------------------------------------------------------------------------

When to use VIRTUAL PRIVATE GATEWAY (VGW)

Use VGW when connecting your ON-PREMISES datacenter to AWS via VPN or Direct Connect.
You need VGW if:
✔️ You want site-to-site VPN
✔️ Your office network must reach AWS EC2 privately
✔️ You want hybrid cloud

📌 VGW use cases
On-prem → AWS secure private communication
Database in AWS accessed by on-prem app
Extend datacenter into AWS
No internet involved

📌 Example Scenario
You have a corporate network in Bangalore and you want to reach EC2 in AWS securely (no internet).
You create:
Customer gateway (on-prem)
Virtual Private Gateway (AWS)
Build VPN tunnel
→ Use VGW


-----------------------------------------------------------------------------------------------------------------
2 When to use NAT GATEWAY (Private Subnet → Internet)

Use NAT Gateway when instances in a PRIVATE subnet need OUTBOUND internet access, but should NOT be reachable from the internet.
You need NAT Gateway if your PRIVATE EC2 wants to:
Download OS patches
Access S3 public endpoints (without VPC endpoint)
Pull Docker images
Access external APIs (Stripe, RazorPay, etc.)

📌 Why not use IGW directly?
Private subnet → cannot use IGW
NAT Gateway → allows outbound only
Internet cannot reach the EC2

📌 Example Scenario
Your application servers run in private subnet for security.They must update Ubuntu packages or call 3rd-party APIs.They should NOT be exposed to internet.
→ Use NAT Gateway

-----------------------------------------------------------------------------------------------------------------


VPC:
The IP addresses for your virtual private cloud (VPC) are represented using Classless Inter-Domain Routing (CIDR) notation. A VPC must have an associated IPv4 CIDR block. You can optionally associate additional IPv4 CIDR blocks and one or more IPv6 CIDR blocks. 

Virtual private clouds (VPC)
A VPC is a virtual network that closely resembles a traditional network that you'd operate in your own data center. After you create a VPC, you can add subnets.
Subnets
A subnet is a range of IP addresses in your VPC. A subnet must reside in a single Availability Zone. After you add subnets, you can deploy AWS resources in your VPC.
IP addressing
You can assign IP addresses, both IPv4 and IPv6, to your VPCs and subnets. You can also bring your public IPv4 addresses and IPv6 GUA addresses to AWS and allocate them to resources in your VPC, such as EC2 instances, NAT gateways, and Network Load Balancers.
Routing
Use route tables to determine where network traffic from your subnet or gateway is directed.
Gateways and endpoints
A gateway connects your VPC to another network. For example, use an internet gateway to connect your VPC to the internet. Use a VPC endpoint to connect to AWS services privately, without the use of an internet gateway or NAT device.
Peering connections
Use a VPC peering connection to route traffic between the resources in two VPCs.
Traffic Mirroring
Copy network traffic from network interfaces and send it to security and monitoring appliances for deep packet inspection.
Transit gateways
Use a transit gateway, which acts as a central hub, to route traffic between your VPCs, VPN connections, and Direct Connect connections.


IPv4
Basics
	• 32-bit address which are written as 4 numbers Example: 192.168.1.10
Size
	• Total addresses ≈ 4.3 billion
	• Many already used → address exhaustion
Characteristics
	• Most widely used
	• Uses NAT heavily
	• Simple and well supported
Example
192.168.0.0/24


IPv6
Basics
	• 128-bit address and Written in hexadecimal
	• Example: 2001:0db8:85a3::8a2e:0370:7334
Size
	• Extremely large address space
	• Practically unlimited
Characteristics
	• No address exhaustion
	• No NAT required
	• Built-in support for security and auto-configuration
	






Why we still use IPv4 in AWS, even though IPv6 is available
1️⃣ Hybrid reality (biggest reason)
Most AWS environments are not cloud-only.
They connect to:
	• On-prem data centers, Corporate networks, VPNs (which are IPv4-only) and Legacy systems
So teams keep the same IP protocol everywhere to avoid complexity.
	Mixing IPv6 in AWS and IPv4 on-prem creates operational friction.

2️⃣ AWS networking is IPv4-first by design
Even today:
	• Every VPC must have IPv4,     Many AWS services still depend internally on IPv4,    IPv6 in AWS is additive, not primary
           So IPv6 is optional, IPv4 is mandatory.

4️⃣ No strong internal business benefit
IPv6 benefits:
	• Address exhaustion and End-to-end connectivity
But inside AWS:
	• Private IPv4 ranges are huge,       NAT solves scale,       Address exhaustion is not a problem
So no ROI for internal migration.

5️⃣ Security and compliance caution
Security teams are conservative:
	• IPv6 increases attack surface
	• Dual-stack = double rules
So IPv6 adoption is slow.

6️⃣ Operational simplicity
Running:
	• IPv4 only → simple
	• IPv4 + IPv6 → complex
	• IPv6 only → impractical
Enterprises choose lowest-risk path.

7️⃣ Application limitations
Many applications:
	• Log IPv4 only
	• Whitelist IPv4
	• Have hardcoded assumptions
Changing this across microservices is expensive.

Where IPv6 is actually used in AWS (realistic)
	• Internet-facing ALBs
	• Global public services
	• Mobile traffic endpoints
But:
	Edge = IPv6
	Core = IPv4


CIDR BLOCK:
The IP addresses for your virtual private cloud (VPC) are represented using Classless Inter-Domain Routing (CIDR) notation. A VPC must have an associated IPv4 CIDR block. You can optionally associate additional IPv4 CIDR blocks and one or more IPv6 CIDR block.
“A CIDR block defines the IP address range for a network. In AWS, CIDR blocks are used to allocate IP ranges for VPCs and subnets. The subnet mask determines how many IPs are available. Smaller CIDR numbers provide larger address space, and AWS uses CIDR-based networking to control IP allocation, routing, and isolation.”
In our project we are using ipv4 CIDR   because Site-to-Site VPN in AWS is IPv4-only. 

Subnet sizing for IPv4
The allowed IPv4 CIDR block size for a subnet is between a /28 netmask and /16 netmask

Subnet sizing for IPv6
If you've associated an IPv6 CIDR block with your VPC, you can associate an IPv6 CIDR block with an existing subnet in your VPC, or when you create a new subnet. Possible IPv6 netmask lengths are between /44 and /64 in increments of /4.






CIDR Calculation:
Always keep 32 in mind. Because 32 bits ips address range. 8 + 8 + 8 + 8 -> 32 

/16	32 - 16	16	2 ^ 16	65536 ( 65K )
/20	32 - 20	12	2 ^ 12	4096 (4K)
/24	32 - 24	8	2 ^ 8	256
/28	32 - 28	4	2 ^ 4	16

Watch this video : AWS VPC Basics - Understanding what is VPC and Calculating CIDR for VPC and Subnets 





IP Range :  10.0.0.0/24
32-24 -> 8 --> 2^8 -. 256Ips.
Network address: 10.0.0.0
First usable IP: 10.0.0.1
Last usable IP:  10.0.0.254
Broadcast IP:    10.0.0.255
AWS reserves 5 IPs per subnet:

10.0.0.0    (network)
10.0.0.1    (router)
10.0.0.2    (DNS)
10.0.0.3    (future use)
10.0.0.255  (broadcast)


---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Extra explaination for CIDR blocks ( 10.0.0.0 ).

IPv4 address has 32 bits
Network Bits = Fixed part of the IP. This portion cannot change. It defines the network/subnet itself.
Host Bits = Changeable part of the IP

This portion changes. It determines devices (EC2, Lambda ENIs, etc.) inside the network.

10.0.0.0/24
	Total bits = 32
	/24 means 24 bits are network bits
	Remaining bits = 32 − 24 = 8 host bits

Meaning:
First 24 bits = network (fixed)
Last 8 bits = host (changeable)

Those 8 bits can make 2⁸ = 256 IPs
That is why a /24 subnet has 256 IPs
Usable for EC2 = 251 IPs (AWS reserves 5)


If we chose /16 it means first units are used for networking and cannot be changed 


choose starting IPs like 10.x.x.x, 172.16.x.x, 192.168.x.x because
Because these IP ranges are reserved exclusively for private networks, according to a global networking standard (RFC 1918).
10.0.0.0 – 10.255.255.255 (/8)
Very large range → 16 million IPs
Used by big companies (Amazon, banks, enterprises)

172.16.0.0 – 172.31.255.255 (/12)
Medium range → 1 million IPs

192.168.0.0 – 192.168.255.255 (/16)
Most common in home Wi-Fi → 65,536 IPs




--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Each region -> Multiple VPC 
Global services : S3, IAM, R53 
Region services : S3 -> though s3 is regional service, it can be accessed globally 
Az level : EC2, RDS 

Availability Zones : 
Each Region has at least three Availability Zones. 
The code for an Availability Zone is its Region code followed by a letter identifier. For example, us-east-2a, us-east-2b, and us-east-2c are the Availability Zones in the us-east-2 Region.

VPC, Subnet - AZ - Region -> These are one to one mapping. Meaning a subnet can be placed 
A VPC always spans multiple AZs within the same Region
1 AZ can have multiple subnets 
1 subnet can be in only one AZ and cannot be shared with other AZ 

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

EC2 Key pair concept:
1. What is a Public Key?
A file that can be shared openly. It is not secret and Anyone can see it. Cannot be used to log in but Used only to verify that the private key is valid
- When you create an EC2 key pair: AWS stores the public key automatically on the EC2 instance in this file:
   ~/.ssh/authorized_keys
This happens during EC2 creation.

You never download the public key from AWS because AWS already installs it on the instance for you.

2. What is a Private Key?
A file that must be kept secret. It is Never shared with anyone. Used by you to authenticate to the server. it Stored on your computer. This file is downloaded once during key pair creation.
Example:
mykey.pem


✅ 1. When you create an SSH key pair in AWS. A public key and private key are generated.
Private key (.pem file) → downloaded and stored only on your machine.
Public key → AWS automatically installs it on your EC2 instance in:
~/.ssh/authorized_keys     (You do NOT manually store it.)

✅ 2. Sharing public keys
Public keys can be shared with anyone safely.
Private keys must NEVER be shared.

✅ 3. When a new person wants to access your EC2 instance:
	✔ Step 1 — The new person generates their own key pair
	On their laptop, they run something like:
	ssh-keygen
	This creates:
	Their private key → stays with them
	Their public key → they will send this to you
	They do NOT use your private key.

	✔ Step 2 — They send you their public key
	✔ Step 3 — You log in to the EC2 using your private key
	You SSH into the instance:
	ssh -i mykey.pem ec2-user@<public-ip>

	✔ Step 4 — You add their public key into your EC2:
	sudo nano ~/.ssh/authorized_keys
	Paste their public key at the end of the file.
	Now the EC2 has both public keys:
	Yours
	The new person’s

	✔ Step 5 — Now they can SSH using their private key:
	ssh -i their_private_key ec2-user@<public-ip>


Authentication works because:
EC2 has their public key
They use their private key
SSH verifies and grants access

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------







 









--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
EC2 




--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Datalake :

“Amazon S3 is an object storage service, while a Data Lake is an architecture built on top of storage like S3 to enable large-scale analytics and machine learning.”
“S3 is not a Data Lake by itself, but it is commonly used as the storage layer for a Data Lake.”

Relationship between S3 and Data Lake (key concept)
	S3 is the foundation of most AWS Data Lakes
A Data Lake typically uses:
	• S3 → Storage
	• Glue → Metadata & catalog
	• Athena → Query
	• EMR / Spark → Processing
	• Lake Formation → Security & governance
So:
	• S3 ≠ Data Lake
	• Data Lake = S3 + services + rules

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
MFA:
MFA (Multi-Factor Authentication) adds an extra layer of security by requiring more than one way to prove your identity when logging in.

MFA in AWS (important for interviews)
In AWS, MFA usually means:
	• Password +
	• Time-based OTP (TOTP)
The three authentication factors (must know)
1️⃣ Something you know
	• Password
	• PIN
2️⃣ Something you have
	• Mobile phone (OTP)
	• Hardware token
	• Authenticator app
3️⃣Something you are
	• Fingerprint
	• Face ID
👉 MFA = at least two different factors


--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
CI CD deployment:

Step 1 : Git hub -> code is hosted   ( Code commit )
Step 2 : Git hub triggers a webhook, which invokes the jenkins pipeline. Jenkins is the orchestrator ( CI part - continuous integration ) . (Code pipeline ). 
step 3 : within the jenkins pipeline, build process. Use docker, sonarcube, maven (java app )  ( Code build ) 
step 4 : take the build executable and deploy into EC2 or kubernetis ( shell script to deploy the app to ec2 ) ( Code deploy )  

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
CI CD
• Code change happens
	• Source code or infra YAML in git or software changes in BAMS
• CI phase
	• Pipeline pulls:
		○ App source code
		○ Infra YAML files
	• Builds a Docker image on the CI runner ( Git runner ) .
	• Pushes the image to the registry
• CD phase (this is the key)
	• Pipeline registers a new ECS task definition revision
	• ECS service is updated to use the new revision
	• This image is the single deployable unit for all envs.
	• This image is the single deployable unit for all envs
	
• ECS handles deployment
	• New tasks are started with the new image
	• Old tasks continue running until new ones are healthy
	• Traffic is shifted automatically
	• Old tasks are stopped

	“Our CI/CD pipeline builds a Docker image from the source code and pushes it to a registry. During deployment, we create a new ECS task definition revision using our infrastructure YAML files. ECS then launches new tasks with the new image and replaces the old ones after health checks, enabling zero-downtime deployment and easy rollback.”. We use A GitLab runner, which is a long running ec2 instance deployed on ci cd account. All the ci docker jmage is built on this instance.
	Old tasks keep running → New tasks start → Health check → Traffic switches → Old tasks stop
	
How ECS uses this image (important distinction)
During CD:
	• ECS task definition references the image URI in the registry
	• ECS pulls the image from ECR
	• ECS runs containers based on that image
👉 ECS never sees source code, only the image.

ECS Task vs ECS Service (Simple Explanation)
🔹 ECS Task
A Task is:
	• A single running instance of your application
	• Created from a Task Definition
	• Runs once (or until it stops/crashes)
👉 Think of a task as “one container (or group of containers) running”
🔹 ECS Service
A Service is:
	• A manager for tasks
	• Ensures desired number of tasks are always running
	• Restarts tasks if they fail
	• Supports auto-recovery, load balancing, and scaling
👉 Think of a service as “a controller that keeps tasks alive”




3 tier architecture :
“In a 3-tier AWS architecture, we deploy the load balancer in public subnets, application servers in private subnets, and the database in isolated private subnets. Traffic enters through an Internet Gateway to the ALB, flows securely to the application tier, and then to the database. Outbound access from private subnets is provided via a NAT Gateway, and security is enforced using security groups.”
A standard production VPC has:
	• At least 2 Availability Zones (for HA)
	• Public subnets
	• Private subnets

VPC
├── Public Subnet (AZ1)
├── Public Subnet (AZ2)
├── Private App Subnet (AZ1)
├── Private App Subnet (AZ2)
├── Private DB Subnet (AZ1)
├── Private DB Subnet (AZ2)


Tier-wise placement (this is key)
🔹 Presentation Tier (Frontend)
Where: Public Subnets Why: Needs internet access
AWS services:
	• CloudFront,  ALB (internet-facing), S3 (static frontend)
User → CloudFront → ALB (Public Subnet)


🔹 Application Tier (Business Logic)
Where: Private Subnets Why: Should NOT be directly exposed to internet
AWS services:
	• EC2 (Auto Scaling), ECS / EKS, Lambda (VPC-based)
ALB → App Tier (Private Subnet)


🔹 Data Tier (Database)
Where: Private Subnets (DB subnets) Why: Maximum isolation
AWS services:
	• RDS (Multi-AZ) DynamoDB (VPC endpoint) Aurora
App Tier → DB Tier (Private Subnet)


--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

What is ELK?
ELK = Elasticsearch + Logstash + Kibana
It is a log collection, storage, search, and visualization stack.
In short: ELK helps you collect logs, search them fast, and see them in dashboards.

Why ELK is needed (real problem)
In real systems you have:
	• EC2 logs
	• Application logs
	• Nginx / Apache logs
	• Database logs
	• Kubernetes logs
1️⃣ Elasticsearch (E)
What it is:
A searchable database optimized for logs
What it does:
	• Stores logs as JSON documents
	• Indexes them for fast search
2️⃣ Logstash (L)
What it is:
A log processing & transformation engine
What it does:
	• Collects logs from many sources
	• Parses and cleans logs
	• Converts raw logs into structured JSON
	• Sends logs to Elasticsearch
3️⃣ Kibana (K)
What it is:
A UI & dashboard tool
What it does:
	• Visualizes Elasticsearch data
	• Shows:
		○ Charts
		○ Graphs
		○ Dashboards
		○ Search UI


--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------



--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


Definitions:
What is an IAM role: 
An IAM role is an AWS identity that provides temporary permissions to users or services without using access keys.
“We attach an IAM role to the EC2 instance, which allows the instance to assume the role via STS and obtain temporary credentials from the Metadata service. These credentials are rotated automatically and used by the AWS SDK to access S3 securely.”

Can you attach a policy directly to a user
“Yes, it’s technically possible for IAM users, but it’s not recommended. We use IAM roles with STS for temporary credentials. The root user should never be used except for account-level tasks. In production, we never use root or long-term user credentials — everything is role-based with STS”

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
A Security Group is a virtual firewall that controls network traffic to and from AWS resources.
What it controls
	• IP addresses, Ports, Protocols, Source / destination
Works at
	• Network level, Instance / ENI level
Example
	• Allow HTTP (80) from internet
	• Allow MySQL (3306) only from app servers

“Security Groups protect resources at the network layer, and IAM enforces least privilege at the identity layer.”



--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


Questions :
How do you give Account A access to S3 in Account B?
Create an IAM role in Account B with permissions to access S3 and a trust policy that allows Account A to assume the role using STS.

How can an EC2 instance access S3 securely without storing access keys?
By attaching an IAM role to the EC2 instance, which allows it to assume the role via STS and obtain temporary credentials from the instance metadata service.


Your Jenkins/GitHub Actions pipeline needs to deploy to AWS. How do you give access securely?
Use IAM roles with STS instead of storing access keys. For GitHub Actions, use OIDC to assume an IAM role. For Jenkins, configure assume-role with temporary credentials.

Why not just use S3 bucket policies instead of roles?
Bucket policies are fine for simple cases, but IAM roles with STS provide better security, flexibility, and scalability for cross-account access.

How do you immediately stop an application from accessing AWS?
Detach or modify the IAM role permissions. Changes take effect immediately without restarting services.

Your EC2 instance needs to upload files to S3.
How does the EC2 instance get credentials, and where are those credentials stored?
The EC2 instance gets temporary credentials by assuming an IAM role via AWS STS.
These credentials are provided through the Instance Metadata Service and are not stored permanently on the instance; they are rotated automatically


How does CodePipeline access AWS services?
Using IAM service roles with defined permissions.

How do you deploy to another AWS account?
Use cross-account IAM roles and STS AssumeRole from the pipeline.


What AWS permission does the pipeline need to create a new task definition revision?
ecs:RegisterTaskDefinition
ecs:RegisterTaskDefinition
ecs:UpdateService
elasticloadbalancing:Describe*

Explain 3-tier architecture in AWS”
“In a 3-tier architecture, the presentation layer handles user requests, the application layer processes business logic, and the data layer manages storage. In AWS, this is commonly implemented using CloudFront and ALB for the frontend, EC2 or ECS for the application tier, and RDS or DynamoDB for the database tier.”

Can tiers be in different subnets?
	Yes. Typically, frontend in public subnet, app and DB in private subnets.

An app has IAM permission to access RDS, but it cannot connect. Why?
IAM controls API access, but database connections depend on Security Groups and ports.

Can you allow EC2 to access S3 using a Security Group rule?
No. Security Groups cannot grant AWS service permissions. S3 access is controlled by IAM policies.

EC2 has the correct IAM role, but S3 access still fails. Why?
IAM allows the action, but network connectivity may be blocked. If the EC2 is in a private subnet, it needs internet access or an S3 VPC endpoint.

EC2 is in a public subnet, but users can’t reach it. What could be wrong?
The Security Group may not allow inbound traffic, even though the subnet is public.

Which one protects against hackers scanning ports?
Question: IAM or Security Group?
Security Group — it blocks unwanted network traffic at the instance level.

Which one protects against leaked credentials?
Question: IAM or Security Group?
IAM — it enforces authentication, authorization, and least privilege.

How do IAM and Security Groups work together?
IAM controls who can perform AWS actions, and Security Groups control which network traffic is allowed. Both are required for a secure system.

“IAM enforces identity-based permissions, while Security Groups enforce network-level access — they solve different security problems.”


“For AWS services like S3 or DynamoDB, we use VPC Endpoints to keep traffic private and avoid NAT costs. NAT Gateways are only used when internet access is required.”
Scenario 1 EC2 in private subnet needs S3 access
✅ Best answer: VPC Endpoint (Gateway)
Why ? 
No internet needed, More secure, Cheaper

Scenario 2️: EC2 needs OS updates from yum/apt
✅ Best answer: NAT Gateway
Why? External internet access required

Scenario 3: ECS tasks pulling images from ECR
✅ Best answer: Interface VPC Endpoints for:
ECR API, ECR DKR,  STS


EC2 in private subnet cannot access DynamoDB. What do you check?
Answer:
	1. IAM role attached to EC2, DynamoDB VPC Endpoint exists, Route table has DynamoDB endpoint entry
EC2 in private subnet cannot access DynamoDB. What do you check?
Answer:
	1. IAM role attached to EC2,  DynamoDB VPC Endpoint exists,  Route table has DynamoDB endpoint entry





--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------



	
	
<img width="1104" height="15640" alt="image" src="https://github.com/user-attachments/assets/3640cb69-a760-49d6-9b63-7e1b459e92a0" />
