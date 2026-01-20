# Stock Market Kafka Data Engineering Project

A lightweight end-to-end data engineering pipeline that streams stock market data using **Apache Kafka**, processes it in real time, and persists consumed data to **Amazon S3**.

---

## Architecture Overview

**Data Flow**:

CSV Dataset → Kafka Producer → Kafka Topic → Kafka Consumer → Amazon S3

* **Kafka Producer**: Randomly samples stock market records from a CSV file and publishes them as JSON messages to a Kafka topic.
* **Kafka Broker**: Handles message streaming and durability.
* **Kafka Consumer**: Subscribes to the topic, consumes messages, and stores them in S3 for downstream analytics.

---

## Tech Stack

* Python 3.x
* Apache Kafka
* kafka-python
* Pandas
* AWS S3 (via boto3)

---

## Project Structure

```
.
├── kafka_producer.py        # Publishes stock data to Kafka
├── kafka_consumer.py        # Consumes data from Kafka
├── data/
│   └── indexProcessed.csv   # Stock market dataset
├── requirements.txt
└── README.md
```

---

## Prerequisites

* Running Kafka cluster (local or EC2-based)
* AWS account with an S3 bucket
* AWS credentials configured locally (`~/.aws/credentials`)

---

## Installation

```bash
pip install -r requirements.txt
```

---

## Kafka Producer

Streams random stock records from a CSV file to Kafka.

```bash
python kafka_producer.py
```

Key behavior:

* Reads from `indexProcessed.csv`
* Sends JSON messages to a Kafka topic at fixed intervals

---

## Kafka Consumer (S3 Sink)

Consumes Kafka messages and uploads them to S3 as JSON or CSV files.

### Example S3 Consumer Logic

```python
import json
from json import dump, json
from kafka import KafkaConsumer

s3 = boto3.client('s3')
BUCKET = "kafka-stk-market"

consumer = KafkaConsumer(
    "demo_test",
    bootstrap_servers=["<EC2-PUBLIC-IP>:9092"],
    value_deserializer=lambda m: json.loads(m.decode("utf-8")),
    auto_offset_reset="earliest"
)

for msg in consumer:
    s3.put_object(
        Bucket=BUCKET,
        Key=f"kafka-data/{msg.offset}.json",
        Body=json.dumps(msg.value)
    )
```

---

