# Troubleshooting data ingestion

To troubleshoot data ingestion a few tools may come in handy:

## Postman with collections

In the tools/postman-collections folder, you can find a set of collections to call the various endpoints separately

- **FHIR**: a set of requests to the FHIR server to validate what data is in the FHIR database, and also to delete or modify data if neccessary
- **IOMT connector**: allows you to send data directly to the IOMT connector EventHub to test templates etc.
- **Isansys Lifeguard**: a set of requests to the Isansys Lifeguard server to see the format and contents of incoming data. Useful when debugging/developing the Isansys sync function.

The postman tool can be installed from [www.postman.com](https://www.postman.com/)

1. Open `Postman` and create a new `Workspace`.
1. Import the available collections from the `tools/postman-collections` folder.
1. Import the `Alderhey Dev Environment` Postman environment from the `tools/postman-collections` folder.
1. Edit the environment to copy the default values to current values.
1. Add the client secret for the `AH-anywhere-engine-debug-dev` client app to the environment (current value of clientSecret). You can fetch the application ID and client Secret from the shared Key Vault `kv-ah-shared-cred-dev` located in the resource group `rg-shared-credentials` (`debugApplicationId` and `debugSecret`).

> NOTE: For now you can get the secret from your friendly admin - the specifics of this authentication will be changed in future sprints.

Before executing any `FHIR` or `IOMT connector` requests, you need to execute the respective `Get Azure AD token` requests, which will populate the bearerToken for any future requests.

> NOTE: You can also duplicate the environment, modify the values and use the same requests for the test environment or similar.

## Investigating problems with IOMT normalization or FHIR conversion

### Send data to the IOMT connector

1. In `Postman`, open the `IOMT connector` collection.
1. Execute the `Get Azure AD token` request.
1. Execute the `Send data to the IOMT connector` request, adding the data you want to send in the body of the request.

The request should return a `201 Created` response.

![postman to IOMT](../assets/postman-toiomt.png)

### Verify that data makes it to the devicedata eventhub

1. Find the `evhns-ic-anywhere-engine-dev` Event Hubs Namespace inside the `rg-anywhere-engine-dev` resource group the Azure portal.
1. Under `Overview` verify that we seem to be receiving messages
1. Under `Entities` and `Event Hubs` select the `devicedata` Event Hub
1. Under `Features` select `Process data` and select `Explore` on `Enable real time insights from events`
1. In the `Query` view, you may need to select the `create` button in the `Input preview` pane if permission to view data is not granted already.

![device data eventhub](../assets/devicedata.png)

### Verify that the Normalize Device Data function can process the incoming data correctly

1. Find the `func-ic-anywhere-engine-dev` Function App inside the `rg-anywhere-engine-dev` resource group the Azure portal.
1. Under `Functions` select the `NormalizeDeviceData` function
1. Select `Monitor` to see the Invocation Traces
1. Verify that the latest invocations show `Success` - For any `Errors` you may need to investigate the logs by clicking on the respective runs for more details

![normalize data function](../assets/normalize-data-function.png)

### Verify that the Normalized device data is correct

1. Find the `evhns-ic-anywhere-engine-dev` Event Hubs Namespace inside the `rg-anywhere-engine-dev` resource group the Azure portal.
1. Under `Overview` verify that we seem to be receiving messages
1. Under `Entities` and `Event Hubs` select the `normalizeddata` Event Hub
1. Under `Features` select `Process data` and select `Explore` on `Enable real time insights from events`
1. In the `Query` view, you may need to select the `create` button in the `Input preview` pane if permission to view data is not granted already.

![device data eventhub](../assets/normalizeddata.png)

### Verify that the FHIR function can process the incoming data correctly

1. Find the `func-ic-anywhere-engine-dev` Function App inside the `rg-anywhere-engine-dev` resource group the Azure portal.
1. Under `Functions` select the `MeasurementCollectionToFhir` function
1. Select `Monitor` to see the Invocation Traces
1. Verify that the latest invocations show `Success` - For any `Errors` you may need to investigate the logs by clicking on the respective runs for more details

![normalize data function](../assets/iomt-to-fhir-function.png)

### Verify that the data is correct in the FHIR database

1. In `Postman`, open the `FHIR` collection.
1. Execute the `Get Azure AD token` request.
1. Execute the `Observations\Get Observations for Patient by identifier` request, with any of the patients identifiers as the patient query value.

![postman FHIR observations](../assets/postman-fhir-observations.png)
