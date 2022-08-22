# mqtt-telegraf-helm-charts
Install MQTT broker on K8s and consume data with telegraf and insert data into InfluxDB

# Source
MQTT helm charts taken from  https://github.com/t3n/helm-charts.git

clone the helm charts 
``` https://github.com/chennv4/mqtt-telegraf-helm-charts.git ```

# Create Namespace
```kubectl create namespace mqtt```

# Install MQTT Broker
```helm upgrade --install mqtt-broker mosquitto -n mqtt```

# Telegraf for reading MQTT messages and inserting into InfluxDB
Open telegraf-config.yaml and configure MQTT broker, topic details
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

