# Standard templates

In order to quickly on-board new devices, we have a set of standard templates that can be used assuming that the device manufacturer sends data in the following format.

```json
{
  "patientId":"AH-12345",
  "deviceType":"some-device-type",
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

> NOTE: the json may optionally contain additional properties, but those will be ignored. API management, along with the StandardApi Azure function will inject the deviceType before forwarding the message to the IOMT connector.

Standard templates are provided for the following vitals:

- Body Height (m)
- Body Mass Index (kg/m2)
- Body Temperature (C)
- Body Weight (kg)
- Diastolic Blood Pressure (mmHg)
- Heart Rate / pulse (1/min)
- Oxygen Saturation (%)
- Respiratory Rate (1/min)
- Systolic Blood Pressure (mmHg)

The data message can contain one or more measurements. Here is an example message containing all the supported vitals.

```json
{
    "deviceType": "AH-12345",
    "patientId":"some-device-type",
    "measurements":[
        {
            "BMI":25.0,
            "height":2.0,
            "weight":65.0,
            "temperature":36.9,
            "diastolic":100,
            "systolic":120,
            "heartRate":65,
            "pulse":65,
            "oxygenSaturation":95.0,
            "respirationRate":66,
            "timestamp":"2022-03-13T17:16:00.0000Z"
        }
    ]
}
```

The standard templates (and any custom templates) can be found in the `mapping-templates` folder.
