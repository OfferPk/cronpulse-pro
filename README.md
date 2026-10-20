# CronPulse Pro (v1.0.0)

> Universal Cron AST Parser, Natural Language Translator, Next-10 Execution Visualizer & Distributed Lock Synthesizer.

---

## What is CronPulse Pro?

**CronPulse Pro** is an architectural developer studio and CLI utility built to eliminate cron scheduling errors, timezone confusion, and overlapping multi-replica task executions. Supporting **5-field UNIX**, **6-field Quartz / Spring / AWS EventBridge**, and **7-field Enterprise** formats, it provides instant human-readable English translations, countdowns across global timezones, and battle-tested distributed mutex boilerplate for Redis, PostgreSQL, and Kubernetes.

---

## Key Features

1. **Universal Cron AST Parser**:  
     
   - Evaluates 5-field UNIX standard, 6-field Quartz (with seconds), and 7-field Enterprise formats.  
   - Comprehensive support for steps (`*/15`), ranges (`9-17`), lists (`MON,WED,FRI`), and special wildcards (`?`, `L`, `W`, `#`).

   

2. **Fluent Natural Language Explanations**:  
     
   - Translates complex expressions into clear, unambiguous English ("Every 15 minutes, Monday through Friday, 09:00 AM to 05:00 PM UTC").

   

3. **Multi-Timezone Execution Schedule Visualizer**:  
     
   - Calculates the **Next 10 Execution Times** with millisecond accuracy.  
   - Displays relative countdowns ("in 14m 22s") and duration deltas between consecutive runs.  
   - Switchable across UTC, Local Browser Time, US/Eastern, UK/London, and UAE/Dubai (GST \+04:00).

   

4. **Distributed Lock & Concurrency Shield Synthesizer**:  
     
   - **Redis Distributed Mutex**: Atomic `SET key token NX EX` with token-matching Lua release script for Node.js and Python.  
   - **PostgreSQL Advisory Lock**: Non-blocking `pg_try_advisory_lock` stored procedure for relational databases.  
   - **Kubernetes CronJob Manifest**: Drop-in YAML configured with `concurrencyPolicy: Forbid` and resource bounds.  
   - **Golang (robfig/cron/v3)**: High-throughput Go worker with Redis mutex.

   

5. **Standalone Python CLI Tool (`cronpulse-cli.py`)**:  
     
   - Translate expressions directly in the terminal (`cronpulse-cli.py explain`).  
   - Query upcoming run dates (`cronpulse-cli.py next -n 5`).  
   - Automated syntax validation for CI/CD pipelines (returns exit code `0` on pass, `1` on error).

---

## Directory Structure

cronpulse-pro/

├── source-code/

│   ├── index.html                 \# Complete Studio Interface

│   ├── cronpulse.css              \# Dark Studio UI

│   ├── cron-engine.js             \# Universal Cron AST Parser & Evaluator

│   ├── lock-engine.js             \# Distributed Lock Synthesizer

│   ├── app.js                     \# UI Controller & Event Orchestrator

│   └── presets/

│       ├── financial-settlement.json  \# Daily Midnight Settlement Preset

│       ├── saas-metering.json         \# 15-Minute SaaS Metering Preset

│       └── high-freq-heartbeat.json   \# 30-Second Heartbeat Preset

├── build/

│   ├── cronpulse-cli.py           \# Standalone Python CLI Evaluator & Linter

│   └── cronpulse-pro-v1.0.0.zip   \# Distributable Product Archive

├── documentation/

│   ├── ARCHITECTURE.md            \# AST Grammar & Concurrency Specifications

│   ├── USER\_GUIDE.md               \# User & Integration Guide

│   └── MONETIZATION\_AND\_MARKETING\_PLAYBOOK.md \# Commercial Playbook

├── screenshots/

│   ├── schedule-preview.svg       \# Execution Schedule Timeline Preview

│   └── studio-preview.svg         \# Studio Interface Preview

└── README.md                      \# Master Repository Documentation

---

## Quick Start

### Running the Web Studio

Open `source-code/index.html` in any browser. No build tools or backend dependencies required.

### Running Automated Tests

python3 tests/test\_cronpulse.py

### Running the CLI Tool

python3 build/cronpulse-cli.py explain "\*/15 9-17 \* \* 1-5"

python3 build/cronpulse-cli.py next "0 0 \* \* \*" \-n 5

---

## Specifications & Standards

- **Formats**: POSIX Crontab (5-field), Quartz / Spring (6-field), Enterprise (7-field).  
- **Concurrency Protection**: Redis atomic Lua mutex & PostgreSQL advisory locks.  
- **Dependencies**: Zero external runtime dependencies.