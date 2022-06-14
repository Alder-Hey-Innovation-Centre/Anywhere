# How IOMT templates work

IOMT mappings occur in two steps:

1. Device Content Mapping (Normalization) - where we map from the incoming format into a common format for observations using JsonPath expressions.
2. FHIR mapping - where we map from the common format into the FHIR format.

The devicecontent.json file and the fhirmapping.json file with the templates are stored in the IOMT connectors storage account in the `template` container, and the actual conversion is happening in the IOMT connectors Azure Function.

See the [IOMT documentation](https://github.com/microsoft/iomt-fhir/blob/master/docs/Configuration.md#examples) for more information and examples.

## Device Content Mapping

The templates for mapping incoming json to the intermediate format we need to write are in `devicecontent.json`

### Incoming data

For Isansys (for example) the messages from the Isansys sync look like this:

```json
{
  "patientId":"Patient1",
  "deviceType":"Lifeguard",
  "measurements":[
    {
      "heartRate":60.0,
      "timestamp":"2019-02-01T20:46:01.8750000Z"
    },
    {
      "heartRate":60.0,
      "timestamp":"2019-02-01T21:46:01.8750000Z"
    },
    {
      "heartRate":60.0,
      "timestamp":"2019-02-01T22:46:01.8750000Z"
    }
  ]
}
```

and similar for other type of measurements.

### Mapping

This is mapped using a CalculatedContent template in `devicecontent.json`.

```json
        {
            "templateType": "CalculatedContent",
            "template": {
                "typeName": "heartRate",
                "typeMatchExpression": "$..[?(@heartRate)]",
                "deviceIdExpression":{
                    "value": "join('_', [Body.deviceType, Body.patientId])",
                    "language": "JmesPath"
                },
                "patientIdExpression": "$.Body.patientId",
                "timestampExpression": "$.matchedToken.timestamp",
                "values": [
                    {
                        "required": "true",
                        "valueExpression": "$.matchedToken.heartRate",
                        "valueName": "heartRate"
                    }
                ]
            }
        },
```

That looks for heartRate in the incoming json, and where it finds it, records the heartrate, the timestamp, and the deviceId and patientId from the original message. The original message with 3 observations generates 3 outgoing readings.

The deviceID is a combination of the manufacturer and the patientId ex. "Isansys_Patient1".

Some things to note:

1. The message coming from the EventHub will be wrapped in an EventHub json envelope

    ```json
    {
        "Body": { message },
        "Properties": {},
        "SystemProperties": {}
    }
    ```

    So to address items in the message we need to address them with `$.Body.propertyname` - ex. `$.Body.deviceType`

1. When the IOMT connector finds a match, it will create a message like so - with the matched token as a separate item, so we can address anything in the matched token with `$.matchedToken.heartRate` or `$.matchedToken.timestamp` etc.

```json
{
    "Body": {
        "humanReadableDeviceId": "ABC123",
        "patientId": "Patient1",
        "manufacturer": "Isansys",
        "measurements": [
            {
                "heartRate": 60.0,
                "timestamp": "2019-02-01T20:46:01.8750000Z"
            },
            {
                "heartRate": 60.0,
                "timestamp": "2019-02-01T21:46:01.8750000Z"
            },
            {
                "heartRate": 60.0,
                "timestamp": "2019-02-01T22:46:01.8750000Z"
            }
        ]
    },
    "Properties": {},
    "SystemProperties": {},
    "matchedToken": {
        "heartRate": 60.0,
        "timestamp": "2019-02-01T20:46:01.8750000Z"
    }
}
```

## FHIR Mapping

The FHIR mapping takes the intermediate format and converts it to the FHIR format.

We can see examples of this in the `fhirmapping.json` file.

For each template (typeName) and each value (valueName) we define a template describing how the value will be mapped.

Here we define the loinc code and unit to be used as well as the sampling rate. For the example below we set the sampling rate to `quantity` indicating that we will not group them, but rather send in all the values as individual observations.

```json
{
    "templateType": "CodeValueFhir",
    "template": {
        "codes": [
            {
                "code": "2708-6",
                "system": "http://loinc.org",
                "display": "Oxygen saturation"
            }
        ],
        "periodInterval": 0,
        "typeName": "oxygenSaturation",
        "value": {
            "unit": "%",
            "valueName": "oxygenSaturation",
            "valueType": "quantity"
        }
    }
},
```

The `MeasurementCollectionToFhir` function is triggered periodically to gather all normalized data from the event hub and create FHIR observations from them.

## Combining templates

In the devicecontent.json we add all our templates to the `templates` array. The current plan is to store each template separately and join them to the devicecontent.json in a CD pipeline.
