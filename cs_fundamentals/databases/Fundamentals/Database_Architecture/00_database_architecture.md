# Database Architecture

## Overview

This section covers how databases work internally - from query processing to storage management. Understanding database internals helps you write better queries, design efficient schemas, and troubleshoot performance issues.

## Topics Covered

1. **[Storage Engine Internals](01_storage_engine_internals.md)** - How data is physically stored on disk
2. **[Buffer Pool and Caching](02_buffer_pool_and_caching.md)** - Memory management for performance
3. **[Write-Ahead Logging](03_write_ahead_logging.md)** - Durability and crash recovery
4. **[Query Processing Pipeline](04_query_processing_pipeline.md)** - How queries are parsed, optimized, and executed

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DATABASE ARCHITECTURE                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                      APPLICATION / CLIENT                            │   │
│   └────────────────────────────────┬────────────────────────────────────┘   │
│                                    │ SQL Query                              │
│   ┌────────────────────────────────▼────────────────────────────────────┐   │
│   │                         QUERY PROCESSOR                              │   │
│   │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐    │   │
│   │  │  Parser  │─▶│ Analyzer │─▶│Optimizer │─▶│ Execution Engine │    │   │
│   │  └──────────┘  └──────────┘  └──────────┘  └──────────────────┘    │   │
│   └────────────────────────────────┬────────────────────────────────────┘   │
│                                    │                                         │
│   ┌────────────────────────────────▼────────────────────────────────────┐   │
│   │                      TRANSACTION MANAGER                             │   │
│   │  ┌─────────────┐  ┌──────────────┐  ┌───────────────────────────┐  │   │
│   │  │Lock Manager │  │ MVCC Engine  │  │ Recovery Manager (WAL)    │  │   │
│   │  └─────────────┘  └──────────────┘  └───────────────────────────┘  │   │
│   └────────────────────────────────┬────────────────────────────────────┘   │
│                                    │                                         │
│   ┌────────────────────────────────▼────────────────────────────────────┐   │
│   │                        STORAGE ENGINE                                │   │
│   │  ┌─────────────┐  ┌──────────────┐  ┌───────────────────────────┐  │   │
│   │  │ Buffer Pool │  │Index Manager │  │   Space Manager           │  │   │
│   │  │  (Cache)    │  │ (B-Tree/Hash)│  │   (Pages/Extents)         │  │   │
│   │  └─────────────┘  └──────────────┘  └───────────────────────────┘  │   │
│   └────────────────────────────────┬────────────────────────────────────┘   │
│                                    │                                         │
│   ┌────────────────────────────────▼────────────────────────────────────┐   │
│   │                          DISK STORAGE                                │   │
│   │           Data Files    │    Index Files    │    Log Files          │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Learning Objectives

After completing this section, you will be able to:
- Explain how data is organized and stored in database files
- Understand the role of buffer pools and caching strategies
- Describe write-ahead logging and its importance for durability
- Trace a query through the processing pipeline
- Make informed decisions about database configuration and tuning
