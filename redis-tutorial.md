# Interactive Redis CLI Tutorial

This guide provides hands-on practice with Redis commands. Make sure you've followed the setup instructions in README.md before starting.

## Data Types Overview

### 1. Lists (Completed)
Lists are implemented as linked lists in Redis, making them perfect for:
- Task queues
- Social media timelines
- Recent activity logs

For detailed examples and best practices, see `redis-lists.md`

### 2. Sets (Next Session)
- Perfect for unique collections
- Commands we'll cover: SADD, SMEMBERS, SISMEMBER, SREM
- Use cases: tags, unique visitors, etc.

### 3. Hashes
- Store object-like data
- Commands we'll cover: HSET, HGET, HGETALL, HDEL
- Use cases: user profiles, product details, etc.

### 4. Sorted Sets
- Ranked/scored items
- Commands we'll cover: ZADD, ZRANGE, ZRANK
- Use cases: leaderboards, priority queues

### 5. Pub/Sub
- Real-time messaging
- Commands we'll cover: PUBLISH, SUBSCRIBE
- Use cases: chat systems, notifications

## Tips
- Use `HELP @type` to see all commands for a specific type (e.g., `HELP @list`)
- Use `HELP COMMAND` to get help for specific commands
- Use `KEYS *` to see all existing keys
- Use `TYPE key` to check the type of a key
- Use `DEL key` to delete a key

## To exit Redis CLI
```
exit
# or press Ctrl+D
``` 