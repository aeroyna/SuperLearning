# Databases Course Overview

## Learning Path

### Phase 1: Fundamentals

- **[1. Introduction to Databases](Fundamentals/Introduction_to_Databases/00_introduction_to_databases.md)**
  - [1.1 What is a Database](Fundamentals/Introduction_to_Databases/01_what_is_a_database.md)
  - [1.2 DBMS Types and History](Fundamentals/Introduction_to_Databases/02_dbms_types_and_history.md)
  - [1.3 Database Use Cases](Fundamentals/Introduction_to_Databases/03_database_use_cases.md)

- **[2. Data Models](Fundamentals/Data_Models/00_data_models.md)**
  - [2.1 Relational Model](Fundamentals/Data_Models/01_relational_model.md)
  - [2.2 Document Model](Fundamentals/Data_Models/02_document_model.md)
  - [2.3 Key-Value Model](Fundamentals/Data_Models/03_key_value_model.md)
  - [2.4 Graph Model](Fundamentals/Data_Models/04_graph_model.md)
  - [2.5 Wide-Column Model](Fundamentals/Data_Models/05_wide_column_model.md)

- **[3. Database Architecture](Fundamentals/Database_Architecture/00_database_architecture.md)**
  - [3.1 Storage Engine Internals](Fundamentals/Database_Architecture/01_storage_engine_internals.md)
  - [3.2 Buffer Pool and Caching](Fundamentals/Database_Architecture/02_buffer_pool_and_caching.md)
  - [3.3 Write-Ahead Logging](Fundamentals/Database_Architecture/03_write_ahead_logging.md)
  - [3.4 Query Processing Pipeline](Fundamentals/Database_Architecture/04_query_processing_pipeline.md)

- **[4. SQL Basics](Fundamentals/SQL_Basics/00_sql_basics.md)**
  - [4.1 DDL Data Definition Language](Fundamentals/SQL_Basics/01_ddl_data_definition_language.md)
  - [4.2 DML Data Manipulation Language](Fundamentals/SQL_Basics/02_dml_data_manipulation_language.md)
  - [4.3 DQL Data Query Language](Fundamentals/SQL_Basics/03_dql_data_query_language.md)
  - [4.4 DCL and TCL](Fundamentals/SQL_Basics/04_dcl_and_tcl.md)
  - [4.5 Joins Deep Dive](Fundamentals/SQL_Basics/05_joins_deep_dive.md)
  - [4.6 Subqueries and CTEs](Fundamentals/SQL_Basics/06_subqueries_and_ctes.md)
  - [4.7 Aggregate Functions and Grouping](Fundamentals/SQL_Basics/07_aggregate_functions_and_grouping.md)
  - [4.8 Window Functions](Fundamentals/SQL_Basics/08_window_functions.md)

- **[5. Normalization](Fundamentals/Normalization/00_normalization.md)**
  - [5.1 Functional Dependencies](Fundamentals/Normalization/01_functional_dependencies.md)
  - [5.2 First Normal Form](Fundamentals/Normalization/02_first_normal_form.md)
  - [5.3 Second Normal Form](Fundamentals/Normalization/03_second_normal_form.md)
  - [5.4 Third Normal Form](Fundamentals/Normalization/04_third_normal_form.md)
  - [5.5 BCNF and Higher Forms](Fundamentals/Normalization/05_bcnf_and_higher_forms.md)
  - [5.6 Denormalization](Fundamentals/Normalization/06_denormalization.md)

- **[6. ER Modeling](Fundamentals/ER_Modeling/00_er_modeling.md)**
  - [6.1 Entities and Attributes](Fundamentals/ER_Modeling/01_entities_and_attributes.md)
  - [6.2 Relationships and Cardinality](Fundamentals/ER_Modeling/02_relationships_and_cardinality.md)
  - [6.3 ER to Relational Mapping](Fundamentals/ER_Modeling/03_er_to_relational_mapping.md)
  - [6.4 Advanced ER Concepts](Fundamentals/ER_Modeling/04_advanced_er_concepts.md)

- **[7. ACID and Transactions](Fundamentals/ACID_and_Transactions/00_acid_and_transactions.md)**
  - [7.1 ACID Properties Deep Dive](Fundamentals/ACID_and_Transactions/01_acid_properties_deep_dive.md)
  - [7.2 Transaction Lifecycle](Fundamentals/ACID_and_Transactions/02_transaction_lifecycle.md)
  - [7.3 Isolation Levels](Fundamentals/ACID_and_Transactions/03_isolation_levels.md)
  - [7.4 Concurrency Control](Fundamentals/ACID_and_Transactions/04_concurrency_control.md)
  - [7.5 Locking Mechanisms](Fundamentals/ACID_and_Transactions/05_locking_mechanisms.md)
  - [7.6 MVCC Deep Dive](Fundamentals/ACID_and_Transactions/06_mvcc_deep_dive.md)

- **[8. Keys and Constraints](Fundamentals/Keys_and_Constraints/00_keys_and_constraints.md)**
  - [8.1 Key Types](Fundamentals/Keys_and_Constraints/01_key_types.md)
  - [8.2 Database Constraints](Fundamentals/Keys_and_Constraints/02_constraints.md)

### Phase 2: MySQL

- **[9. MySQL Architecture](MySQL/MySQL_Architecture/00_mysql_architecture.md)**
  - [9.1 Server Architecture](MySQL/MySQL_Architecture/01_server_architecture.md)
  - [9.2 InnoDB Storage Engine](MySQL/MySQL_Architecture/02_innodb_storage_engine.md)
  - [9.3 MyISAM vs InnoDB](MySQL/MySQL_Architecture/03_myisam_vs_innodb.md)
  - [9.4 MySQL Memory Structures](MySQL/MySQL_Architecture/04_mysql_memory_structures.md)

- **[10. MySQL Indexing](MySQL/MySQL_Indexing/00_mysql_indexing.md)**
  - [10.1 B-Tree Indexes](MySQL/MySQL_Indexing/01_btree_indexes.md)
  - [10.2 Hash Indexes](MySQL/MySQL_Indexing/02_hash_indexes.md)
  - [10.3 Full-Text Indexes](MySQL/MySQL_Indexing/03_fulltext_indexes.md)
  - [10.4 Covering Indexes](MySQL/MySQL_Indexing/04_covering_indexes.md)
  - [10.5 Index Optimization](MySQL/MySQL_Indexing/05_index_optimization.md)

- **[11. MySQL Query Optimization](MySQL/MySQL_Query_Optimization/00_mysql_query_optimization.md)**
  - [11.1 EXPLAIN and Query Plans](MySQL/MySQL_Query_Optimization/01_explain_and_query_plans.md)
  - [11.2 Query Profiling](MySQL/MySQL_Query_Optimization/02_query_profiling.md)
  - [11.3 Slow Query Log](MySQL/MySQL_Query_Optimization/03_slow_query_log.md)
  - [11.4 Optimization Techniques](MySQL/MySQL_Query_Optimization/04_optimization_techniques.md)

- **[12. MySQL Replication](MySQL/MySQL_Replication/00_mysql_replication.md)**
  - [12.1 Binary Log Replication](MySQL/MySQL_Replication/01_binary_log_replication.md)
  - [12.2 Master-Slave Setup](MySQL/MySQL_Replication/02_master_slave_setup.md)
  - [12.3 Group Replication](MySQL/MySQL_Replication/03_group_replication.md)
  - [12.4 Replication Lag and Troubleshooting](MySQL/MySQL_Replication/04_replication_lag_and_troubleshooting.md)

- **[13. MySQL Administration](MySQL/MySQL_Administration/00_mysql_administration.md)**
  - [13.1 User Management and Security](MySQL/MySQL_Administration/01_user_management_and_security.md)
  - [13.2 Backup and Recovery](MySQL/MySQL_Administration/02_backup_and_recovery.md)
  - [13.3 Performance Tuning](MySQL/MySQL_Administration/03_performance_tuning.md)
  - [13.4 Monitoring and Diagnostics](MySQL/MySQL_Administration/04_monitoring_and_diagnostics.md)

### Phase 3: PostgreSQL

- **[14. PostgreSQL Architecture](PostgreSQL/PostgreSQL_Architecture/00_postgresql_architecture.md)**
  - [14.1 Process Architecture](PostgreSQL/PostgreSQL_Architecture/01_process_architecture.md)
  - [14.2 Memory Architecture](PostgreSQL/PostgreSQL_Architecture/02_memory_architecture.md)
  - [14.3 Storage and TOAST](PostgreSQL/PostgreSQL_Architecture/03_storage_and_toast.md)
  - [14.4 WAL and Checkpoints](PostgreSQL/PostgreSQL_Architecture/04_wal_and_checkpoints.md)

- **[15. PostgreSQL Advanced Features](PostgreSQL/PostgreSQL_Advanced_Features/00_postgresql_advanced_features.md)**
  - [15.1 Advanced Data Types](PostgreSQL/PostgreSQL_Advanced_Features/01_advanced_data_types.md)
  - [15.2 JSONB and Document Storage](PostgreSQL/PostgreSQL_Advanced_Features/02_jsonb_and_document_storage.md)
  - [15.3 Full-Text Search](PostgreSQL/PostgreSQL_Advanced_Features/03_fulltext_search.md)
  - [15.4 Extensions and Foreign Data Wrappers](PostgreSQL/PostgreSQL_Advanced_Features/04_extensions_and_fdw.md)

- **[16. PostgreSQL Indexing](PostgreSQL/PostgreSQL_Indexing/00_postgresql_indexing.md)**
  - [16.1 B-Tree Internals](PostgreSQL/PostgreSQL_Indexing/01_btree_internals.md)
  - [16.2 Hash, GIN, and GiST Indexes](PostgreSQL/PostgreSQL_Indexing/02_hash_gin_gist.md)
  - [16.3 BRIN and Bloom Indexes](PostgreSQL/PostgreSQL_Indexing/03_brin_and_bloom.md)
  - [16.4 Partial and Expression Indexes](PostgreSQL/PostgreSQL_Indexing/04_partial_expression_indexes.md)
  - [16.5 Index-Only Scans](PostgreSQL/PostgreSQL_Indexing/05_index_only_scans.md)

- **[17. PostgreSQL Concurrency](PostgreSQL/PostgreSQL_Concurrency/00_postgresql_concurrency.md)**
  - [17.1 MVCC Deep Dive](PostgreSQL/PostgreSQL_Concurrency/01_mvcc_deep_dive.md)
  - [17.2 Transaction Isolation](PostgreSQL/PostgreSQL_Concurrency/02_transaction_isolation.md)
  - [17.3 Locks and Deadlocks](PostgreSQL/PostgreSQL_Concurrency/03_locks_and_deadlocks.md)
  - [17.4 Vacuum and Bloat](PostgreSQL/PostgreSQL_Concurrency/04_vacuum_and_bloat.md)

- **[18. PostgreSQL Replication and HA](PostgreSQL/PostgreSQL_Replication_HA/00_replication_and_ha.md)**
  - [18.1 Streaming Replication](PostgreSQL/PostgreSQL_Replication_HA/01_streaming_replication.md)
  - [18.2 Logical Replication](PostgreSQL/PostgreSQL_Replication_HA/02_logical_replication.md)
  - [18.3 High Availability](PostgreSQL/PostgreSQL_Replication_HA/03_high_availability.md)
  - [18.4 Backup and Recovery](PostgreSQL/PostgreSQL_Replication_HA/04_backup_and_recovery.md)

### Phase 4: NoSQL Databases

- **[19. MongoDB](NoSQL/MongoDB/00_mongodb.md)**
  - [19.1 Document Model and BSON](NoSQL/MongoDB/01_document_model_and_bson.md)
  - [19.2 CRUD Operations](NoSQL/MongoDB/02_crud_operations.md)
  - [19.3 Aggregation Pipeline](NoSQL/MongoDB/03_aggregation_pipeline.md)
  - [19.4 Indexing Strategies](NoSQL/MongoDB/04_indexing_strategies.md)
  - [19.5 Sharding and Replication](NoSQL/MongoDB/05_sharding_and_replication.md)
  - [19.6 Schema Design Patterns](NoSQL/MongoDB/06_schema_design_patterns.md)

- **[20. Redis](NoSQL/Redis/00_redis.md)**
  - [20.1 Data Structures](NoSQL/Redis/01_data_structures.md)
  - [20.2 Persistence Options](NoSQL/Redis/02_persistence_options.md)
  - [20.3 Pub/Sub and Streams](NoSQL/Redis/03_pubsub_and_streams.md)
  - [20.4 Clustering and Sentinel](NoSQL/Redis/04_clustering_and_sentinel.md)
  - [20.5 Caching Patterns](NoSQL/Redis/05_caching_patterns.md)
  - [20.6 Lua Scripting](NoSQL/Redis/06_lua_scripting.md)

- **[21. Cassandra](NoSQL/Cassandra/00_cassandra.md)**
  - [21.1 Data Modeling](NoSQL/Cassandra/01_data_modeling.md)
  - [21.2 CQL and Operations](NoSQL/Cassandra/02_cql_and_operations.md)
  - [21.3 Partitioning Strategy](NoSQL/Cassandra/03_partitioning_strategy.md)
  - [21.4 Consistency Tuning](NoSQL/Cassandra/04_consistency_tuning.md)
  - [21.5 Cluster Operations](NoSQL/Cassandra/05_cluster_operations.md)

- **[22. Neo4j](NoSQL/Neo4j/00_neo4j.md)**
  - [22.1 Graph Concepts](NoSQL/Neo4j/01_graph_concepts.md)
  - [22.2 Cypher Language](NoSQL/Neo4j/02_cypher_language.md)
  - [22.3 Graph Algorithms](NoSQL/Neo4j/03_graph_algorithms.md)
  - [22.4 Deployment](NoSQL/Neo4j/04_deployment.md)

### Phase 5: Indexing and Query Optimization

- **[23. Indexing Deep Dive](Indexing_and_Optimization/Indexing_Deep_Dive/00_indexing_deep_dive.md)**
  - [23.1 B-Tree Internals](Indexing_and_Optimization/Indexing_Deep_Dive/01_btree_internals.md)
  - [23.2 LSM Trees](Indexing_and_Optimization/Indexing_Deep_Dive/02_lsm_trees.md)
  - [23.3 Bloom Filters](Indexing_and_Optimization/Indexing_Deep_Dive/03_bloom_filters.md)
  - [23.4 Skip Lists](Indexing_and_Optimization/Indexing_Deep_Dive/04_skip_lists.md)
  - [23.5 Inverted Indexes](Indexing_and_Optimization/Indexing_Deep_Dive/05_inverted_indexes.md)

- **[24. Query Optimization](Indexing_and_Optimization/Query_Optimization/00_query_optimization.md)**
  - [24.1 Cost-Based Optimization](Indexing_and_Optimization/Query_Optimization/01_cost_based_optimization.md)
  - [24.2 Query Execution Plans](Indexing_and_Optimization/Query_Optimization/02_query_execution_plans.md)
  - [24.3 Join Algorithms](Indexing_and_Optimization/Query_Optimization/03_join_algorithms.md)
  - [24.4 Statistics and Cardinality](Indexing_and_Optimization/Query_Optimization/04_statistics_and_cardinality.md)

### Phase 6: Distributed Databases

- **[25. Distributed Systems Fundamentals](Distributed_Databases/Distributed_Fundamentals/00_distributed_fundamentals.md)**
  - [25.1 CAP Theorem](Distributed_Databases/Distributed_Fundamentals/01_cap_theorem.md)
  - [25.2 PACELC Theorem](Distributed_Databases/Distributed_Fundamentals/02_pacelc_theorem.md)
  - [25.3 Consistency Models](Distributed_Databases/Distributed_Fundamentals/03_consistency_models.md)
  - [25.4 Consensus Algorithms](Distributed_Databases/Distributed_Fundamentals/04_consensus_algorithms.md)

- **[26. Sharding and Partitioning](Distributed_Databases/Sharding_and_Partitioning/00_sharding_and_partitioning.md)**
  - [26.1 Horizontal vs Vertical Partitioning](Distributed_Databases/Sharding_and_Partitioning/01_horizontal_vs_vertical_partitioning.md)
  - [26.2 Sharding Strategies](Distributed_Databases/Sharding_and_Partitioning/02_sharding_strategies.md)
  - [26.3 Consistent Hashing](Distributed_Databases/Sharding_and_Partitioning/03_consistent_hashing.md)
  - [26.4 Rebalancing and Resharding](Distributed_Databases/Sharding_and_Partitioning/04_rebalancing_and_resharding.md)

- **[27. Replication](Distributed_Databases/Replication/00_replication.md)**
  - [27.1 Single-Leader Replication](Distributed_Databases/Replication/01_single_leader_replication.md)
  - [27.2 Multi-Leader Replication](Distributed_Databases/Replication/02_multi_leader_replication.md)
  - [27.3 Leaderless Replication](Distributed_Databases/Replication/03_leaderless_replication.md)
  - [27.4 Conflict Resolution](Distributed_Databases/Replication/04_conflict_resolution.md)

- **[28. NewSQL and Distributed SQL](Distributed_Databases/NewSQL/00_newsql.md)**
  - [28.1 CockroachDB](Distributed_Databases/NewSQL/01_cockroachdb.md)
  - [28.2 TiDB](Distributed_Databases/NewSQL/02_tidb.md)
  - [28.3 Spanner and F1](Distributed_Databases/NewSQL/03_spanner_and_f1.md)
  - [28.4 Vitess](Distributed_Databases/NewSQL/04_vitess.md)

### Phase 7: Database Design and Best Practices

- **[29. Schema Design](Database_Design/Schema_Design/00_schema_design.md)**
  - [29.1 Design Methodology](Database_Design/Schema_Design/01_design_methodology.md)
  - [29.2 Common Schema Patterns](Database_Design/Schema_Design/02_common_schema_patterns.md)
  - [29.3 Temporal Data Modeling](Database_Design/Schema_Design/03_temporal_data_modeling.md)
  - [29.4 Multi-Tenancy Patterns](Database_Design/Schema_Design/04_multi_tenancy_patterns.md)

- **[30. Database Security](Database_Design/Database_Security/00_database_security.md)**
  - [30.1 Authentication and Authorization](Database_Design/Database_Security/01_authentication_and_authorization.md)
  - [30.2 Encryption at Rest and In Transit](Database_Design/Database_Security/02_encryption.md)
  - [30.3 SQL Injection Prevention](Database_Design/Database_Security/03_sql_injection_prevention.md)
  - [30.4 Auditing and Compliance](Database_Design/Database_Security/04_auditing_and_compliance.md)

- **[31. Migrations and Versioning](Database_Design/Migrations_and_Versioning/00_migrations_and_versioning.md)**
  - [31.1 Schema Migration Strategies](Database_Design/Migrations_and_Versioning/01_schema_migration_strategies.md)
  - [31.2 Zero-Downtime Migrations](Database_Design/Migrations_and_Versioning/02_zero_downtime_migrations.md)
  - [31.3 Migration Tools](Database_Design/Migrations_and_Versioning/03_migration_tools.md)
  - [31.4 Data Migration Patterns](Database_Design/Migrations_and_Versioning/04_data_migration_patterns.md)

- **[32. Backup and Disaster Recovery](Database_Design/Backup_and_Recovery/00_backup_and_recovery.md)**
  - [32.1 Backup Strategies](Database_Design/Backup_and_Recovery/01_backup_strategies.md)
  - [32.2 Point-in-Time Recovery](Database_Design/Backup_and_Recovery/02_point_in_time_recovery.md)
  - [32.3 Disaster Recovery Planning](Database_Design/Backup_and_Recovery/03_disaster_recovery_planning.md)
  - [32.4 Testing Recovery Procedures](Database_Design/Backup_and_Recovery/04_testing_recovery_procedures.md)

### Phase 8: Real-World Applications

- **[33. Database Selection](Real_World/Database_Selection/00_database_selection.md)**
  - [33.1 Choosing the Right Database](Real_World/Database_Selection/01_choosing_the_right_database.md)
  - [33.2 Polyglot Persistence](Real_World/Database_Selection/02_polyglot_persistence.md)
  - [33.3 Migration Between Databases](Real_World/Database_Selection/03_migration_between_databases.md)

- **[34. Case Studies](Real_World/Case_Studies/00_case_studies.md)**
  - [34.1 E-Commerce Database Design](Real_World/Case_Studies/01_ecommerce_database_design.md)
  - [34.2 Social Media Database Design](Real_World/Case_Studies/02_social_media_database_design.md)
  - [34.3 Analytics and Data Warehousing](Real_World/Case_Studies/03_analytics_and_data_warehousing.md)
  - [34.4 Real-Time Systems](Real_World/Case_Studies/04_real_time_systems.md)

- **[35. Performance Engineering](Real_World/Performance_Engineering/00_performance_engineering.md)**
  - [35.1 Benchmarking Databases](Real_World/Performance_Engineering/01_benchmarking_databases.md)
  - [35.2 Load Testing](Real_World/Performance_Engineering/02_load_testing.md)
  - [35.3 Capacity Planning](Real_World/Performance_Engineering/03_capacity_planning.md)
  - [35.4 Monitoring and Alerting](Real_World/Performance_Engineering/04_monitoring_and_alerting.md)
