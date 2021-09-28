{% assign feature = "Platform Integrations" %}{% include templates/pe-feature-banner.md %}



## Kafka Integration

Kafka Integration — is an open-source distributed software message broker under the Apache foundation. It is written in the Java and Scala programming languages.

Designed as a distributed, horizontally scalable system that provides capacity growth both with an increase in the number and load from the sources, and the number of subscriber systems. Subscribers can be combined into groups. Supports the ability to temporarily store data for subsequent batch processing.

In some scenarios, Kafka can be used instead of a message queue, in cases where there is no stable connection between the device and ThingsBoard.
#
 ![image](/images/user-guide/integrations/mqtt-integration.png)
#
## Kafka Installation
[Apache Kafka](https://kafka.apache.org/) is an open-source stream-processing software platform.

### Install ZooKeeper
Kafka uses ZooKeeper, so you need to first install ZooKeeper server:
```shell
sudo apt-get install zookeeper
```
{: .copy-code}

### Install Kafka
```shell
wget https://dlcdn.apache.org/kafka/3.0.0/kafka_2.13-3.0.0.tgz

tar xzf kafka_2.13-3.0.0.tgz

mv kafka_2.13-3.0.0 /usr/local/kafka
```
{: .copy-code}


### Setup ZooKeeper Systemd Unit file
Create systemd unit file for Zookeeper:
```shell
sudo nano /etc/systemd/system/zookeeper.service
```
{: .copy-code}

Add below content:
```shell
[Unit]
Description=Apache Zookeeper server
Documentation=http://zookeeper.apache.org
Requires=network.target remote-fs.target
After=network.target remote-fs.target

[Service]
Type=simple
ExecStart=/usr/local/kafka/bin/zookeeper-server-start.sh /usr/local/kafka/config/zookeeper.properties
ExecStop=/usr/local/kafka/bin/zookeeper-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
```
{: .copy-code}

### Setup Kafka Systemd Unit file
Create systemd unit file for Kafka:
```shell
sudo nano /etc/systemd/system/kafka.service
```
{: .copy-code}

Add the below content. Make sure to replace "**PUT_YOUR_JAVA_PATH**" with your real JAVA_HOME path as per the Java installed on your system, by default like “/usr/lib/jvm/java-11-openjdk-xxx”:
```shell
[Unit]
Description=Apache Kafka Server
Documentation=http://kafka.apache.org/documentation.html
Requires=zookeeper.service

[Service]
Type=simple
Environment="JAVA_HOME=PUT_YOUR_JAVA_PATH"
ExecStart=/usr/local/kafka/bin/kafka-server-start.sh /usr/local/kafka/config/server.properties
ExecStop=/usr/local/kafka/bin/kafka-server-stop.sh

[Install]
WantedBy=multi-user.target
```
{: .copy-code}

Start ZooKeeper and Kafka:
```shell
sudo systemctl start zookeeper

sudo systemctl start kafka
```
{: .copy-code}

## Kafka installation using Docker
Create docker-compose file for services:
```shell
nano docker-compose.yml
```
{: .copy-code}

Add the following line to the yml file:
```shell
services:
  zookeeper:
    restart: always
    image: "zookeeper:3.5"
    ports:
      - "2181:2181"
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=zookeeper:2888:3888;zookeeper:2181
  kafka:
    restart: always
    image: wurstmeister/kafka:13-3.0.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```
{: .copy-code}

Where:
- restart: always - automatically start Kafka service in case of system reboot and restart in case of failure.;
- Ports: 2181:2181 - connect local port 2181 to exposed internal zookeeper port zookeeper;
- Ports: 9092:9092 - connect local port 9092 to exposed internal kafka port kafka;

Then, from the folder with this file, run the command:
```shell
docker-compose up -d
```
{: .copy-code}

### Create Uplink Converter

```js
// Decode an uplink message from a buffer
// payload - array of bytes
// metadata - key/value object

/** Decoder **/
// decode payload to string
var payloadStr = decodeToJson(payload);
// decode payload to JSON
// var data = decodeToJson(payload);
// var groupName = 'thermostat devices';
// use assetName and assetType instead of deviceName and deviceType
// to automatically create assets instead of devices.
// var assetName = 'Asset A';
// var assetType = 'building';
// Result object with device/asset attributes/telemetry data

   var result = {
// Use deviceName and deviceType or assetName and assetType, but not both.
   deviceName: payloadStr.deviceName,
   deviceType: payloadStr.deviceType,
// assetName: assetName,
// assetType: assetType,
   attributes: payloadStr.attributes,
   telemetry: payloadStr.telemetry
};

/** Helper functions **/
function decodeToString(payload) {
   return String.fromCharCode.apply(String, payload);
}
function decodeToJson(payload) {
   // covert payload to string.
   var str = decodeToString(payload);
   // parse string to JSON
   var data = JSON.parse(str);
   return data;
}
return result;
```
{: .copy-code}
You can change the parameters and decoder code when creating a converter or editing. If the converter has already been created, click the pencil icon to edit it. Copy the sample converter configuration (or use your own configuration) and paste it into the decoder function. Then save the changes by clicking the checkmark icon.


ТУТ КАРТИНКИ

### Create Integration
After creating the Uplink converter, it is possible to create an integration.

ТУТ КАРТИНКИ

With these settings, the integration will request updates from the Kafka broker every 5 seconds. And if set a topic does not exist at the broker, it will be created automatically.

### Send test Uplink message
You can simulate a message from a device or server using a terminal. To send an uplink message, you need a Kafka endpoint URL from the integration.
```shell
echo "{\"deviceName\":\"SN-111\",\"deviceType\":\"default\",\"attributes\":{\"model\":\"Model A\"},\"telemetry\":[{\"ts\":1527863143000,\"values\":{\"battery\":9.99,\"temperature\":27.99}},{\"ts\":1527863044000,\"values\":{\"battery\":9.99,\"temperature\":99.99}}]}" | /usr/local/kafka/bin/kafka-console-producer.sh --broker-list YOUR_KAFKA_ENDPOINT_URL:9092 --topic my-topic > /dev/null
```
{: .copy-code}

тут скрин

You can also check through the console what data came to Kafka
```shell
/usr/local/kafka/bin/kafka-console-consumer.sh --bootstrap-server YOUR_KAFKA_ENDPOINT_URL:9092 --topic my-topic --from-beginning
```
{: .copy-code}

### Advanced Usage: Create Downlink Converter

### **Дописать**


Kafka Node sends messages to Kafka brokers. Expect messages with any message type. Will send record via Kafka producer to Kafka server.


You can also connect an additional module Kafka Streams API for processing the calculation of incoming or outgoing Kafka data

## Next steps

{% assign currentGuide = "ConnectYourDevice" %}{% include templates/multi-project-guides-banner.md %}