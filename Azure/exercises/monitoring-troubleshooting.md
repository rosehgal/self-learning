# Monitoring and Troubleshooting

## Exercise 1

1. Create a Ubuntu 16.04 VM of size "Standard_B1s" with a managed disk. Make sure Boot diagnostics and Guest OS diagnostics are enabled.
2. SSH into the machine and install Apache web server.
3. In Azure monitor, Create a line chart, add the "Network In" as well as the "Network Out" of the VM created earlier as the metrics.
4. Provide a title for your chart and pin it to your dashboard.
5. Export your chart data and download it as an excel file.
6. Explore the resource health of your VM.

### Results

- Go to Monitor > Metrics (preview) > Choose your subscription, resource group and resource as the VM.
- Select Network In and Network Out from available metrics.

![](/Azure/images/mon-ex1.png)

## Exercise 2

1. Create a Ubuntu 16.04 VM of size "Standard_B1s" with a managed disk. Make sure Boot diagnostics and Guest OS diagnostics are enabled.
2. Download the screenshot of your VM and check whether the login screen is shown.
3. Download the serial log of the VM and go through it.
4. Change the password of your VM using the Azure portal as well as the Azure CLI.
5. Run a shell command inside your VM using the Azure portal as well as the Azure CLI.
6. Access the Azure serial console and stop the SSH service. Try to SSH into the VM and also check whether the serial console still works.
7. Check if the serial console can be accessed through Azure CLI or SDKs.
8. Download the list of effective security rules of your VM.

### Results

Files:
[Screenshot](/Azure/files/monex1-vm1.cec1467c-0d79-42fe-82bb-82218d52927a.screenshot.bmp),
[Serial Log](/Azure/files/monex1-vm1.cec1467c-0d79-42fe-82bb-82218d52927a.serialconsole.log),
[Effective Security rules](/Azure/files/monex1-vm1971_EffectiveSecurityRules.csv)

- Step 2 and 3

  - Go to Virtual machines > [your VM] > Boot diagnostics and download the screenshot and serial log.

- Step 4

  - Go to Virtual machines > [your VM] > Reset password
  - `az vm user update -g monex1 -n monex1-vm1 -u <username> -p "<password>"`

- Step 5
> https://docs.microsoft.com/en-us/azure/virtual-machines/linux/run-command

  - Go to Virtual machines > [your VM] > Run command
  - For CLI

```bash
$ az vm run-command invoke -g monex1 -n monex1-vm1  --command-id RunShellScript --scripts "echo hi; ip a; hostname -i"
 - Running ..
{
  "endTime": "2018-08-05T11:31:27.990809+00:00",
  "error": null,
  "name": "e20f4d1f-5a24-44b5-98e6-c0033d2e354a",
  "output": [
    {
      "code": "ProvisioningState/succeeded",
      "displayStatus": "Provisioning succeeded",
      "level": "Info",
      "message": "Enable succeeded: \n[stdout]\nhi\n1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000\n    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00\n    inet 127.0.0.1/8 scope host lo\n       valid_lft forever preferred_lft forever\n    inet6 ::1/128 scope host \n       valid_lft forever preferred_lft forever\n2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000\n    link/ether 00:0d:3a:1f:17:42 brd ff:ff:ff:ff:ff:ff\n    inet 10.0.1.4/24 brd 10.0.1.255 scope global eth0\n       valid_lft forever preferred_lft forever\n    inet6 fe80::20d:3aff:fe1f:1742/64 scope link \n       valid_lft forever preferred_lft forever\n10.0.1.4\n\n[stderr]\n"
    }
  ],
  "startTime": "2018-08-05T11:31:08.990637+00:00",
  "status": "Succeeded"
}
```

- Step 6

  - An already connected SSH session does not stop on shutting down the SSH service.
  - A new SSH session cannot be established.
  - The existing serial console works as long as it is not disconnected. New serial consoles do not work.
    - If a new serial console is opened, the existing serial console connection gets closed.

- Step 7

  - Serial Console cannot be accessed via CLI. ???
  - Use 'Run command' to restart ssh.

- Step 8

  - Go to Virtual machines > [your VM] > Networking > Effective security rules > Download

## Exercise 3

1. Create a Ubuntu 16.04 VM of size "Standard_B1s" with a managed disk. Make sure Boot diagnostics and Guest OS diagnostics are enabled.
2. Create a new alert rule with your VM as the target.
3. Use CPU percentage of the VM as the alert criteria such that whenever the average CPU percentage is greater than 75 % for a period of one minute, then the alert should be triggered.
4. Create and choose an action group that has the following actions.
  1. Send an email and a SMS to your email ID and your phone number.
  2. Send a notification to your webhook URI (if possible, write a simple web server that accepts a HTTP POST request and print the request body).
5. Also, explore the other action types available in the alert action group. Finally, Save and create an alert rule.
6. SSH into your VM, install the "stress" utility and increase the CPU utilization to hundred percent.
7. After few minutes, check whether you have received a notification from azure.

### Results

- Go to Monitor > Alerts > New Alert Rule

- Step 4.2
  - Create a file with the following contents on a server with 'nodejs 8.x' or higher installed and run the server with the command `node <file>`

```javascript
const http = require('http')
const port = 80

const requestHandler = (request, response) => {
  console.log(request.url)
  body = ''
  request.on('data', function (data) {
    body += data;
  });
  request.on('end', function () {
    console.log("Body: " + body);
  });
  response.end('Hello Node.js Server!')
}

const server = http.createServer(requestHandler)

server.listen(port, (err) => {
  if (err) {
    return console.log('something bad happened', err)
  }

  console.log(`server is listening on ${port}`)
})
```

Response in server logs for the incoming webhook. The alert rule gets triggered when the alert criteria is activated as well as deactivated.

```json
/
Body: {"schemaId":"AzureMonitorMetricAlert","data":{
  "version": "2.0",
  "status": "Activated",
  "context": {
    "timestamp": "2018-08-05T13:24:48.2134694Z",
    "id": "/subscriptions/fda10a14-c1d3-4e91-9b04-b097c287cb38/resourceGroups/monex1/providers/microsoft.insights/metricAlerts/VM%20CPU%20excess%20util",
    "name": "VM CPU excess util",
    "description": "",
    "conditionType": "SingleResourceMultipleMetricCriteria",
    "condition": {
      "windowSize": "PT1M",
      "allOf": [
        {
          "metricName": "Percentage CPU",
          "dimensions": [
            {
              "name": "ResourceId",
              "value": "cec1467c-0d79-42fe-82bb-82218d52927a"
            }
          ],
          "operator": "GreaterThan",
          "threshold": "75",
          "timeAggregation": "PT1M",
          "metricValue": 1.0
        }
      ]
    },
    "subscriptionId": "fda10a14-c1d3-4e91-9b04-b097c287cb38",
    "resourceGroupName": "monex1",
    "resourceName": "monex1-vm1",
    "resourceType": "Microsoft.Compute/virtualMachines",
    "resourceId": "/subscriptions/fda10a14-c1d3-4e91-9b04-b097c287cb38/resourceGroups/monex1/providers/Microsoft.Compute/virtualMachines/monex1-vm1",
    "portalLink": "https://portal.azure.com/#resource//subscriptions/fda10a14-c1d3-4e91-9b04-b097c287cb38/resourceGroups/monex1/providers/Microsoft.Compute/virtualMachines/monex1-vm1"
  },
  "properties": {}
}}
/
Body: {"schemaId":"AzureMonitorMetricAlert","data":{
  "version": "2.0",
  "status": "Deactivated",
  "context": {
    "timestamp": "2018-08-05T13:37:48.3244319Z",
    "id": "/subscriptions/fda10a14-c1d3-4e91-9b04-b097c287cb38/resourceGroups/monex1/providers/microsoft.insights/metricAlerts/VM%20CPU%20excess%20util",
    "name": "VM CPU excess util",
    "description": "",
    "conditionType": "SingleResourceMultipleMetricCriteria",
    "condition": {
      "windowSize": "PT1M",
      "allOf": [
        {
          "metricName": "Percentage CPU",
          "dimensions": [
            {
              "name": "ResourceId",
              "value": "cec1467c-0d79-42fe-82bb-82218d52927a"
            }
          ],
          "operator": "GreaterThan",
          "threshold": "75",
          "timeAggregation": "PT1M",
          "metricValue": 0.0
        }
      ]
    },
    "subscriptionId": "fda10a14-c1d3-4e91-9b04-b097c287cb38",
    "resourceGroupName": "monex1",
    "resourceName": "monex1-vm1",
    "resourceType": "Microsoft.Compute/virtualMachines",
    "resourceId": "/subscriptions/fda10a14-c1d3-4e91-9b04-b097c287cb38/resourceGroups/monex1/providers/Microsoft.Compute/virtualMachines/monex1-vm1",
    "portalLink": "https://portal.azure.com/#resource//subscriptions/fda10a14-c1d3-4e91-9b04-b097c287cb38/resourceGroups/monex1/providers/Microsoft.Compute/virtualMachines/monex1-vm1"
  },
  "properties": {}
}}
```

## Exercise 4

1. Create a new alert rule targeting all virtual machines in your resource group.
2. Add an alert criteria such that whenever a new VM is created in your resource group, the alert should be triggered.
3. Create and choose an action group that has the following actions. Then, save and create the alert rule
  1. Send an email and a SMS to your email ID and your phone number.
  2. Send a notification to your webhook URI (if possible, write a simple web server that accepts a HTTP POST request and print the request body).
4. Launch a new Ubuntu 16.04 VM in your resource group.
5. Wait for few minutes and check whether you have received a notification from azure.

### Results

- Webhook response

```json
{
  "schemaId": "Microsoft.Insights/activityLogs",
  "data": {
    "status": "Activated",
    "context": {
      "activityLog": {
        "authorization": {
          "action": "Microsoft.Compute/virtualMachines/write",
          "scope": "/subscriptions/fda10a14-c1d3-4e91-9b04-b097c287cb38/resourcegroups/monex1/providers/Microsoft.Compute/virtualMachines/vm-del1"
        },
        "channels": "Debug",
        "claims": "{\"aud\":\"https://management.core.windows.net/\",\"iss\":\"https://sts.windows.net/3cbcc3d3-094d-4006-9849-0d11d61f484d/\",\"iat\":\"1533475902\",\"nbf\":\"1533475902\",\"exp\":\"1533479802\",\"http://schemas.microsoft.com/claims/authnclassreference\":\"1\",\"aio\":\"42BgYBB3fF7jwSS76YVFvZB0XnUS38oHvxa/5oiy6J6s+H5dcTYA\",\"appid\":\"04b07795-8ddb-461a-bbee-02f9e1bf7b46\",\"appidacr\":\"0\",\"e_exp\":\"262800\",\"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname\":\"XXX\",\"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname\":\"XXX\",\"groups\":\"b97a2df7-d8ac-4a63-a2b2-9ff4e2ec9e44,095da0d0-d887-4fd9-962a-38ead0ae2bea,8b5a6b59-9a0e-451d-b052-bc536a2f6092,d6d3b443-0033-4137-be36-331650a121c0,acad2e11-94fc-4036-b152-c26d6ef64a3d,9444a7b9-e9c3-4514-bbc7-40689ea04cbd,7cd0a2ba-d4b7-4a44-8672-4f066c635496,a6c239bd-566e-45ff-91b0-2c5a583f5288,e062f601-e0df-4287-acac-b52811d7dde8,b794e453-3ca2-416f-bc67-3def9b2033a7,6ca8a8d3-3378-44f4-a890-7cd512c4d0be,24de82ec-385a-41d8-947b-02826ff8f3ce,b4a0f3dd-8154-4543-ad45-3bc83611e635,37a82a63-9705-4da1-93b9-22e8965dd175,88d07b84-b966-442d-8acc-47cc9be534bf,4359237d-4658-4c00-a05a-9982c46bddd7,c78c518c-b860-4406-94ce-20e1ded08436,9ef132e4-2deb-476f-aa37-89af8f045fb2,c521c129-11b9-43e0-9946-1190394aa49d,dd67a34a-55f2-4364-a462-904b3f9d7784,a2c7b2b8-463a-42bc-993f-f269326f04ae,47a17e9f-807b-4fee-929c-0b106a56b2fa,28f45272-3528-45de-b2cd-09dd5d1c9af5,574440ff-5e61-4021-9993-f5e4643f47b9,2e56ec39-cf5f-4014-a77e-78c1e6607dfe\",\"in_corp\":\"true\",\"ipaddr\":\"49.206.1.11\",\"name\":\"XXX XXX\",\"http://schemas.microsoft.com/identity/claims/objectidentifier\":\"78b87a97-9330-4d56-a655-da983b3a40e0\",\"onprem_sid\":\"S-1-5-21-1592216029-2136481655-317593308-892113\",\"puid\":\"1003BFFD9925BBF8\",\"http://schemas.microsoft.com/identity/claims/scope\":\"user_impersonation\",\"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier\":\"_AfT8-RqJuVD3dnP4tz4o5xtqyxoN-8seK1cXT-tmZg\",\"http://schemas.microsoft.com/identity/claims/tenantid\":\"3cbcc3d3-094d-4006-9849-0d11d61f484d\",\"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name\":\"example@email.com\",\"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn\":\"example@email.com\",\"uti\":\"UMXgtBBneUaMFBKhCGESAA\",\"ver\":\"1.0\"}",
        "caller": "example@email.com",
        "correlationId": "e7d65fe4-5dda-415d-b06a-81701df46e6b",
        "description": "",
        "eventSource": "Administrative",
        "eventTimestamp": "2018-08-05T14:01:01.7175736+00:00",
        "eventDataId": "890cb015-22e3-4c7b-9d7c-a500409767e5",
        "level": "Informational",
        "operationName": "Microsoft.Compute/virtualMachines/write",
        "operationId": "902353fe-2d3a-45c3-912b-fb44ac4963a4",
        "properties": {
          "statusCode": "Accepted",
          "statusMessage": "\"Resource provisioning is in progress.\""
        },
        "resourceId": "/subscriptions/fda10a14-c1d3-4e91-9b04-b097c287cb38/resourcegroups/monex1/providers/Microsoft.Compute/virtualMachines/vm-del1",
        "resourceGroupName": "monex1",
        "resourceProviderName": "Microsoft.Compute",
        "status": "Accepted",
        "subStatus": "",
        "subscriptionId": "fda10a14-c1d3-4e91-9b04-b097c287cb38",
        "submissionTimestamp": "2018-08-05T14:01:23.1006139+00:00",
        "resourceType": "Microsoft.Compute/virtualMachines"
      }
    },
    "properties": {}
  }
}
```

## Exercise 5

1. Explore the Azure advisor and go through the recommendations.
2. Download and view the CSV and PDF formats of Azure Advisor recommendations.
3. Explore the Azure security centre and find out the security recommendations suggested.
4. Evaluate the features and pricing models of Azure security centre.
5. Try to access the Azure Advisor recommendations using Azure CLI.

### Results

Files:
[Advisor CSV](/Azure/files/Advisor_2018-08-05T13_48_02.911Z.csv),
[Advisor PDF](/Azure/files/Advisor_2018-08-05T13_48_12.174Z.pdf)

- Step 1 and 2

  - Go to Advisor and download the recommendations.

- Step 3 and 4

![](/Azure/images/mon-ex5.png)
![](/Azure/images/mon-ex5-pricing.png)

- Step 5

```bash
$ az advisor recommendation list
[
  {
    "category": "Cost",
    "id": "/subscriptions/fda10a14-c1d3-4e91-9b04-b097c287cb38/providers/Microsoft.Advisor/recommendations/60db5849-6bb0-9afa-e258-4459154e0b58",
    "impact": "High",
    "impactedField": "Microsoft.ReservedInstances/reservedInstances",
    "impactedValue": "3 virtual machines",
    "lastUpdated": "2018-08-03T02:02:07.510575+00:00",
    "metadata": null,
    "name": "60db5849-6bb0-9afa-e258-4459154e0b58",
    "recommendationTypeId": null,
    "risk": "Error",
    "shortDescription": {
      "problem": "Buy virtual machine reserved instances to save money over pay-as-you-go costs",
      "solution": "Buy virtual machine reserved instances to save money over pay-as-you-go costs"
    },
    "suppressionIds": null,
    "type": "Microsoft.Advisor/recommendations"
  },
  {
    "category": "Cost",
    "id": "/subscriptions/fda10a14-c1d3-4e91-9b04-b097c287cb38/providers/Microsoft.Advisor/recommendations/557e03b1-961e-96f1-b321-155fe410d3cb",
    "impact": "High",
    "impactedField": "Microsoft.ReservedInstances/reservedInstances",
    "impactedValue": "1 virtual machine",
    "lastUpdated": "2018-08-05T02:02:41.220490+00:00",
    "metadata": null,
    "name": "557e03b1-961e-96f1-b321-155fe410d3cb",
    "recommendationTypeId": null,
    "risk": "Error",
    "shortDescription": {
      "problem": "Buy virtual machine reserved instances to save money over pay-as-you-go costs",
      "solution": "Buy virtual machine reserved instances to save money over pay-as-you-go costs"
    },
    "suppressionIds": null,
    "type": "Microsoft.Advisor/recommendations"
  },
  {
    "category": "Security",
    "id": "/subscriptions/fda10a14-c1d3-4e91-9b04-b097c287cb38/providers/Microsoft.Advisor/recommendations/efe94c7e-9115-327d-15f2-9f45304136ee",
    "impact": "High",
    "impactedField": "",
    "impactedValue": "",
    "lastUpdated": "2018-08-05T13:45:14.339706+00:00",
    "metadata": null,
    "name": "efe94c7e-9115-327d-15f2-9f45304136ee",
    "recommendationTypeId": null,
    "risk": "None",
    "shortDescription": {
      "problem": "Improve the security of your Azure resources",
      "solution": "Follow Security Center recommendations"
    },
    "suppressionIds": null,
    "type": "Microsoft.Advisor/recommendations"
  },
  {
    "category": "Security",
    "id": "/subscriptions/fda10a14-c1d3-4e91-9b04-b097c287cb38/providers/Microsoft.Advisor/recommendations/0503f2b4-c080-1643-7e65-ff61e246ce42",
    "impact": "High",
    "impactedField": "",
    "impactedValue": "",
    "lastUpdated": "2018-08-05T13:45:14.339706+00:00",
    "metadata": null,
    "name": "0503f2b4-c080-1643-7e65-ff61e246ce42",
    "recommendationTypeId": null,
    "risk": "None",
    "shortDescription": {
      "problem": "Improve the security of your Azure resources",
      "solution": "Follow Security Center recommendations"
    },
    "suppressionIds": null,
    "type": "Microsoft.Advisor/recommendations"
  },
  {
    "category": "Security",
    "id": "/subscriptions/fda10a14-c1d3-4e91-9b04-b097c287cb38/resourceGroups/MONEX1/providers/Microsoft.Compute/virtualMachines/monex1-vm1/providers/Microsoft.Advisor/recommendations/5f1395a7-241c-c7b0-f925-fe95ff5b1eec",
    "impact": "High",
    "impactedField": "Microsoft.Compute/virtualMachines",
    "impactedValue": "monex1-vm1",
    "lastUpdated": "2018-08-05T13:45:14.340708+00:00",
    "metadata": null,
    "name": "5f1395a7-241c-c7b0-f925-fe95ff5b1eec",
    "recommendationTypeId": null,
    "resourceGroup": "MONEX1",
    "risk": "None",
    "shortDescription": {
      "problem": "Improve the security of your Azure resources",
      "solution": "Follow Security Center recommendations"
    },
    "suppressionIds": null,
    "type": "Microsoft.Advisor/recommendations"
  },
  {
    "category": "Security",
    "id": "/subscriptions/fda10a14-c1d3-4e91-9b04-b097c287cb38/resourceGroups/monex1/providers/Microsoft.Network/publicIPAddresses/monex1-vm1-ip/providers/Microsoft.Advisor/recommendations/07f89f7b-6d3a-09e0-1d56-e02c850a4027",
    "impact": "High",
    "impactedField": "Microsoft.Network/publicIPAddresses",
    "impactedValue": "monex1-vm1-ip",
    "lastUpdated": "2018-08-05T13:45:14.340708+00:00",
    "metadata": null,
    "name": "07f89f7b-6d3a-09e0-1d56-e02c850a4027",
    "recommendationTypeId": null,
    "resourceGroup": "monex1",
    "risk": "None",
    "shortDescription": {
      "problem": "Improve the security of your Azure resources",
      "solution": "Follow Security Center recommendations"
    },
    "suppressionIds": null,
    "type": "Microsoft.Advisor/recommendations"
  },
  {
    "category": "Security",
    "id": "/subscriptions/fda10a14-c1d3-4e91-9b04-b097c287cb38/resourceGroups/monex1/providers/Microsoft.Network/virtualNetworks/monex1-vnet/subnets/default/providers/Microsoft.Advisor/recommendations/c78e891a-5139-a85f-94d2-8304474a0984",
    "impact": "High",
    "impactedField": "Microsoft.Network/virtualNetworks/subnets",
    "impactedValue": "monex1-vnet/default",
    "lastUpdated": "2018-08-05T13:45:14.340708+00:00",
    "metadata": null,
    "name": "c78e891a-5139-a85f-94d2-8304474a0984",
    "recommendationTypeId": null,
    "resourceGroup": "monex1",
    "risk": "None",
    "shortDescription": {
      "problem": "Improve the security of your Azure resources",
      "solution": "Follow Security Center recommendations"
    },
    "suppressionIds": null,
    "type": "Microsoft.Advisor/recommendations"
  },
  {
    "category": "Security",
    "id": "/subscriptions/fda10a14-c1d3-4e91-9b04-b097c287cb38/resourceGroups/MONEX1/providers/Microsoft.Compute/virtualMachines/monex1-vm1/providers/Microsoft.Advisor/recommendations/79c0f72b-fc36-3272-b4f0-82a9e491369c",
    "impact": "High",
    "impactedField": "Microsoft.Compute/virtualMachines",
    "impactedValue": "monex1-vm1",
    "lastUpdated": "2018-08-05T13:45:14.339706+00:00",
    "metadata": null,
    "name": "79c0f72b-fc36-3272-b4f0-82a9e491369c",
    "recommendationTypeId": null,
    "resourceGroup": "MONEX1",
    "risk": "None",
    "shortDescription": {
      "problem": "Improve the security of your Azure resources",
      "solution": "Follow Security Center recommendations"
    },
    "suppressionIds": null,
    "type": "Microsoft.Advisor/recommendations"
  },
  {
    "category": "Security",
    "id": "/subscriptions/fda10a14-c1d3-4e91-9b04-b097c287cb38/resourceGroups/MONEX1/providers/Microsoft.Compute/virtualMachines/monex1-vm1/providers/Microsoft.Advisor/recommendations/85ff8225-7af7-16c5-aa40-010d7b8db98e",
    "impact": "High",
    "impactedField": "Microsoft.Compute/virtualMachines",
    "impactedValue": "monex1-vm1",
    "lastUpdated": "2018-08-05T13:45:14.340708+00:00",
    "metadata": null,
    "name": "85ff8225-7af7-16c5-aa40-010d7b8db98e",
    "recommendationTypeId": null,
    "resourceGroup": "MONEX1",
    "risk": "None",
    "shortDescription": {
      "problem": "Improve the security of your Azure resources",
      "solution": "Follow Security Center recommendations"
    },
    "suppressionIds": null,
    "type": "Microsoft.Advisor/recommendations"
  },
  {
    "category": "Security",
    "id": "/subscriptions/fda10a14-c1d3-4e91-9b04-b097c287cb38/resourceGroups/monex1/providers/Microsoft.Network/publicIPAddresses/monex1-vm1-ip/providers/Microsoft.Advisor/recommendations/bd70c240-e549-fc06-0431-0175794ef7d5",
    "impact": "High",
    "impactedField": "Microsoft.Network/publicIPAddresses",
    "impactedValue": "monex1-vm1-ip",
    "lastUpdated": "2018-08-05T13:45:14.340708+00:00",
    "metadata": null,
    "name": "bd70c240-e549-fc06-0431-0175794ef7d5",
    "recommendationTypeId": null,
    "resourceGroup": "monex1",
    "risk": "None",
    "shortDescription": {
      "problem": "Improve the security of your Azure resources",
      "solution": "Follow Security Center recommendations"
    },
    "suppressionIds": null,
    "type": "Microsoft.Advisor/recommendations"
  },
  {
    "category": "HighAvailability",
    "id": "/subscriptions/fda10a14-c1d3-4e91-9b04-b097c287cb38/providers/Microsoft.Advisor/recommendations/34db6971-02d3-c5bd-6740-b6c3670c54c8",
    "impact": "Low",
    "impactedField": "Microsoft.Subscriptions/subscriptions",
    "impactedValue": "fda10a14-c1d3-4e91-9b04-b097c287cb38",
    "lastUpdated": "2018-08-05T10:42:15.758435+00:00",
    "metadata": null,
    "name": "34db6971-02d3-c5bd-6740-b6c3670c54c8",
    "recommendationTypeId": null,
    "risk": "Warning",
    "shortDescription": {
      "problem": "Create an Azure service health alert",
      "solution": "Create an Azure service health alert"
    },
    "suppressionIds": null,
    "type": "Microsoft.Advisor/recommendations"
  }
]
```
