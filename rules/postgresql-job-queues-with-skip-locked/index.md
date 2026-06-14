# PostgreSQL job queues with SKIP LOCKED

[Source](https://dev.to/polliog/i-replaced-redis-with-postgresql-and-its-faster-4942)

PostgreSQL:

```sql
CREATE TABLE jobs (
  id BIGSERIAL PRIMARY KEY,
  queue TEXT NOT NULL,
  payload JSONB NOT NULL,
  attempts INT DEFAULT 0,
  max_attempts INT DEFAULT 3,
  scheduled_at TIMESTAMPTZ DEFAULT NOW(),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_jobs_queue ON jobs(queue, scheduled_at) 
WHERE attempts < max_attempts;
```

Enqueue:

```sql
INSERT INTO jobs (queue, payload)
VALUES ('send-email', '{"to": "user@example.com", "subject": "Hi"}');
Worker (dequeue):

WITH next_job AS (
  SELECT id FROM jobs
  WHERE queue = $1
    AND attempts < max_attempts
    AND scheduled_at <= NOW()
  ORDER BY scheduled_at
  LIMIT 1
  FOR UPDATE SKIP LOCKED
)
UPDATE jobs
SET attempts = attempts + 1
FROM next_job
WHERE jobs.id = next_job.id
RETURNING *;
```

The magic: FOR UPDATE SKIP LOCKED

This makes PostgreSQL a lock-free queue:

- Multiple workers can pull jobs concurrently
- No job is processed twice
- If a worker crashes, job becomes available again
- Benchmark: PostgreSQL is 3x time versus Redis

## JavaScript usage example

```js
// queue.js
class PostgresQueue {
  constructor(pool) {
    this.pool = pool;
  }

  async enqueue(queue, payload, scheduledAt = new Date()) {
    await this.pool.query(
      'INSERT INTO jobs (queue, payload, scheduled_at) VALUES ($1, $2, $3)',
      [queue, payload, scheduledAt]
    );
  }

  async dequeue(queue) {
    const result = await this.pool.query(
      `WITH next_job AS (
        SELECT id FROM jobs
        WHERE queue = $1
          AND attempts < max_attempts
          AND scheduled_at <= NOW()
        ORDER BY scheduled_at
        LIMIT 1
        FOR UPDATE SKIP LOCKED
      )
      UPDATE jobs
      SET attempts = attempts + 1
      FROM next_job
      WHERE jobs.id = next_job.id
      RETURNING jobs.*`,
      [queue]
    );

    return result.rows[0];
  }

  async complete(jobId) {
    await this.pool.query('DELETE FROM jobs WHERE id = $1', [jobId]);
  }

  async fail(jobId, error) {
    await this.pool.query(
      `UPDATE jobs
       SET attempts = max_attempts,
           payload = payload || jsonb_build_object('error', $2)
       WHERE id = $1`,
      [jobId, error.message]
    );
  }
}

module.exports = PostgresQueue;
```