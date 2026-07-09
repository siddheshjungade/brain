# DBMS & SQL Roadmap

Welcome to the Database Management Systems (DBMS) and SQL knowledge base. This roadmap is designed to guide you through database architecture, relational design, transactional integrity (ACID), indexing, query tuning, and distributed systems scaling (replication, partitioning, sharding).

---

## 🛠 Phase 1: Core Fundamentals
Understanding how databases store data, differentiate from file systems, and structure relational tables.
* **[Database Architectures](file:///Users/sid/workspace/brain/DBMS/1-Fundamentals/Database%20Architectures.md)**: File systems vs. DBMS, 3-Schema Architecture, physical/logical data independence, and RDBMS vs. NoSQL vs. NewSQL.
* **[Relational Model & Keys](file:///Users/sid/workspace/brain/DBMS/1-Fundamentals/Relational%20Model%20%26%20Keys.md)**: Relations, constraints, the keys hierarchy (Candidate, Primary, Foreign, Composite), and Relational Algebra foundations.

---

## 📐 Phase 2: Database Design & Normalization
Designing robust schemas to prevent redundancy and data anomalies.
* **[ER Diagrams](file:///Users/sid/workspace/brain/DBMS/2-Design%20%26%20Normalization/ER%20Diagrams.md)**: Entities, attributes, relationships, cardinalities, weak entity sets, and ER-to-relational schema mapping.
* **[Normalization](file:///Users/sid/workspace/brain/DBMS/2-Design%20%26%20Normalization/Normalization.md)**: Anomalies, Functional Dependencies, Normal Forms (1NF, 2NF, 3NF, BCNF), lossless decomposition, and dependency preservation.

---

## 🔄 Phase 3: Transactions & Concurrency Control
Ensuring reliability, consistency, and safe concurrent access in high-throughput environments.
* **[ACID Properties](file:///Users/sid/workspace/brain/DBMS/3-Transactions%20%26%20Concurrency/ACID%20Properties.md)**: Atomicity, Consistency, Isolation, and Durability internals.
* **[Concurrency Control](file:///Users/sid/workspace/brain/DBMS/3-Transactions%20%26%20Concurrency/Concurrency%20Control.md)**: Concurrency issues (dirty reads, phantom reads), serializability, lock-based protocols (2PL, Strict 2PL), deadlock handling, and SQL isolation levels.
* **[Recovery & Logging](file:///Users/sid/workspace/brain/DBMS/3-Transactions%20%26%20Concurrency/Recovery%20%26%20Logging.md)**: Write-Ahead Logging (WAL), checkpoints, deferred/immediate updates, and the ARIES recovery algorithm.

---

## ⚡ Phase 4: Storage, Indexing & Scaling
How databases achieve physical performance and scale horizontally.
* **[Indexing](file:///Users/sid/workspace/brain/DBMS/4-Storage%20%26%20Indexing/Indexing.md)**: Primary/Secondary indexes, Dense vs. Sparse indexing, and B-Trees vs. B+ Trees internals.
* **[Query Optimization](file:///Users/sid/workspace/brain/DBMS/4-Storage%20%26%20Indexing/Query%20Optimization.md)**: Parsing, cost-based optimizers, relational equivalence, and reading execution plans (`EXPLAIN`).
* **[Sharding, Replication & Partitioning](file:///Users/sid/workspace/brain/DBMS/4-Storage%20%26%20Indexing/Sharding%20Replication%20Partitioning.md)**: Replication models (single/multi-leader, leaderless), partitioning (range, hash, local/global indexes), sharding key selection, 2PC, and CAP/PACELC theorems.

---

## 💾 Phase 5: SQL Reference
Practical guide to Structured Query Language from basic DDL/DML to complex analytical queries.
* **[DDL DML TCL](file:///Users/sid/workspace/brain/DBMS/5-SQL%20Reference/1.DDL%20DML%20TCL.md)**: Definition, manipulation, and transaction control commands with constraints.
* **[Joins & Subqueries](file:///Users/sid/workspace/brain/DBMS/5-SQL%20Reference/2.Joins%20%26%20Subqueries.md)**: Inner/Outer/Cross/Self Joins, grouping, correlated subqueries, and Common Table Expressions (CTEs).
* **[Advanced SQL](file:///Users/sid/workspace/brain/DBMS/5-SQL%20Reference/3.Advanced%20SQL.md)**: Window functions (ranks, offsets), database views, stored procedures, and triggers.

---

## 📅 4-Week Study Sprint

| Week | Focus Area | Key Goal |
| :--- | :--- | :--- |
| **Week 1** | Fundamentals & Normalization | Design ER schemas, normalize tables up to BCNF, compute FDs. |
| **Week 2** | Concurrency & Recovery | Master serializability, 2PL timelines, write WAL logs, and isolation levels. |
| **Week 3** | Indexing, Query Plans & Tuning | Draw B+ Trees, analyze execution plans (`EXPLAIN`), optimize query performance. |
| **Week 4** | Distributed Systems & SQL | Master advanced window functions, CTEs, sharding strategies, and CAP trade-offs. |

---

## 📚 Top Resources
1. **Database System Concepts (Silberschatz, Korth, Sudarshan)**: The classic standard textbook for core DB engines.
2. **Designing Data-Intensive Applications (Martin Kleppmann)**: Essential for replication, partitioning, transactions, and distributed databases.
3. **Use The Index, Luke (Marcus Winand)**: A developer-centric guide to database indexing and SQL tuning.
