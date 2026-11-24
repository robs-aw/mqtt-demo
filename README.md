# MQTT Demo

A Lightweight MQTT telemetry simulation for demonstrating real-time data ingestion, storage, and analysis.

This project simulates multiple sensors publishing cumulative readings to an MQTT broker. A subscriber consumes the messages, writes them to JSON Lines (`.jsonl`) files, and the resulting data can be explored using DuckDB, Pandas, and Jupyter notebooks.

## Features

- Simulates multiple IoT sensors
- Publishes sensor telemetry to MQTT topics
- Subscribes to telemetry streams using MQTT wildcards
- Persists incoming messages to JSONL files
- Buffers and periodically flushes data to disk
- Includes exploratory analysis notebook using DuckDB
- Environment-variable driven configuration

## Architecture

```text
┌─────────────┐
│  Publisher  │
│  (Sensors)  │
└──────┬──────┘
       │ MQTT
       ▼
┌─────────────┐
│ MQTT Broker │
└──────┬──────┘
       │ MQTT
       ▼
┌─────────────┐
│ Subscriber  │
│ JSONL Files │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ DuckDB /    │
│ Pandas EDA  │
└─────────────┘
```

## Project Structure

```text
.
├── data/
│   └── .gitkeep
├── notebooks/
│   └── eda.ipynb
├── src/
│   └── mqtt_demo/
│       ├── __init__.py
│       ├── publisher.py
│       └── subscriber.py
├── .example.env
├── pyproject.toml
└── README.md
```

## Installation

### Prerequisites

- Python 3.13+
- MQTT broker (defaults to `test.mosquitto.org`)
- uv (recommended) or pip

### Clone the Repository

```bash
git clone <repository-url>
cd mqtt-demo
```

### Create Environment

Using `uv`:

```bash
uv sync
```

Or using `pip`:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -e .
```

## Configuration

Copy the example environment file:

```bash
cp .example.env .env
```

Example configuration:

```env
MQTT_HOST=test.mosquitto.org
MQTT_PORT=1883
MQTT_NAMESPACE=training/my-unique-namespace

NUMBER_OF_SENSORS=5
NUMBER_OF_SAMPLES=100

PUBLISH_INTERVAL=1
FLUSH_INTERVAL_SECONDS=10
```

### Configuration Parameters

| Variable | Description | Default |
|-----------|-------------|-----------|
| MQTT_HOST | MQTT broker hostname | test.mosquitto.org |
| MQTT_PORT | MQTT broker port | 1883 |
| MQTT_NAMESPACE | Topic namespace used by publisher and subscriber | Required |
| NUMBER_OF_SENSORS | Number of simulated sensors | Required |
| NUMBER_OF_SAMPLES | Number of readings generated per sensor | Required |
| PUBLISH_INTERVAL | Delay between publish cycles (seconds) | 1 |
| FLUSH_INTERVAL_SECONDS | Subscriber buffer flush interval | 60 |

> **Important:** Use a unique `MQTT_NAMESPACE` to avoid consuming data from other users when using a public broker.

## Running the Subscriber

Start the subscriber before the publisher:

```bash
python -m mqtt_demo.subscriber
```

Example output:

```text
Connected to MQTT broker
Subscribed to training/my-namespace/sensor/+
```

Incoming messages are buffered and periodically written to:

```text
data/sensors_YYYYMMDD_HHMMSS.jsonl
```

## Running the Publisher

In a separate terminal:

```bash
python -m mqtt_demo.publisher
```

Example log:

```text
Published to training/my-namespace/sensor/sensor_0:
{"sensor_id":0,"sequence_no":1,"timestamp":"...","cumulative_value":0.52}
```

## Data Format

Each published message contains:

```json
{
  "sensor_id": 0,
  "sequence_no": 1,
  "timestamp": "2026-11-23T14:43:24.878746+00:00",
  "cumulative_value": 0.1801
}
```

The subscriber enriches records with metadata:

```json
{
  "sensor_id": 0,
  "sequence_no": 1,
  "timestamp": "2026-11-23T14:43:24.878746+00:00",
  "cumulative_value": 0.1801,
  "_topic": "training/my-namespace/sensor/sensor_0",
  "_received_at": "2026-11-23T14:43:24.934903+00:00"
}
```

## Data Analysis

The included notebook demonstrates how to query collected telemetry data using DuckDB.

Example:

```python
import duckdb

con = duckdb.connect()

df = con.execute("""
    SELECT *
    FROM read_json_auto(
        '../data/*.jsonl',
*       records=true
    )
""").df()```
