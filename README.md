HybridCloud Corp — Hybrid Cloud Network Infrastructure
IPsec Site-to-Site VPN | Cisco IOS | strongSwan | Docker | LocalStack
Project Overview

This project demonstrates the design and implementation of a hybrid cloud network architecture for HybridCloud Corp. An on-premise Cisco router is connected toa simulated 
cloud environment via an encrypted IPsec Site-to-Site VPN tunnel, enabling secure bidirectional communication between the on-premise network and cloud-hosted services.

The project replicates real-world enterprise hybrid cloud deployments, where organizations maintain local infrastructure while extending workloads into a cloud environment — 
all without exposing traffic over the public internet in plaintext.
Architecture
        ON-PREMISE SIDE                          CLOUD SIDE
        172.16.1.0/24                            10.0.0.0/24

  ┌─────────────────────┐                  ┌─────────────────────┐
  │  PC0 / Ubuntu Server│                  │    Cloud Server     │
  │    172.16.1.10      │                  │     10.0.0.10       │
  └────────┬────────────┘                  └──────────┬──────────┘
           │                                          │
  ┌────────┴────────────┐                  ┌──────────┴──────────┐
  │     SW1 Switch      │                  │  strongSwan Gateway  │
  │     (Layer 2)       │                  │     10.0.0.1        │
  └────────┬────────────┘                  └──────────┬──────────┘
           │                                          │
  ┌────────┴────────────┐                  ┌──────────┴──────────┐
  │   Cisco R1 Router   │                  │     LocalStack      │
  │  172.16.1.1 (LAN)   │◄══IPsec Tunnel══►│   (AWS Simulation)  │
  │  203.0.113.1 (WAN)  │   AES-256/SHA    │     10.0.0.20       │
  └─────────────────────┘                  └─────────────────────┘

  Network Design
Device	Role	IP Address	Tool
PC0	On-premise host	172.16.1.10/24	Packet Tracer VPCS
SW1	Layer-2 LAN switch	—	Packet Tracer
Cisco R1	On-premise IPsec gateway	172.16.1.1 / 203.0.113.1	Packet Tracer IOS
Cloud-GW	Cloud-side IPsec endpoint	10.0.0.1/24	Docker + strongSwan
Cloud Server	Cloud-side target host	10.0.0.10/24	Docker Alpine
LocalStack	AWS cloud simulation	10.0.0.20/24	Docker LocalStack 3.0
Cloud-GW	Cloud-side IPsec peer	203.0.113.2 (WAN)	Docker
IPsec Tunnel Parameters
Parameter	Value
IKE Version	IKEv1 (Main Mode)
Encryption	AES-256
Hash / Integrity	SHA (HMAC)
DH Group	Group 5 (1536-bit)
Authentication	Pre-Shared Key (PSK)
ESP Transform	ESP-AES / ESP-SHA-HMAC
PFS	Enabled
IKE Lifetime	86400s (24 hours)
ESP Lifetime	3600s (1 hour)
Tunnel Mode	Default (tunnel)

Project Structure
HybridCloud/
│
├── packet-tracer/
│   └── HybridCloud_Final.pkt          ← Packet Tracer topology file
│
├── cisco-configs/
│   └── R1OnPrem-running-config.txt    ← Full Cisco IOS running configuration
│
├── docker/
│   ├── docker-compose.yml             ← Cloud-side container definitions
│   ├── Dockerfile.gateway             ← Custom strongSwan image (Ubuntu 22.04)
│   └── ipsec/
│       ├── ipsec.conf                 ← strongSwan tunnel configuration
│       ├── ipsec.secrets              ← Pre-shared key
│       └── strongswan.conf            ← Daemon configuration
│
├── screenshots/
│   ├── isakmp-sa-active.png           ← show crypto isakmp sa (QM_IDLE)
│   ├── cloud-to-onprem-ping.png       ← Ping from cloud to 172.16.1.10
│   └── docker-compose-ps.png          ← All containers running
│
└── README.md                          ← This file

Setup Instructions
Prerequisites
Cisco Packet Tracer (v8.x or later)
Docker Desktop for Windows (with WSL2 backend)
Windows 10/11 with WSL2 enabled
Part 1 — On-Premise Side (Packet Tracer)

1. Open the topology

File → Open → HybridCloud_Final.pkt

2. Verify device IPs

PC0: 172.16.1.10/24, gateway 172.16.1.1
R1OnPrem Gi0/0: 172.16.1.1/24
R1OnPrem Gi0/1: 203.0.113.1/252
Cloud-GW Gi0/1: 203.0.113.2/252
PC1: 10.0.0.10/24, gateway 10.0.0.1

3. Verify IPsec tunnel status on R1OnPrem

enable
show crypto isakmp sa

Expected output:

dst           src           state     conn-id  status
203.0.113.2   203.0.113.1   QM_IDLE   XXXX     ACTIVE

4. Test connectivity On PC1 (cloud side) → Command Prompt:

ping 172.16.1.10
Part 2 — Cloud Side (Docker)

1. Navigate to the docker folder

powershell
cd C:\HybridLab2\hybridcloud-v2-ubuntu

2. Build and start all containers

powershell
docker compose up -d --build

3. Verify all containers are running

powershell
docker compose ps

Expected:

cloud-gateway    Up
cloud-server     Up
localstack       Up (healthy)

4. Verify strongSwan is running

powershell
docker exec cloud-gateway ipsec --version
docker exec cloud-gateway ipsec statusall

5. Test internal cloud connectivity

powershell
docker exec cloud-server ping -c 4 10.0.0.1

6. Verify LocalStack (AWS simulation)

powershell
curl http://localhost:4566/_localstack/health

Expected: "s3": "available", "lambda": "available", ...

Key Configuration Files
Cisco R1 — IPsec Summary
crypto isakmp policy 10
 encr aes 256
 hash sha
 authentication pre-share
 group 5

crypto isakmp key HybridCloud@123 address 203.0.113.2

crypto ipsec transform-set HYBRID esp-aes esp-sha-hmac

ip access-list extended CRYPTO-ACL
 permit ip 172.16.1.0 0.0.0.255 10.0.0.0 0.0.0.255

crypto map HYBRID-MAP 10 ipsec-isakmp
 set peer 203.0.113.2
 set transform-set HYBRID
 match address CRYPTO-ACL
strongSwan — ipsec.conf Summary
conn cloud-to-onprem
    keyexchange=ikev1
    left=10.0.0.1
    leftsubnet=10.0.0.0/24
    right=203.0.113.1
    rightsubnet=172.16.1.0/24
    authby=secret
    auto=start

    Verification Commands
Cisco Router (Packet Tracer)
show crypto isakmp sa        ← Phase 1 status (QM_IDLE = active)
show crypto ipsec sa         ← Phase 2 SA + packet counters
show ip route                ← Routing table
show running-config          ← Full configuration
Docker / strongSwan
powershell
docker exec cloud-gateway ipsec status
docker exec cloud-gateway ipsec statusall
docker exec cloud-server ping -c 4 10.0.0.1
docker compose ps
docker logs cloud-gateway
Troubleshooting
Problem	Cause	Fix
Invalid input detected in Packet Tracer	Wrong CLI mode	Type enable then configure terminal first
Tunnel shows MM_NO_STATE	PSK mismatch or peer unreachable	Verify pre-shared key matches on both sides
QM_IDLE but ping fails	ACL not matching traffic	Check permit ip subnet ranges in CRYPTO-ACL
Docker container restarting	Missing kernel modules or bad image	Use Ubuntu-based strongSwan image, not community images
LocalStack license error	Pulled Pro version	Pin to localstack/localstack:3.0 (community free)
WSL2 file mount error	Windows path issue	Use Dockerfile COPY instead of volume mounts
Simulated AWS Services (LocalStack)
Service	Status	Use Case
S3	Available	File storage simulation
Lambda	Available	Serverless function simulation
SQS	Available	Message queue simulation
SNS	Available	Notification service simulation
DynamoDB	Available	NoSQL database simulation

Technologies Used
Cisco IOS 15.4 — Router configuration, IPsec/IKEv1
Cisco Packet Tracer — On-premise network simulation
strongSwan 5.9.5 — Linux IPsec implementation (cloud gateway)
Docker & Docker Compose — Container orchestration
Ubuntu 22.04 — Base OS for strongSwan container
LocalStack 3.0 — AWS cloud services simulation
Alpine Linux — Lightweight cloud server container
Windows 11 + WSL2 — Host environment

Project Outcome
Successfully designed and deployed a hybrid cloud network demonstrating:
Secure IPsec Site-to-Site VPN tunnel between on-premise and cloud environments
Bidirectional connectivity verified with successful pings across both network segments
Cloud-side AWS service simulation via LocalStack with S3, Lambda, and DynamoDB available
Industry-standard encryption (AES-256) and authentication (PSK/SHA) applied throughout
Multi-container cloud infrastructure deployed and managed via Docker Compose
Author ZAINAB IMRAN

HybridCloud Corp — Network Infrastructure Lab Submitted as part of Network Engineering coursework.

This project uses Cisco Packet Tracer as an approved substitute for GNS3 due to IOS image availability constraints in the lab environment. EOF
