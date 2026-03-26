# Airline Metrics Data Pipeline (ClickHouse)

Distributed analytical data pipeline in ClickHouse for computing airline performance metrics (ASK, RASK, Yield, Load Factor) with near real-time aggregation.

## 📌 Overview
This project implements a multi-layer data pipeline designed to transform raw ticket 
and boarding passes datasets into a query-optimized analytical data mart.

The system supports:
1) BI dashboards;
2) ad-hoc analytics;
3) Fast business metric computation;

The design prioritizes performance, scalability, and cost efficiency.

## 🎯 Problem

Airline analytics requires computing key performance metrics:
1) ASK (Available Seat Kilometers);
2) RASK (Revenue per ASK);
3) Yield (Revenue per RPK);
4) Load Factor (RPK / ASK);

These metrics depend on multiple heterogeneous data sources:
- append-only data (flights)
- mutable data (flight statuses)
- high-volume data (tickets, boarding passes)

Direct querying leads to:
- heavy JOIN operations
- slow query performance
- poor scalability

## ⚙️ Solution

A distributed ClickHouse pipeline was designed with:
- staged data transformations (raw → wide → aggregated)
- materialized views for incremental processing
- pre-aggregated tables for low-latency queries
- cluster-aware data modeling (replication + sharding)


## 🏗 Architecture
```
Sources
 ├── flights
 ├── flight_statuses
 ├── ticket_flights
 ├── boarding_passes

        ↓

Raw Layer
 ├── flights_raw (replicated)
 ├── ticket_flights_raw (sharded)
 ├── boarding_passes_raw (sharded)

        ↓ (Materialized Views)

Wide Layer
 ├── ticket_flights_wide
 ├── boarding_passes_wide

        ↓ (Materialized Views)

Aggregated Layer
 └── flight_metrics (pre-aggregated KPIs)
```

## ⚡ Technology Choice: Why ClickHouse

*Interactive performance*

The choice of database was driven by analytical workload requirements rather than convenience.
The system is designed for:
- BI dashboards;
- frequent ad-hoc queries;

Traditional OLTP databases (PostgreSQL) struggle with such workloads, 
while batch systems (Hadoop) introduce high latency.ClickHouse provides: 
- sub-second query execution;
- efficient columnar storage;
- high compression;

*Predictable cost model*

Unlike cloud-based DWH solutions, ClickHouse enables:
- predictable infrastructure costs;
- full control over resource usage;
- efficient handling of variable query workloads;

*Trade-offs*

1) Increased operational complexity;
2) Need to manage cluster infrastructure;

These trade-offs are justified by:
- high performance;
- scalability;
- cost efficiency;

## 🔧 Key Engineering Decisions

*1.Dictionaries instead of JOINs*

- airports and aircrafts converted into dictionaries;
- reduced runtime JOIN overhead;

*2.Hybrid cluster architecture*

1) Replication for critical tables (flights, statuses);
2) Sharding for high-volume tables (tickets, boarding passes);

*3.Materialized Views as pipeline engine*

- incremental data processing;
- automatic updates on insert;
- near real-time aggregation;

*4. Pre-aggregation strategy*

- central aggregated table: `flight_metrics`;
- projection for efficient grouped queries;


## 📊 Output Dataset

The final dataset includes:
- `route` — flight route;
- `dep_city`, `arr_city` — cities;
- `total_ask` — capacity;
- `total_passengers` — transported passengers;
- `total_revenue` — total revenue;
- `yield` — revenue per passenger-km;
- `load_factor` — seat occupancy ratio;
- `rask` — revenue per seat-km;

## 📈 Business Classification

Routes are categorized into: `Healthy`,  `Demand Issue`, `Pricing Issue`, `Poor`;

**Based on**:
`Yield` ≥ 11;
`Load Factor` ≥ 70%;

## Performance and limitaions

**Performance Considerations**:
- minimized JOIN operations;
- leveraged columnar storage;
- used pre-aggregation for heavy metrics;
- optimized for large-scale data growth;

**Limitations**: batch-oriented ingestion assumptions, limited handling of late-arriving data, no streaming ingestion layer

## 🔮 Future Improvements

1) Streaming ingestion (e.g., Kafka integration);
2) Data quality validation layer;
3) Improved partitioning strategy;
4) BI tool integration;
5) Add `chproxy`;

Open to your feedback, criticism and suggestions!
