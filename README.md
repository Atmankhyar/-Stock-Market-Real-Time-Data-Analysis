# Kafka Stock Market Project

This project streams stock market records from a CSV file into an Apache Kafka topic, consumes the messages, and writes them to Amazon S3 as JSON files.

## Architecture

![Project architecture](Architecture.jpg)

## Project Structure

```text
.
|-- Architecture.jpg
|-- KafkaProducer.ipynb
|-- KafkaConsumer.ipynb
|-- command_kafka.txt
|-- indexProcessed.csv
`-- Kafka-stock-market-project.pem
```

## Requirements

- Python 3.8+
- Jupyter Notebook
- Apache Kafka
- Java 8 or newer
- AWS account with access to the target S3 bucket

Python packages:

```bash
pip install pandas kafka-python s3fs jupyter
```

## How to Run

### 1. Start Kafka

Install and extract Kafka on the machine that will run the broker:

```bash
wget https://downloads.apache.org/kafka/3.3.1/kafka_2.12-3.3.1.tgz
tar -xvf kafka_2.12-3.3.1.tgz
cd kafka_2.12-3.3.1
```

Start ZooKeeper in one terminal:

```bash
bin/zookeeper-server-start.sh config/zookeeper.properties
```

Start Kafka in a second terminal:

```bash
export KAFKA_HEAP_OPTS="-Xmx256M -Xms128M"
bin/kafka-server-start.sh config/server.properties
```

If Kafka is running on EC2, update `config/server.properties` so `advertised.listeners` points to the EC2 public IP:

```text
advertised.listeners=PLAINTEXT://<EC2_PUBLIC_IP>:9092
```

Also make sure the EC2 security group allows inbound traffic on port `9092` from your client IP.

### 2. Create the Kafka Topic

In a third terminal, create the topic used by the notebooks:

```bash
bin/kafka-topics.sh --create \
  --topic demo_testing2 \
  --bootstrap-server <EC2_PUBLIC_IP>:9092 \
  --replication-factor 1 \
  --partitions 1
```

To confirm the topic exists:

```bash
bin/kafka-topics.sh --list --bootstrap-server <EC2_PUBLIC_IP>:9092
```

### 3. Set Up the Python Environment

From the project folder:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install pandas kafka-python s3fs jupyter
jupyter notebook
```

### 4. Configure the Notebooks

Open both notebooks in Jupyter:

- `KafkaProducer.ipynb`
- `KafkaConsumer.ipynb`

Update the Kafka broker address in both notebooks:

```python
bootstrap_servers=['<EC2_PUBLIC_IP>:9092']
```

The notebooks currently use this Kafka topic:

```text
demo_testing2
```

In `KafkaConsumer.ipynb`, update the S3 bucket path to your bucket:

```python
s3://<YOUR_BUCKET_NAME>/stock_market_<count>.json
```

Use AWS credentials through environment variables, AWS CLI configuration, or an IAM role. Avoid hard-coding access keys in notebooks.

### 5. Run the Pipeline

1. Run `KafkaConsumer.ipynb` so it is ready to listen for messages.
2. Run `KafkaProducer.ipynb` to read rows from `indexProcessed.csv` and send them to Kafka.
3. Check the configured S3 bucket for generated `stock_market_*.json` files.

Stop the producer notebook manually when enough records have been streamed.

## Useful Kafka Commands

Start a console producer:

```bash
bin/kafka-console-producer.sh \
  --topic demo_testing2 \
  --bootstrap-server <EC2_PUBLIC_IP>:9092
```

Start a console consumer:

```bash
bin/kafka-console-consumer.sh \
  --topic demo_testing2 \
  --bootstrap-server <EC2_PUBLIC_IP>:9092 \
  --from-beginning
```

