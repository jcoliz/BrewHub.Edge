# Telegraf Issue #13338 Repro

This is a repro for [Telegraf Issue #13338](https://github.com/influxdata/telegraf/issues/13338)

1. Put up Mosquitto version

```
PS BrewHub\src\brewhub-edge\issue-13338> docker compose -f .\docker-compose-eclipse.yml up
```

2. Start generating MQTT traffic

```
PS iot\MqttSender> dotnet run
[13:06:05 INF] Successfully connected.
[13:06:05 INF] Message recieved: {"Timestamp":1685477165712,"Seq":0,"Model":"dtmi:com:example:TemperatureController;2","WorkingSet":197760}
[13:06:05 INF] Message recieved: {"Timestamp":1685477165712,"Seq":0,"Model":"dtmi:com:example:Thermostat;1","Temperature":60}
[13:06:05 INF] Message recieved: {"Timestamp":1685477165712,"Seq":0,"Model":"dtmi:com:example:Thermostat;1","Temperature":64.86289650954788}
```

3. Examine docker logs

```
mosquitto  | 1685477150: mosquitto version 2.0.15 starting
mosquitto  | 1685477150: Config loaded from /mosquitto/config/mosquitto.conf.
mosquitto  | 1685477150: Opening ipv4 listen socket on port 1883.
mosquitto  | 1685477150: Opening ipv6 listen socket on port 1883.
mosquitto  | 1685477150: mosquitto version 2.0.15 running
mosquitto  | 1685477151: New connection from 172.19.0.4:58738 on port 1883.
mosquitto  | 1685477151: New client connected from 172.19.0.4:58738 as telegraf (p2, c0, k60).
mosquitto  | 1685477165: New connection from 172.19.0.1:43282 on port 1883.
mosquitto  | 1685477165: New client connected from 172.19.0.1:43282 as mqttsender (p2, c1, k15).

telegraf   | 2023-05-30T20:05:51Z I! Loading config: /etc/telegraf/telegraf.conf
telegraf   | 2023-05-30T20:05:51Z I! Starting Telegraf 1.26.3
telegraf   | 2023-05-30T20:05:51Z I! Available plugins: 235 inputs, 9 aggregators, 27 processors, 22 parsers, 57 outputs, 2 secret-stores
telegraf   | 2023-05-30T20:05:51Z I! Loaded inputs: mqtt_consumer
telegraf   | 2023-05-30T20:05:51Z I! Loaded aggregators:
telegraf   | 2023-05-30T20:05:51Z I! Loaded processors:
telegraf   | 2023-05-30T20:05:51Z I! Loaded secretstores:
telegraf   | 2023-05-30T20:05:51Z I! Loaded outputs: file influxdb_v2
telegraf   | 2023-05-30T20:05:51Z I! Tags enabled: host=8fe78e84ddc6
telegraf   | 2023-05-30T20:05:51Z I! [agent] Config: Interval:10s, Quiet:false, Hostname:"8fe78e84ddc6", Flush Interval:10s
telegraf   | 2023-05-30T20:05:51Z D! [agent] Initializing plugins
telegraf   | 2023-05-30T20:05:51Z D! [agent] Connecting outputs
telegraf   | 2023-05-30T20:05:51Z D! [agent] Attempting connection to [outputs.influxdb_v2]
telegraf   | 2023-05-30T20:05:51Z D! [agent] Successfully connected to outputs.influxdb_v2
telegraf   | 2023-05-30T20:05:51Z D! [agent] Attempting connection to [outputs.file]
telegraf   | 2023-05-30T20:05:51Z D! [agent] Successfully connected to outputs.file
telegraf   | 2023-05-30T20:05:51Z D! [agent] Starting service inputs
telegraf   | 2023-05-30T20:05:51Z I! [inputs.mqtt_consumer] Connected [tcp://mqtt:1883]
telegraf   | 2023-05-30T20:06:01Z D! [outputs.file] Buffer fullness: 0 / 10000 metrics
telegraf   | 2023-05-30T20:06:01Z D! [outputs.influxdb_v2] Buffer fullness: 0 / 10000 metrics
telegraf   | 2023-05-30T20:06:11Z D! [outputs.file] Wrote batch of 33 metrics in 107.567µs
telegraf   | 2023-05-30T20:06:11Z D! [outputs.file] Buffer fullness: 0 / 10000 metrics
telegraf   | mqtt_consumer,host=8fe78e84ddc6,topic=spB1.0/testing/NDATA/device-1 value="{\"Timestamp\":1685477165712,\"Seq\":0,\"Model\":\"dtmi:com:example:TemperatureController;2\",\"WorkingSet\":197760}" 1685477165790826492
telegraf   | mqtt_consumer,host=8fe78e84ddc6,topic=spB1.0/testing/DDATA/device-1/thermostat1 value="{\"Timestamp\":1685477165712,\"Seq\":0,\"Model\":\"dtmi:com:example:Thermostat;1\",\"Temperature\":60}" 1685477165792887332
telegraf   | mqtt_consumer,host=8fe78e84ddc6,topic=spB1.0/testing/DDATA/device-1/thermostat2 value="{\"Timestamp\":1685477165712,\"Seq\":0,\"Model\":\"dtmi:com:example:Thermostat;1\",\"Temperature\":64.86289650954788}" 1685477165793700996
```

We can see that all the Seq:0 entries are logged.

4. Take that down

```
PS BrewHub\src\brewhub-edge\issue-13338> docker compose -f .\docker-compose-eclipse.yml down
```

5. Put up the VerneMQ version

```
docker compose -f .\docker-compose-vernemq.yml up
```

6. Again run the producer, and examine logs

```
telegraf  | mqtt_consumer,host=a0f64c339772,topic=spB1.0/testing/NDATA/device-1 value="{\"Timestamp\":1685477519536,\"Seq\":0,\"Model\":\"dtmi:com:example:TemperatureController;2\",\"WorkingSet\":194400}" 1685477519599114464
telegraf  | mqtt_consumer,host=a0f64c339772,topic=spB1.0/testing/DDATA/device-1/thermostat1 value="{\"Timestamp\":1685477519536,\"Seq\":0,\"Model\":\"dtmi:com:example:Thermostat;1\",\"Temperature\":47.90943073464691}" 1685477519599274098
telegraf  | mqtt_consumer,host=a0f64c339772,topic=spB1.0/testing/DDATA/device-1/thermostat2 value="{\"Timestamp\":1685477519536,\"Seq\":0,\"Model\":\"dtmi:com:example:Thermostat;1\",\"Temperature\":54.158233816355185}" 1685477519600995600
```

Oh! It's working now!!

Ok, fine, I will start by making incremental changes toward the world I want.

First thing, I put in the vernemq config I'm actually using. All good.

Aha! I cleared the volumes, cleared the containers, and now it's back to not working!

Ok, went back a step to using NO vernemq config.
Ran it once, and it now it's back to working.