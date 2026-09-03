# Week 9 - Day 17: Messaging and Streaming

## Name
Sanket Dangat


## Tasks Completed
- [x] Watched/read the weekly content
- [x] Completed hands-on labs
- [x] Added screenshots or proof
- [ ] Posted on LinkedIn
- [x] Cleaned up AWS resources


## Architecture

## Event Driven Order Processing

![Architecture](diagram/event-driven-order-processing-architecture.gif)

---

## Kinesis Firehos S3 Streaming

![Architecture](diagram/kinesis-firehose-s3-architecture.gif)


## Result

Successfully completed the Messaging and Streaming Practical covering Amazon SQS Standard queues, visibility timeout, Dead-Letter Queues (DLQ) and redrive, FIFO ordering, Amazon SNS fanout with subscription filtering, Amazon EventBridge custom event buses and rules, EventBridge Scheduler, Kinesis Data Streams, Amazon Data Firehose, and Amazon S3 delivery.

Demonstrated the complete streaming flow: `Kinesis Data Streams → Amazon Data Firehose → Amazon S3`

Amazon MQ and Amazon MSK were reviewed as walkthrough/selection topics only, with no broker or cluster created.

---

## Resources Created

- SQS Dead-Letter Queue `cloudadhar-orders-dlq-day17`
- SQS Standard Queue `cloudadhar-orders-standard-day17`
- SQS FIFO Queue `cloudadhar-orders-fifo-day17.fifo`
- SQS Priority Queue `cloudadhar-priority-orders-day17`
- SNS Topic `cloudadhar-orders-topic-day17`
- EventBridge Custom Bus `cloudadhar-orders-bus-day17`
- EventBridge Rule `cloudadhar-high-value-orders-rule-day17`
- EventBridge Scheduler `cloudadhar-payment-reminder-day17`
- Kinesis Data Stream `cloudadhar-clickstream-day17`
- Amazon Data Firehose `cloudadhar-clickstream-firehose-day17`
- Private S3 Bucket `cloudadhar-day17-streaming-<ACCOUNT-ID>-ap-south-1-an`
- IAM execution/service roles created by AWS Console for EventBridge Scheduler and Firehose

---

## Screenshots

### Part A - Amazon SQS

#### 1. SQS Dead-Letter Queue

Created the Standard DLQ `cloudadhar-orders-dlq-day17` with 30-second visibility timeout, 14-day retention, and SSE-SQS encryption.

![SQS DLQ](screenshots/01_SQS_DLQ.png)

---

#### 2. Standard SQS Queue

Created `cloudadhar-orders-standard-day17` with a 30-second visibility timeout, 4-day message retention, 20-second long polling, and SSE-SQS encryption.

![Standard SQS](screenshots/02_SQS_Standard.png)

---

#### 3. Standard Queue DLQ Configuration

Configured `cloudadhar-orders-standard-day17` to use `cloudadhar-orders-dlq-day17` as its Dead-Letter Queue with a maximum receive count of `3`.

![SQS DLQ Configuration](screenshots/03_SQS_DLQ_Configuration.png)

---

#### 4. SQS Visibility Timeout Test

Sent a test message and polled it without deleting it to demonstrate the visibility timeout. The message became in flight, remained temporarily invisible during the 30-second visibility timeout, and became visible again after the timeout expired.

![SQS Message In Flight](screenshots/04a_SQS_Message_In_Flight.png)

![SQS Message Visible Again](screenshots/04b_SQS_Message_Visible_Again.png)

---

#### 5. SQS DLQ and Redrive

Repeatedly received the test message without deleting it, observed the receive count increase, verified that the message moved to the DLQ after exceeding the configured maximum receive count, and successfully redrove the message back to the source queue.

![SQS Message in DLQ](screenshots/05a_SQS_Message_In_DLQ.png)

![SQS DLQ Redrive](screenshots/05b_SQS_DLQ_Redrive.png)

---

#### 6. FIFO Queue Configuration

Created `cloudadhar-orders-fifo-day17.fifo` with FIFO ordering, 120-second visibility timeout, and content-based deduplication disabled.

![SQS FIFO Configuration](screenshots/06_SQS_FIFO_Configuration.png)

---

#### 7. FIFO Ordering Test

- Sent `Payment received` and `Order shipped` using the same Message Group ID `(order-O-2001)`.
- The first poll received `"Payment received"` from the FIFO queue. While the message remained in flight, subsequent polling did not release `"Order shipped"`.
- After the visibility timeout expired, the newly received `Payment received` message was successfully deleted using its current receipt handle.
- The next poll then released `"Order shipped"`, confirming that FIFO ordering was maintained within the same message group.

![SQS_FIFO_Messages_Sent](screenshots/07a_SQS_FIFO_Messages_Sent.png)

![SQS_FIFO_Payment_Received](screenshots/07b_SQS_FIFO_Payment_Received.png)

![SQS_FIFO_Ordering_Blocked](screenshots/07c_SQS_FIFO_Ordering_Blocked.png)

![SQS_FIFO_Order_Shipped](screenshots/07d_SQS_FIFO_Order_Shipped.png)

---

#### 8. Create the Priority Queue

Created the Standard priority queue `cloudadhar-priority-orders-day17` with a 30-second visibility timeout, 4-day message retention, 20-second receive message wait time, and SSE-SQS encryption. Added the project tag and created the queue.

![SQS Priority Queue](screenshots/08_SQS_Priority_Queue.png)

---

### Part B - Amazon SNS

#### 9. SNS Topic and Subscriptions

Created SNS topic `cloudadhar-orders-topic-day17` and configured subscriptions for both the Standard orders queue and Priority orders queue.

![SNS Topic Subscriptions](screenshots/09_SNS_Topic_Subscriptions.png)

---

#### 10. SNS HIGH-Priority Filter

Configured the Priority queue subscription with a message attribute filter allowing only messages where `priority = HIGH`.

![SNS HIGH Filter](screenshots/10_SNS_HIGH_Filter.png)

---

#### 11. SNS Fanout and Filtering Test

Published **NORMAL** and **HIGH** priority orders to the SNS topic and verified that subscription filtering routed the messages to the expected SQS queues.

**NORMAL order:** Delivered to the **Standard orders queue** only.

![SNS NORMAL Order Filtering Test](screenshots/11_SNS_Normal_Filtering_Test.png)

**HIGH order:** Delivered to both the **Standard orders queue** and **Priority orders queue**.

![SNS HIGH Order Fanout and Filtering Test](screenshots/11_SNS_Fanout_Test.png)

---

### Part C - Amazon EventBridge

#### 12. EventBridge Custom Event Bus

Created the custom EventBridge event bus `cloudadhar-orders-bus-day17`.

![EventBridge Custom Bus](screenshots/12_EventBridge_Custom_Bus.png)

---

#### 13. EventBridge High-Value Rule

Created `cloudadhar-high-value-orders-rule-day17` with an event pattern matching orders where the amount is greater than `5000`.

![EventBridge Rule Pattern](screenshots/13_EventBridge_Rule_Pattern.png)

---

#### 14. EventBridge Negative Test

Sent an order event with amount `2500` and verified that it did not match the rule and no message was delivered to the Priority queue.

![EventBridge Negative Test](screenshots/14_EventBridge_Negative_Test.png)

---

#### 15. EventBridge Positive Test

Sent an order event with amount `7500` and verified that it matched the rule and was successfully delivered to the Priority SQS queue.

![EventBridge Positive Test](screenshots/15_EventBridge_Positive_Test.png)

---

### Part D - EventBridge Scheduler

#### 16. EventBridge Scheduler

Created the one-time schedule `cloudadhar-payment-reminder-day17` with Amazon SQS as the target.

![EventBridge Scheduler](screenshots/16_EventBridge_Scheduler.png)

---

#### 17. Scheduler to SQS Result

Verified that the scheduled payment reminder was successfully delivered to the Priority SQS queue.

![Scheduler SQS Result](screenshots/17_Scheduler_SQS_Result.png)

---

### Part E - Kinesis Data Streams and Amazon Data Firehose

#### 18. Kinesis Data Stream

Created `cloudadhar-clickstream-day17` using On-demand capacity mode with 1-day record retention.

![Kinesis Stream](screenshots/18_Kinesis_Stream.png)

---

#### 19. Kinesis Data Viewer

Produced clickstream records and verified them using the Kinesis Data Viewer. Records using the same partition key were verified in the same shard.

![Kinesis Data Viewer](screenshots/19_Kinesis_Data_Viewer.png)

---

#### 20. Firehose to S3 Configuration

Created `cloudadhar-clickstream-firehose-day17` with Kinesis Data Streams as the source and Amazon S3 as the destination.

![Firehose S3 Configuration](screenshots/20_Firehose_S3_Configuration.png)

---

#### 21. Complete Kinesis → Firehose → S3 Flow

Sent records to Kinesis, delivered them through Amazon Data Firehose, and verified that the records were successfully stored as objects in the private S3 bucket.

![Kinesis Firehose S3 Result](screenshots/21_Kinesis_Firehose_S3_Result.png)

---

### Part F - Amazon MQ and Amazon MSK

#### 22. Amazon MQ

- Reviewed Amazon MQ → Brokers → Create broker without creating a broker.
- Inspected RabbitMQ and ActiveMQ engines, broker sizing, deployment options, VPC networking, private access, authentication, encryption, and maintenance.
- **Selection:** Amazon MQ is suitable for applications that already use RabbitMQ or ActiveMQ and require compatibility with their existing broker protocols and semantics.

#### 23. Amazon MSK

- Reviewed Amazon MSK → Clusters → Create cluster without creating a cluster.
- Inspected MSK Provisioned and Serverless, Kafka versions, topics, partitions, consumer groups, VPC networking, authentication, encryption, monitoring, and Multi-AZ architecture.
- **Selection:** Amazon MSK is suitable for applications that require Apache Kafka APIs, Kafka clients, consumer groups, offsets, retained topics, or Kafka ecosystem compatibility.

> **Key Difference:** Amazon MQ is for RabbitMQ/ActiveMQ workloads, while Amazon MSK is for Apache Kafka workloads.

---

## Cleanup

**Day 17 cleanup should be performed only after all required evidence has been captured.**

1. Stop any running Firehose demo-data delivery.
2. Delete the Firehose stream `cloudadhar-clickstream-firehose-day17`.
3. Delete the Kinesis Data Stream `cloudadhar-clickstream-day17`.
4. Empty the S3 bucket `cloudadhar-day17-streaming-<ACCOUNT-ID>-ap-south-1-an`.
5. Delete the S3 bucket `cloudadhar-day17-streaming-<ACCOUNT-ID>-ap-south-1-an`.
6. Delete the EventBridge rule `cloudadhar-high-value-orders-rule-day17`.
7. Delete the custom EventBridge event bus `cloudadhar-orders-bus-day17`.
8. Verify that the one-time EventBridge Scheduler `cloudadhar-payment-reminder-day17` has been automatically deleted after completion; if it still exists, delete it manually.
9. Delete the SNS subscriptions from `cloudadhar-orders-topic-day17`.
10. Delete the SNS topic `cloudadhar-orders-topic-day17`.
11. Purge and delete the SQS Priority queue `cloudadhar-priority-orders-day17`.
12. Purge and delete the SQS FIFO queue `cloudadhar-orders-fifo-day17.fifo`.
13. Disable the dead-letter queue configuration on `cloudadhar-orders-standard-day17`, then purge and delete the SQS Standard queue.
14. Purge and delete the SQS Dead-Letter Queue `cloudadhar-orders-dlq-day17`.
15. Delete IAM execution/service roles created specifically for the Day 17 Scheduler, EventBridge, or Firehose resources, only after confirming they are no longer in use.
16. Verify that no Day 17 Kinesis streams, Firehose streams, S3 buckets, EventBridge rules/buses/schedules, SNS topics/subscriptions, or SQS queues remain.
17. Verify that no temporary IAM roles or permissions created specifically for Day 17 remain.
18. Verify that no Amazon MQ broker or Amazon MSK cluster was accidentally created.
19. Verify that no billable Day 17 resources remain in Mumbai (`ap-south-1`).