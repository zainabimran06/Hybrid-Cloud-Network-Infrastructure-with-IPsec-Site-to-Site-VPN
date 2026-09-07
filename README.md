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

  
