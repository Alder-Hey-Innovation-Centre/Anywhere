# Data Flow from Device to Portal

The following image shows the data flow from a device to the portal.

## Using the Standard API

1. Device manufacturer calls the standard API with json containing the patient observation. Each request is limited to 10KB (roughly 100 measurements)

    > See the [json health data ADR](../architecture-decisions/0010-json-health-data.md) for more details.

1. API Management adds the deviceType based on the API key
1. API Management forwards the call to the Standard API Azure function, which in turn adds the deviceType to the json data, and forwards it to the IOMT connector
1. The IOMT connectors `NormalizeDeviceData` Azure function normalizes (converts to a common format) the data, based on the devicecontent templates.

    > See [IOMT templates](IOMT-templates/how-templates-work.md) for more inforamtion.

1. The IOMT connectors `MeasurementCollectionToFhir` Azure function converts the data to FHIR, and sends it to the FHIR server.

    > See [IOMT templates](IOMT-templates/how-templates-work.md) for more information

1. The AlderHey Portal reads the data from FHIR and presents it to the clinicians or patients

## Using Isansys Sync

1. The Isansys Sync Azure function periodically (once a minute) calls the on-premise Lifeguard server to sync the data for all Isansys devices connected to Lifeguard.
1. Isansys Sync sends the data to the IOMT connector, and continues with steps 3-6 above.

    > For more information on the inner workings on the Isansys Sync Azure function, see [Isansys Sync Azure function README](https://dev.azure.com/AlderHey/AlderHey%20Anywhere/_git/IsansysSync)

![Data flow](../assets/dataflow.png)
