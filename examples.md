# Example Configuration for Schema Registry Transfer (SMT)
# Compatible with Kafka 3.8 and Confluent 7.0.1

## Basic Configuration

### Example 1: Minimal Configuration (No Authentication)
```json
{
  "name": "my-connector-with-schema-transfer",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
    "tasks.max": "1",
    "connection.url": "jdbc:mysql://localhost:3306/mydb",
    "mode": "incrementing",
    "incrementing.column.name": "id",
    "topic.prefix": "mysql-",
    
    "transforms": "SchemaTransfer",
    "transforms.SchemaTransfer.type": "cricket.jmoore.kafka.connect.transforms.SchemaRegistryTransfer",
    "transforms.SchemaTransfer.src.schema.registry.url": "http://source-schema-registry:8081",
    "transforms.SchemaTransfer.dest.schema.registry.url": "http://dest-schema-registry:8081"
  }
}
```

### Example 2: With Basic Authentication (User Info)
```json
{
  "name": "my-connector-with-auth",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "tasks.max": "1",
    "database.hostname": "postgres",
    "database.port": "5432",
    "database.user": "postgres",
    "database.password": "postgres",
    "database.dbname": "mydb",
    "database.server.name": "dbserver1",
    
    "transforms": "SchemaTransfer",
    "transforms.SchemaTransfer.type": "cricket.jmoore.kafka.connect.transforms.SchemaRegistryTransfer",
    
    "transforms.SchemaTransfer.src.schema.registry.url": "https://source-schema-registry:8081",
    "transforms.SchemaTransfer.src.basic.auth.credentials.source": "USER_INFO",
    "transforms.SchemaTransfer.src.schema.registry.basic.auth.user.info": "source-user:source-password",
    
    "transforms.SchemaTransfer.dest.schema.registry.url": "https://dest-schema-registry:8081",
    "transforms.SchemaTransfer.dest.basic.auth.credentials.source": "USER_INFO",
    "transforms.SchemaTransfer.dest.schema.registry.basic.auth.user.info": "dest-user:dest-password"
  }
}
```

### Example 3: With SSL/TLS
```json
{
  "name": "my-connector-with-ssl",
  "config": {
    "connector.class": "org.apache.kafka.connect.file.FileStreamSourceConnector",
    "tasks.max": "1",
    "file": "/tmp/test.txt",
    "topic": "connect-test",
    
    "transforms": "SchemaTransfer",
    "transforms.SchemaTransfer.type": "cricket.jmoore.kafka.connect.transforms.SchemaRegistryTransfer",
    
    "transforms.SchemaTransfer.src.schema.registry.url": "https://source-schema-registry:8081",
    "transforms.SchemaTransfer.src.schema.registry.ssl.truststore.location": "/etc/kafka/secrets/kafka.client.truststore.jks",
    "transforms.SchemaTransfer.src.schema.registry.ssl.truststore.password": "test1234",
    "transforms.SchemaTransfer.src.schema.registry.ssl.keystore.location": "/etc/kafka/secrets/kafka.client.keystore.jks",
    "transforms.SchemaTransfer.src.schema.registry.ssl.keystore.password": "test1234",
    "transforms.SchemaTransfer.src.schema.registry.ssl.key.password": "test1234",
    
    "transforms.SchemaTransfer.dest.schema.registry.url": "https://dest-schema-registry:8081",
    "transforms.SchemaTransfer.dest.schema.registry.ssl.truststore.location": "/etc/kafka/secrets/kafka.client.truststore.jks",
    "transforms.SchemaTransfer.dest.schema.registry.ssl.truststore.password": "test1234",
    "transforms.SchemaTransfer.dest.schema.registry.ssl.keystore.location": "/etc/kafka/secrets/kafka.client.keystore.jks",
    "transforms.SchemaTransfer.dest.schema.registry.ssl.keystore.password": "test1234",
    "transforms.SchemaTransfer.dest.schema.registry.ssl.key.password": "test1234"
  }
}
```

### Example 4: Complete Configuration with All Parameters
```json
{
  "name": "my-connector-full-config",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
    "tasks.max": "1",
    "connection.url": "jdbc:postgresql://localhost:5432/mydb",
    "mode": "timestamp",
    "timestamp.column.name": "updated_at",
    "topic.prefix": "pg-",
    
    "transforms": "SchemaTransfer",
    "transforms.SchemaTransfer.type": "cricket.jmoore.kafka.connect.transforms.SchemaRegistryTransfer",
    
    "transforms.SchemaTransfer.src.schema.registry.url": "https://source-schema-registry:8081,https://source-schema-registry-backup:8081",
    "transforms.SchemaTransfer.src.basic.auth.credentials.source": "USER_INFO",
    "transforms.SchemaTransfer.src.schema.registry.basic.auth.user.info": "admin:secret123",
    
    "transforms.SchemaTransfer.dest.schema.registry.url": "https://dest-schema-registry:8081,https://dest-schema-registry-backup:8081",
    "transforms.SchemaTransfer.dest.basic.auth.credentials.source": "USER_INFO",
    "transforms.SchemaTransfer.dest.schema.registry.basic.auth.user.info": "admin:secret456",
    
    "transforms.SchemaTransfer.schema.capacity": "5000",
    "transforms.SchemaTransfer.transfer.message.keys": "true",
    "transforms.SchemaTransfer.include.message.headers": "true"
  }
}
```

### Example 5: Multi-Transform Chain
```json
{
  "name": "my-connector-multi-transform",
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "tasks.max": "1",
    "database.hostname": "mysql",
    "database.port": "3306",
    "database.user": "debezium",
    "database.password": "dbz",
    "database.server.id": "184054",
    "database.server.name": "dbserver1",
    "database.include.list": "inventory",
    
    "transforms": "unwrap,SchemaTransfer,route",
    
    "transforms.unwrap.type": "io.debezium.transforms.ExtractNewRecordState",
    "transforms.unwrap.drop.tombstones": "false",
    
    "transforms.SchemaTransfer.type": "cricket.jmoore.kafka.connect.transforms.SchemaRegistryTransfer",
    "transforms.SchemaTransfer.src.schema.registry.url": "http://source-sr:8081",
    "transforms.SchemaTransfer.dest.schema.registry.url": "http://dest-sr:8081",
    "transforms.SchemaTransfer.schema.capacity": "1000",
    
    "transforms.route.type": "org.apache.kafka.connect.transforms.RegexRouter",
    "transforms.route.regex": "([^.]+)\\.([^.]+)\\.([^.]+)",
    "transforms.route.replacement": "$3"
  }
}
```

## Configuration Parameters

### Required Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
`src.schema.registry.url` | List | URL(s) of the source Schema Registry. This can be a comma-separated list for HA. |
`dest.schema.registry.url` | List | URL(s) of the destination Schema Registry. This can be a comma-separated list for HA. |

### Optional Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
`src.basic.auth.credentials.source` | String | "" | Credential source for basic authentication in the source. Values: `USER_INFO`, `URL`, `SASL_INHERIT` |
`src.schema.registry.basic.auth.user.info` | Password | "" | Username and password in `username:password` format for the source registry |
`dest.basic.auth.credentials.source` | String | "" | Credential source for basic authentication in the destination. Values: `USER_INFO`, `URL`, `SASL_INHERIT` |
`dest.schema.registry.basic.auth.user.info` | Password | "" | Username and password in `username:password` format for destination registry |
`schema.capacity` | Int | 100 | Maximum size of the schema cache. Adjust according to the volume of different schemas. |
`transfer.message.keys` | Boolean | true | If `true`, also transfers the message key schemas. |
`include.message.headers` | Boolean | true | If `true`, preserves the Kafka Connect headers in transformed messages. |

### SSL/TLS Parameters (Optional)

**For Source Registry:**
| Parameter | Description |
|-----------|-------------|
`src.schema.registry.ssl.truststore.location` | Path to the truststore for validating server certificates |
`src.schema.registry.ssl.truststore.password` | Truststore Password |
`src.schema.registry.ssl.keystore.location` | Path to the keystore for client authentication |
`src.schema.registry.ssl.keystore.password` | Keystore Password |
`src.schema.registry.ssl.key.password` | Private Key Password |

**For Destination Registry:**
| Parameter | Description |
|-----------|-------------|
`dest.schema.registry.ssl.truststore.location` | Path to the truststore for server certificate validation |
`dest.schema.registry.ssl.truststore.password` | Truststore Password |
`dest.schema.registry.ssl.keystore.location` | Path to the keystore for client authentication |
`dest.schema.registry.ssl.keystore.password` | Keystore password |
`dest.schema.registry.ssl.key.password` | Private key password |

## Common Use Cases

### Use Case 1: Replication Between Clusters
Replicate data from an on-premises cluster to the cloud while keeping schemas synchronized.

```json
{
  "transforms.SchemaTransfer.src.schema.registry.url": "http://onprem-registry:8081",
  "transforms.SchemaTransfer.dest.schema.registry.url": "https://cloud-registry.cloud.provider.com:443",
  "transforms.SchemaTransfer.dest.basic.auth.credentials.source": "USER_INFO",
  "transforms.SchemaTransfer.dest.schema.registry.basic.auth.user.info": "${file:/secrets/sr-creds.properties:cloud.sr.auth}"
}
```

### Case 2: Schema Registry Migration
Migrating from an old Schema Registry to a new one without downtime.

```json
{
  "transforms.SchemaTransfer.src.schema.registry.url": "http://old-registry:8081",
  "transforms.SchemaTransfer.dest.schema.registry.url": "http://new-registry:8081",
  "transforms.SchemaTransfer.schema.capacity": "10000"
}
```

### Case 3: Multi-Tenant with Separate Registries
Moving data between different tenants that use separate registries.

```json
{
  "transforms.SchemaTransfer.src.schema.registry.url": "http://tenant-a-registry:8081",
  "transforms.SchemaTransfer.src.basic.auth.credentials.source": "USER_INFO",
  "transforms.SchemaTransfer.src.schema.registry.basic.auth.user.info": "tenant-a:password-a",
  
  "transforms.SchemaTransfer.dest.schema.registry.url": "http://tenant-b-registry:8081",
  "transforms.SchemaTransfer.dest.basic.auth.credentials.source": "USER_INFO",
  "transforms.SchemaTransfer.dest.schema.registry.basic.auth.user.info": "tenant-b:password-b"
}
```

### Case 4: DR (Disaster Recovery)
Maintain a synchronized backup registry for disaster recovery.

```json
{
  "transforms.SchemaTransfer.src.schema.registry.url": "http://primary-registry:8081,http://primary-registry-2:8081",
  "transforms.SchemaTransfer.dest.schema.registry.url": "http://dr-registry:8081,http://dr-registry-2:8081",
  "transforms.SchemaTransfer.schema.capacity": "5000"
}
```

## Performance Tips

### 1. Adjusting Cache Size
```json
// For few schemas (<100 different schemas)
"transforms.SchemaTransfer.schema.capacity": "100"

// For many schemas (100-1000)
"transforms.SchemaTransfer.schema.capacity": "1000"

// For high volume (>1000 schemas)
"transforms.SchemaTransfer.schema.capacity": "5000"
```

### 2. Key Transfer
If your messages don't have keys with Avro schemas, disable key transfer:
```json
"transforms.SchemaTransfer.transfer.message.keys": "false"
```

### 3. Headers
If you don't need to preserve Connect headers, disable them:
```json
"transforms.SchemaTransfer.include.message.headers": "false"
```

## Troubleshooting

### Enable Debug Logging
En `connect-log4j.properties`:
```properties
log4j.logger.cricket.jmoore.kafka.connect.transforms=DEBUG
log4j.logger.io.confluent.kafka.schemaregistry=DEBUG
```

### Verify Configuration
```bash
# Ver configuración activa del conector
curl http://localhost:8083/connectors/my-connector/config | jq

# Ver status
curl http://localhost:8083/connectors/my-connector/status | jq
```

### JMX Monitoring
Useful metrics for monitoring:
- `kafka.connect:type=task-metrics,connector=*,task=*`
- `kafka.connect:type=connector-task-metrics,connector=*,task=*`

## Environment Variables and Secrets

### Using Environment Variables
```json
{
  "transforms.SchemaTransfer.src.schema.registry.url": "${env:SOURCE_SR_URL}",
  "transforms.SchemaTransfer.dest.schema.registry.url": "${env:DEST_SR_URL}"
}
```

### Using Secrets Files
```json
{
  "transforms.SchemaTransfer.src.schema.registry.basic.auth.user.info": "${file:/secrets/creds.properties:src.auth}",
  "transforms.SchemaTransfer.dest.schema.registry.basic.auth.user.info": "${file:/secrets/creds.properties:dest.auth}"
}
```

File `/secrets/creds.properties`:
```properties
src.auth=source-user:source-password
dest.auth=dest-user:dest-password
```
