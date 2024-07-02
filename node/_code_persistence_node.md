```
# Database connection
descriptor: pip-services:connection:mongodb:default:3.0
connection:
  host: ${{MONGO_HOST}}{$unless MONGO_HOST}localhost{$unless}
  port: ${{MONGO_PORT}}{$unless MONGO_PORT}27017{$unless}
credentials:
  host: ${{MONGO_USER}}{$unless MONGO_USER}mongo{$unless}
  port: ${{MONGO_PASS}}{$unless MONGO_PASS}pwd123{$unless}
options:
  max_pool_size: 10
  keep_alive: true

```

```
# Persistence with default connection
descriptor: myservice:mypersistence:mongodb:persist1:1.0

# Persistence linked to specific connection
descriptor: myservice:mypersistence:mongodb:persist2:1.0
references:
  connection: pip-services:connection:mongodb:conn1:3.0

```

```

```
