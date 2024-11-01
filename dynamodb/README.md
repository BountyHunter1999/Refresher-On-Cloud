# DynamoDB Core Concepts

## Table of Contents
- [Tables, Items, and Attributes](#tables-items-and-attributes)
  - [Tables](#tables)
  - [Items](#items)
  - [Attributes](#attributes)
- [Indexes](#indexes)
  - [Primary Key Types](#primary-key-types)
- [Secondary Indexes](#secondary-indexes)
  - [Global Secondary Index (GSI)](#global-secondary-index-gsi)
  - [Local Secondary Index (LSI)](#local-secondary-index-lsi)
- [Partition Key and Sort Key Deep Dive](#partition-key-and-sort-key-deep-dive)
  - [Partition Key (Hash Key)](#partition-key-hash-key)
  - [Sort Key (Range Key)](#sort-key-range-key)
  - [Composite Key Examples](#composite-key-examples)
  - [Best Practices](#best-practices)
- [Design Strategies](#design-strategies)
  - [Key Selection Guide](#key-selection-guide)
  - [Performance Optimization](#performance-optimization)

Amazon DynamoDB is a fully managed, serverless NoSQL database service that provides fast and predictable performance with seamless scalability. Key features include:

- **Serverless**: No servers to manage, scales automatically
- **Performance**: Single-digit millisecond latency at any scale
- **Regional**: Available in multiple regions, region-specific endpoints
- **Scalability**: Handles >10 trillion requests per day and >20 million requests per second
- **Durability**: Data is replicated across multiple AZs with 99.999999999% durability
- **Enterprise Ready**:
  - ACID transactions
  - Point-in-time recovery
  - On-demand backup and restore
  - Encryption at rest

**Schema Flexibility**:
- **Schemaless Design**: No need to predefine attributes except for the primary key
- **Required Schema Elements**:
  - Table name
  - Primary key (must be defined at table creation)
    - Partition key (required)
    - Sort key (optional) (we can't add this after the table is created)
  - Capacity mode (provisioned or on-demand)
    - **Provisioned Capacity Mode**:
      - Specify Read Capacity Units (RCU) and Write Capacity Units (WCU)
      - More cost-effective for predictable workloads
      - Can use auto-scaling to adjust capacity
      - 1 RCU = 1 strongly consistent read/sec for items up to 4KB
      - 1 WCU = 1 write/sec for items up to 1KB
      - **Use when**: Workloads are predictable, and consistent performance is required.
    
    - **On-Demand Capacity Mode**:
      - Pay-per-request pricing
      - Automatically scales up/down based on actual usage
      - No capacity planning needed
      - Best for unpredictable workloads
      - Higher per-request cost than provisioned
      - Can switch between modes once per 24 hours
      - **Use when**: Workloads are random and unpredictable.
- **Optional Schema Elements**:
  - Secondary indexes (GSI/LSI)
  - TTL settings
  - Encryption settings
  - Stream settings
  - Tags

**Use Cases**:
- Gaming leaderboards
- Session management
- IoT device data
- E-commerce shopping carts
- Real-time big data
- Mobile/web backends

## Tables, Items, and Attributes

### Tables
- A table is a collection of data
- Each table must have a primary key that uniquely identifies each item
- Tables in DynamoDB are schemaless (except for the primary key)

### Items
- An item is a group of attributes that is uniquely identifiable
- Similar to a "row" in a relational database
- Each item in a table must have the primary key attributes
- Maximum item size is 400KB
- No limit to the number of items in a table

### Attributes
- Fundamental data elements of DynamoDB
- Similar to "columns" in a relational database
- Each item is composed of one or more attributes
- Types include:
  - Scalar Types (String, Number, Boolean, Null, Binary)
    - String: "Hello World"
    - Number: 42, 3.14, -10
    - Boolean: true, false
    - Null: null
    - Binary: Base64-encoded binary data
  - Document Types (List, Map)
    - List: ["apple", "banana", 123, true]
    - Map: {
        "name": "John",
        "age": 30,
        "address": {
          "street": "123 Main St",
          "city": "Seattle"
        }
      }
  - Set Types (String Set, Number Set, Binary Set)
    - String Set: {"apple", "banana", "cherry"}
    - Number Set: {1, 2, 3, 4, 5}
    - Binary Set: {01010101, 10101010}

## Indexes

### Primary Key Types
1. **Partition Key (Simple Primary Key)**
   - Single attribute
   - Must be unique for each item
   - Used to distribute data across partitions

2. **Partition Key and Sort Key (Composite Primary Key)**
   - Combination of two attributes
   - Multiple items can have the same partition key but must have different sort keys
   - Useful for organizing related items together

## Secondary Indexes

### Global Secondary Index (GSI)
- Index with partition key and optional sort key different from the base table
- Instead of querying with partition key and sort key, you can query with the GSI
- Useful when we need to do quick lookups based on a different attribute
- Otherwise, we'll do a scan and filter
  - Infeasible due to RCU/WCU costs, and latency  
- **Can be created or deleted any time**
- We pay for additional storage and throughput
- Has its own provisioned throughput settings
- Maximum of 20 GSIs per table
- Creation:
  - Define the primary key (partition key and sort key(optional))
  - Index name
  - Projected attributes (all, none, or specific attributes), we can select a subset of attributes to be projected into the index
  - Define read/write capacity
- Creating a GSI clones our primary table using our new Partition Key, but keeps two tables in sync
  - The GSI partition key requires uniform data distribution
  - Define RCU/WCU capacity separately on the Index
  - Throttling
- Gotchas:
  - Writes to the main table result in write to the GSI, doubling the cost
  - Writes aren't guaranteed to be reflected in the GSI immediately
  - We can't delete a GSI if it's being used by a query
  - Race condition due to eventual consistency - our GSI can potentially return stale data
  - Keep the WCU capacity on our GSI tables >= the WCU capacity on our base table

### Local Secondary Index (LSI)
- **Definition**: An LSI is an index that shares the same partition key as the base table but has a different sort key. This allows for more flexible querying of data within the same partition.
  
- **Creation**: 
  - LSIs must be defined at the time of table creation. You cannot add an LSI to an existing table.
  - You specify a different sort key for the LSI, which allows you to query the data in different ways.

- **Provisioned Throughput**:
  - LSIs share the provisioned throughput settings with the base table. This means that the read and write capacity units (RCUs and WCUs) are shared between the table and its LSIs.
  - This can be cost-effective since you don't need to provision additional throughput for the index.

- **Cost**:
  - There is no additional cost for using LSIs beyond the storage cost for the additional data in the index.

- **Limitations**:
  - You can have a maximum of 5 LSIs per table.
  - Since LSIs share throughput with the base table, heavy use of an LSI can impact the performance of the base table.

- **Use Cases**:
  - LSIs are useful when you need to query data in different ways without duplicating data across multiple tables.
  - For example, if you have a table of customer orders, you might use an LSI to query orders by order date, while the base table's sort key is by order ID.

- **Example**:
  Suppose you have a table with a partition key of `customerId` and a sort key of `orderId`. You could create an LSI with the same `customerId` partition key but a sort key of `orderDate` to allow querying orders by date.

- **Querying**:
  - You can perform queries on an LSI using the same query operations available for the base table, such as `=`, `<`, `<=`, `>`, `>=`, `BETWEEN`, and `BEGINS_WITH`.
  - This allows for efficient range queries within the same partition.

- **Consistency**:
  - Queries on LSIs can be eventually consistent or strongly consistent, just like queries on the base table.

- **Best Practices**:
  - Use LSIs when you need to query data in different ways without incurring additional throughput costs.
  - Plan your LSIs carefully during table design, as they cannot be added later.
  - Monitor the performance impact of LSIs on your base table's throughput.

## Partition Key and Sort Key Deep Dive

### Partition Key (Hash Key)
- Acts as the primary mechanism for data distribution
- DynamoDB uses the partition key's value as input to an internal hash function
- Determines the physical partition where the item will be stored
- Best practices:
  - Choose keys with high cardinality (many distinct values)
  - Avoid "hot" partition keys that could lead to throttling
    - when a partition key is accessed too frequently, it can become a bottleneck and lead to throttling
  - Common choices: userId, deviceId, sessionId

### Sort Key (Range Key)
- Enables:
  - Hierarchical relationships
  - One-to-many relationships
  - Complex queries within a partition
- Query Operations:
  - Can use operators: =, <, <=, >, >=, BETWEEN, BEGINS_WITH
  - Supports efficient range queries
- Common patterns:
  - Timestamp: For time-series data
  - Status#Timestamp: For filtering by status and time
  - Type#Name: For grouping similar items

### Composite Key Examples
1. **E-commerce Orders:**
   - Partition Key: customerId
   - Sort Key: orderDate#orderId
   ```
   customerId: "12345"
   sortKey: "2024-03-20#ORD-789"
   ```

2. **User Sessions:**
   - Partition Key: userId
   - Sort Key: sessionType#timestamp
   ```
   userId: "U789"
   sortKey: "mobile#2024-03-20T10:30:00"
   ```

3. **Time-Series Data:**
   - Partition Key: deviceId
   - Sort Key: timestamp
   ```
   deviceId: "DEVICE-001"
   sortKey: "2024-03-20T10:30:00.000Z"
   ```

### Best Practices
- Design keys based on access patterns
- Use composite sort keys for flexible querying
- Avoid hot partitions by distributing data evenly
- Consider future growth and query requirements
- Use GSI/LSI when additional access patterns are needed

## Design Strategies

### Key Selection Guide

1. **When to Use Partition Key Only:**
   - For quick lookups of known, globally unique keys
   - Ideal for:
     - UUID lookups
     - Email address lookups
     - Product ID lookups
   ```
   // Example: User lookup by UUID
   PartitionKey: userId (uuid)
   ```

2. **When to Use Partition Key + Sort Key:**
   - For non-unique keys
   - When range-based queries are needed
   - Examples of range operations:
     - Get all orders for a customer between dates
     - Find all products in a category with price > X
   ```
   // Example: Customer orders
   PartitionKey: customerId
   SortKey: orderDate
   ```

### Performance Optimization

1. **Key Composition Strategies:**
   - Use prefixes/suffixes to distribute data
   ```
   // Instead of
   PartitionKey: "2024-03"
   
   // Use
   PartitionKey: "ORDER#2024-03"
   PartitionKey: "PRODUCT#2024-03"
   ```

2. **DynamoDB Accelerator (DAX)**
   - In-memory caching layer for DynamoDB
   - Benefits:
     - Bypasses the 3000 RCU partition limit
     - Microsecond latency for cached reads
     - Ideal for read-heavy workloads
   - Considerations:
     - Backed by EC2 instances (additional cost)
     - Best for read-heavy and repetitive request patterns
     - Not suitable for strongly consistent read requirements
