# Project 3: HTTP over RPC (HTTP Proxy)

**Group ID:** 16 
**Course:** Distributed Systems 2026

## 📋 Overview

HTTP Proxy server using RPC for distributed web content fetching. Implements concepts from multiple chapters:

| Chapter | Concept | Implementation |
|---------|---------|----------------|
| 3 | RPC | XML-RPC for proxy-worker communication |
| 4 | MapReduce | Log analytics with Map-Shuffle-Reduce |
| 5 | DFS | Shared file cache across workers |
| 6 | Cloud | Docker containers, docker-compose |

---

## 🏗️ Architecture

```
                         ┌─────────────────┐
                         │   HTTP Client   │
                         └────────┬────────┘
                                  │
                                  ▼
                    ┌─────────────────────────┐
                    │     Proxy Server        │
                    │     (port 8080)         │
                    │  ┌─────────────────┐    │
                    │  │  Load Balancer  │    │
                    │  │  (Round-robin)  │    │
                    │  └─────────────────┘    │
                    │  ┌─────────────────┐    │
                    │  │  Access Logger  │────┼──► logs/access.log
                    │  │  (MapReduce)    │    │        │
                    │  └─────────────────┘    │        ▼
                    └────────────┬────────────┘   ┌──────────┐
                                 │                │Analytics │
              ┌──────────────────┼──────────────┐ │(MapReduce│
              ▼                  ▼              ▼ └──────────┘
        ┌──────────┐       ┌──────────┐   ┌──────────┐
        │ Worker 1 │       │ Worker 2 │   │ Worker 3 │
        │ (RPC)    │       │ (RPC)    │   │ (RPC)    │
        └────┬─────┘       └────┬─────┘   └────┬─────┘
             │                  │              │
             └──────────────────┼──────────────┘
                                │
                     ┌──────────▼──────────┐
                     │   Shared Cache      │
                     │   (DFS Volume)      │
                     │   cache_data/       │
                     └─────────────────────┘
                                │
                                ▼
                     ┌─────────────────────┐
                     │      Internet       │
                     └─────────────────────┘
```

---

## 📁 Project Structure

```
Project3_HTTP_RPC/
├── docker-compose.yml          # Container orchestration
├── Dockerfile                  # Container image
├── requirements.txt            # Dependencies
├── README.md                   # This file
│
├── src/                        # Source code
│   ├── rpc_proxy_server.py     # Proxy + Load Balancer + Logger
│   ├── rpc_worker.py           # RPC Worker + DFS Cache
│   ├── analytics.py            # MapReduce Log Analysis
│   └── test_proxy.py           # Test suite
│
└── report/                     # Report files
    ├── report.pdf
    └── report.pptx
```

---

## 🚀 Quick Start (Docker)

### 1. Start entire system with ONE command:

```bash
cd Project3_HTTP_RPC
docker-compose up --build
```

This starts:
- 3 RPC Workers (ports 8001, 8002, 8003)
- 1 Proxy Server (port 8080)

### 2. Test the proxy:

```bash
# Simple test
curl -x http://localhost:8080 http://example.com

# Run test suite
docker-compose exec proxy python test_proxy.py
```

### 3. Run MapReduce Analytics:

```bash
docker-compose --profile tools run analytics
```

### 4. Stop system:

```bash
docker-compose down
```

---

## 🖥️ Local Development (Without Docker)

### Terminal 1, 2, 3 - Start Workers:

```bash
cd src
python rpc_worker.py 8001  # Terminal 1
python rpc_worker.py 8002  # Terminal 2
python rpc_worker.py 8003  # Terminal 3
```

### Terminal 4 - Start Proxy:

```bash
python rpc_proxy_server.py 8080
```

### Terminal 5 - Test:

```bash
curl -x http://localhost:8080 http://example.com
python test_proxy.py
```

### Run Analytics:

```bash
python analytics.py
```

---

## ✨ Features by Chapter

### Chapter 3: RPC Communication

- XML-RPC protocol between Proxy and Workers
- Functions: `fetch_url()`, `health_check()`, `clear_cache()`, `get_stats()`

### Chapter 4: MapReduce (Log Analytics)

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│    MAP      │────►│   SHUFFLE   │────►│   REDUCE    │
│ Parse logs  │     │ Group by    │     │ Aggregate   │
│ Emit pairs  │     │ key         │     │ counts      │
└─────────────┘     └─────────────┘     └─────────────┘
```

Output includes:
- Top 10 domains accessed
- Cache hit/miss ratio
- Worker load distribution
- Response time statistics
- HTTP status code distribution

### Chapter 5: Distributed File System

- Shared cache directory (`cache_data/`)
- All workers read/write to same location
- Cache files: `{md5_hash}.json`
- TTL-based expiration (60 seconds)

### Chapter 6: Cloud/Virtualization

- Dockerized containers
- Docker Compose orchestration
- Shared volumes for DFS simulation
- Automatic container networking

---

## 📊 Sample Output

### Proxy Server:

```
[Proxy] Starting on port 8080
[Proxy] Worker OK: http://worker1:8001 (Cache: 5 entries)
[Proxy] Worker OK: http://worker2:8001 (Cache: 5 entries)
[Proxy] Worker OK: http://worker3:8001 (Cache: 5 entries)
[Proxy] Active workers: 3/3

[Proxy] GET http://example.com -> 200 [FRESH] 245ms via worker-1
[Proxy] GET http://example.com -> 200 [CACHED] 12ms via worker-2
```

### MapReduce Analytics:

```
============================================================
ANALYTICS REPORT
============================================================

📊 TOP 10 DOMAINS (Most Accessed):
----------------------------------------
   1. example.com                       15 ██████████████████████████████
   2. httpbin.org                        8 ████████████████

📦 CACHE STATISTICS:
----------------------------------------
  Cache Hits:      12 (80.0%)
  Cache Misses:     3 (20.0%)
  [🟢🟢🟢🟢🟢🟢🟢🟢🔴🔴]

⚙️ WORKER DISTRIBUTION (Load Balancing):
----------------------------------------
  worker-1                               5 ██████████
  worker-2                               5 ██████████
  worker-3                               5 ██████████
```

---

## 🔧 Configuration

### Environment Variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `WORKERS` | localhost:8001,... | Comma-separated worker URLs |
| `CACHE_DIR` | /app/cache_data | DFS cache directory |
| `LOG_DIR` | /app/logs | MapReduce log directory |
| `WORKER_ID` | worker-{port} | Worker identifier |

---

## 👥 Team Responsibilities

| Member | Role | Tasks |
|--------|------|-------|
| 1 | Tech Lead + Cloud | `rpc_proxy_server.py`, `Dockerfile`, `docker-compose.yml` |
| 2 | Backend + MapReduce | `rpc_proxy_server.py` (logging), `analytics.py` |
| 3 | Performance + DFS | `rpc_worker.py` (file-based cache) |
| 4 | Testing + Demo | `test_proxy.py`, demo script |

