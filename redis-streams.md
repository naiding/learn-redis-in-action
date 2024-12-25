# Redis Streams Deep Dive

Redis Streams are a log data structure introduced in Redis 5.0. Think of them as an append-only log with built-in support for consumer groups, making them perfect for event sourcing, message queues, and real-time analytics.

## 1. Fundamentals

### Key Differences from Lists
- Append-only log structure (vs. linked list)
- Each entry has a unique ID (vs. simple values)
- Support for consumer groups (vs. single consumer)
- Persistence of consumed messages (vs. removal after consumption)
- Built-in last-delivered tracking

### Message ID Format
- IDs are in format `timestamp-sequence` (e.g., `1735108392022-0`)
- Timestamp part is milliseconds since Unix epoch
- Sequence part handles multiple messages in same millisecond
- Never generate your own IDs unless you fully understand the format
- Using `*` for auto-generation is strongly recommended

### Basic Operations

#### Adding Entries
```redis
# Add a single entry with auto-generated ID
XADD mystream * sensor-id 1234 temperature 19.8 humidity 0.6

# Add with custom ID
XADD mystream 1234567-0 name "John" age "30"

# Add multiple field-value pairs
XADD mystream * user_id 100 action "login" device "mobile"
```

#### Reading Entries
```redis
# Read all entries
XREAD COUNT 2 STREAMS mystream 0

# Read new entries (blocking)
XREAD BLOCK 5000 STREAMS mystream $

# Read from multiple streams
XREAD STREAMS stream1 stream2 0 0
```

## 2. Consumer Groups

### Creating and Managing Groups
```redis
# Create a consumer group
XGROUP CREATE mystream mygroup $

# Create consumer group from beginning
XGROUP CREATE mystream mygroup 0

# Delete a consumer group
XGROUP DESTROY mystream mygroup
```

### Reading as Consumer Group
```redis
# Read as consumer "consumer1" in group "mygroup"
XREADGROUP GROUP mygroup consumer1 COUNT 1 STREAMS mystream >

# Read pending messages
XREADGROUP GROUP mygroup consumer1 STREAMS mystream 0
```

### Blocking Reads with Consumer Groups
```redis
# Block until new messages arrive (wait forever)
XREADGROUP GROUP mygroup consumer1 BLOCK 0 STREAMS mystream >

# Block with timeout (wait 5000 milliseconds)
XREADGROUP GROUP mygroup consumer1 BLOCK 5000 STREAMS mystream >

# Block on multiple streams
XREADGROUP GROUP mygroup consumer1 BLOCK 5000 STREAMS stream1 stream2 > >
```

Key Points:
- `BLOCK 0` means wait indefinitely
- `BLOCK milliseconds` specifies maximum wait time
- Returns `nil` if timeout is reached
- Only blocks the client connection, not Redis itself
- Perfect for real-time processing with minimal latency

### Consumer Group Behavior
- Messages are not deleted when acknowledged
- Each consumer group maintains its own position in the stream
- A message can be in different states for different consumer groups
- Consumer groups can start reading from any point in the stream
- Dead consumers still hold their pending messages

### Acknowledging Messages
```redis
# Acknowledge processing
XACK mystream mygroup 1526569495631-0

# Get pending entries info
XPENDING mystream mygroup
```

## 3. Reliability and Durability

### Consumer Groups and Message Acknowledgment
```redis
# Create a consumer group that starts from the beginning
XGROUP CREATE orders order-processors 0 MKSTREAM

# Consumer 1 reads a message
XREADGROUP GROUP order-processors consumer1 COUNT 1 STREAMS orders >
# Returns: message-id: 1735108392022-0

# Message stays in pending state until acknowledged
XPENDING orders order-processors

# After successful processing, acknowledge the message
XACK orders order-processors 1735108392022-0
```

### Message Recovery
```redis
# List messages pending for more than 1 minute
XPENDING orders order-processors - + 10 consumer1 60000

# Claim messages from failed consumer
XCLAIM orders order-processors new-consumer 60000 1735108392022-0

# Check for stalled consumers
XINFO CONSUMERS orders order-processors

# Reassign messages from dead consumer
XGROUP DELCONSUMER orders order-processors dead-consumer
```

### Error Recovery Patterns

#### Dead Letter Pattern
```redis
# After max retries, move to dead letter stream
XADD orders:dead-letter * 
  original_id 1735108392022-0 
  error "Processing timeout" 
  retries 3

# Monitor dead letter queue
XLEN orders:dead-letter
```

#### Retry Pattern
```redis
# Track retry attempts
XADD orders:retry * 
  message_id 1735108392022-0 
  retry_count 1 
  next_retry 1703424180

# Process retries
XREADGROUP GROUP retry-processor worker1 COUNT 10 STREAMS orders:retry >
```

### Durability Guarantees

#### Persistence Options
```redis
# Check last save time
LASTSAVE

# Force immediate save
SAVE
```

#### AOF (Append Only File)
- Every write command is logged
- Stream data can be reconstructed
- Provides better durability guarantees

#### RDB (Redis Database)
- Point-in-time snapshots
- Backup and restore capability
- Faster restart times

### High Availability
- Stream data is replicated to replicas
- Automatic failover with Redis Sentinel
- No message loss during master failure
- Cluster mode support for distribution

## 4. Performance and Memory Management

### Performance Characteristics
- XADD: O(1)
- XREAD: O(N) where N is number of entries returned
- XRANGE: O(log(N)) for seeking + O(M) for returning M elements
- XLEN: O(1)

### Memory Management
- Streams grow indefinitely unless trimmed
- Set appropriate MAXLEN limits based on your use case
- Monitor stream size and memory usage
- Consider the trade-off between history retention and memory usage
- Use `~` for approximate trimming in high-throughput scenarios

### Stream Size Management
```redis
# Trim by length
XADD mystream MAXLEN 1000 * field value

# Approximate trim (faster)
XADD mystream MAXLEN ~ 1000 * field value

# Manually trim the stream
XTRIM mystream MAXLEN 1000
```

### Performance Considerations
- Blocking operations only block the client, not Redis
- Long-running consumers should implement heartbeat mechanisms
- Large COUNT values in XREAD/XREADGROUP can impact latency
- Consider batching for high-throughput scenarios
- Monitor and adjust consumer count based on lag

## 5. Common Use Cases

### Event Sourcing
```redis
# Store events
XADD orders * order_id 1234 status "created" amount 50.00
XADD orders * order_id 1234 status "paid" payment_method "credit"
XADD orders * order_id 1234 status "shipped" tracking "ABC123"

# Read order history
XRANGE orders - + COUNT 10
```

### Real-time Analytics

#### User Behavior Analysis
```redis
# Track user session
XADD user_sessions * 
  session_id "sess123" 
  user_id 100 
  page "/products" 
  referrer "google.com" 
  device "mobile"

# Track feature usage
XADD feature_usage * 
  feature_id "dark_mode" 
  user_id 100 
  action "toggle_on" 
  platform "ios"
```

#### Performance Monitoring
```redis
# Monitor API latency
XADD api_metrics * 
  endpoint "/api/v1/products" 
  response_time 150 
  status 200 
  region "us-west"

# Track error rates
XADD error_events * 
  service "payment" 
  error_type "timeout" 
  user_id 100 
  trace_id "abc123"
```

#### Real-time Aggregations
```redis
# Consumer processing events for 5-minute windows
XREADGROUP GROUP metrics_aggregator worker1 COUNT 1000 STREAMS api_metrics >

# Example aggregated data structure (using other Redis types)
HINCRBY "metrics:5min:endpoint:/api/products" "total_requests" 1
HINCRBY "metrics:5min:endpoint:/api/products" "total_time" 150
```

### Message Queue with Multiple Consumers
```redis
# Setup consumer group
XGROUP CREATE notifications mygroup $

# Consumer 1
XREADGROUP GROUP mygroup consumer1 BLOCK 0 STREAMS notifications >

# Consumer 2
XREADGROUP GROUP mygroup consumer2 BLOCK 0 STREAMS notifications >
```

## 6. Common Pitfalls and Best Practices

### Common Pitfalls

#### Message Loss
- Not setting appropriate MAXLEN values
- Slow consumers with aggressive trimming
- Not handling consumer failures properly
- Missing message acknowledgments

#### Memory Issues
- Unbounded stream growth
- Too many pending messages
- Not cleaning up dead consumers
- Keeping too much history

#### Performance Problems
- Too many consumers in a group
- Processing messages one by one
- Not batching writes in high-throughput cases
- Inefficient consumer error handling

#### Operational Issues
- Not monitoring consumer lag
- Missing consumer group management
- Inadequate error handling
- Poor retry strategies

### Best Practices

#### Message Processing
- Always acknowledge after successful processing
- Implement idempotent consumers
- Use unique message IDs for deduplication
- Handle partial failures gracefully

#### Consumer Design
- Implement heartbeat mechanism
- Set appropriate timeout values
- Handle backpressure properly
- Log consumer state changes

#### Monitoring
- Track pending message counts
- Monitor consumer lag
- Alert on processing delays
- Watch for repeated failures

#### Recovery Procedures
- Document recovery steps
- Test failure scenarios
- Maintain operation runbooks
- Regular disaster recovery drills

## 7. Production Readiness Checklist
- [ ] Appropriate MAXLEN values set
- [ ] Consumer group recovery procedures documented
- [ ] Monitoring and alerting in place
- [ ] Dead letter queues implemented
- [ ] Retry strategies defined
- [ ] Backup and recovery tested
- [ ] Performance benchmarks established
- [ ] Consumer scaling strategy defined
- [ ] Error handling procedures documented
- [ ] Resource limits set and monitored

## 8. When to Use Streams vs Lists
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