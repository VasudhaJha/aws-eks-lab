# Project 02: Application Deployment & Services

**Estimated Time**: 45-60 minutes

## Learning Objectives

By completing this lab, you will:

- Deploy a multi-tier application using existing Docker images
- Understand and implement all three Kubernetes Service types (ClusterIP, NodePort, LoadBalancer)
- Practice service discovery and inter-pod communication
- Learn deployment scaling and basic troubleshooting
- Work with labels and selectors for service routing
- Understand pod networking in a real application context

## Prerequisites

- Completed Lab 01 (Basic EKS Cluster Setup)
- EKS cluster running with worker nodes
- `kubectl` configured and working
- Basic understanding of Docker containers

## What You'll Build

The classic Voting App - a distributed application with multiple services:

- Voting Frontend: Web interface for casting votes (Python Flask)
- Redis: In-memory database for storing votes
- Worker: Background processor that moves votes from Redis to PostgreSQL
- PostgreSQL: Persistent database for storing processed votes
- Results Frontend: Web interface for viewing vote results (Node.js)

## Architecture Overview

```bash
                Internet Users
                      |
            ┌─────────┴─────────┐
            ↓                   ↓
    [LoadBalancer]      [LoadBalancer]
      (External)          (External)
            ↓                   ↑
       ┌─────────┐         ┌─────────┐
       │   POD   │         │   POD   │
       │voting-app│         │result-app│
       │(Python) │         │(Node.js)│
       └─────────┘         └─────────┘
            ↓                   ↑
            ↓                   ↑
       [ClusterIP]         [ClusterIP]
         Service             Service
            ↓                   ↑
            ↓                   ↑
       ┌─────────┐         ┌─────────┐
       │   POD   │         │   POD   │
       │  redis  │         │postgres │
       │         │         │         │
       └─────────┘         └─────────┘
            ↑                   ↑
            │                   │
            └──── ┌─────────┐ ──┘
                  │   POD   │
                  │ worker  │
                  │         │
                  └─────────┘

```

### Component Interactions

- Voting Flow: `Users → LoadBalancer → Voting Pod → Redis Pod (stores votes)`
- Processing: `Worker Pod reads from Redis → processes votes → writes to PostgreSQL`
- Results Flow: `Users → LoadBalancer → Results Pod → PostgreSQL (reads processed data)`

### Key Points

- Worker Pod connects to BOTH Redis (read) and PostgreSQL (write)
- Redis and PostgreSQL use ClusterIP services (internal only)
- Only voting-app and result-app need external access (LoadBalancer)