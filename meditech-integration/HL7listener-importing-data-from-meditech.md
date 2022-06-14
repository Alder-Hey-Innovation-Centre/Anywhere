# Importing data from meditech with the HL7listener

The HL7Listner is a .NET 6 console application running as a container app, that is used for receiving messages from Meditech (EHR system) in HL7 format over private network. Application converts the HL7 messages to FHIR bundles and then submits the bundle message to the FHIR server.

Logical flow of the application:

- The console app opens a TCP/IP listener on a port specified in the environment variables (default port for HL7 is 2575).
- App receives an HL7 message and verifies that it's format is valid.
- App creates a JSON message containing the HTML encoded HL7 message that is sends to the FHIR servers $convert-data endpoint
- App receives a FHIR bundle from the $convert-data endpoint
- App sends the received FHIR bundle to FHIR server, committing  HL7 data to FHIR

For debugging or troubleshooting see [HL7 Troubleshooting](./HL7-troubleshooting.md)

## Configuration

The [appsettings.json](appsettings.json) file is used for configuration. Each of the configuration values can be overwritten via environment variables by using a pattern ```HL7Config__VariableName=VariableValue``` e.g. variable for port nested under HL7Config can be overwritten as ```HL7Config__Port=2575```.

| Config name | Description | Values | Default |
|---|---|---|---|
| Logging:LogLevel:Default | The default log output of the HL7Listener. | Debug, Information, Warning, Error| Information|
| Logging:LogLevel:Microsoft | The log output level for any namespaces starting with Microsoft. | Debug, Information, Warning, Error| Warning|
| Logging:LogLevel:System | The log output level for any namespaces starting with System. | Debug, Information, Warning, Error| Warning|
| Hl7Config:Port | The TCP port number on which HL7Listener is listening. | 0-65535| 2575|
| Hl7Config:FhirServerBaseUrl | The URL of the FHIR server the data is submitted to. | [https://FHIR_SERVER_NAME.azurehealthcareapis.com](https://FHIRSERVERNAME.azurehealthcareapis.com)| |
| Hl7Config:Hl7ToJsonTemplateCollectionReference | See [templateCollectionReference](https://docs.microsoft.com/en-us/azure/healthcare-apis/fhir/convert-data#parameter-resource). |ACR_NAME.azurecr.io/TEMPLATES_CONTAINER:VERSION||
| Hl7Config:Hl7ToJsonRootTemplate | See [rootTemplate](https://docs.microsoft.com/en-us/azure/healthcare-apis/fhir/convert-data#parameter-resource).|ADT_A01, ADT_A31, ADT_A**|ADT_A01|

Log level explained:

- Debug includes everything.
- Information includes information, warning and error.
- Warning includes warning and error.
- Error for errors only.

## Authentication

The HL7Listener is accessing the FHIR server which requires an Azure Active Directory identity.

We are using a Service Principal to access the FHIR server.

> Ideally we would use Managed Identity to authenticate between HL7Listener and FHIR, but we are hosting the HL7Listener in an Azure Container Instance which does not support Managed Idenity when deployed on private VNET ([product feature request](https://feedback.azure.com/d365community/idea/3d0a8ded-0c25-ec11-b6e6-000d3a4f0858)). The service principal should be switched to Manged Identity once the feature is available ([how to](https://docs.microsoft.com/en-us/azure/container-instances/container-instances-managed-identity)).

We are using [Azure Identity library](https://docs.microsoft.com/en-us/dotnet/api/overview/azure/identity-readme) for authentication. The library automatically picks up the context the process is running in. When running locally it will use the locally authenticated user's credentials, but we can also overwrite this context, by setting the environment variables of the Service Principal:

```bash
AZURE_TENANT_ID=...
AZURE_CLIENT_ID=...
AZURE_CLIENT_SECRET=...
```

The HL7Listener needs the [RBAC roles](https://docs.microsoft.com/en-us/azure/healthcare-apis/configure-azure-rbac):

- ```FHIR Data Converter``` to convert from HL7 to FHIR bundle.
- ```FHIR Data Writer``` to submit the FHIR bundle to the FHIR server.
