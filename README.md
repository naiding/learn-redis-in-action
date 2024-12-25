# Learn Redis

This repository contains a Docker-based Redis setup for learning Redis commands.

## Prerequisites
- Docker
- Docker Compose

## Setup Instructions

1. Clone this repository

2. Start Redis using Docker Compose:
```bash
docker compose up -d
```

This will:
- Start a Redis container
- Expose Redis on port 6379
- Create a persistent volume for data storage

3. Connect to Redis CLI:
```bash
docker compose exec redis redis-cli
```

4. When done, you can stop Redis with:
```bash
docker compose down
```

## Tutorial
Once connected, follow the instructions in `redis-tutorial.md` for an interactive guide to Redis commands. 