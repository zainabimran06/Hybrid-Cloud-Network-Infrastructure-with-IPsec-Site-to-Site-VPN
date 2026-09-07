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
