/\*\*  
 \* CronPulse Pro \- Distributed Lock & Worker Wrapper Synthesizer  
 \* Prevents duplicate job execution across multi-replica microservices  
 \*/

class LockEngine {  
  /\*\*  
   \* Generates Node.js (node-cron / ioredis) distributed lock worker.  
   \*/  
  static generateNodeRedisLock(cronExpr, jobName \= 'billing\_settlement') {  
    return \`// Node.js Distributed Cron Worker with Atomic Redis Lock  
// Prevents duplicate execution across multiple Kubernetes Pods / Replicas  
import cron from 'node-cron';  
import Redis from 'ioredis';  
import { randomUUID } from 'crypto';

const redis \= new Redis(process.env.REDIS\_URL || 'redis://localhost:6379');  
const CRON\_EXPRESSION \= '\${cronExpr}';  
const LOCK\_KEY \= 'cron:lock:\${jobName}';  
const LOCK\_TTL\_SECONDS \= 300; // 5-minute auto-release

// Atomic Lua script to release lock only if the random token matches  
const RELEASE\_LOCK\_LUA \= \\\`  
if redis.call("get", KEYS\[1\]) \== ARGV\[1\] then  
    return redis.call("del", KEYS\[1\])  
else  
    return 0  
end  
\\\`;

cron.schedule(CRON\_EXPRESSION, async () \=\> {  
  const lockToken \= randomUUID();  
  console.log(\\\`\[\\\${new Date().toISOString()}\] Attempting to acquire lock for \\\${LOCK\_KEY}...\\\`);

  // Acquire lock atomically with SET NX EX  
  const acquired \= await redis.set(LOCK\_KEY, lockToken, 'NX', 'EX', LOCK\_TTL\_SECONDS);

  if (\!acquired) {  
    console.log(\\\`\[\\\${new Date().toISOString()}\] Lock already held by another replica. Skipping execution.\\\`);  
    return;  
  }

  try {  
    console.log(\\\`\[\\\${new Date().toISOString()}\] Lock acquired. Starting job execution...\\\`);  
      
    // \--- YOUR WORKER LOGIC HERE \---  
    await executeJob();

    console.log(\\\`\[\\\${new Date().toISOString()}\] Job completed successfully.\\\`);  
  } catch (err) {  
    console.error(\\\`\[\\\${new Date().toISOString()}\] Job execution failed:\\\`, err);  
  } finally {  
    // Release lock safely  
    await redis.eval(RELEASE\_LOCK\_LUA, 1, LOCK\_KEY, lockToken);  
    console.log(\\\`\[\\\${new Date().toISOString()}\] Lock released.\\\`);  
  }  
});

async function executeJob() {  
  // Simulate task workload  
  await new Promise(resolve \=\> setTimeout(resolve, 2000));  
}  
\`;  
  }

  /\*\*  
   \* Generates Python (APScheduler / redis-py) distributed lock worker.  
   \*/  
  static generatePythonRedisLock(cronExpr, jobName \= 'billing\_settlement') {  
    return \`\# Python Distributed Cron Worker with Atomic Redis Lock  
import time  
import uuid  
import redis  
from apscheduler.schedulers.blocking import BlockingScheduler

r \= redis.Redis.from\_url("redis://localhost:6379/0", decode\_responses=True)  
LOCK\_KEY \= "cron:lock:\${jobName}"  
LOCK\_TTL \= 300 \# seconds

RELEASE\_SCRIPT \= """  
if redis.call("get", KEYS\[1\]) \== ARGV\[1\] then  
    return redis.call("del", KEYS\[1\])  
else  
    return 0  
end  
"""  
release\_lock \= r.register\_script(RELEASE\_SCRIPT)

def run\_guarded\_job():  
    lock\_token \= str(uuid.uuid4())  
    print(f"\[{time.strftime('%X')}\] Attempting to acquire lock for {LOCK\_KEY}...")

    \# Acquire lock atomically  
    acquired \= r.set(LOCK\_KEY, lock\_token, nx=True, ex=LOCK\_TTL)  
    if not acquired:  
        print(f"\[{time.strftime('%X')}\] Lock held by another worker instance. Bailing out.")  
        return

    try:  
        print(f"\[{time.strftime('%X')}\] Lock acquired. Executing task...")  
        \# \--- YOUR TASK CODE HERE \---  
        time.sleep(2)  
        print(f"\[{time.strftime('%X')}\] Task completed successfully.")  
    except Exception as e:  
        print(f"\[{time.strftime('%X')}\] Task encountered error: {e}")  
    finally:  
        release\_lock(keys=\[LOCK\_KEY\], args=\[lock\_token\])  
        print(f"\[{time.strftime('%X')}\] Lock released.")

if \_\_name\_\_ \== "\_\_main\_\_":  
    scheduler \= BlockingScheduler()  
    \# Parsing cron: '\${cronExpr}'  
    scheduler.add\_job(run\_guarded\_job, 'cron', minute='\${cronExpr.split(' ')\[0\]}', hour='\${cronExpr.split(' ')\[1\] if len(cronExpr.split(' ')) \> 1 else '\*'}')  
    print("Scheduler initialized. Waiting for trigger...")  
    scheduler.start()  
\`;  
  }

  /\*\*  
   \* Generates PostgreSQL Advisory Lock wrapper.  
   \*/  
  static generatePostgreSqlLock(cronExpr, lockId \= 987654\) {  
    return \`-- PostgreSQL Transaction-Level Advisory Lock Worker  
\-- Guarantees single-worker execution across multiple database connections

CREATE OR REPLACE FUNCTION run\_scheduled\_cron\_\${lockId}()   
RETURNS void AS \$\$  
DECLARE  
    lock\_obtained boolean;  
BEGIN  
    \-- pg\_try\_advisory\_lock returns immediately with TRUE if lock was acquired, FALSE if busy  
    SELECT pg\_try\_advisory\_lock(\${lockId}) INTO lock\_obtained;

    IF NOT lock\_obtained THEN  
        RAISE NOTICE 'Cron job already executing on another connection. Skipping.';  
        RETURN;  
    END IF;

    BEGIN  
        RAISE NOTICE 'Advisory lock % acquired. Executing business logic...', \${lockId};

        \-- \--- YOUR SQL BATCH / STORED PROCEDURE HERE \---  
        PERFORM pg\_sleep(2);

        RAISE NOTICE 'Scheduled batch execution finished.';  
    EXCEPTION WHEN OTHERS THEN  
        RAISE WARNING 'Batch failed: %', SQLERRM;  
    END;

    \-- Release advisory lock  
    PERFORM pg\_advisory\_unlock(\${lockId});  
END;  
\$\$ LANGUAGE plpgsql;

\-- Execute function  
SELECT run\_scheduled\_cron\_\${lockId}();  
\`;  
  }

  /\*\*  
   \* Generates Kubernetes CronJob YAML with concurrencyPolicy: Forbid.  
   \*/  
  static generateK8sCronJobYaml(cronExpr, jobName \= 'cronpulse-worker') {  
    return \`\# Kubernetes CronJob Manifest with Concurrency Shield  
apiVersion: batch/v1  
kind: CronJob  
metadata:  
  name: \${jobName}  
  labels:  
    app.kubernetes.io/name: \${jobName}  
    app.kubernetes.io/managed-by: cronpulse-pro  
spec:  
  schedule: "\${cronExpr}"  
  \# CRITICAL: Forbid prevents overlapping pods if previous run hasn't finished  
  concurrencyPolicy: Forbid  
  successfulJobsHistoryLimit: 3  
  failedJobsHistoryLimit: 5  
  jobTemplate:  
    spec:  
      backoffLimit: 2  
      template:  
        spec:  
          restartPolicy: OnFailure  
          containers:  
          \- name: worker  
            image: ghcr.io/your-org/\${jobName}:latest  
            resources:  
              requests:  
                cpu: "100m"  
                memory: "128Mi"  
              limits:  
                cpu: "500m"  
                memory: "256Mi"  
            env:  
            \- name: NODE\_ENV  
              value: "production"  
\`;  
  }

  /\*\*  
   \* Generates Golang (robfig/cron/v3) wrapper.  
   \*/  
  static generateGoLock(cronExpr, jobName \= 'cron\_task') {  
    return \`// Golang robfig/cron/v3 with Redis Distributed Mutex  
package main

import (  
	"context"  
	"fmt"  
	"time"

	"github.com/google/uuid"  
	"github.com/redis/go-redis/v9"  
	"github.com/robfig/cron/v3"  
)

var ctx \= context.Background()

func main() {  
	rdb := redis.NewClient(\&redis.Options{Addr: "localhost:6379"})  
	c := cron.New()

	c.AddFunc("\${cronExpr}", func() {  
		lockKey := "cron:lock:\${jobName}"  
		token := uuid.New().String()

		// Attempt atomic lock  
		ok, err := rdb.SetNX(ctx, lockKey, token, 5\*time.Minute).Result()  
		if err \!= nil || \!ok {  
			fmt.Println("Job already running on another node. Skipping.")  
			return  
		}

		defer func() {  
			// Release lock  
			rdb.Eval(ctx, \\\`  
				if redis.call("get", KEYS\[1\]) \== ARGV\[1\] then  
					return redis.call("del", KEYS\[1\])  
				else  
					return 0  
				end  
			\\\`, \[\]string{lockKey}, token)  
		}()

		fmt.Println("Lock acquired. Executing task...")  
		time.Sleep(2 \* time.Second)  
		fmt.Println("Task completed.")  
	})

	c.Start()  
	select {}  
}  
\`;  
  }  
}

if (typeof module \!== 'undefined' && module.exports) {  
  module.exports \= LockEngine;  
}  
