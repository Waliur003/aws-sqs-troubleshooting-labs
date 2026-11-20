# Amazon DynamoDB Troubleshooting Labs

**Production-Style AWS Troubleshooting | Cloud Engineering | Cloud Security Engineering**

## Overview

This repository contains 10 hands-on Amazon DynamoDB troubleshooting labs based on real-world cloud engineering and cloud security scenarios.

The purpose of this project is to develop practical skills in investigating DynamoDB errors, identifying root causes, restoring database operations, monitoring performance, and securing AWS database resources.

Each lab simulates a production-style incident involving DynamoDB tables, access permissions, data validation, partition keys, throughput, secondary indexes, DynamoDB Streams, or disaster recovery.

The troubleshooting process uses AWS CLI, Linux Bash, Python when appropriate, IAM, CloudWatch, and other relevant AWS services.

Rather than only learning DynamoDB concepts, this repository focuses on identifying problems, reproducing failures, investigating configurations, applying remediation, and verifying recovery.

## Learning Objectives

By completing these labs, the goal is to understand:

- DynamoDB table operations and resource identification.
- IAM permissions for DynamoDB.
- DynamoDB partition keys and sort keys.
- DynamoDB item structure and attribute types.
- DynamoDB PutItem, GetItem, Query, and Scan operations.
- Common DynamoDB exceptions.
- Provisioned and on-demand capacity modes.
- Read and write capacity concepts.
- Conditional writes and optimistic concurrency.
- Global Secondary Indexes (GSIs).
- DynamoDB Streams and Lambda event source mappings.
- Point-in-time recovery (PITR).
- DynamoDB backup and restore.
- Hot partitions and uneven workloads.
- CloudWatch metrics and performance analysis.
- Secure database configuration.
- Incident investigation and recovery.

---

## AWS Services and Tools

| Service / Tool | Purpose |
|---|---|
| Amazon DynamoDB | Primary NoSQL database |
| AWS IAM | Access permissions and least privilege |
| Amazon CloudWatch | Metrics, logging, and monitoring |
| AWS Lambda | DynamoDB Streams processing |
| DynamoDB Streams | Change data capture |
| AWS Backup / DynamoDB PITR | Recovery concepts |
| AWS CLI v2 | Resource management and troubleshooting |
| Linux Bash | Command-line automation |
| Python / Boto3 | Application-level investigation when needed |
| jq | JSON filtering and inspection |

**AWS Region:** `us-east-1`

**Primary environment:** WSL2 Ubuntu Linux

---

## Repository Structure

```text
aws-dynamodb-troubleshooting-labs/
│
├── README.md
├── .gitignore
│
├── 01-table-not-found/
│   └── README.md
│
├── 02-putitem-access-denied/
│   └── README.md
│
├── 03-validation-exception/
│   └── README.md
│
├── 04-wrong-partition-key/
│   └── README.md
│
├── 05-provisioned-throughput-exceeded/
│   └── README.md
│
├── 06-conditional-check-failed/
│   └── README.md
│
├── 07-gsi-query-not-returning-results/
│   └── README.md
│
├── 08-dynamodb-streams-not-triggering/
│   └── README.md
│
├── 09-point-in-time-recovery-investigation/
│   └── README.md
│
└── 10-dynamodb-hot-partition/
    └── README.md
```

Lab folders and their documentation will be added as each practical exercise is completed.

---

## Troubleshooting Lab Roadmap

| Lab | Troubleshooting Scenario | Main Investigation |
|---|---|---|
| 01 | Table Not Found | Incorrect table name, Region, or missing resource |
| 02 | PutItem AccessDenied | IAM permissions and authorization |
| 03 | ValidationException | Invalid item structure and attribute types |
| 04 | Wrong Partition Key | Primary key schema and item retrieval |
| 05 | Provisioned Throughput Exceeded | Capacity settings and request throttling |
| 06 | ConditionalCheckFailedException | Conditional expressions and data conflicts |
| 07 | GSI Query Not Returning Results | Index keys, query conditions, and index contents |
| 08 | DynamoDB Streams Not Triggering | Stream configuration and Lambda triggers |
| 09 | Point-in-Time Recovery Investigation | PITR configuration and restoration |
| 10 | DynamoDB Hot Partition | Uneven partition-key access and performance |

---

## Lab 01 — Table Not Found

### Production Incident

An application attempts to access a DynamoDB table that does not exist in the configured AWS account or Region.

The request fails with:

`ResourceNotFoundException`

### Investigation Areas

- DynamoDB table name.
- AWS account and Region.
- Existing table inventory.
- Table status.
- AWS CLI configuration.
- Application environment variables.

### Expected Outcome

Identify the missing or incorrectly referenced table and restore successful access.

---

## Lab 02 — PutItem AccessDenied

### Production Incident

An application attempts to write an item to DynamoDB but receives:

`AccessDeniedException`

### Investigation Areas

- IAM identity and permissions.
- DynamoDB PutItem authorization.
- Identity-based and resource-based policies where applicable.
- Explicit denies.
- Table ARN.
- CloudTrail investigation.

### Expected Outcome

Identify the missing or denied permission and restore the authorized write operation using least privilege.

---

## Lab 03 — ValidationException

### Production Incident

An application attempts to write or retrieve data using a request that violates DynamoDB validation requirements.

The operation fails with:

`ValidationException`

### Investigation Areas

- Table key schema.
- Attribute names and data types.
- Missing required key attributes.
- Invalid expression syntax.
- Incorrect API request structure.

### Expected Outcome

Correct the invalid request and verify successful DynamoDB operations.

---

## Lab 04 — Wrong Partition Key

### Production Incident

An application uses an incorrect partition-key value or key structure when retrieving an item.

The requested item cannot be found, or DynamoDB rejects the malformed key.

### Investigation Areas

- Partition key.
- Sort key.
- Primary key schema.
- GetItem requests.
- Query operations.
- Exact key values and attribute types.

### Expected Outcome

Identify the correct primary key and retrieve the intended item.

---

## Lab 05 — Provisioned Throughput Exceeded

### Production Incident

An application experiences DynamoDB throttling when request demand exceeds available provisioned capacity or related service limits.

Possible error:

`ProvisionedThroughputExceededException`

### Investigation Areas

- Read Capacity Units (RCUs).
- Write Capacity Units (WCUs).
- Provisioned capacity.
- Throttling metrics.
- Consumed capacity.
- Auto scaling.
- Application retry behavior.

### Expected Outcome

Identify the throttling cause and restore reliable operations through appropriate capacity configuration or workload adjustments.

---

## Lab 06 — ConditionalCheckFailedException

### Production Incident

An application attempts to update or write an item using a condition that evaluates to false.

The operation fails with:

`ConditionalCheckFailedException`

### Investigation Areas

- ConditionExpression.
- Attribute existence checks.
- Conditional PutItem operations.
- Conditional UpdateItem operations.
- Version attributes.
- Optimistic concurrency control.

### Expected Outcome

Identify the failed condition and correct the write logic without accidentally overwriting valid data.

---

## Lab 07 — GSI Query Not Returning Results

### Production Incident

An application queries a DynamoDB Global Secondary Index but does not receive the expected records.

### Investigation Areas

- GSI name and status.
- Index partition key.
- Index sort key.
- KeyConditionExpression.
- Missing indexed attributes.
- Sparse indexes.
- Index projection.
- Eventual consistency.

### Expected Outcome

Identify why the items are not returned and correct the index query or data model.

---

## Lab 08 — DynamoDB Streams Not Triggering

### Production Incident

An application expects AWS Lambda to process DynamoDB item changes, but the expected processing does not occur.

### Investigation Areas

- DynamoDB Streams configuration.
- Stream ARN.
- Stream view type.
- Lambda event source mapping.
- IAM permissions.
- Lambda execution logs.
- Event source mapping state.
- Processing errors and retries.

### Expected Outcome

Restore the intended change-processing workflow and verify that new DynamoDB changes trigger Lambda.

---

## Lab 09 — Point-in-Time Recovery Investigation

### Production Incident

A critical DynamoDB item is accidentally deleted or modified.

The team must investigate the table's point-in-time recovery settings and determine whether recovery is possible.

### Investigation Areas

- PITR enablement.
- Available recovery window.
- Earliest and latest restorable times.
- RestoreTableToPointInTime.
- Restoring into a new table.
- Data verification.
- Recovery procedures.

### Expected Outcome

Restore the required data to a new table when recovery is available and verify the restored records before considering any production cutover.

---

## Lab 10 — DynamoDB Hot Partition

### Production Incident

A DynamoDB workload experiences uneven traffic distribution because a small number of partition-key values receive a disproportionate share of read or write requests.

### Investigation Areas

- Partition-key access patterns.
- Uneven request distribution.
- DynamoDB throttling.
- Consumed capacity.
- CloudWatch metrics.
- Contributor Insights where available.
- Partition-key design.
- Write sharding.
- Workload distribution.

### Expected Outcome

Identify the concentrated traffic pattern and evaluate remediation techniques that improve workload distribution.

---

## Standard Troubleshooting Methodology

Each lab follows a consistent production-style workflow.

### 1. Incident Scenario

Describe the production issue, affected application, symptoms, and expected behavior.

### 2. Architecture

Document the broken and healthy system architecture.

### 3. Environment Preparation

Configure AWS CLI, Linux Bash variables, AWS Region, and required tools.

### 4. Resource Deployment

Create the required AWS resources.

### 5. Fault Injection

Intentionally reproduce the target error in a safe lab environment.

### 6. Investigation

Use AWS CLI, CloudWatch, logs, metrics, and resource configurations to identify the problem.

### 7. Root Cause Analysis

Explain why the failure occurred and identify the configuration or application behavior responsible.

### 8. Remediation

Apply the smallest appropriate change to restore the service.

### 9. Recovery Verification

Confirm the operation succeeds and collect supporting evidence.

### 10. Cleanup

Remove temporary AWS resources to avoid unnecessary costs.

---

## Security Best Practices

This repository emphasizes Cloud Security Engineering throughout the labs.

Important principles include:

- Use IAM least privilege.
- Avoid unrestricted DynamoDB permissions.
- Verify AWS account and Region before changing resources.
- Use appropriate encryption settings.
- Protect sensitive data.
- Avoid committing credentials or secrets.
- Review resource-based access policies where applicable.
- Use CloudTrail for audit investigations.
- Enable appropriate monitoring.
- Protect production data from accidental deletion.
- Understand backup and recovery limitations.
- Verify resources before cleanup.

**All experiments should use temporary tables and fabricated data, never production customer records.**

---

## Cost Management

DynamoDB costs depend on capacity mode, data storage, requests, and optional features.

Potential charges include:

- DynamoDB read and write capacity.
- On-demand requests.
- Table and index storage.
- DynamoDB Streams consumption.
- Point-in-time recovery.
- Backups and restores.
- Lambda invocations.
- CloudWatch monitoring.

Labs should use small data sets and bounded test workloads.

Avoid uncontrolled request loops, large tables, unnecessary long-lived resources, or unnecessary backup retention.

Delete temporary resources once each lab is complete.

---

## Skills Demonstrated

By working through this repository, the following skills are practiced:

- Amazon DynamoDB operations.
- DynamoDB schema troubleshooting.
- NoSQL data modeling.
- IAM permission investigation.
- Read and write capacity management.
- CloudWatch monitoring.
- Database performance troubleshooting.
- DynamoDB query investigation.
- Conditional expressions.
- Global Secondary Indexes.
- DynamoDB Streams.
- Lambda integration.
- Point-in-time recovery.
- Incident response.
- Root cause analysis.
- Production remediation.
- Cloud Security Engineering.
- AWS CLI automation.
- Linux troubleshooting.
- AWS cost management.

---

## Interview Preparation

These labs help prepare for technical interviews involving:

- Junior Cloud Engineer.
- Cloud Operations Engineer.
- Cloud Security Engineer.
- DevOps Engineer.
- Platform Engineer.
- Infrastructure Engineer.
- AWS Cloud Engineer.

Example questions:

**Why does DynamoDB return ResourceNotFoundException?**

The specified table or index may not exist in the account and Region being used, or the referenced resource may not be in the expected state.

**How do you troubleshoot DynamoDB AccessDeniedException?**

Identify the caller, inspect relevant IAM and resource policies, check explicit denies, and verify that the required DynamoDB action is authorized for the intended resource.

**What causes DynamoDB throttling?**

Request traffic may exceed available capacity or applicable table, index, or partition-level limits.

**What is the difference between a partition key and a sort key?**

The partition key is used to determine data distribution. An optional sort key allows multiple items with the same partition-key value to be ordered and queried within that item collection.

**What is a DynamoDB Global Secondary Index?**

A GSI provides an alternative key schema for querying data.

**What is DynamoDB point-in-time recovery?**

PITR allows a table to be restored to a selected point within its available recovery window.

---

## Documentation Standards

Each completed lab includes a detailed `README.md` covering:

- Overview.
- Lab information.
- Production incident scenario.
- AWS architecture.
- Relevant AWS concepts.
- Prerequisites and safety.
- AWS CLI deployment.
- Fault injection.
- Investigation.
- CloudWatch evidence.
- Root cause analysis.
- Remediation.
- Recovery verification.
- Security considerations.
- Cost management.
- Resource cleanup.
- Interview questions.
- Lessons learned.
- Official AWS documentation.

The repository prioritizes practical investigation, clear explanations, secure remediation, and reproducible AWS CLI commands.

---

## Official AWS Documentation

- [Amazon DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
- [DynamoDB API Reference](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/Welcome.html)
- [DynamoDB AWS CLI Reference](https://docs.aws.amazon.com/cli/latest/reference/dynamodb/)
- [DynamoDB Security](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/security.html)
- [DynamoDB Capacity Modes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadWriteCapacityMode.html)
- [DynamoDB Global Secondary Indexes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html)
- [DynamoDB Streams](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.html)
- [DynamoDB Point-in-Time Recovery](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/PointInTimeRecovery.html)
- [DynamoDB CloudWatch Metrics](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/metrics-dimensions.html)
- [Amazon DynamoDB Pricing](https://aws.amazon.com/dynamodb/pricing/)

---

**Repository Focus:** Building practical AWS troubleshooting, incident investigation, performance analysis, recovery, and cloud security skills through 10 production-style Amazon DynamoDB labs.
