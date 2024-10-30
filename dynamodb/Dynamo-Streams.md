# DynamoDB Streams

DynamoDB Streams is a feature that captures data modification events in DynamoDB tables in real-time, enabling powerful event-driven architectures and data processing workflows.

## Core Concepts

### Event Types
- **INSERT**: Captures new items added to the table
- **MODIFY**: Records updates to existing items
- **DELETE**: Tracks items removed from the table

### Stream Records
- Contain before and after images of modified items
- Stored for 24 hours
- Processed in order within each partition key
- Can be processed by AWS Lambda or custom applications

## Configuration

### View Types
- **KEYS_ONLY**: Only captures the key attributes of modified items
- **NEW_IMAGE**: Records the entire item after the modification
- **OLD_IMAGE**: Records the entire item before the modification
- **NEW_AND_OLD_IMAGES**: Captures both the old and new versions of the item

### Stream Settings
- Enabled at the table level
- Can be enabled/disabled at any time
- Streams are region-specific
- Each stream has a unique Amazon Resource Name (ARN)

## Common Use Cases
1. **Cross-region Replication**
   - Replicate data changes to tables in different regions
   - Maintain backup copies for disaster recovery

2. **Audit Trails**
   - Track all data modifications
   - Maintain compliance and security records

3. **Event-driven Applications**
   - Trigger Lambda functions based on data changes
   - Build reactive architectures

4. **Real-time Analytics**
   - Process changes for analytics in real-time
   - Maintain derived data structures

5. **Cache Invalidation**
   - Update caches when source data changes
   - Maintain consistency across systems

## Stream Processing

### Lambda Integration
Table -> Stream -> Lambda -> Other AWS Services

### Performance Characteristics
- Each shard processes up to 1,000 events per second
- Automatic scaling based on table activity
- Stream records are retained for 24 hours

### Best Practices
1. **Monitoring**
   - Track Iterator Age metric
   - Monitor Lambda execution metrics
   - Set up alerts for processing delays

2. **Error Handling**
   - Implement retry logic
   - Handle duplicate records gracefully
   - Use Dead Letter Queues for failed events

3. **Performance**
   - Process events in batches when possible
   - Consider using parallel processing
   - Optimize Lambda function performance

4. **Security**
   - Use IAM roles with least privilege
   - Encrypt sensitive data
   - Regularly audit access patterns

## Limitations
- 24-hour retention period for stream records
- Maximum of 2 concurrent Lambda functions per stream
- Stream records must be processed in sequence within a shard
- Streams are region-specific
