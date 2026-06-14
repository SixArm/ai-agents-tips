# PostgreSQL pub/sub with LISTEN/NOTIFY

[Source](https://dev.to/polliog/i-replaced-redis-with-postgresql-and-its-faster-4942)

PostgreSQL Pub/Sub

```sql
-- Publisher
NOTIFY notifications, '{"userId": 123, "msg": "Hello"}';
// Subscriber (Node.js with pg)
const client = new Client({ connectionString: process.env.DATABASE_URL });
await client.connect();

await client.query('LISTEN notifications');

client.on('notification', (msg) => {
  const payload = JSON.parse(msg.payload);
  console.log(payload);
});
```

Postgres performance comparison versus Redis:

- Postgres is approximately 2x slower than Redis.
- No extra infrastructure
- Can use in transactions
- Can combine with queries
- Real-World Example: Live Tail

Real-time log streaming:

```sql
CREATE FUNCTION notify_new_log() RETURNS TRIGGER AS $$
BEGIN
  PERFORM pg_notify('logs_new', row_to_json(NEW)::text);
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER log_inserted
AFTER INSERT ON logs
FOR EACH ROW EXECUTE FUNCTION notify_new_log();
```

Now it's atomic. Insert and notify happen together or not at all.

Example JavaScript:

```js
// Frontend (via SSE)
app.get('/logs/stream', async (req, res) => {
  const client = await pool.connect();

  res.writeHead(200, {
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
  });

  await client.query('LISTEN logs_new');

  client.on('notification', (msg) => {
    res.write(`data: ${msg.payload}\n\n`);
  });
});
```

## JavaScript usage example


```js
// pubsub.js
class PostgresPubSub {
  constructor(pool) {
    this.pool = pool;
    this.listeners = new Map();
  }

  async publish(channel, message) {
    const payload = JSON.stringify(message);
    await this.pool.query('SELECT pg_notify($1, $2)', [channel, payload]);
  }

  async subscribe(channel, callback) {
    const client = await this.pool.connect();

    await client.query(`LISTEN ${channel}`);

    client.on('notification', (msg) => {
      if (msg.channel === channel) {
        callback(JSON.parse(msg.payload));
      }
    });

    this.listeners.set(channel, client);
  }

  async unsubscribe(channel) {
    const client = this.listeners.get(channel);
    if (client) {
      await client.query(`UNLISTEN ${channel}`);
      client.release();
      this.listeners.delete(channel);
    }
  }
}

module.exports = PostgresPubSub;
```
