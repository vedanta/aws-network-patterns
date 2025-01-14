# AWS VPC Networking Patterns and Best Practices

A comprehensive guide to AWS VPC networking patterns, subnet designs, and connectivity options.

## Table of Contents
1. [Basic VPC with Public/Private Subnets](#1-basic-vpc-with-publicprivate-subnets)
2. [Multi-AZ VPC Design](#2-multi-az-vpc-design)
3. [VPC with Isolated Subnets](#3-vpc-with-isolated-subnets)
4. [Transit Gateway Network](#4-transit-gateway-network)
5. [VPC Peering Design](#5-vpc-peering-design)
6. [Hybrid Connectivity](#6-hybrid-connectivity)
7. [Security Layer Design](#7-security-layer-design)
8. [VPC Endpoints Pattern](#8-vpc-endpoints-pattern)

## 1. Basic VPC with Public/Private Subnets

**Components:**
- Public subnets with Internet Gateway
- Private subnets with NAT Gateway
- Route tables for each subnet type
- Network ACLs and Security Groups

**Use Cases:**
- Web applications
- Basic two-tier architectures
- Development environments

```mermaid
graph TB
    subgraph VPC[VPC 10.0.0.0/16]
        IG[Internet Gateway]
        
        subgraph Public[Public Subnet 10.0.1.0/24]
            NGW[NAT Gateway]
            BH[Bastion Host]
        end
        
        subgraph Private[Private Subnet 10.0.2.0/24]
            APP[Application Servers]
            DB[Database]
        end
        
        IG --- Public
        Public --- NGW
        NGW --- Private
    end
    
    Internet((Internet)) --- IG
```

## 2. Multi-AZ VPC Design

**Components:**
- Subnets in multiple Availability Zones
- Redundant NAT Gateways
- Cross-AZ load balancing
- Fault-tolerant design

**Use Cases:**
- Production workloads
- High availability requirements
- Disaster recovery scenarios

```mermaid
graph TB
    subgraph VPC[VPC 10.0.0.0/16]
        IG[Internet Gateway]
        ALB[Application Load Balancer]
        
        subgraph AZ1[Availability Zone 1]
            PS1[Public 10.0.1.0/24]
            PRS1[Private 10.0.2.0/24]
            NGW1[NAT Gateway 1]
            APP1[App Servers 1]
        end
        
        subgraph AZ2[Availability Zone 2]
            PS2[Public 10.0.3.0/24]
            PRS2[Private 10.0.4.0/24]
            NGW2[NAT Gateway 2]
            APP2[App Servers 2]
        end
        
        IG --- ALB
        ALB --- PS1
        ALB --- PS2
        PS1 --- NGW1
        PS2 --- NGW2
        NGW1 --- PRS1
        NGW2 --- PRS2
        PRS1 --- APP1
        PRS2 --- APP2
    end
    
    Internet((Internet)) --- IG
```

## 3. VPC with Isolated Subnets

**Components:**
- No internet connectivity
- VPC Endpoints for AWS services
- Internal load balancers
- Strict network isolation

**Use Cases:**
- Regulated workloads
- Data processing systems
- Internal services

```mermaid
graph TB
    subgraph VPC[VPC 10.0.0.0/16]
        subgraph Isolated[Isolated Subnet 10.0.1.0/24]
            APP[Application Servers]
            DB[Databases]
            EP[VPC Endpoints]
        end
        
        subgraph Services[AWS Services]
            S3[S3]
            DDB[DynamoDB]
            SM[Secrets Manager]
        end
        
        EP --- S3
        EP --- DDB
        EP --- SM
        APP --- EP
        APP --- DB
    end
```

## 4. Transit Gateway Network

**Components:**
- Central Transit Gateway
- Multiple VPC attachments
- Route table associations
- Network segmentation

**Use Cases:**
- Multi-VPC architectures
- Centralized networking
- Hub-and-spoke topology

```mermaid
graph TB
    TG[Transit Gateway]
    
    subgraph VPC1[Production VPC]
        PROD[Production Workloads]
    end
    
    subgraph VPC2[Development VPC]
        DEV[Development Workloads]
    end
    
    subgraph VPC3[Shared Services VPC]
        SVC[Shared Services]
    end
    
    TG --- VPC1
    TG --- VPC2
    TG --- VPC3
    VPC1 --- PROD
    VPC2 --- DEV
    VPC3 --- SVC
```

## 5. VPC Peering Design

**Components:**
- VPC Peering Connections
- Cross-VPC routing
- Security group references
- CIDR planning

**Use Cases:**
- Resource sharing
- Cross-account access
- Regional connectivity

```mermaid
graph LR
    subgraph VPC1[VPC A 10.0.0.0/16]
        A[Resources A]
    end
    
    subgraph VPC2[VPC B 172.16.0.0/16]
        B[Resources B]
    end
    
    subgraph VPC3[VPC C 192.168.0.0/16]
        C[Resources C]
    end
    
    VPC1 --- VPC2
    VPC2 --- VPC3
    VPC1 -.-> VPC3
```

## 6. Hybrid Connectivity

**Components:**
- Direct Connect
- Site-to-Site VPN
- Customer Gateway
- Virtual Private Gateway

**Use Cases:**
- On-premises integration
- Hybrid cloud
- Disaster recovery

```mermaid
graph TB
    subgraph AWS[AWS Cloud]
        VPC[VPC]
        VGW[Virtual Private Gateway]
        DX[Direct Connect]
    end
    
    subgraph OnPrem[On-Premises]
        DC[Data Center]
        CGW[Customer Gateway]
        Router[Router]
    end
    
    VGW --- DX
    VGW -.-> CGW
    DX --- Router
    CGW --- DC
    Router --- DC
```

## 7. Security Layer Design

**Components:**
- Network ACLs
- Security Groups
- Web Application Firewall
- Network Firewall

**Use Cases:**
- Defense in depth
- Compliance requirements
- Traffic filtering

```mermaid
graph TB
    subgraph VPC[VPC Security Layers]
        WAF[WAF] --> ALB[Application Load Balancer]
        ALB --> NACL1[NACL]
        NACL1 --> SG1[Security Group]
        SG1 --> APP[Application Tier]
        APP --> NACL2[NACL]
        NACL2 --> SG2[Security Group]
        SG2 --> DB[Database Tier]
    end
```

## 8. VPC Endpoints Pattern

**Components:**
- Interface Endpoints
- Gateway Endpoints
- Endpoint Services
- Private connectivity

**Use Cases:**
- AWS service access
- Private API access
- Service provider integration

```mermaid
graph TB
    subgraph VPC[Private VPC]
        subgraph Private[Private Subnet]
            APP[Applications]
            
            subgraph Endpoints[VPC Endpoints]
                GE[Gateway Endpoints]
                IE[Interface Endpoints]
            end
        end
    end
    
    subgraph AWS[AWS Services]
        S3[(S3)]
        DDB[(DynamoDB)]
        ECR[ECR]
        SM[Secrets Manager]
    end
    
    GE --> S3
    GE --> DDB
    IE --> ECR
    IE --> SM
    APP --> GE
    APP --> IE
```

## Best Practices

1. **CIDR Planning**
   - Use non-overlapping CIDR ranges
   - Plan for future expansion
   - Reserve space for new subnets

2. **High Availability**
   - Deploy across multiple AZs
   - Use redundant NAT Gateways
   - Implement fault-tolerant designs

3. **Security**
   - Implement defense in depth
   - Use security groups and NACLs
   - Enable VPC Flow Logs
   - Implement least privilege access

4. **Cost Optimization**
   - Use Gateway Endpoints for S3/DynamoDB
   - Optimize NAT Gateway placement
   - Consider Traffic Flow costs

## Contributing
Feel free to contribute to this documentation by submitting pull requests or opening issues for improvements.

## License
This documentation is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
