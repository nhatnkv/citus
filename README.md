# Citus Cluster with Docker Compose on Local

This repository demonstrates how to set up a Citus cluster (PostgreSQL + Citus extension) using Docker Compose. The cluster consists of one **Coordinator** and four **Worker** nodes, all running in containers on a single Docker network.

---

## Table of Contents

- [Features](#features)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
  - [1. Clone the repository](#1-clone-the-repository)
  - [2. Launch the cluster](#2-launch-the-cluster)
  - [3. Register Worker nodes](#3-register-worker-nodes)
  - [4. Create a distributed table](#4-create-a-distributed-table)
  - [5. Verify cluster status](#5-verify-cluster-status)
- [Scaling & Maintenance](#scaling--maintenance)
- [Cleanup](#cleanup)
- [License](#license)

---

## Features

- **Single Coordinator** for query planning and routing
- **Four Worker nodes** for distributed data storage and parallel execution
- **Horizontal scaling**: add more workers by updating Docker Compose
- **Exposed Coordinator port** (`5432`) for client connections

---

## Prerequisites

- **Docker Engine** >= 20.10
- **Docker Compose** >= 1.29
- **Git** installed

---

## Getting Started

### 1. Clone the repository

```bash
git clone git@github.com:nhatnkv/citus.git
cd citus
```

### 2. Launch the cluster

```bash
docker compose -f citus.yml up
```

### 3. Register Worker nodes

Connect to the Coordinator container:

```bash
docker exec -it citus_coordinator psql -U postgres
```

In the `psql` prompt, run:

```sql
-- Add each worker node
DELETE FROM pg_dist_node;
SELECT master_add_node('worker1', 5432);
SELECT master_add_node('worker2', 5432);
SELECT master_add_node('worker3', 5432);
SELECT master_add_node('worker4', 5432);
```

### 4. Create a distributed table

Assuming you have a `products` table in the `postgres` database:

```sql
-- Example schema (if not present):
CREATE TABLE IF NOT EXISTS products (
  id BIGSERIAL PRIMARY KEY,
  name TEXT,
  price NUMERIC,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Convert to a distributed table with 20 shards
SELECT create_distributed_table(
  'products',        -- table name
  'id',              -- distribution column
);
```

## Scaling & Maintenance

- **Add more workers**: update `citus.yml` with new `workerX` services and rerun [Register Worker nodes](#3-register-worker-nodes).
- **Rebalance shards**:

  ```sql
  SELECT rebalance_table_shards('products');
  ```
---

## Cleanup

Stop and remove containers, network, and volumes:

```bash
docker compose -f citus.yml down
```

---

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
