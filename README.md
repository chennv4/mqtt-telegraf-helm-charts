# mqtt-telegraf-helm-charts
Install MQTT broker on K8s and consume data with telegraf and insert data into InfluxDB

# Source
MQTT helm charts taken from  https://github.com/t3n/helm-charts.git

clone the helm charts 
``` 

git clone https://github.com/chennv4/mqtt-telegraf-helm-charts.git

```

# Create Namespace
```kubectl create namespace mqtt```

# Install MQTT Broker
```helm upgrade --install mqtt-broker mosquitto -n mqtt```

# Telegraf for reading MQTT messages and inserting into InfluxDB
Open telegraf-config.yaml and configure MQTT broker, topic details

Configure influxdb details
```
[[outputs.influxdb]]
      ## The full HTTP or UDP URL for your InfluxDB instance.
      ##
      ## Multiple URLs can be specified for a single cluster, only ONE of the
      ## urls will be written to each interval.
      # urls = ["unix:///var/run/influxdb.sock"]
      # urls = ["udp://127.0.0.1:8089"]
      # urls = ["http://127.0.0.1:8086"]
      urls = ["http://project-metrics.observer.svc.cluster.local:8086"]

      ## The target database for metrics; will be created as needed.
      database = "observer"

      ## If true, no CREATE DATABASE queries will be sent.  Set to true when using
      ## Telegraf with a user without permissions to create databases or when the
      ## database already exists.
      # skip_database_creation = false

      ## Name of existing retention policy to write to.  Empty string writes to
      ## the default retention policy.  Only takes effect when using HTTP.
      # retention_policy = ""

      ## Write consistency (clusters only), can be: "any", "one", "quorum", "all".
      ## Only takes effect when using HTTP.
      # write_consistency = "any"

      ## Timeout for HTTP messages.
      # timeout = "5s"

      ## HTTP Basic Auth
      username = "9Xp3wEnsVE"
      password = "rufq8or8P1"
   ```
   
   Configure input MQTT consumer
```
[[inputs.mqtt_consumer]]
       servers = ["tcp://10.246.153.250:1883"]

       ## Topics that will be subscribed to.
       #data_format = "influx"
       topics = ["sensors/#"]
       qos = 2
       connection_timeout = "300s"
       persistent_session = false

       data_format = "json" # invokes the parser -- lines following are parser config

```

Deploy telegraf

```
kubectl apply -f telegraf-config.yaml -n mqtt
kubectl apply -f telegraf-secrets.yaml -n mqtt
kubectl apply -f telegraf-deployment.yaml -n mqtt
```
# Publish message 
```mosquitto_pub -h 10.246.153.250 -p 1883 -t sensors -m "{\"sensor-id\": 2, \"type\": \"temp-sensor\", \"value\": 33.29}" -q 0 -i mqtt-client-id -d```

# Consume or Subscribe messages
```mosquitto_sub -h 10.246.153.250  -v -t 'sensors'```

