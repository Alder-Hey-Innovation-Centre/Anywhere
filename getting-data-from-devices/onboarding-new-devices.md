# Onboarding new devices

In order to on-board a new device, or device vendor, you need to:

1. Create a new API key in API management and provide the vendor with this API key.
1. Discuss with the vendor, if they can conform to the standard message format, or if you need to add custom templates.
1. Create new custom templates if needed.

## Create a new API key in API management

1. In Azure Portal - locate the API Management service - and select Subscriptions

   ![API Management](../assets/api-management.png)

1. Select `Add subscription` and fill out the details. The Name of the subscription will be used as the deviceType in the measurement messages, so this should preferably be fairly short, unique and avoid spaces or special characters.

    ![Add subscription](../assets/api-management-new-subscription.png)

1. By selecting `...` you can show/hide the keys, delete the key or otherwise manage it. You need to copy the primary (or secondary) key and give it to the vendor.

    ![API management keys](../assets/show-hide-keys.png)

1. Once the API key is created, you can use it to send messages to the device on the URL `https://apim-anywhere-engine-dev.azure-api.net/StandardAPI/` by adding `Ocp-Apim-Subscription-Key` to the header of the request.

    ![Send message](../assets/key-for-standard-api.png)

1. When the vendor sends a measurement message to the device, a `deviceType` with the same name as the Name of the subscription will be added to the message.

    ![Message with deviceType](../assets/send-data-to-standard-api.png)

## The standard message format

A number of standard mapping templates are already available in the `mapping-templates` folder.

A list of available mappings and expected data input is available in the document on [standard templates](IOMT-templates/standard-templates.md).

If the vendor is unable to conform to the standard message format, you will need to [create custom templates](IOMT-templates/creating-custom-templates.md).

## Sample app

A sample app using the standard API and the templates provided out-of-the-box can be found in `/spikes/AnywhereQtSampleApp/` folder. See this [README file](https://dev.azure.com/AlderHey/AlderHey%20Anywhere/_git/AlderHeyAnywhere?path=/spikes/AnywhereQtSampleApp/README.md) to learn more.

![Silly advert](../assets/silly-advert.png)
