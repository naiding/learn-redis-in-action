# Learn Redis

This repository contains a Docker-based Redis setup for learning Redis commands.

## Prerequisites
- Docker
- Docker Compose

## Setup Instructions

1. Clone this repository

2. Start Redis and RedisInsight:
```bash
docker compose up -d
```

This will:
- Start a Redis container on port 6379
- Start RedisInsight on port 5540
- Create persistent volumes for both services

## Accessing Redis

### Using Redis CLI
```bash
# Add this alias to your ~/.zshrc or ~/.bashrc
alias rcli="docker compose exec redis redis-cli"

# Then you can use Redis CLI with:
rcli PING  # Should return PONG
```

### Using RedisInsight (GUI)

1. Open RedisInsight in your browser:
   - URL: http://localhost:5540

2. Click "Add Redis Database" and enter:
   - Host: `redis` (service name from docker-compose)
   - Port: `6379`
   - Name: `Learn Redis` (or any name you prefer)
   - Username: leave empty
   - Password: leave empty
   - Leave all other settings as default

3. Click "Add Redis Database" to connect

## Stopping the Services
```bash
docker compose down
```

## Tutorial
Once connected, follow the instructions in `redis-tutorial.md` for an interactive guide to Redis commands. 