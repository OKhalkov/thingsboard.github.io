{% assign feature = "Platform Integrations" %}{% include templates/pe-feature-banner.md %}



## Kafka Integration

Kafka — is an open-source distributed software message broker under the Apache foundation. It is written in the Java and Scala programming languages.

Designed as a distributed, horizontally scalable system that provides capacity growth both with an increase in the number and load from the sources, and the number of subscriber systems. Subscribers can be combined into groups. Supports the ability to temporarily store data for subsequent batch processing.

In some scenarios, Kafka can be used instead of a message queue, in cases where there is no stable connection between the device and an instance.

![image](/images/user-guide/integrations/kafka/Kafka_main.png)

{% capture authorizationTypes %}
Kafka<br/><small>Common installation</small>%,%kafka%,%templates/integration/kafka/kafka-common-installation%br%
Kafka in docker container<br/>%,%basic-credential%,%templates/integration/kafka/kafka-docker-installation%br%
Kafka Confluent<br/><small>Cloud solution</small>%,%basic-credential%,%/templates/integration/kafka/kafka-confluent{% endcapture %}

{% include content-toggle.html content-toggle-id="IntegrationKafka" toggle-spec=authorizationTypes %}

**Create Uplink Converter**

To create **Uplink converter**, go to the **Data Converters** section and click **Add Data Converter**, then **Create New Converter**. Name it "**Uplink (Kafka)**" and select the **uplink** type. Use debug mode when you need to parse decoder events.

**NOTE**. While debug mode is very useful for development and troubleshooting, leaving it enabled in production mode can significantly increase the disk space used by the database since all debug data is stored there. After debugging is complete, it is highly recommended turning off debug mode.

You can use the following code, copy it to the decoder function section:

```js
// Decode an uplink message from a buffer
// payload - array of bytes
// metadata - key/value object

/** Decoder **/
// decode payload to JSON
var payloadJsn = decodeToJson(payload);

// decode payload to String
// var payloadStr = decodeToString(payload);

// var groupName = 'thermostat devices';
// use assetName and assetType instead of deviceName and deviceType
// to automatically create assets instead of devices.
// var assetName = 'Asset A';
// var assetType = 'building';

// Result object with device/asset attributes/telemetry data
   var result = {
// Use deviceName and deviceType or assetName and assetType, but not both.
   deviceName: payloadJsn.deviceName,
   deviceType: payloadJsn.deviceType,
// assetName: assetName,
// assetType: assetType,
   attributes: payloadJsn.attributes,
   telemetry: payloadJsn.telemetry
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

Example of payload:
```json
{
        "deviceName":"SN-111",
        "deviceType":"default",
        "attributes":{
            "model":"Model A"
        },
        "telemetry":[
            {
                "ts":1607731932000,
                "values":{
                    "battery":3.99,
                    "temperature":27.05
                }
            },
            {
                "ts":1607775132000,
                "values":{
                    "battery":3.98,
                    "temperature":27.06
        }}]
}
```
{: .copy-code}

{% include images-gallery.html imageCollection="Create Uplink Converter" %}

You can change the parameters and decoder code when creating a converter or editing. If the converter has already been created, click the pencil icon to edit it. Copy the sample converter configuration (or use your own configuration) and paste it into the decoder function. Then save the changes by clicking the checkmark icon.


**Create Integration**
 
After creating the Uplink converter, it is possible to create an integration. Required fields: Name, Type, Topics

{% include images-gallery.html imageCollection="Kafka Integration" %}

With these settings, the integration will request updates from the Kafka broker every 5 seconds. And if set a topic does not exist at the broker, it will be created automatically.

**Send test Uplink message**

You can simulate a message from a device or server using a terminal. To send an uplink message, you need a Kafka endpoint URL from the integration.
```shell
echo "{\"deviceName\":\"SN-111\",\"deviceType\":\"default\",\"attributes\":{\"model\":\"Model A\"},\"telemetry\":[{\"ts\":1527863143000,\"values\":{\"battery\":9.99,\"temperature\":27.99}},{\"ts\":1527863044000,\"values\":{\"battery\":9.99,\"temperature\":99.99}}]}" | /usr/local/kafka/bin/kafka-console-producer.sh --broker-list YOUR_KAFKA_ENDPOINT_URL:9092 --topic my-topic > /dev/null
```
{: .copy-code}

Result:

{% include images-gallery.html imageCollection="Kafka_integration_test_send_msg_result" %}

Also, you can check through the terminal what data came to Kafka.
```yml
/usr/local/kafka/bin/kafka-console-consumer.sh --bootstrap-server YOUR_KAFKA_ENDPOINT_URL:9092 --topic my-topic --from-beginning
```
{: .copy-code}

**Advanced Usage: Kafka Producer (Downlink)**

To get functionality such as Kafka Producer, you need to use the [Kafka Rule Node](https://thingsboard.io/docs/pe/user-guide/rule-engine-2-0/external-nodes/#kafka-node) in which you can specify Bootstrap servers, Topic and other parameters to connect to the Kafka broker:

With this Node, you can send the preprocessed data to the required Kafka topic.

**Note**: using the same broker for uplink and downlink connections can lead to data loops.

## Next steps

{% assign currentGuide = "ConnectYourDevice" %}{% include templates/multi-project-guides-banner.md %}