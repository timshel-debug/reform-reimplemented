# Security and Trust Boundaries

## AuthN/AuthZ

* TODO: confirm identity provider and claims (ASM-0002). 
* Enforce:
  * Customers can access only their own designs (NFR-0004). 
  * Admin endpoints require Admin/Operations role (REQ-0009). 
  * Rollout/routing flag mutation endpoints require elevated Admin/Operations role and produce immutable audit entries (`who/when/what`). 

## Trust boundaries

* Browser (untrusted) → Platform/UI → Decoration API (trusted).
* Decoration API/Workers → DB/Object/Queue (trusted).
* Legacy adapter → Third-party (untrusted external dependency). 
* Routing policy control plane and audit log store remain in trusted zone; read access is least-privilege and write access is restricted to rollout operators.

## Trust boundary diagram

```mermaid
flowchart LR
  subgraph Untrusted[Untrusted Zone]
    browser[Customer Browser]
    third[Third-Party<br/>Legacy Service]
  end
  
  subgraph DMZ[DMZ]
    alb[ALB/WAF]
  end
  
  subgraph Trusted[Trusted Zone - AWS VPC]
    subgraph App[Application Layer]
      api[Decoration API<br/>AuthN/AuthZ]
      workers[Workers]
    end
    subgraph Data[Data Layer]
      rds[(RDS<br/>Private Subnet)]
      s3[(S3<br/>Bucket Policies)]
      sqs[[SQS]]
    end
  end
  
  browser -->|HTTPS| alb
  alb -->|AuthN| api
  api --> rds
  api --> s3
  api --> sqs
  workers --> sqs
  workers --> s3
  api -.->|Outbound Only| third
  
  style Untrusted fill:#ffcccc
  style DMZ fill:#ffffcc
  style Trusted fill:#ccffcc
```
