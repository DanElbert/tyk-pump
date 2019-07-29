# Tyk Pump

[![Build Status](https://travis-ci.org/TykTechnologies/tyk-pump.svg?branch=master)](https://travis-ci.org/TykTechnologies/tyk-pump)

Tyk Pump is a pluggable analytics purger to move Analytics generated by your Tyk nodes to any back-end.

## Back ends currently supported:

- MongoDB (to replace built-in purging)
- CSV (updated, now supports all fields)
- ElasticSearch (2.0+)
- Graylog
- InfluxDB
- Moesif
- Splunk
- StatsD
- DogStatsD
- Hybrid (Tyk RPC)
- Prometheus
- Logz.io
- Kafka

## Configuration:

Create a `pump.conf` file:

```
{
  "analytics_storage_type": "redis",
  "analytics_storage_config": {
    "type": "redis",
    "host": "localhost",
    "port": 6379,
    "hosts": null,
    "username": "",
    "password": "",
    "database": 0,
    "optimisation_max_idle": 100,
    "optimisation_max_active": 0,
    "enable_cluster": false
  },
  "purge_delay": 1,
  "pumps": {
    "dummy": {
      "type": "dummy",
      "meta": {
        
      }
    },
    "mongo": {
      "type": "mongo",
      "meta": {
        "collection_name": "tyk_analytics",
        "mongo_url": "mongodb://username:password@{hostname:port},{hostname:port}/{db_name}"
      }
    },
    "mongo-pump-aggregate": {
      "name": "mongo-pump-aggregate",
      "meta": {
	"mongo_url": "mongodb://username:password@{hostname:port},{hostname:port}/{db_name}",
	"use_mixed_collection": true
      }
    },
    "csv": {
      "type": "csv",
      "meta": {
        "csv_dir": "./"
      }
    },
    "elasticsearch": {
      "type": "elasticsearch",
      "meta": {
        "index_name": "tyk_analytics",
        "elasticsearch_url": "localhost:9200",
        "enable_sniffing": false,
        "document_type": "tyk_analytics",
        "rolling_index": false,
        "extended_stats": false,
        "version": "5"
      }
    },
    "influx": {
      "type": "influx",
      "meta": {
        "database_name": "tyk_analytics",
        "address": "http//localhost:8086",
        "username": "root",
        "password": "root",
        "fields": [
          "request_time"
        ],
        "tags": [
          "path",
          "response_code",
          "api_key",
          "api_version",
          "api_name",
          "api_id",
          "raw_request",
          "ip_address",
          "org_id",
          "oauth_id"
        ]
      }
    },
    "moesif": {
      "type": "moesif",
      "meta": {
        "application_id": ""
      }
    },
    "splunk": {
      "type": "splunk",
      "meta": {
        "collector_token": "<token>",
        "collector_url": "<url>",
        "ssl_insecure_skip_verify": false,
        "ssl_cert_file": "<cert-path>",
        "ssl_key_file": "<key-path>",
        "ssl_server_name": "<server-name>"
      }
    },
    "statsd": {
      "type": "statsd",
      "meta": {
        "address": "localhost:8125",
        "fields": [
          "request_time"
        ],
        "tags": [
          "path",
          "response_code",
          "api_key",
          "api_version",
          "api_name",
          "api_id",
          "raw_request",
          "ip_address",
          "org_id",
          "oauth_id"
        ]
      }
    },
    "dogstatsd": {
      "name": "dogstatsd",
      "meta": {
        "address": "localhost:8125",
        "namespace": "pump",
        "async_uds": true,
        "async_uds_write_timeout_seconds": 2,
        "buffered": true,
        "buffered_max_messages": 32
      }
    },
    "prometheus": {
      "type": "prometheus",
      "meta": {
        "listen_address": "localhost:9090",
        "path": "/metrics"
      }
    },
    "graylog": {
      "type": "graylog",
      "meta": {
        "host": "10.60.6.15",
        "port": 12216,
        "tags": [
          "method",
          "path",
          "response_code",
          "api_key",
          "api_version",
          "api_name",
          "api_id",
          "org_id",
          "oauth_id",
          "raw_request",
          "request_time",
          "raw_response"
        ]
      }
    },
    "hybrid": {
      "type": "hybrid",
      "meta": {
        "rpc_key": "5b5fd341e6355b5eb194765e",
        "api_key": "008d6d1525104ae77240f687bb866974",
        "connection_string": "localhost:9090",
	"aggregated": false,
        "use_ssl": false,
        "ssl_insecure_skip_verify": false,
        "group_id": "",
        "call_timeout": 30,
        "ping_timeout": 60,
        "rpc_pool_size": 30
      }
    },
    "logzio": {
      "type": "logzio",
      "meta": {
        "token": "<YOUR-LOGZ.IO-TOKEN>"
      }
    },
    "kafka": {
      "type": "kafka",
      "meta": {
        "broker": [
            "localhost:9092"
        ],
        "client_id": "tyk-pump",
        "topic": "tyk-pump",
        "timeout": 60,
        "compressed": true,
        "meta_data": {
            "key": "value"
        }
      }
    }
  },
  "uptime_pump_config": {
    "collection_name": "tyk_uptime_analytics",
    "mongo_url": "mongodb://username:password@{hostname:port},{hostname:port}/{db_name}"
  },
  "dont_purge_uptime_data": false
}
```

Settings are the same as for the original `tyk.conf` for redis and for mongoDB.

### Tyk Dashboard

The Tyk Dashboard uses the "mongo-pump-aggregate" collection to display analytics.  This is different than the standard "mongo" pump plugin that will store individual analytic items into mongo.  The aggregate functionality was built to be fast, as querying raw analytics is expensive in large data sets.

### Elasticsearch Config

`"index_name"` - The name of the index that all the analytics data will be placed in. Defaults to "tyk_analytics"

`"elasticsearch_url"` - If sniffing is disabled, the URL that all data will be sent to. Defaults to "http://localhost:9200"

`"enable_sniffing"` - If sniffing is enabled, the "elasticsearch_url" will be used to make a request to get a list of all the nodes in the cluster, the returned addresses will then be used. Defaults to false

`"document_type"` - The type of the document that is created in ES. Defaults to "tyk_analytics"

`"rolling_index"` - Appends the date to the end of the index name, so each days data is split into a different index name. E.g. tyk_analytics-2016.02.28 Defaults to false

`"extended_stats"` - If set to true will include the following additional fields: Raw Request, Raw Response and User Agent.

`"version"` - Specifies the ES version. Use "3" for ES 3.X, "5" for ES 5.X, "6" for ES 6.X. Defaults to "3".

### Moesif Config
[Moesif](https://www.moesif.com) is a logging and analytics service for APIs. The Moesif pump will
move analytics data from Tyk to Moesif.

`"application_id"` - Moesif App Id JWT. Multiple api_id's will go under the same app id.

### Prometheus
Prometheus is an open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach.

Add the following section to expose "/metrics" endpoint:
```
"prometheus": {
        "type": "prometheus",
	"meta": {
		"listen_address": "localhost:9090",
		"path": "/metrics"
	}
},
```

Tyk expose the following counters:
- tyk_http_status{code, api}
- tyk_http_status_per_path{code, api, path, method}
- tyk_http_status_per_key{code, key}
- tyk_http_status_per_oauth_client{code, client_id}

And the following Histogram for latencies:
- tyk_latency{type, api}

### DogStatsD

- `address`: address of the datadog agent including host & port
- `namespace`: prefix for your metrics to datadog
- `async_uds`: Enable async UDS over UDP https://github.com/Datadog/datadog-go#unix-domain-sockets-client
- `async_uds_write_timeout_seconds`: Integer write timeout in seconds if `async_uds: true`
- `buffered`: Enable buffering of messages
- `buffered_max_messages`: Max messages in single datagram if `buffered: true`. Default 16
- `sample_rate`: default 1 which equates to 100% of requests. To sample at 50%, set to 0.5

```json
"dogstatsd": {
  "name": "dogstatsd",
  "meta": {
    "address": "localhost:8125",
    "namespace": "pump",
    "async_uds": true,
    "async_uds_write_timeout_seconds": 2,
    "buffered": true,
    "buffered_max_messages": 32,
    "sample_rate": 0.5
  }
},
```

On startup, you should see the loaded configs when initializing the dogstatsd pump
```
[May 10 15:23:44]  INFO dogstatsd: initializing pump
[May 10 15:23:44]  INFO dogstatsd: namespace: pump.
[May 10 15:23:44]  INFO dogstatsd: sample_rate: 50%
[May 10 15:23:44]  INFO dogstatsd: buffered: true, max_messages: 32
[May 10 15:23:44]  INFO dogstatsd: async_uds: true, write_timeout: 2s
```

### Kafka Config

`"broker"` - The list of brokers used to discover the partitions available on the kafka cluster. E.g. "localhost:9092"

`"client_id"` - Unique identifier for client connections established with Kafka.

`"topic"` - The topic that the writer will produce messages to.

`"timeout"` - Timeout is the maximum amount of time will wait for a connect or write to complete. 

`"compressed"` - Enable "github.com/golang/snappy" codec to be used to compress Kafka messages. By default is false

`"meta_data"` - Can be used to set custom metadata inside the kafka message

## Compiling & Testing

1. Download dependent packages:

```
go get -t -d -v ./...
```

2. Compile:

```
go build -v ./...
```

3. Test

```
go test -v ./...
```
