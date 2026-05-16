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

## Notebook Overview (`amazon-reviews.ipynb`)

The notebook walks through a full NLP pipeline on the Amazon Reviews dataset using PySpark, from raw data ingestion to TF-IDF feature extraction.

### What It Does

**1. Data Ingestion**
Reads `train.csv` directly from HDFS (`hdfs://namenode:9000/amazon_reviews/train.csv`) into a Spark DataFrame with schema inference.

**2. Preprocessing**
- Renames columns to `polarity`, `review_heading`, and `review_body`
- Converts the polarity column into a binary label: `1` for positive (polarity = 2), `0` for negative
- Concatenates the heading and body into a single `text` field

**3. Exploratory Data Analysis (EDA)**
- Row count and schema inspection
- Null/missing value check across all columns
- Label distribution — counts of positive vs. negative reviews
- Top 20 most frequent words across the corpus, visualized as a bar chart

**4. Text Cleaning & Tokenization**
- Lowercases all text and strips non-alphabetic characters
- Tokenizes into word arrays using PySpark's `Tokenizer`
- Explodes words to compute global word frequencies

**5. TF-IDF Feature Extraction**
A full MLlib `Pipeline` with four stages:
- `RegexTokenizer` — splits on non-word characters, lowercases
- `StopWordsRemover` — filters out common English stopwords
- `HashingTF` — hashes filtered tokens into a 10,000-feature vector
- `IDF` — weights features by inverse document frequency (min doc frequency = 2)

The final `df_tfidf` DataFrame contains a `features` column (sparse TF-IDF vectors) and a `label` column, ready for downstream ML model training.

### Data Schema

```
polarity        integer   (1 = negative, 2 = positive → mapped to 0/1)
review_heading  string
review_body     string
label           integer   (0 = negative, 1 = positive)
text            string    (heading + body, concatenated)
```

### Loading Data into HDFS

Before running the notebook, upload the dataset CSV to HDFS:

```bash
# Copy the CSV into the namenode container
docker cp train.csv namenode:/tmp/train.csv

# Create the HDFS directory and move the file
docker exec namenode hdfs dfs -mkdir -p /amazon_reviews
docker exec namenode hdfs dfs -put /tmp/train.csv /amazon_reviews/train.csv

# Verify
docker exec namenode hdfs dfs -ls /amazon_reviews
```

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
├── README.md
└── notebooks/
    └── amazon-reviews.ipynb   # Full EDA + TF-IDF pipeline (place at /home/jovyan/work inside Jupyter)
```

Your local directory is automatically mounted into the Jupyter container, so any notebooks or data files you place here are accessible inside the environment.
