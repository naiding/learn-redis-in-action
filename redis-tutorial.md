# Interactive Redis CLI Tutorial

This guide provides hands-on practice with Redis commands. Make sure you've followed the setup instructions in README.md before starting.

## Lists Tutorial (Completed)

### Basic List Operations
```redis
# Add items to the end of list
RPUSH todos "Learn Redis Lists"
RPUSH todos "Practice Redis Commands" "Build Something Cool"

# View all items
LRANGE todos 0 -1

# Add to beginning of list
LPUSH todos "First Priority Task"

# Get list length
LLEN todos

# Remove and return last item
RPOP todos

# Remove and return first item
LPOP todos
```

### Advanced List Operations
```redis
# Insert before specific item
LINSERT todos BEFORE "Learn Redis Lists" "Read Redis Documentation"

# Get item by position
LINDEX todos 1

# Update item at position
LSET todos 0 "Updated First Task"

# Trim list to specific range
LTRIM todos 0 1
```

## Next Topics To Explore

### 1. Sets (Next Session)
- Perfect for unique collections
- Commands we'll cover: SADD, SMEMBERS, SISMEMBER, SREM
- Use cases: tags, unique visitors, etc.

### 2. Hashes
- Store object-like data
- Commands we'll cover: HSET, HGET, HGETALL, HDEL
- Use cases: user profiles, product details, etc.

### 3. Sorted Sets
- Ranked/scored items
- Commands we'll cover: ZADD, ZRANGE, ZRANK
- Use cases: leaderboards, priority queues

### 4. Pub/Sub
- Real-time messaging
- Commands we'll cover: PUBLISH, SUBSCRIBE
- Use cases: chat systems, notifications

## Tips
- Use `HELP @list` to see all list-related commands
- Use `HELP COMMAND` to get help for specific commands
- Use `KEYS *` to see all existing keys
- Use `TYPE key` to check the type of a key
- Use `DEL key` to delete a key

## To exit Redis CLI
```
exit
# or press Ctrl+D
``` 