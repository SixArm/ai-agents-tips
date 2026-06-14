# PostgreSQL sessions with JSONB

[Source](https://dev.to/polliog/i-replaced-redis-with-postgresql-and-its-faster-4942)

PostgreSQL:

```sql
CREATE TABLE sessions (
  id TEXT PRIMARY KEY,
  data JSONB NOT NULL,
  expires_at TIMESTAMPTZ NOT NULL
);

CREATE INDEX idx_sessions_expires ON sessions(expires_at);

-- Insert/Update
INSERT INTO sessions (id, data, expires_at)
VALUES ($1, $2, NOW() + INTERVAL '24 hours')
ON CONFLICT (id) DO UPDATE
  SET data = EXCLUDED.data,
      expires_at = EXCLUDED.expires_at;

-- Read
SELECT data FROM sessions
WHERE id = $1 AND expires_at > NOW();
```

Bonus: JSONB Operators

You can query inside the session:

```sql
-- Find all sessions for a specific user
SELECT * FROM sessions
WHERE data->>'userId' = '123';

-- Find sessions with specific role
SELECT * FROM sessions
WHERE data->'user'->>'role' = 'admin';
```

You can't do this with Redis!
