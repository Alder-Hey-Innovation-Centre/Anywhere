# HL7 Listener Troubleshooting

This is the format used by Meditech, and we need to convert it to FHIR to import the patients into the FHIR server.

The full process for ingesting HL7 data from Meditech is outlined in the [Importing data from meditech with the HL7listener](./HL7listener-importing-data-from-meditech.md) document.

## Troubleshooting

If messages aren't coming through from Meditech to FHIR, firstly check that the container instance is up and running, this should be named `aci-hl7-anywhereengine[environment]` e.g. `aci-hl7-anywhereenginedev`
The status should show as running:

![Container instance screenshot](../assets/hl7-container-instance-status.png)

If the status is running, check the container instance logs:

![Container instance logs screenshot](../assets/hl7-container-instance-logs.png)

If you see something that looks like this:

```hl7
Received data: 'MSH|^~\&|HIS|RLC|||201608040915||ADT^A01|
```

Then the HL7 listener is correctly receiving data from Meditech.

> Note that you may see data that looks like this:
>
> ```hl7
> Received data: 'ieU��random1random2random3random4/
>      9�,*'
>```
>
> This is not necessarily indicative of successful data from Meditech.

## Application Insights

If data is observed being received in the logs then we know the connection to Meditech is working OK.  The next step is to check Application Insights.  This can also be done in the case that data is not being observed in the logs as a double check.

Find the Application Insights resource in the same resource group as you're currently working e.g.`appi-anywhere-engine-dev`.
Navigate to the *Logs* section in the *Monitoring* group and run the following query:

```text
traces
| where cloud_RoleName contains "HL7Listener"
```

It should look as follows:

![Application Insights screen shot with query](../assets/hl7-application-insights.png)

You should see data being received as below:

![Application Insights screen shot with logs](../assets/hl7-application-insights-logs.png)

If you see any errors or warnings in the list this may indicate either a problem sending the message to the FHIR server or an invalid message received from Meditech.  To view more information in the logs set the log level to debug as described in the *Debugging* section.

## Running Locally

Set the configuration as in the [Configuration document](./HL7listener-importing-data-from-meditech.md#configuration).

Requirement: Ensure you have dotnet installed on the machine used for debugging. You can download dotnet from [here](https://dotnet.microsoft.com/en-us/download).

- From Visual Studio - open the AlderHeyAnywhere.sln and press F5
- From command prompt - from the HL7Listener folder run ```dotnet run```
- From Visual Studio Code - follow these [instructions](https://docs.microsoft.com/en-us/dotnet/core/tutorials/debugging-with-visual-studio-code?pivots=dotnet-6-0)

## Testing by Sending in a HL7 Message

HL7 is a very compact format for representing structured health data. Each message is a sequence of segments, each of which is a sequence of fields. NK1 for example, denotes the Next of Kin - followed by a set of values for different fields.
To test the application, send in a HL7 message. To do so, download and start the [SmartHL7 Sender](https://foldda.com/2020/02/22/download-foldda-integrator/) tool. Load the HL7 message by pressing the Load button, set the URL and port of the listener, and then press start to send the message.

```hl7
MSH|^~\&|MRI|RLC|||201608040916||ADT^A31|798821|D|2.4|||AL|NE||||||
EVN||201608040915|||FMCINTYRE^McIntyre^Frank|201608040915||
PID|1||AH0000512^^^^MR^RLC~3337777333^^^NHS^NH^RLC~A0-B20160804091508120^^^^PI^RLC~A00000516^^^^HUB^RLC||INTERFACE^PATIENTONE^^^MASTER^^L~INTERFACE^TESTPATIENTONE^^^^^M|TESTPAT1|20090314|M||A|^^^^^^P||||EN|S|BU||3337777333|||||||||||N|||||||||
PD1|||^^N81020^PENKETH HEALTH CENTRE^THE HEALTH CENTRE^HONITON WAY^PENKETH, WARRINGTON^WA5 2EY^01925 725644|G9506392||||||||||||||||||
NK1|1|INTERFACE,MOTHER|M|224 MUIRHEAD AVENUE^WEST DERBY^LIVERPOOL^MERSEYSIDE^L13 0BA|111-2222|22233322222|NOK|||||||||||||||||||||||||||||||
NK1|2|INTERFACE,MOTHER|M|224 MUIRHEAD AVENUE^WEST DERBY^LIVERPOOL^MERSEYSIDE^L13 0BA|111-2222|22233322222|NOT|||||||||||||||||||||||||||||||
ROL|1|AD|PP|G9506392^JA^PALIN^^^^^^^^^^XX|||||||||
ROL|2|AD|FHCP|G9506392^JA^PALIN^^^^^^^^^^XX|||||||||
GT1|1|||||||||||||||||||||||||||||||||||||||||||||||||||||||
```

![alt text](../assets/SmartHL7_Sender.png)

Links:

- You can view and verify content of the HL7 message by using [Online HL7 message parser](https://hl7messageparser.azurewebsites.net/).
- Sample messages can be found [here](https://alderheynhsuk.sharepoint.com/sites/AHMicrosoftMindwave/Shared%20Documents/Forms/AllItems.aspx?csf=1&web=1&e=w83z13&xsdata=MDN8MDF8fGYxN2U2ZGY1ODlhMjQyNzViODI4OTI4OWFhNzhkMDVlfDcyZjk4OGJmODZmMTQxYWY5MWFiMmQ3Y2QwMTFkYjQ3fDF8MHw2Mzc3ODI4MjI5MDk0MTg3MDl8R29vZHxWR1ZoYlhOVFpXTjFjbWwwZVZObGNuWnBZMlY4ZXlKV0lqb2lNQzR3TGpBd01EQWlMQ0pRSWpvaVYybHVNeklpTENKQlRpSTZJazkwYUdWeUlpd2lWMVFpT2pFeGZRPT0%3D&sdata=Q0lzMlVna3lIZStJY2hRYk5zVGhud29ObGtadkQ0WFhHR2JneitHK2FETT0%3D&ovuser=72f988bf%2D86f1%2D41af%2D91ab%2D2d7cd011db47%2Canlybeck%40microsoft%2Ecom&OR=Teams%2DHL&CT=1642685500719&sourceId=&params=%7B%22AppName%22%3A%22Teams%2DDesktop%22%2C%22AppVersion%22%3A%2227%2F21110108720%22%7D&cid=233e7961%2D0db6%2D4caf%2D8e11%2De506fc69b5d2&RootFolder=%2Fsites%2FAHMicrosoftMindwave%2FShared%20Documents%2FGeneral%2FMeditech&FolderCTID=0x012000E72A5F74C4C01043969762165F3045C3). One of the sample files is stored in the repo [here](../assets/ADT_MT_ADM_TO_OV_A01.txt), for easier testing.

## Sample Requests

To test the FHIR endpoint send a POST request to the `https://<fhir-url>/$convert-data` endpoint:

- **inputData** is the HL7 message to be converted.
- **templateCollectionReference** is the reference to the template collection in the container registry.
- **rootTemplate** indicates which template to use for the conversion.

```json
{
    "resourceType": "Parameters",
    "parameter": [
        {
            "name": "inputData",
            "valueString": "MSH|^~\\&|HIS|RLC|||201608090839||ADT^A05|799913|D|2.4|||AL|NE||||||\nEVN||201608090839|||FMCINTYRE^McIntyre^Frank|201608091040||\nPID|1||AH0000512^^^^MR^RLC~3337777333^^^NHS^NH^RLC~A0-B20160804091508120^^^^PI^RLC~A00000516^^^^HUB^RLC||INTERFACE^PATIENTONE^^^MASTER^^L~INTERFACE^TESTPATIENTONE^^^^^M|TESTPAT1|20090314|M||A|224 MUIRHEAD AVENUE^WEST DERBY^LIVERPOOL^MERSEYSIDE^L13 0BA^^P||1111111||EN|S|BU|V00000001910|3337777333|||||||||||N|||||||||\nPD1|||^^N81020^PENKETH HEALTH CENTRE^THE HEALTH CENTRE^HONITON WAY^PENKETH, WARRINGTON^WA5 2EY^01925 725644|G9506392||||||||||||||||||\nNK1|1|INTERFACE^MOTHER|M^Mother|224 MUIRHEAD AVENUE^WEST DERBY^LIVERPOOL^MERSEYSIDE^L13 0BA|111-2222|22233322222|NOK|||||||||||||||||||||||||||||||\nNK1|2|INTERFACE^MOTHER|M^Mother|224 MUIRHEAD AVENUE^WEST DERBY^LIVERPOOL^MERSEYSIDE^L13 0BA|111-2222|22233322222|NOT|||||||||||||||||||||||||||||||\nPV1|1|P|OPDG||||PLOSTY^Losty^Paul^^^Prof^^^^^^^XX|||PAES||||||||CLI||NHS|||||||||||||||||||RLC||PRE|||201608091040||||||||G9506392^JA^PALIN^^^^^^^^^^XX|\nPV2|||1ST APPOINTMENT|||||||||||||||||||||||||||||||||N||||||||||||\nROL|1|AD|AT|PLOSTY^Losty^Paul^^^Prof^^^^^^^XX|||||||||\nROL|2|AD|FHCP|G9506392^JA^PALIN^^^^^^^^^^XX|||||||||\nROL|3|AD|PP|G9506392^JA^PALIN^^^^^^^^^^XX|||||||||\nAL1|1|MA|3^NO  KNOWN ALLERGIES^^NO  KNOWN ALLERGIES^^allergy.id|MI||20160804|\nDRG|5.0|||||||||||\n"
        },
        {
            "name": "inputDataType",
            "valueString": "Hl7v2"
        },
        {
            "name": "templateCollectionReference",
            "valueString": "crateampoc.azurecr.io/fhirtemplates:default"
        },
        {
            "name": "rootTemplate",
            "valueString": "ADT_A01"
        }
    ]
}
```

The results of the conversion request are then posted to the `https://<fhir-url>/` endpoint.

## Resources

- [How to convert data to FHIR](https://docs.microsoft.com/en-us/azure/healthcare-apis/azure-api-for-fhir/convert-data#customize-templates)
- [How to create a conversion template](https://docs.microsoft.com/en-us/azure/healthcare-apis/azure-api-for-fhir/convert-data#create-a-conversion-template)
- [FHIR Converter VS Code Extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-health-fhir-converter)

## Useful VS Code Extensions

- [FHIR Converter VS Code Extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-health-fhir-converter) - tools an instructions for how to create, pull, and push templates to the container registry. Also allows you to easily work with templates locally to convert HL7 messages to FHIR resources. (See the instructions in the README.md file for more information.)
- [FHIR Tools VS Code Extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-health-fhir-tools) - tools for working with FHIR resources.
- [HL7 Language Support for VS Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-health-hl7-language-support) - language support for HL7 messages, hover to see information on fields and values.
