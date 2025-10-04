# Database Documentation

## Overview

Welcome to the comprehensive Database documentation. This guide covers essential database concepts, SQL and NoSQL databases, interview questions, and best practices for working with databases.

## Table of Contents

1. [SQL vs NoSQL Comparison](SQL-vs-NoSQL.md)
2. [Database Interview Questions](Database-Interview-Questions.md)

## What is a Database?

A database is an organized collection of structured information or data, typically stored electronically in a computer system. Databases are managed by Database Management Systems (DBMS), which provide an interface for users to interact with the data.

## Types of Databases

### 1. Relational Databases (SQL)
- Store data in tables with rows and columns
- Use SQL (Structured Query Language) for queries
- Examples: MySQL, PostgreSQL, Oracle, SQL Server, SQLite

### 2. NoSQL Databases
- Store data in various formats (document, key-value, graph, column-family)
- Designed for specific data models and access patterns
- Examples: MongoDB, Redis, Cassandra, DynamoDB

## Key Database Concepts

### ACID Properties
- **Atomicity**: All operations in a transaction succeed or all fail
- **Consistency**: Database remains in a valid state before and after transaction
- **Isolation**: Concurrent transactions don't interfere with each other
- **Durability**: Committed transactions are permanently saved

### CAP Theorem (for Distributed Systems)
- **Consistency**: All nodes see the same data at the same time
- **Availability**: Every request receives a response
- **Partition Tolerance**: System continues to operate despite network partitions

*You can only guarantee 2 out of 3 properties*

### BASE (NoSQL Alternative to ACID)
- **Basically Available**: System guarantees availability
- **Soft State**: State may change over time without input
- **Eventually Consistent**: System will become consistent over time

## Database Design Principles

### Normalization
The process of organizing data to reduce redundancy and improve data integrity:

1. **1NF (First Normal Form)**: Eliminate repeating groups, create separate table for related data
2. **2NF (Second Normal Form)**: Meet 1NF + remove partial dependencies
3. **3NF (Third Normal Form)**: Meet 2NF + remove transitive dependencies
4. **BCNF (Boyce-Codd Normal Form)**: Stricter version of 3NF
5. **4NF & 5NF**: Handle multi-valued and join dependencies

### Denormalization
Intentionally introducing redundancy for performance optimization in read-heavy applications.

## Common Use Cases

### When to Use SQL Databases
- Complex queries and relationships
- ACID compliance is critical
- Structured data with clear schema
- Financial transactions, banking systems
- Traditional enterprise applications

### When to Use NoSQL Databases
- Flexible schema requirements
- Horizontal scalability needed
- High-volume, high-velocity data
- Real-time applications
- Content management systems
- IoT and time-series data

## Performance Optimization

### Indexing
- B-Tree indexes for range queries
- Hash indexes for equality searches
- Full-text indexes for text search
- Composite indexes for multiple columns

### Query Optimization
- Use EXPLAIN to analyze query execution
- Avoid SELECT * when possible
- Use appropriate JOINs
- Leverage indexes effectively
- Partition large tables

### Caching
- Application-level caching (Redis, Memcached)
- Database query caching
- Result set caching

## Security Best Practices

1. **Authentication & Authorization**: Implement proper user access controls
2. **Encryption**: Encrypt data at rest and in transit
3. **SQL Injection Prevention**: Use parameterized queries
4. **Principle of Least Privilege**: Grant minimum necessary permissions
5. **Regular Backups**: Implement automated backup strategies
6. **Audit Logging**: Track database access and changes

## Popular Database Technologies

### Relational Databases

#### MySQL
- Open-source, widely used
- Good for web applications
- Strong community support

#### PostgreSQL
- Advanced open-source database
- Supports complex queries and JSON
- ACID compliant with strong consistency

#### SQL Server
- Microsoft's enterprise database
- Excellent integration with .NET
- Advanced analytics capabilities

#### Oracle
- Enterprise-grade database
- High performance and scalability
- Comprehensive features but expensive

### NoSQL Databases

#### MongoDB
- Document-oriented database
- Flexible schema with JSON-like documents
- Horizontal scaling with sharding

#### Redis
- In-memory key-value store
- Ultra-fast performance
- Used for caching, sessions, real-time analytics

#### Cassandra
- Wide-column store
- Highly scalable and available
- Good for time-series data

#### DynamoDB
- AWS managed NoSQL database
- Serverless with auto-scaling
- Single-digit millisecond performance

## Database Patterns

### Master-Slave Replication
- Master handles writes
- Slaves handle reads
- Improves read scalability

### Multi-Master Replication
- Multiple nodes accept writes
- Conflict resolution needed
- Higher availability

### Sharding
- Horizontal partitioning of data
- Distributes data across multiple servers
- Improves scalability

### Connection Pooling
- Reuse database connections
- Reduces connection overhead
- Improves performance

## Learning Resources

### Books
- "Database System Concepts" by Silberschatz, Korth, and Sudarshan
- "SQL Performance Explained" by Markus Winand
- "Designing Data-Intensive Applications" by Martin Kleppmann

### Online Resources
- [W3Schools SQL Tutorial](https://www.w3schools.com/sql/)
- [MongoDB University](https://university.mongodb.com/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [SQLBolt](https://sqlbolt.com/) - Interactive SQL lessons

### Practice Platforms
- LeetCode Database Problems
- HackerRank SQL Challenges
- SQLZoo
- Mode Analytics SQL Tutorial

## Contributing

This documentation is designed to be a comprehensive resource for database learning and interview preparation. For detailed comparisons and interview questions, please refer to the specific documentation files listed in the Table of Contents.

## Last Updated

October 2025

---

*Happy Learning!*
