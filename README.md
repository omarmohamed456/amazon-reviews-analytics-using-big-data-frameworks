# Amazon Reviews Analytics Using Big Data Frameworks

A self-contained big data environment for analyzing Amazon customer reviews using Apache Hadoop (HDFS) and Apache Spark, running entirely via Docker Compose.

Dataset: [Amazon Reviews — Kaggle](https://www.kaggle.com/datasets/kritanjalijain/amazon-reviews)

---

## Stack Overview

| Service | Image | Purpose |
|---|---|---|
| HDFS Namenode | `bde2020/hadoop-namenode:2.0.0-hadoop3.2.1-java8` | Manages HDFS metadata and directory structure |
| HDFS Datanode | `bde2020/hadoop-datanode:2.0.0-hadoop3.2.1-java8` | Stores distributed data blocks |
| Spark Master | `apache/spark:3.5.1` | Coordinates the Spark cluster |
| Spark Worker | `apache/spark:3.5.1` | Executes Spark tasks |
| Jupyter (PySpark) | `quay.io/jupyter/pyspark-notebook:latest` | Notebook environment with PySpark pre-installed |

### Spark Capabilities (Built-in)

- Spark SQL — structured data querying via DataFrames or SQL
- Structured Streaming — real-time data processing
- MLlib — distributed machine learning
- PySpark — Python API, available directly in Jupyter

---

## Requirements

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running
- At least 8 GB RAM allocated to Docker

---

## Getting Started

### Start the Stack

```bash
docker compose up -d
```

### Stop the Stack

```bash
docker compose down
```

### Tear Down Completely (remove all containers, images, volumes, networks)

```bash
docker system prune -a
```

When prompted, type `y` and press Enter to confirm.

---

## Verifying the Setup

### Step 1 — Check Running Containers

```bash
docker ps
```

You should see all five containers with status `Up`:

```
namenode
datanode
spark-master
spark-worker
jupyter
```

If any container shows `Exited`, check its logs:

```bash
docker logs <container-name>
```

### Step 2 — Hadoop HDFS UI

Open in your browser:

```
http://localhost:9870
```

You should see the Hadoop NameNode dashboard. If it loads, HDFS is working.

### Step 3 — Spark UI

Open in your browser:

```
http://localhost:8080
```

You should see the Spark Master UI with 1 worker connected. If the worker appears, the Spark cluster is working.

### Step 4 — Jupyter Notebook

Open in your browser:

```
http://localhost:8888
```

If it prompts for a token, run:

```bash
docker logs jupyter
```

Look for a line like:

```
http://127.0.0.1:8888/lab?token=abc123...
```

Copy the token from that URL and paste it into the browser prompt.

---

## Port Reference

| Service | Port | URL |
|---|---|---|
| HDFS Namenode UI | 9870 | http://localhost:9870 |
| HDFS RPC | 9000 | — |
| Spark Master UI | 8080 | http://localhost:8080 |
| Spark Master RPC | 7077 | — |
| Datanode UI | 9864 | http://localhost:9864 |
| Jupyter | 8888 | http://localhost:8888 |

---

## Project Structure

```
.
├── docker-compose.yml
└── notebooks/          # Place your .ipynb files here (mounted into Jupyter at /home/jovyan/work)
```

Your local directory is automatically mounted into the Jupyter container, so any notebooks or data files you place here are accessible inside the environment.