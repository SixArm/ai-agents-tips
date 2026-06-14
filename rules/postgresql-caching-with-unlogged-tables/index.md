# PostgreSQL caching with UNLOGGED tables

[Source](https://dev.to/polliog/i-replaced-redis-with-postgresql-and-its-faster-4942)

PostgreSQL:

```sql
CREATE UNLOGGED TABLE cache (
  key TEXT PRIMARY KEY,
  value JSONB NOT NULL,
  expires_at TIMESTAMPTZ NOT NULL
);

CREATE INDEX idx_cache_expires ON cache(expires_at);
```

Insert:

```sql
INSERT INTO cache (key, value, expires_at)
VALUES ($1, $2, NOW() + INTERVAL '1 hour')
ON CONFLICT (key) DO UPDATE
  SET value = EXCLUDED.value,
      expires_at = EXCLUDED.expires_at;
```

Read:

```sql
SELECT value FROM cache
WHERE key = $1 AND expires_at > NOW();
Cleanup (run periodically):

DELETE FROM cache WHERE expires_at < NOW();
```

Why use UNLOGGED tables?

- Skip the Write-Ahead Log (WAL)
- Much faster writes
- Don't survive crashes (perfect for cache!)

## JavaScript usage example

```js
// cache.js
class PostgresCache {
  constructor(pool) {
    this.pool = pool;
  }

  async get(key) {
    const result = await this.pool.query(
      'SELECT value FROM cache WHERE key = $1 AND expires_at > NOW()',
      [key]
    );
    return result.rows[0]?.value;
  }

  async set(key, value, ttlSeconds = 3600) {
    await this.pool.query(
      `INSERT INTO cache (key, value, expires_at)
       VALUES ($1, $2, NOW() + INTERVAL '${ttlSeconds} seconds')
       ON CONFLICT (key) DO UPDATE
         SET value = EXCLUDED.value,
             expires_at = EXCLUDED.expires_at`,
      [key, value]
    );
  }

  async delete(key) {
    await this.pool.query('DELETE FROM cache WHERE key = $1', [key]);
  }

  async cleanup() {
    await this.pool.query('DELETE FROM cache WHERE expires_at < NOW()');
  }
}

module.exports = PostgresCache;
```
