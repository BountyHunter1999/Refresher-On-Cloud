# DynamoDB Core Concepts

## Table of Contents
- [Tables, Items, and Attributes](#tables-items-and-attributes)
  - [Tables](#tables)
  - [Items](#items)
  - [Attributes](#attributes)
- [Indexes](#indexes)
  - [Primary Key Types](#primary-key-types)
  - [Secondary Indexes](#secondary-indexes)
- [Partition Key and Sort Key Deep Dive](#partition-key-and-sort-key-deep-dive)
  - [Partition Key (Hash Key)](#partition-key-hash-key)
  - [Sort Key (Range Key)](#sort-key-range-key)
  - [Composite Key Examples](#composite-key-examples)
  - [Best Practices](#best-practices)
- [Design Strategies](#design-strategies)
  - [Key Selection Guide](#key-selection-guide)
  - [Performance Optimization](#performance-optimization)

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
  - Document Types (List, Map)
  - Set Types (String Set, Number Set, Binary Set)

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

### Secondary Indexes

1. **Global Secondary Index (GSI)**
   - Index with partition key and optional sort key different from the base table
   - Can be created or deleted any time
   - Has its own provisioned throughput settings
   - Maximum of 20 GSIs per table

2. **Local Secondary Index (LSI)**
   - Index that shares partition key with base table but has different sort key
   - Must be created when creating the table
   - Shares provisioned throughput with base table
   - Maximum of 5 LSIs per table

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
