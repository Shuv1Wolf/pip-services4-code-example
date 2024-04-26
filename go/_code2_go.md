config.yml
```
---
# Container descriptor
- descriptor: "pip-services:context-info:default:default:1.0"
  name: "hello-world" 
  description: "HelloWorld microservice"

# Console logger
- descriptor: "pip-services:logger:console:default:1.0" 
  level: "trace"

# Performance counter that post values to log
- descriptor: "pip-services:counters:log:default:1.0"

# Service
- descriptor: "hello-world:service:default:default:1.0"
  default_name: "World"

# Shared HTTP Endpoint
- descriptor: "pip-services:endpoint:http:default:1.0" 
  connection:
    protocol: http
    host: 0.0.0.0
    port: 8080

# HTTP controller V1
- descriptor: "hello-world:controller:http:default:1.0"

# Hearbeat controller
- descriptor: "pip-services:heartbeat-controller:http:default:1.0"

# Status controller
- descriptor: "pip-services:status-controller:http:default:1.0"

```
