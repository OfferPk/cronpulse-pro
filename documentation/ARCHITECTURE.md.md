# CronPulse Pro — Architectural Specification & Distributed Lock Blueprint

## 1\. Executive Architectural Overview

CronPulse Pro resolves the two fundamental challenges of cron-based task orchestration in modern cloud environments:

1. **Cron Syntax Incompatibility & Misinterpretation**: Different cloud runtimes and frameworks interpret cron expressions with subtle, dangerous discrepancies (e.g. 0-indexed vs 1-indexed days of week, 5-field UNIX vs 6-field Quartz with seconds, and AWS EventBridge mutual exclusion rules between Day-of-month and Day-of-week). A single misunderstood field leads to critical tasks executing at wrong hours or multiplying exponentially.  
2. **The Multi-Replica Overlapping Execution Hazard**: Modern web applications deploy with horizontal auto-scaling (multiple Kubernetes Pods, ECS tasks, or serverless workers). When scheduled background workers are embedded in application code without a distributed mutex, all active replicas fire the cron task simultaneously, triggering duplicate customer billings, duplicate database migrations, and database connection exhaustion.

---

## 2\. AST Grammar & Parsing Specifications

CronPulse Pro parses cron strings into an Abstract Syntax Tree (AST) supporting three dominant formats:

### 2.1 Supported Standards

- **UNIX Standard 5-Field**: `[Minute] [Hour] [Day-of-Month] [Month] [Day-of-Week]`  
- **Quartz / Spring 6-Field**: `[Seconds] [Minute] [Hour] [Day-of-Month] [Month] [Day-of-Week]`  
- **Enterprise 7-Field**: `[Seconds] [Minute] [Hour] [Day-of-Month] [Month] [Day-of-Week] [Year]`

### 2.2 Field Value Bounds

| Field | Standard Bounds | Wildcard & Operators |
| :---- | :---- | :---- |
| **Seconds** | 0–59 | `*`, `,`, `-`, `/` |
| **Minutes** | 0–59 | `*`, `,`, `-`, `/` |
| **Hours** | 0–23 | `*`, `,`, `-`, `/` |
| **Day of Month** | 1–31 | `*`, `,`, `-`, `/`, `?`, `L`, `W` |
| **Month** | 1–12 or JAN–DEC | `*`, `,`, `-`, `/` |
| **Day of Week** | 0–7 (0 & 7 \= SUN) or SUN–SAT | `*`, `,`, `-`, `/`, `?`, `L`, `#` |
| **Year** | 2020–2099 | `*`, `,`, `-`, `/` |

---

## 3\. Distributed Mutex & Concurrency Guarantees

To ensure exactly-once execution across \$K\$ replicas, CronPulse Pro synthesizes two proven distributed lock architectures:

### 3.1 Redis Atomic Mutex (`SET NX EX` \+ Token-Safe Release)

- **Acquire Phase**:  
    
  SET cron:lock:\<job\_id\> \<random\_uuid\> NX EX \<ttl\_seconds\>  
    
  - `NX`: Ensures only the first replica acquires the key.  
  - `EX`: Automatically evicts the lock if the holding worker crashes, preventing permanent deadlock.  
- **Release Phase**: Releasing a lock with a simple `DEL` is dangerous—if a job runs longer than `TTL`, a second worker acquires the lock, and the first worker could accidentally delete the second worker's lock. CronPulse Pro synthesizes an atomic Lua release script:  
    
  if redis.call("get", KEYS\[1\]) \== ARGV\[1\] then  
    
      return redis.call("del", KEYS\[1\])  
    
  else  
    
      return 0  
    
  end

### 3.2 PostgreSQL Advisory Locks

For applications utilizing relational databases without Redis, CronPulse Pro synthesizes PostgreSQL transaction-level advisory locks:

SELECT pg\_try\_advisory\_lock(987654);

- Non-blocking: returns `TRUE` if lock was successfully obtained, `FALSE` if already held by another database worker.  
- Automatically cleaned up if the connection terminates abruptly.