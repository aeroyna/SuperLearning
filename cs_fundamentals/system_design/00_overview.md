# System Design - Interview Preparation

A comprehensive System Design learning path designed for technical interviews at top companies (Google, Microsoft, Amazon, Meta, Netflix, Uber, etc.). This module covers both **High Level Design (HLD)** for distributed systems architecture and **Low Level Design (LLD)** for object-oriented design.

## Target Audience

This curriculum is designed for candidates preparing for:
- Senior/Staff Software Engineering Interviews at FAANG/MAANG companies
- System Design rounds (typically 45-60 minutes)
- Object-Oriented Design rounds
- Architecture discussions in behavioral rounds

---

## Learning Path

### [1. Fundamentals](Fundamentals/00_fundamentals.md)

- **[1. Scalability](Fundamentals/Scalability/00_scalability.md)**
  - [1.1 Horizontal vs Vertical Scaling](Fundamentals/Scalability/01_horizontal_vs_vertical_scaling.md)
  - [1.2 Load Balancing](Fundamentals/Scalability/02_load_balancing.md)
  - [1.3 Database Replication](Fundamentals/Scalability/03_database_replication.md)

- **[2. CAP Theorem & Distributed Systems](Fundamentals/CAP_Theorem/00_cap_theorem.md)**
  - [2.1 Consistency vs Availability](Fundamentals/CAP_Theorem/01_consistency_availability_tradeoffs.md)
  - [2.2 Eventual Consistency](Fundamentals/CAP_Theorem/02_eventual_consistency.md)
  - [2.3 Consensus Algorithms](Fundamentals/CAP_Theorem/03_consensus_algorithms.md)

- **[3. Database Fundamentals](Fundamentals/Database_Fundamentals/00_database_fundamentals.md)**
  - [3.1 SQL vs NoSQL](Fundamentals/Database_Fundamentals/01_sql_vs_nosql.md)
  - [3.2 Database Indexing](Fundamentals/Database_Fundamentals/02_database_indexing.md)
  - [3.3 Database Sharding](Fundamentals/Database_Fundamentals/03_database_sharding.md)
  - [3.4 Database Partitioning](Fundamentals/Database_Fundamentals/04_database_partitioning.md)

- **[4. Caching](Fundamentals/Caching/00_caching.md)**
  - [4.1 Caching Strategies](Fundamentals/Caching/01_caching_strategies.md)
  - [4.2 Cache Eviction Policies](Fundamentals/Caching/02_cache_eviction_policies.md)
  - [4.3 Distributed Caching](Fundamentals/Caching/03_distributed_caching.md)

- **[5. Networking Basics](Fundamentals/Networking_Basics/00_networking_basics.md)**
  - [5.1 HTTP/HTTPS & REST](Fundamentals/Networking_Basics/01_http_rest.md)
  - [5.2 WebSockets & Long Polling](Fundamentals/Networking_Basics/02_websockets_long_polling.md)
  - [5.3 gRPC & Protocol Buffers](Fundamentals/Networking_Basics/03_grpc_protobuf.md)

---

### [2. High Level Design (HLD)](HLD/00_hld.md)

- **[6. Core Components](HLD/Core_Components/00_core_components.md)**
  - [6.1 Load Balancers](HLD/Core_Components/01_load_balancers.md)
  - [6.2 Content Delivery Networks (CDN)](HLD/Core_Components/02_cdn.md)
  - [6.3 Message Queues](HLD/Core_Components/03_message_queues.md)
  - [6.4 API Gateway](HLD/Core_Components/04_api_gateway.md)
  - [6.5 Service Discovery](HLD/Core_Components/05_service_discovery.md)
  - [6.6 Reverse Proxy](HLD/Core_Components/06_reverse_proxy.md)

- **[7. Architecture Patterns](HLD/Architecture_Patterns/00_architecture_patterns.md)**
  - [7.1 Monolithic vs Microservices](HLD/Architecture_Patterns/01_monolithic_vs_microservices.md)
  - [7.2 Event-Driven Architecture](HLD/Architecture_Patterns/02_event_driven_architecture.md)
  - [7.3 CQRS Pattern](HLD/Architecture_Patterns/03_cqrs.md)
  - [7.4 Saga Pattern](HLD/Architecture_Patterns/04_saga_pattern.md)
  - [7.5 Circuit Breaker Pattern](HLD/Architecture_Patterns/05_circuit_breaker.md)

- **[8. HLD Case Studies](HLD/Case_Studies/00_case_studies.md)**
  - [8.1 Design URL Shortener (TinyURL)](HLD/Case_Studies/01_design_url_shortener.md)
  - [8.2 Design Rate Limiter](HLD/Case_Studies/02_design_rate_limiter.md)
  - [8.3 Design Twitter/X](HLD/Case_Studies/03_design_twitter.md)
  - [8.4 Design Instagram](HLD/Case_Studies/04_design_instagram.md)
  - [8.5 Design WhatsApp/Messenger](HLD/Case_Studies/05_design_whatsapp.md)
  - [8.6 Design YouTube/Netflix](HLD/Case_Studies/06_design_youtube.md)
  - [8.7 Design Uber/Lyft](HLD/Case_Studies/07_design_uber.md)
  - [8.8 Design Notification System](HLD/Case_Studies/08_design_notification_system.md)
  - [8.9 Design Search Autocomplete](HLD/Case_Studies/09_design_search_autocomplete.md)
  - [8.10 Design Distributed Cache](HLD/Case_Studies/10_design_distributed_cache.md)
  - [8.11 Design Web Crawler](HLD/Case_Studies/11_design_web_crawler.md)
  - [8.12 Design Ticket Booking System](HLD/Case_Studies/12_design_ticket_booking.md)

- **[9. HLD Interview Framework](HLD/Interview_Framework/00_interview_framework.md)**
  - [9.1 Requirements Gathering](HLD/Interview_Framework/01_requirements_gathering.md)
  - [9.2 Back-of-Envelope Estimation](HLD/Interview_Framework/02_estimation_calculations.md)
  - [9.3 High Level Design Steps](HLD/Interview_Framework/03_hld_steps.md)
  - [9.4 Deep Dive Strategies](HLD/Interview_Framework/04_deep_dive_strategies.md)

---

### [3. Low Level Design (LLD)](LLD/00_lld.md)

- **[10. SOLID Principles](LLD/SOLID_Principles/00_solid_principles.md)**
  - [10.1 Single Responsibility Principle](LLD/SOLID_Principles/01_single_responsibility.md)
  - [10.2 Open/Closed Principle](LLD/SOLID_Principles/02_open_closed.md)
  - [10.3 Liskov Substitution Principle](LLD/SOLID_Principles/03_liskov_substitution.md)
  - [10.4 Interface Segregation Principle](LLD/SOLID_Principles/04_interface_segregation.md)
  - [10.5 Dependency Inversion Principle](LLD/SOLID_Principles/05_dependency_inversion.md)

- **[11. Design Patterns](LLD/Design_Patterns/00_design_patterns.md)**

  - **[11.1 Creational Patterns](LLD/Design_Patterns/Creational/00_creational.md)**
    - [11.1.1 Singleton](LLD/Design_Patterns/Creational/01_singleton.md)
    - [11.1.2 Factory Method](LLD/Design_Patterns/Creational/02_factory.md)
    - [11.1.3 Abstract Factory](LLD/Design_Patterns/Creational/03_abstract_factory.md)
    - [11.1.4 Builder](LLD/Design_Patterns/Creational/04_builder.md)
    - [11.1.5 Prototype](LLD/Design_Patterns/Creational/05_prototype.md)

  - **[11.2 Structural Patterns](LLD/Design_Patterns/Structural/00_structural.md)**
    - [11.2.1 Adapter](LLD/Design_Patterns/Structural/01_adapter.md)
    - [11.2.2 Decorator](LLD/Design_Patterns/Structural/02_decorator.md)
    - [11.2.3 Facade](LLD/Design_Patterns/Structural/03_facade.md)
    - [11.2.4 Proxy](LLD/Design_Patterns/Structural/04_proxy.md)
    - [11.2.5 Composite](LLD/Design_Patterns/Structural/05_composite.md)

  - **[11.3 Behavioral Patterns](LLD/Design_Patterns/Behavioral/00_behavioral.md)**
    - [11.3.1 Strategy](LLD/Design_Patterns/Behavioral/01_strategy.md)
    - [11.3.2 Observer](LLD/Design_Patterns/Behavioral/02_observer.md)
    - [11.3.3 Command](LLD/Design_Patterns/Behavioral/03_command.md)
    - [11.3.4 State](LLD/Design_Patterns/Behavioral/04_state.md)
    - [11.3.5 Chain of Responsibility](LLD/Design_Patterns/Behavioral/05_chain_of_responsibility.md)
    - [11.3.6 Template Method](LLD/Design_Patterns/Behavioral/06_template_method.md)

- **[12. LLD Case Studies](LLD/Case_Studies/00_case_studies.md)**
  - [12.1 Design Parking Lot](LLD/Case_Studies/01_design_parking_lot.md)
  - [12.2 Design Elevator System](LLD/Case_Studies/02_elevator_system.md)
  - [12.3 Design Library Management System](LLD/Case_Studies/03_library_management.md)
  - [12.4 Design Vending Machine](LLD/Case_Studies/04_vending_machine.md)
  - [12.5 Design Tic-Tac-Toe](LLD/Case_Studies/05_tic_tac_toe.md)
  - [12.6 Design Chess Game](LLD/Case_Studies/06_chess_game.md)
  - [12.7 Design Movie Booking System](LLD/Case_Studies/07_booking_system.md)
  - [12.8 Design LRU Cache](LLD/Case_Studies/08_lru_cache.md)
  - [12.9 Design Snake Game](LLD/Case_Studies/09_snake_game.md)
  - [12.10 Design File System](LLD/Case_Studies/10_file_system.md)

- **[13. UML Diagrams](LLD/UML_Diagrams/00_uml_diagrams.md)**
  - [13.1 Class Diagrams](LLD/UML_Diagrams/01_class_diagrams.md)
  - [13.2 Sequence Diagrams](LLD/UML_Diagrams/02_sequence_diagrams.md)
  - [13.3 Use Case Diagrams](LLD/UML_Diagrams/03_use_case_diagrams.md)

---

### [4. Interview Preparation](Interview_Prep/00_interview_prep.md)

- **[14. HLD Interview Template](Interview_Prep/01_hld_interview_template.md)**
- **[15. LLD Interview Template](Interview_Prep/02_lld_interview_template.md)**
- **[16. Common Mistakes](Interview_Prep/03_common_mistakes.md)**
- **[17. Estimation Cheatsheet](Interview_Prep/04_estimation_cheatsheet.md)**

---

### [5. Google System Design Interview Prep](Google_Prep/00_google_overview.md)

Google's system design interviews have specific characteristics and expectations. This section is tailored for Google L4-L6+ interviews.

- **[18. Google Interview Format](Google_Prep/01_interview_format.md)**
  - Interview structure and expectations
  - L4 vs L5 vs L6+ differences
  - Evaluation criteria (Googleyness + Leadership)

- **[19. Google-Specific Topics](Google_Prep/02_google_topics.md)**
  - Spanner & global consistency
  - Bigtable & wide-column stores
  - MapReduce & distributed processing
  - Borg & container orchestration
  - Pub/Sub & messaging at scale

- **[20. Google Case Studies](Google_Prep/03_google_case_studies.md)**
  - Design Google Search
  - Design Google Maps
  - Design Google Drive
  - Design Gmail
  - Design YouTube (Google-specific considerations)
  - Design Google Photos

- **[21. Google Infrastructure Deep Dives](Google_Prep/04_infrastructure_deep_dives.md)**
  - Colossus (Google File System successor)
  - Chubby (distributed lock service)
  - Megastore & Spanner evolution
  - Zanzibar (authorization system)

- **[22. Mock Interview Scenarios](Google_Prep/05_mock_scenarios.md)**
  - Sample 45-minute interview walkthroughs
  - Common follow-up questions
  - How to handle ambiguity

---

## Quick Reference

- **[HLD Interview Template](Interview_Prep/01_hld_interview_template.md)** - Step-by-step framework for HLD interviews
- **[LLD Interview Template](Interview_Prep/02_lld_interview_template.md)** - Step-by-step framework for LLD interviews
- **[Estimation Cheatsheet](Interview_Prep/04_estimation_cheatsheet.md)** - Common numbers for back-of-envelope calculations
- **[Common Mistakes](Interview_Prep/03_common_mistakes.md)** - Pitfalls to avoid in system design interviews

---

## Study Plan Recommendations

### 2-Week Intensive (HLD Focus)
Focus on: Scalability fundamentals, CAP theorem, Core components (LB, CDN, Cache, MQ), 5 key HLD case studies (URL Shortener, Rate Limiter, Twitter, WhatsApp, Uber)

### 2-Week Intensive (LLD Focus)
Focus on: SOLID Principles, Top 10 Design Patterns, 5 key LLD case studies (Parking Lot, Elevator, LRU Cache, Chess, Booking System)

### 4-Week Comprehensive
Add: All fundamentals, All architecture patterns, All HLD + LLD case studies, UML Diagrams

### 8-Week Complete
Add: Deep dive into each case study, Mock interviews, Cross-referencing with DSA patterns

---

## Key Topics by Frequency (Based on Interview Data)

### Most Frequently Asked (HLD)
1. Design URL Shortener
2. Design Rate Limiter
3. Design Twitter/Feed System
4. Design Chat System (WhatsApp)
5. Design Video Streaming (YouTube/Netflix)
6. Design Ride Sharing (Uber)

### Most Frequently Asked (LLD)
1. Design Parking Lot
2. Design Elevator System
3. Design LRU Cache
4. Design Chess/Tic-Tac-Toe
5. Design Booking System

### Key Concepts to Master
1. Scalability & Load Balancing
2. Caching Strategies
3. Database Sharding & Replication
4. Message Queues & Async Processing
5. CAP Theorem Trade-offs
6. SOLID Principles
7. Common Design Patterns (Factory, Strategy, Observer, Singleton)
