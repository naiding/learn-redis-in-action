# Redis Streams Deep Dive

Redis Streams are a log data structure introduced in Redis 5.0. Think of them as an append-only log with built-in support for consumer groups, making them perfect for event sourcing, message queues, and real-time analytics.

## Key Differences from Lists
- Append-only log structure (vs. linked list)
- Each entry has a unique ID (vs. simple values)
- Support for consumer groups (vs. single consumer)
- Persistence of consumed messages (vs. removal after consumption)
- Built-in last-delivered tracking

## Basic Operations

### 1. Adding Entries
```redis
# Add a single entry with auto-generated ID
XADD mystream * sensor-id 1234 temperature 19.8 humidity 0.6

# Add with custom ID
XADD mystream 1234567-0 name "John" age "30"

# Add multiple field-value pairs
XADD mystream * user_id 100 action "login" device "mobile"
```

### 2. Reading Entries
```redis
# Read all entries
XREAD COUNT 2 STREAMS mystream 0

# Read new entries (blocking)
XREAD BLOCK 5000 STREAMS mystream $

# Read from multiple streams
XREAD STREAMS stream1 stream2 0 0
```

## Consumer Groups

### 1. Creating and Managing Groups
```redis
# Create a consumer group
XGROUP CREATE mystream mygroup $

# Create consumer group from beginning
XGROUP CREATE mystream mygroup 0

# Delete a consumer group
XGROUP DESTROY mystream mygroup
```

### 2. Reading as Consumer Group
```redis
# Read as consumer "consumer1" in group "mygroup"
XREADGROUP GROUP mygroup consumer1 COUNT 1 STREAMS mystream >

# Read pending messages
XREADGROUP GROUP mygroup consumer1 STREAMS mystream 0
```

### 3. Acknowledging Messages
```redis
# Acknowledge processing
XACK mystream mygroup 1526569495631-0

# Get pending entries info
XPENDING mystream mygroup
```

## Common Use Cases

### 1. Event Sourcing
```redis
# Store events
XADD orders * order_id 1234 status "created" amount 50.00
XADD orders * order_id 1234 status "paid" payment_method "credit"
XADD orders * order_id 1234 status "shipped" tracking "ABC123"

# Read order history
XRANGE orders - + COUNT 10
```

### 2. Real-time Analytics
```redis
# Log user actions
XADD user_actions * user_id 100 action "view" product_id 555
XADD user_actions * user_id 100 action "add_to_cart" product_id 555

# Get recent actions
XREVRANGE user_actions + - COUNT 5
```

### 3. Message Queue with Multiple Consumers
```redis
# Setup consumer group
XGROUP CREATE notifications mygroup $

# Consumer 1
XREADGROUP GROUP mygroup consumer1 BLOCK 0 STREAMS notifications >

# Consumer 2
XREADGROUP GROUP mygroup consumer2 BLOCK 0 STREAMS notifications >
```

## Performance Characteristics

### Time Complexity
- XADD: O(1)
- XREAD: O(N) where N is number of entries returned
- XRANGE: O(log(N)) for seeking + O(M) for returning M elements
- XLEN: O(1)

### Memory Efficiency
- Radix tree implementation
- Efficient memory usage
- Automatic ID generation
- Optional entry trimming

## Best Practices

1. **Message IDs**:
   - Use auto-generated IDs when possible
   - IDs are timestamp-based and monotonic
   - Custom IDs should follow same format (timestamp-sequence)

2. **Consumer Groups**:
   - Use for workload distribution
   - Monitor pending entries
   - Implement retry logic for failed processing
   - Regularly check consumer group lag

3. **Stream Size Management**:
   ```redis
   # Trim by length
   XADD mystream MAXLEN 1000 * field value

   # Approximate trim (faster)
   XADD mystream MAXLEN ~ 1000 * field value
   ```

4. **Error Handling**:
   - Track failed message processing
   - Implement dead letter queues
   - Monitor consumer group health
   - Handle consumer failures gracefully

## When to Use Streams vs Lists
- Use Streams when you need:
  - Message persistence after consumption
  - Multiple consumers for same messages
  - Built-in consumer groups
  - Message acknowledgment
  - Time-based message retrieval
- Use Lists when you need:
  - Simple FIFO queue
  - Single consumer model
  - Messages to be removed after processing
  - Simpler implementation 