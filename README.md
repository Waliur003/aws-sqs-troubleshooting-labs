# Amazon SQS Troubleshooting Labs

**Production-Style AWS Cloud Engineering & Cloud Security Engineering Troubleshooting**

## Overview

This repository contains 10 hands-on Amazon Simple Queue Service (Amazon SQS) troubleshooting labs.

The goal is to learn how to investigate, reproduce, diagnose, and remediate real-world messaging problems using AWS CLI, Linux Bash, AWS Lambda, IAM, and Amazon CloudWatch.

Amazon SQS is a fully managed message queuing service that helps applications communicate asynchronously.

It is commonly used in production systems for:

- Microservices communication.
- Background job processing.
- Order processing systems.
- Event-driven architectures.
- Serverless applications.
- Distributed systems.
- Application decoupling.
- Workload buffering.
- Fault-tolerant processing.
- Cloud-native infrastructure.

Although Amazon SQS is managed by AWS, application failures can still occur because of incorrect configurations, insufficient permissions, improperly managed visibility timeouts, missing consumers, or failed message processing.

These labs simulate those situations using dedicated AWS resources.

Each lab follows a production incident investigation process.

The repository is designed for practical Cloud Engineer, Cloud Operations Engineer, DevOps Engineer, and Cloud Security Engineer preparation.

---

## 1. Repository Objectives

By completing these labs, the engineer develops practical experience in:

- Creating and configuring SQS queues.
- Investigating AWS CLI errors.
- Troubleshooting IAM authorization failures.
- Understanding message lifecycle behavior.
- Investigating missing and delayed messages.
- Configuring visibility timeouts.
- Understanding at-least-once delivery.
- Designing idempotent message consumers.
- Configuring dead-letter queues.
- Troubleshooting failed redrive policies.
- Understanding FIFO queue ordering and deduplication.
- Connecting AWS Lambda to SQS.
- Investigating poison messages.
- Monitoring queue backlogs.
- Measuring message processing delays.
- Applying least-privilege IAM permissions.
- Performing root cause analysis.
- Cleaning up AWS resources safely.

---

## 2. Troubleshooting Labs

| Lab | Troubleshooting Scenario | Primary Focus |
|---|---|---|
| 01 | Queue Does Not Exist | Queue URLs, names, Regions, resource discovery |
| 02 | SendMessage AccessDenied | IAM policies, identity, queue permissions |
| 03 | Messages Not Being Received | Visibility, consumers, long polling |
| 04 | Visibility Timeout Misconfiguration | In-flight messages and timeout settings |
| 05 | Messages Processed Multiple Times | At-least-once delivery, idempotency |
| 06 | Dead-Letter Queue Not Receiving Messages | Redrive policies and receive counts |
| 07 | FIFO Message Deduplication | MessageGroupId, MessageDeduplicationId |
| 08 | Lambda SQS Event Source Failure | Event source mapping, IAM, Lambda errors |
| 09 | Poison Message Investigation | Failed processing, DLQ, safe redrive |
| 10 | Queue Backlog and Processing Delay | Queue depth, message age, consumer capacity |

The labs progress from basic queue troubleshooting to production-style distributed message-processing incidents.

---

## 3. AWS Architecture

### Basic Message Queue Architecture

```text
                 Producer Application
                          |
                          v
                     SendMessage
                          |
                          v
                    Amazon SQS
                          |
                          v
                    Message Queue
                          |
                          v
                  Consumer Application
                          |
                          v
                    ReceiveMessage
                          |
                          v
                    Process Message
                          |
                          v
                    DeleteMessage
```

### Production Architecture With Dead-Letter Queue

```text
                     Producer
                        |
                        v
                    Amazon SQS
                    Source Queue
                        |
                        v
                   Message Consumer
                        |
                 +------+------+
                 |             |
                 v             v
              Success        Failure
                 |             |
                 v             v
            DeleteMessage   Retry After
                            Visibility Timeout
                                  |
                                  v
                            Receive Count
                                  |
                                  v
                           maxReceiveCount
                              Exceeded
                                  |
                                  v
                           Dead-Letter Queue
                                  |
                                  v
                             Investigation
                                  |
                                  v
                            Fix and Redrive
```

### Serverless Architecture

```text
                 Order Application
                        |
                        v
                  Amazon SQS
                        |
                        v
               Lambda Event Source
                    Mapping
                        |
                        v
                    AWS Lambda
                        |
                        v
                Process Message Batch
                        |
                  +-----+-----+
                  |           |
                  v           v
               Success     Failure
                  |           |
                  v           v
             Message       Retry / DLQ
             Removed
```

AWS Lambda polls SQS using event source mappings and processes messages in batches. A failed batch can cause messages to become visible again after the visibility timeout, depending on configuration.

---

## 4. Understanding Amazon SQS

Amazon SQS provides managed message queues for decoupled applications.

### Standard Queue

Standard queues support high throughput and at-least-once message delivery.

Characteristics:

- At-least-once delivery.
- Best-effort ordering.
- Possible duplicate delivery.
- High scalability.
- Suitable for general asynchronous processing.

### FIFO Queue

FIFO queues support ordered message processing within message groups and deduplication within the supported deduplication interval.

Characteristics:

- Message group ordering.
- Message deduplication.
- `MessageGroupId` support.
- `MessageDeduplicationId` support.
- Five-minute deduplication interval.
- Suitable for workloads requiring ordered processing.

### Standard vs. FIFO

| Feature | Standard | FIFO |
|---|---|---|
| Delivery | At least once | Deduplicated sends within supported interval |
| Ordering | Best effort | Within message groups |
| Duplicate Delivery | Possible | Deduplication protections, with consumer idempotency still recommended |
| MessageGroupId | Not required | Required |
| MessageDeduplicationId | Not required | Required unless content-based deduplication is enabled |
| Common Use | General background jobs | Ordered workflows |

**Important:** FIFO deduplication does not eliminate the need for idempotent application processing.

---

## 5. Understanding the SQS Message Lifecycle

```text
Producer Sends Message
          |
          v
Message Becomes Available
          |
          v
Consumer Receives Message
          |
          v
Message Becomes In-Flight
          |
          v
Visibility Timeout Begins
          |
     +----+----+
     |         |
     v         v
Processing   Processing
Success      Failure
     |         |
     v         v
DeleteMessage  Timeout Expires
     |         |
     v         v
Message       Message Becomes
Removed       Visible Again
                   |
                   v
                  Retry
```

A successful ReceiveMessage does not delete the message.

The consumer must delete the processed message using its receipt handle.

### Important SQS Concepts

| Concept | Meaning |
|---|---|
| Queue URL | API identifier for a queue |
| Queue ARN | AWS resource identifier |
| Message ID | Identifier for a sent message |
| Receipt Handle | Token used to delete or change visibility of a received message |
| Visibility Timeout | Time a received message is temporarily hidden |
| Long Polling | Waits for messages to arrive |
| Message Retention | How long messages remain available before expiration |
| In-Flight Message | Received but not deleted message |
| DLQ | Stores messages that fail processing repeatedly |
| Redrive Policy | Controls movement to a DLQ |
| ApproximateReceiveCount | Approximate number of receives |

---

## 6. Lab 01 — Queue Does Not Exist

### Production Scenario

An application attempts to access an SQS queue but receives an error indicating the queue does not exist.

### Investigation Areas

- Incorrect queue name.
- Incorrect AWS Region.
- Incorrect AWS account.
- Incorrect queue URL.
- Deleted queue.
- IAM permissions affecting queue discovery.

### AWS CLI Operations

```bash
aws sqs list-queues
aws sqs get-queue-url
aws sqs create-queue
aws sqs get-queue-attributes
```

### Expected Skills

- Queue discovery.
- AWS resource identification.
- Region verification.
- Queue URL troubleshooting.
- Safe queue creation.

---

## 7. Lab 02 — SendMessage AccessDenied

### Production Scenario

An application attempts to publish a message to SQS but lacks sufficient permissions.

### Investigation Areas

- IAM identity policies.
- Queue resource policies.
- Explicit denies.
- Cross-account permissions.
- KMS permissions when using customer-managed encryption keys.

### AWS CLI Operations

```bash
aws sts get-caller-identity
aws sqs send-message
aws sqs get-queue-attributes
```

### Expected Skills

- IAM authorization troubleshooting.
- Queue policy investigation.
- Least-privilege access.
- Secure message publishing.

---

## 8. Lab 03 — Messages Not Being Received

### Production Scenario

An application successfully sends messages to a queue, but its consumer reports no messages available.

### Investigation Areas

- Queue URL.
- Consumer configuration.
- Message visibility.
- Long polling.
- Delivery delay.
- Competing consumers.
- Approximate message counts.

### AWS CLI Operations

```bash
aws sqs send-message
aws sqs receive-message
aws sqs get-queue-attributes
```

### Expected Skills

- Message lifecycle investigation.
- Visibility analysis.
- Long polling.
- Consumer troubleshooting.

---

## 9. Lab 04 — Visibility Timeout Misconfiguration

### Production Scenario

A consumer receives a message but takes longer to process it than the configured visibility timeout.

The message may become visible again before the first consumer finishes processing.

### Investigation Areas

- Queue visibility timeout.
- Processing duration.
- In-flight messages.
- ChangeMessageVisibility.
- Duplicate processing risk.

### AWS CLI Operations

```bash
aws sqs get-queue-attributes
aws sqs set-queue-attributes
aws sqs receive-message
aws sqs change-message-visibility
```

### Expected Skills

- Visibility timeout configuration.
- Processing reliability.
- Message retry behavior.
- In-flight message management.

---

## 10. Lab 05 — Messages Processed Multiple Times

### Production Scenario

An application performs the same operation more than once because a message is received multiple times.

### Investigation Areas

- Standard queue at-least-once delivery.
- Missing DeleteMessage.
- Visibility timeout expiration.
- Consumer crashes.
- Duplicate message handling.
- Idempotent processing.

### AWS CLI Operations

```bash
aws sqs send-message
aws sqs receive-message
aws sqs delete-message
```

### Expected Skills

- At-least-once processing.
- Idempotency design.
- Duplicate message investigation.
- Safe consumer implementation.

---

## 11. Lab 06 — Dead-Letter Queue Not Receiving Messages

### Production Scenario

A message repeatedly fails processing, but the expected dead-letter queue remains empty.

### Investigation Areas

- RedrivePolicy.
- maxReceiveCount.
- DLQ ARN.
- Queue type compatibility.
- Message receive attempts.
- RedriveAllowPolicy.
- Retention settings.

### AWS CLI Operations

```bash
aws sqs create-queue
aws sqs set-queue-attributes
aws sqs get-queue-attributes
aws sqs receive-message
```

### Expected Skills

- Dead-letter queue configuration.
- Redrive troubleshooting.
- Receive-count investigation.
- Failure isolation.

---

## 12. Lab 07 — FIFO Message Deduplication

### Production Scenario

A producer sends a FIFO message, but subsequent sends with the same deduplication identifier do not create additional deliverable messages during the deduplication interval.

### Investigation Areas

- FIFO queue configuration.
- MessageGroupId.
- MessageDeduplicationId.
- Content-based deduplication.
- Deduplication interval.
- Message ordering.

### AWS CLI Operations

```bash
aws sqs create-queue
aws sqs send-message
aws sqs receive-message
```

### Expected Skills

- FIFO queue behavior.
- Message ordering.
- Deduplication investigation.
- Reliable producer design.

---

## 13. Lab 08 — Lambda SQS Event Source Failure

### Production Scenario

Messages accumulate in SQS, but Lambda does not process them successfully.

### Investigation Areas

- Lambda event source mapping.
- IAM execution role.
- Queue visibility timeout.
- Lambda execution timeout.
- Lambda errors.
- Batch failures.
- Partial batch responses.
- CloudWatch Logs.

### AWS CLI Operations

```bash
aws lambda list-event-source-mappings
aws lambda get-event-source-mapping
aws lambda update-event-source-mapping
aws logs tail
```

### Expected Skills

- SQS–Lambda integration.
- Event source mapping troubleshooting.
- IAM permissions.
- Failed batch handling.
- Serverless incident investigation.

---

## 14. Lab 09 — Poison Message Investigation

### Production Scenario

One malformed or invalid message repeatedly causes consumer processing failures.

### Investigation Areas

- Invalid message payloads.
- Application exceptions.
- Retry behavior.
- ApproximateReceiveCount.
- Poison message isolation.
- DLQ processing.
- Safe message redrive.

### Expected Skills

- Poison message identification.
- Failure isolation.
- Consumer validation.
- Retry limits.
- Dead-letter queue recovery.

---

## 15. Lab 10 — Queue Backlog and Processing Delay

### Production Scenario

Messages arrive faster than consumers can process them.

The queue backlog grows, and customers experience processing delays.

### Investigation Areas

- ApproximateNumberOfMessages.
- ApproximateNumberOfMessagesNotVisible.
- ApproximateAgeOfOldestMessage.
- Message arrival rate.
- Processing throughput.
- Consumer concurrency.
- Lambda scaling.
- CloudWatch alarms.

### Expected Skills

- Backlog investigation.
- Queue-depth monitoring.
- Consumer capacity analysis.
- Processing-delay troubleshooting.
- Scaling recommendations.

---

## 16. Production Troubleshooting Workflow

Every lab follows a consistent incident investigation methodology.

### Phase 1 — Prepare

- Verify AWS account and Region.
- Create dedicated lab resources.
- Configure environment variables.
- Identify expected behavior.

### Phase 2 — Establish Baseline

- Verify healthy queue configuration.
- Send and receive test messages where appropriate.
- Confirm expected consumer behavior.

### Phase 3 — Inject Fault

- Introduce one controlled misconfiguration or application failure.
- Reproduce the expected symptom.
- Capture relevant AWS CLI output.

### Phase 4 — Investigate

- Inspect queue attributes.
- Inspect IAM policies.
- Inspect message states.
- Review CloudWatch metrics.
- Identify the underlying cause.

### Phase 5 — Remediate

- Correct the configuration.
- Apply least privilege.
- Preserve message integrity.
- Avoid unnecessary resource replacement.

### Phase 6 — Verify

- Repeat the original operation.
- Confirm the expected message behavior.
- Verify consumer processing.
- Check logs and metrics.

### Phase 7 — Cleanup

- Delete temporary queues.
- Remove temporary IAM and Lambda resources.
- Verify resource deletion.
- Preserve useful source files and documentation.

---

## 17. Development Environment

### Operating System

WSL2 Ubuntu.

### AWS Region

```bash
export AWS_REGION="us-east-1"
export AWS_DEFAULT_REGION="$AWS_REGION"
export AWS_PAGER=""
```

### Required Tools

| Tool | Purpose |
|---|---|
| AWS CLI v2 | AWS resource management |
| Linux Bash | CLI workflows and scripts |
| jq | JSON processing |
| Python 3 | Optional consumer simulations |
| Boto3 | Python AWS SDK |
| AWS Lambda | Event-driven message consumers |
| CloudWatch | Metrics and logs |
| IAM | Access management |
| Git | Version control |
| GitHub | Lab documentation |

### Verify Tools

```bash
aws --version
python3 --version
jq --version
git --version
aws sts get-caller-identity
```

---

## 18. CloudWatch Monitoring

Important SQS CloudWatch metrics include:

| Metric | Purpose |
|---|---|
| ApproximateNumberOfMessagesVisible | Available messages |
| ApproximateNumberOfMessagesNotVisible | In-flight messages |
| ApproximateAgeOfOldestMessage | Approximate age of oldest unprocessed message |
| NumberOfMessagesSent | Messages sent |
| NumberOfMessagesReceived | Message receives |
| NumberOfMessagesDeleted | Message deletion operations |
| NumberOfEmptyReceives | Receives that return no messages |

### Production Monitoring Goals

- Detect growing backlogs.
- Identify delayed consumers.
- Investigate repeated receives.
- Monitor failed processing.
- Detect unexpectedly old messages.
- Identify dead-letter queue activity.
- Improve system reliability.

SQS metrics are approximate and should be interpreted alongside application logs and consumer telemetry.

---

## 19. Cloud Security Engineering Considerations

Amazon SQS is commonly used in security-sensitive distributed applications.

Important security practices include:

### IAM Least Privilege

Grant producers permission to send messages and consumers permission to receive and delete messages only from the intended queues.

### Queue Resource Policies

Use appropriate queue policies to control resource access, especially for cross-account integrations.

### Encryption

Review SQS encryption at rest and encryption in transit.

For customer-managed KMS keys, verify the required KMS permissions and key policy.

### Sensitive Message Data

Avoid placing unnecessary secrets, passwords, or sensitive customer information directly in message bodies.

### Dead-Letter Queue Protection

DLQs may contain failed messages with sensitive data.

Apply appropriate IAM permissions, encryption, and retention settings.

### Idempotency

Use application-level idempotency to prevent duplicate financial, inventory, or order-processing operations.

### Monitoring

Investigate unexpected message volume, consumer failures, permission changes, and growing backlogs.

### Security Principle

Reliable message-processing systems require secure access, correct retry behavior, failure isolation, idempotency, and operational monitoring.

---

## 20. AWS Cost Management

Amazon SQS is billed based on request usage and applicable service features.

Additional costs may arise from:

- Lambda invocations.
- CloudWatch Logs.
- CloudWatch alarms.
- Customer-managed KMS key usage.
- Excessive message retries.
- High-frequency empty polling.

### Cost-Safety Practices

- Use small test messages.
- Avoid uncontrolled producer loops.
- Use bounded message receive attempts.
- Use long polling when appropriate.
- Delete temporary queues after experiments.
- Avoid unnecessary Lambda invocations.
- Monitor application retries.

The labs use disposable resources and controlled workloads rather than production-scale load tests.

---

## 21. Repository Structure

```text
aws-sqs-troubleshooting-labs/
│
├── README.md
├── .gitignore
│
├── 01-queue-does-not-exist/
│   └── README.md
│
├── 02-sendmessage-access-denied/
│   └── README.md
│
├── 03-messages-not-being-received/
│   └── README.md
│
├── 04-visibility-timeout-misconfiguration/
│   └── README.md
│
├── 05-messages-processed-multiple-times/
│   └── README.md
│
├── 06-dlq-not-receiving-messages/
│   └── README.md
│
├── 07-fifo-message-deduplication/
│   └── README.md
│
├── 08-lambda-sqs-event-source-failure/
│   └── README.md
│
├── 09-poison-message-investigation/
│   └── README.md
│
└── 10-queue-backlog-processing-delay/
    └── README.md
```

This is the planned final structure. Individual lab files will be created after completing each hands-on exercise.

### Lab Folder Contents

Depending on the scenario, folders may contain:

```text
XX-lab-name/
├── README.md
├── scripts/
├── policies/
└── diagrams/
```

Source code, scripts, and reusable policies can be committed to GitHub after review.

Do not commit credentials, secrets, unreviewed logs, or sensitive production data.

---

## 22. Expected Skills After Completing All Labs

| Skill Category | Topics |
|---|---|
| Queue Operations | CreateQueue, GetQueueUrl, SendMessage, ReceiveMessage |
| Message Lifecycle | Visibility timeout, in-flight messages, DeleteMessage |
| Reliability | Retries, duplicate delivery, idempotency |
| Failure Handling | DLQ, poison messages, redrive |
| FIFO Queues | Ordering, grouping, deduplication |
| Serverless | SQS–Lambda event source mapping |
| IAM Security | Identity policies, queue policies, KMS |
| Monitoring | CloudWatch metrics, backlog, message age |
| Troubleshooting | AWS CLI diagnostics and root cause analysis |
| Operations | Cleanup, cost management, production incident response |

### Interview Preparation

After these labs, an engineer should be able to explain:

1. Why messages may be missing from a consumer's results.
2. How visibility timeouts affect retry behavior.
3. Why standard SQS messages can be delivered multiple times.
4. Why idempotency is necessary.
5. How dead-letter queues isolate repeatedly failing messages.
6. How FIFO deduplication works.
7. How Lambda polls SQS queues.
8. Why Lambda can repeatedly process failed message batches.
9. How poison messages affect consumer performance.
10. How to investigate and reduce queue-processing delays.

---

## 23. Official AWS Documentation

- [Amazon SQS Developer Guide](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html)
- [AWS CLI SQS Reference](https://docs.aws.amazon.com/cli/latest/reference/sqs/)
- [Amazon SQS Visibility Timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html)
- [Using Dead-Letter Queues](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html)
- [Amazon SQS FIFO Queues](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-fifo-queues.html)
- [Using Lambda with Amazon SQS](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html)
- [Lambda SQS Error Handling](https://docs.aws.amazon.com/lambda/latest/dg/services-sqs-errorhandling.html)
- [Monitoring Amazon SQS with CloudWatch](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-monitoring-using-cloudwatch.html)
- [Amazon SQS Pricing](https://aws.amazon.com/sqs/pricing/)

---

## Conclusion

This repository provides hands-on, production-style troubleshooting experience with Amazon SQS.

The labs progress from basic queue configuration and authorization errors to more advanced message-delivery, retry, dead-letter queue, FIFO, Lambda integration, and backlog scenarios.

The primary goal is not only to understand Amazon SQS concepts but also to develop the practical skills required to investigate messaging incidents, identify root causes, implement safe remediation, and verify recovery.

The repository supports preparation for Cloud Engineering, Cloud Operations, DevOps, and Cloud Security Engineering roles.
