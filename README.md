# web-apps-node-iot-hub-data-visualization

This repo contains code for a web application, which can read temperature and humidity data from IoT Hub and show the real-time data in a line chart on the web page.

## Browser compatiblity

| Browser | Least Version |
| --- | --- |
| IE | 10 |
| Edge | 14 |
| Firefox | 50 |
| Chrome | 49 |
| Safari | 10 |
| Opera | 43 |
| iOS Safari | 9.3 |
| Opera Mini | ALL |
| Android Browser | 4.3 |
| Chrome for Android | 56 |

This tutorial shows how to set up a nodejs website to visualize device data streaming to an [Azure IoT Hub](https://azure.microsoft.com/en-us/services/iot-hub) using the [event hub SDK](https://www.npmjs.com/package/@azure/event-hubs). In this tutorial, you learn how to:

- Create an Azure IoT Hub
- Configure your IoT hub with a device, a consumer group, and use that information for connecting a device and a service application
- On a website, register for device telemetry and broadcast it over a web socket to attached clients
- In a web page, display device data in a chart

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/) before you begin.

> You may follow the manual instructions below, or refer to the Azure CLI notes at the bottom to learn how to automate these steps.

## Sign in to the Azure portal

Sign in to the [Azure portal](https://portal.azure.com/).

## Create and configure your IoT hub

1. [Create](https://mportal.azure.com/#create/Microsoft.IotHub), or [select an existing](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Devices%2FIotHubs), IoT hub.
    - For **Size and Scale**, you may use "F1: Free tier".

1. Select the **Shared access policies** menu item, open the **service** policy, and copy a connection string to be used in later steps.

1. Select **Built-in endpoints -> Events**, add a new consumer group (e.g. "monitoring"), and then change focus to save it. Note the name to be used in later steps.

1. Select **IoT devices**, create a device, and copy device the connection string.

## Send device data

- For quickest results, simulate temperature data using the [Raspberry Pi Azure IoT Online Simulator](https://azure-samples.github.io/raspberry-pi-web-simulator/#Getstarted). Paste in the **device connection string**, and select the **Run** button.

- If you have a physical Raspberry Pi and BME280 sensor, you may measure and report real temperature and humidity values by following the [Connect Raspberry Pi to Azure IoT Hub (Node.js)](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-raspberry-pi-kit-node-get-started) tutorial.

## Run the visualization website

Clone this repo. For a quick start, it is recommended to run the site locally, but you may also deploy it to Azure. Follow the corresponding option below.

### Inspect the code

Server.js is a service-side script that initializes the web socket and event hub wrapper class, and provides a callback to the event hub for incoming messages to broadcast them to the web socket.

Event-hub-reader.js is a service-side script that connects to the IoT hub's event hub using the specified connection string and consumer group, extracts the DeviceId and EnqueuedTimeUtc from metadata, and then relays message using the provided callback method.

Chart-device-data.js is a client-side script that listens on the web socket, keeps track of each DeviceId and stores the the last 50 points of incoming device data. It then binds the selected device data to the chart object.

Index.html handles the UI layout for the web page, and references the necessary scripts for client-side logic.

### Run locally

1. To pass parameters to the website, you may use environment variables or parameters.
    - Open a command prompt or PowerShell terminal and set the environment variables **IotHubConnectionString** and **EventHubConsumerGroup**. Syntax for command prompt is `set key=value` and for PowerShell is `$env:key="value"`.

    - Or, if you are debugging with [VS Code](https://code.visualstudio.com/docs/nodejs/nodejs-debugging), you can edit the launch.json file and add these values in the env property.

1. Run `npm install` to download and install referenced packages.

1. Run the website one of the following ways:
    - From the command-line use `npm start` (if necessary, with the parameters mentioned above)
    - In VS Code, press F5 to start debugging (if necessary, with parameters mentioned above set in launch.json).

1. Watch for console output from the website. Depending on how you started it, you'll see it in the console where you typed `npm start`, or if in VS Code you'll find it in the **DEBUG CONSOLE**.

1. If you are using VS Code, you may set breakpoints in any of the server-side scripts and step through the code to watch the code work.

1. Open a browser to [localhost:3000](http://localhost:3000).

### Use an Azure App Service

The approach here is to create a website in Azure, configure it to deploy using git where it hosts a remote repo, and push your local branch to that repo.

1. Change the code in public/js/chart-device-data.js at line 5 to use `wss://` protocol.
    - This is required because in Azure the website will be hosted over SSL, and we'll want the web socket connection between server and client to also be encrypted.

1. Create a [Web App](https://ms.portal.azure.com/#create/Microsoft.WebSite).
    - OS: Windows
    - Publish: Code
    - App Service Plan: choose the cheapest plan (e.g. Dev / Test | F1)

1. Select **Settings | Configuration**
    1. Select **Application settings** and add key/value pairs for:
        - Add **IotHubConnectionString** and the corresponding value.
        - Add **EventHubConsumerGroup** and the corresponding value.
    1. Select **General settings** and turn **Web socksets** to **On**.

1. Select **Deployment Options**, and configure for a **Local Git** to deploy your web app.

1. Push the repo's code to the git repo URL in last step with:
    - In the **Overview** page, find the **Git clone URL**, using the **App Service build service** build provider. Then run the following commands:

        ```cmd
        git clone https://github.com/Azure-Samples/web-apps-node-iot-hub-data-visualization.git
        cd web-apps-node-iot-hub-data-visualization
        git remote add webapp <Git clone URL>
        git push webapp master:master
        ```

    - When prompted for credentials, select **Deployment Center | Deployment Credentials** in the Azure portal and use the auto-generated app credentials, or create your own.

1. After the push and deploy has finished, you can view the page to see the real-time data chart. Find the URL in **Overview** in the Essentials section.

## Troubleshooting

If you encounter any issues with this sample, try the following steps. If you still encounter issues, drop us a note in the Issues tab.

- Client issues:
  - If a device does not appear in the list, or no graph is being drawn, ensure the sample application is running on your device.
  - In the browser, open the developer tools (in many browsers the F12 key will open it), and find the Console. Look for any warnings or errors printed here.
    - Also, you can debug client-side script in /js/chat-device-data.js.
- Local website issues:
  - Watch the output in the window where node was launched for console output.
  - Debug the server code, namely server.js and /scripts/event-hub-reader.js.
- Azure App Service issues:
  - Open **Monitoring | Diagnostic logs**. Turn Application Logging (File System) to on, Level to Error, and then Save. Then open **Log stream**.
  - Open **Development Tools | Console** and validate node and npm versions with `node -v` and `npm -v`.
  - If you see an error about not finding a package, you may have run the steps out of order. When the site is deployed (with `git push`) the app service runs `npm install` which runs based on the current version of node it has configured. If that is changed in configuration later, you'll need to make a meaningless change to the code and push again.

## CLI documentation

In order to automate the steps to deploy to Azure, consider reading the following documentation and using the corresponding commands.

- [Azure login](https://docs.microsoft.com/en-us/cli/azure/reference-index?view=azure-cli-latest#az-login)
- [Resource group create](https://docs.microsoft.com/en-us/cli/azure/group?view=azure-cli-latest#az-group-create)
- [IoT Hub create](https://docs.microsoft.com/en-us/cli/azure/iot/hub?view=azure-cli-latest#az-iot-hub-create)
- [IoT Hub consumer group](https://docs.microsoft.com/en-us/cli/azure/iot/hub/consumer-group?view=azure-cli-latest#az-iot-hub-consumer-group-create)
- [ServicePlan create](https://docs.microsoft.com/en-us/cli/azure/appservice/plan?view=azure-cli-latest#az-appservice-plan-create)
- [WebApp create](https://docs.microsoft.com/en-us/cli/azure/webapp?view=azure-cli-latest#az-webapp-create)
- [WebApp config](https://docs.microsoft.com/en-us/cli/azure/webapp/config?view=azure-cli-latest#az-webapp-config-set)
- [WebApp appsettings](https://docs.microsoft.com/en-us/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set)
- [WebApp deployment credentials](https://docs.microsoft.com/en-us/cli/azure/webapp/deployment?view=azure-cli-latest#az-webapp-deployment-list-publishing-credentials)
- [WebApp deployment source](https://docs.microsoft.com/en-us/cli/azure/webapp/deployment/source?view=azure-cli-latest)

```az cli
az login --subscription <subscriptionId>

az group create --name <rgName> --location "<location>"

az iot hub create --name <hubName> --resource-group <rgName> --location "<location>" --sku S1
az iot hub consumer-group create --name <consumerGroupName> --hub-name <hubName> --resource-group <rgName>

az appservice plan create --resource-group <rgName> --name <planName> --sku F1 --location "<location>"
az webapp create --name <websiteName> --resource-group <rgName> --plan <planName> --runtime "node|10.6"
az webapp update --name <websiteName> --resource-group <rgName> --https-only true
az webapp config set --name <websiteName> --resource-group <rgName> --web-sockets-enabled true
az webapp config appsettings set --name <websiteName> --resource-group <rgName> --settings IotHubConnectionString=HostName=drwPeaStream.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=BxmjTaz6nlU7BIYDKwdE46pV2L+/Q5T0sW07hRwpI9Y= EventHubConsumerGroup=<consumerGroup>

az webapp deployment list-publishing-credentials --name <websiteName> --resource-group <rgName>
az webapp deployment source config-local-git --name <websiteName> --resource-group <rgName>

git remote add azure <websiteGitUrl>
git push azure master:<localBranch>
```

## Conclusion

In this tutorial, you learned how to:

- Create an Azure IoT Hub
- Configure your IoT hub with a device, a consumer group, and use that information for connecting a device and a service application
- On a website, register for device telemetry and broadcast it over a web socket to attached clients
- In a web page, display device data in a chart