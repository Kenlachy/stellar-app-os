# Multi-Region PostgreSQL Read-Replica Query Balancer

## Overview

The replica balancer provides intelligent routing of database queries across multiple PostgreSQL read-replica instances. It automatically routes read queries to the nearest healthy replica based on region, priority, and latency, while ensuring all write queries go to the primary database.

## Features

- **Automatic Health Checks**: Periodic health checks monitor all database instances
- **Intelligent Routing**: Routes queries based on region preference, priority, and latency
- **Automatic Failover**: Falls back to primary if all replicas are unhealthy
- **Retry Logic**: Configurable retry mechanism for failed queries
- **Connection Pooling**: Efficient connection pooling for all instances
- **Type-Safe**: Full TypeScript support with strict type definitions

## Configuration

### Environment Variables

```bash
# Primary Database
DATABASE_URL=postgres://user:pass@primary-host:5432/dbname
DATABASE_PRIMARY_REGION=us-east-1

# Read Replicas (numbered sequentially)
DATABASE_REPLICA_0_URL=postgres://user:pass@replica-0-host:5432/dbname
DATABASE_REPLICA_0_REGION=us-west-2
DATABASE_REPLICA_0_Priority=10

DATABASE_REPLICA_1_URL=postgres://user:pass@replica-1-host:5432/dbname
DATABASE_REPLICA_1_REGION=eu-west-1
DATABASE_REPLICA_1_Priority=20

# Additional replicas follow the same pattern...
```

### Configuration Options

| Option | Default | Description |
|--------|---------|-------------|
| `healthCheckInterval` | 30000ms | Interval between health checks |
| `healthCheckTimeout` | 5000ms | Timeout for health check queries |
| `maxRetries` | 3 | Maximum retry attempts for failed queries |
| `retryDelay` | 100ms | Delay between retry attempts |

## Usage

### Basic Usage

```typescript
import { query, write } from '@/lib/db/replica-balancer';

// Read query - automatically routed to nearest healthy replica
const result = await query('SELECT * FROM users WHERE id = $1', [userId]);

// Write query - always routed to primary
await write('INSERT INTO users (name, email) VALUES ($1, $2)', [name, email]);
```

### Advanced Usage

```typescript
import { getReplicaBalancer } from '@/lib/db/replica-balancer';

const balancer = getReplicaBalancer();

// Force query to primary
await balancer.query('SELECT * FROM sensitive_data', [], { forcePrimary: true });

// Prefer specific region
await balancer.query('SELECT * FROM users', [], { preferredRegion: 'eu-west-1' });

// Allow fallback to primary if replicas fail
await balancer.query('SELECT * FROM analytics', [], { allowFallback: true });

// Check health status
const healthStatus = balancer.getHealthStatus();
console.log(healthStatus);

// Get specific pool
const pool = balancer.getPool('replica-0');
```

## Health Checks

The balancer performs periodic health checks on all database instances:

- **Check Query**: `SELECT 1` - lightweight query to verify connectivity
- **Metrics**: Records latency and health status for each instance
- **Automatic Recovery**: Instances are marked healthy when checks pass
- **Error Logging**: Failed health checks are logged with error details

## Query Routing Logic

1. **Force Primary**: If `forcePrimary` option is set, use primary
2. **Healthy Replicas**: Filter to only healthy replica instances
3. **Region Preference**: Sort by preferred region if specified
4. **Priority**: Sort by priority (lower number = higher priority)
5. **Latency**: Sort by latency (lower latency preferred)
6. **Fallback**: If no healthy replicas, use primary (if `allowFallback`)

## Error Handling

- **Retry Logic**: Failed queries are retried up to `maxRetries` times
- **Exponential Backoff**: Configurable delay between retries
- **Pool Errors**: Connection pool errors are logged and instances marked unhealthy
- **Graceful Degradation**: Falls back to primary if replicas fail

## Monitoring

### Health Status

```typescript
const balancer = getReplicaBalancer();
const status = balancer.getHealthStatus();

status.forEach(instance => {
  console.log(`${instance.instanceName}:`);
  console.log(`  Healthy: ${instance.isHealthy}`);
  console.log(`  Latency: ${instance.latencyMs}ms`);
  console.log(`  Last Checked: ${instance.checkedAt}`);
  if (instance.error) {
    console.log(`  Error: ${instance.error}`);
  }
});
```

### Pool Statistics

```typescript
const pool = balancer.getPool('replica-0');
console.log('Total connections:', pool.totalCount);
console.log('Idle connections:', pool.idleCount);
console.log('Waiting clients:', pool.waitingCount);
```

## Best Practices

1. **Use Read Replicas for Read-Heavy Workloads**: Analytics, reporting, dashboard queries
2. **Use Primary for Write Operations**: INSERT, UPDATE, DELETE, and critical reads
3. **Configure Appropriate Priorities**: Lower priority for backup replicas
4. **Monitor Health Status**: Set up alerts for unhealthy replicas
5. **Test Failover**: Regularly test failover to primary
6. **Optimize Connection Pools**: Adjust pool size based on workload

## Testing

Run tests with Vitest:

```bash
pnpm test lib/db/replica-balancer.test.ts
```

## Migration from Single Pool

Replace existing pool usage:

```typescript
// Before
import { getPool } from '@/lib/db/client';
const pool = getPool();
const result = await pool.query('SELECT * FROM users');

// After
import { query, write } from '@/lib/db/replica-balancer';
const result = await query('SELECT * FROM users'); // Read
await write('UPDATE users SET name = $1', [name]); // Write
```

## Troubleshooting

### All Queries Going to Primary

- Check replica health status: `balancer.getHealthStatus()`
- Verify replica connection strings are correct
- Check network connectivity to replicas
- Review health check logs for errors

### HighLatency

- Check replica region configuration
- Verify network routing between regions
- Consider adding replicas in closer regions
- Review health check timeout settings

### Connection Pool Exhaustion

- Increase pool size in configuration
- Check for connection leaks in application code
- Monitor pool statistics regularly
- Consider connection timeout settings

## Security Considerations

- Use SSL/TLS for database connections
- Store connection strings in environment variables
- Rotate database credentials regularly
- Limit replica access to read-only operations
- Monitor for unauthorized access attempts

## Performance Optimization

- **Connection Pooling**: Reuse connections efficiently
- **Region-Based Routing**: Minimize cross-region latency
- **Health Check Frequency**: Balance between responsiveness and overhead
- **Retry Configuration**: Adjust based on network conditions
- **Query Optimization**: Ensure queries are efficient regardless of instance

## License

MIT
