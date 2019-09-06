# SAP and Azure integration demo with Microsoft Teams and Outlook365

# Introduction 
The intention of this demo is to implement a basic scenario for integrating and displaying SAP ERP data in a Teams channel or an e-mail generated by Outlook. 
This can be conveniently achieved by consuming OData-services of the SAP Netweaver Gateway Server with a Logic App. 
To make this demo easily reproducible, the public SAP Gateway demo system [E5]( https://blogs.sap.com/2017/12/05/new-sap-gateway-demo-system-available/) is used. 

Ideally this first demo is an inspiration for more advanced Microsoft Modern Workplace integration demos. 
For example creating a Teams based chatbot to communicate with SAP would be one scenario. 

![Architecture]( https://github.com/ROBROICH/SAP_AND_AZURE_TEAMS_DEMO/blob/master/Architekture.png)
 
# Prerequisites 

Optional: 

For testing the SAP OData-services the recommendation is to utilize Visual Studio Code and the [REST Client extension](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) for VS Code.


Basic knowledge how to install Visual Studio Code and the corresponding REST client extension is a prerequisite. 
The REST client is required to acquire the JSON payload, which is later used for parsing the JSON in the Logic App. 

To implement this demo Visual Studio Code is not mandatory, the generated payload for this specific service is stored in GitHub for your convenience.   

Mandatory: 

For implementing the demo an user for the SAP community is required and can be registered [here](https://www.sap.com/community/resources/registration-and-profile.html). 

This SAP community user is the prerequisite to register a [E5](https://blogs.sap.com/2017/12/05/new-sap-gateway-demo-system-available/) demo system user. 

Basic knowledge how to create a Logic App as a prerequisite, since this is not coved in this tutorial. 

# Working with the sales orders service 
For this demo, the SalesOrderSet OData service of the SAP Gateway Demo System will be utilized to display data in Teams:
https://sapes5.sapdevcenter.com/sap/opu/odata/iwbep/GWSAMPLE_BASIC/SalesOrderSet

For testing purposes, the service can be consumed via the browser by clicking on the URL. 
The service returns set of sales orders.
To call the service from the REST Client the following command is required:

```
# @name getSalesOrders
GET https://sapes5.sapdevcenter.com/sap/opu/odata/iwbep/GWSAMPLE_BASIC/SalesOrderSet?$top=100  HTTP/1.1
Authorization: Basic YOURUSER YOURPWD
Accept: application/json;odata=verbose
```


Attention, there is a blank between the username and password!

After executing the GET command the payload for the sales order will be returned. 
This payload will be required in the Logic App pipeline to parse the JSON. 


# Creating the Logic App

1.	The first step is to create a Logic App with the name LAP_SAP_TEAMS_SALESORDER
2.	Create blank app
3.	Search for “recurrence” and insert the corresponding trigger

![Recurrence Trigger]( https://github.com/ROBROICH/SAP_AND_AZURE_TEAMS_DEMO/blob/master/Recurrence_Trigger.png)

4.	Now insert search for HTTP and insert the corresponding action:

![HTML ACTION](https://github.com/ROBROICH/SAP_AND_AZURE_TEAMS_DEMO/blob/master/HTTP_ACTION.png)


Implement the HTTP action to get the top 50 SalesOrders:

```
URI:  https://sapes5.sapdevce
nter.com/sap/opu/odata/iwbep/GWSAMPLE_BASIC/SalesOrderSet?$top=50

Headers: 
Accept : application/json;odata

Authentication: Basic + Your user and pwd for the E5 demosystem

```

![HTML ACTION](https://github.com/ROBROICH/SAP_AND_AZURE_TEAMS_DEMO/blob/master/HTTP_ACTION_1.png)


Now add a Parse JSON action and execute the following steps:
1.	Add the body output of the HTTP request
2.	Maintain the JSON payload of former service call by copying the [content](https://github.com/ROBROICH/SAP_AND_AZURE_TEAMS_DEMO/blob/master/JSON_Payload.txt) of the file. 

![PARSE_JSON](https://github.com/ROBROICH/SAP_AND_AZURE_TEAMS_DEMO/blob/master/ParseJSON.png)

The next step is to initialize a variable with the name ArraySalesOrder of the type array.
The variable is used to store each row or sales order of the parsed JSON resultset. 

![Variable](https://github.com/ROBROICH/SAP_AND_AZURE_TEAMS_DEMO/blob/master/InitializeVariable.png)

After creating the variable a “For each” control is required.
Input for the for each statement is the variable “results” from “Parse JSON”

The next step is to embed a “Compose” action into the "For each" control. 

![FOREACH](https://github.com/ROBROICH/SAP_AND_AZURE_TEAMS_DEMO/blob/master/ForEach1.png)


In addition a “Compose” action has to be inserted into the “For each” element.

As next step this code needs to be inserted as “Inputs” for the “Compose” action:

```
{
    "inputs": {
        "BillingStatus": "@{items('For_each')?['BillingStatus']}",
        "BillingStatusDescription": "@{items('For_each')?['BillingStatusDescription']} ",
        "ChangedAt": "@{items('For_each')?['ChangedAt']}",
        "CreatedAt": "@{items('For_each')?['CreatedAt']}",
        "CurrencyCode": "@{items('For_each')?['CurrencyCode']}",
        "CustomerID": "@{items('For_each')?['CustomerID']}",
        "CustomerName": "@{items('For_each')?['CustomerName']}",
        "DeliveryStatus": "@{items('For_each')?['DeliveryStatus']}",
        "DeliveryStatusDescription": "@{items('For_each')?['DeliveryStatusDescription']}",
        "GrossAmount": "@{items('For_each')?['GrossAmount']}",
        "LifecycleStatus": "@{items('For_each')?['LifecycleStatus']}",
        "LifecycleStatusDescription": "@{items('For_each')?['LifecycleStatusDescription']}",
        "NetAmount": "@{items('For_each')?['NetAmount']}",
        "Note": "",
        "NoteLanguage": "@{items('For_each')?['NoteLanguage']}",
        "SalesOrderID": "@{items('For_each')?['SalesOrderID']}",
        "TaxAmount": "@{items('For_each')?['TaxAmount']}"
    }
}

```

![FOREACH2](https://github.com/ROBROICH/SAP_AND_AZURE_TEAMS_DEMO/blob/master/ForEach2.png)

After the creation of the mappings for the JSON elements in the collection, each record needs to be added to the variable ArraySalesOrder within the “For each” action. To achieve this a “Append to array variable” action needs to be added. The “value” will be the “Outputs” of “Compose”. 

![FOREACH3](https://github.com/ROBROICH/SAP_AND_AZURE_TEAMS_DEMO/blob/master/ForEach3.png)

In order to create a formatted message in Teams we pass the “ArraySalesOrders” variable to a "Ceate HTML Table" action.

![HTMLTABLE](https://github.com/ROBROICH/SAP_AND_AZURE_TEAMS_DEMO/blob/master/HTML_TABLE.png)

The next step is to add the result of the HTML table (“Output”) to a Teams “Post a message V(3)” action: 

![TEAMSMESSAGE](https://github.com/ROBROICH/SAP_AND_AZURE_TEAMS_DEMO/blob/master/TEAMS_MESSAGE.png)

Finally a "Send an email (V2)" Outlook 365 action can utilizes to send the SAP sales order as e-mail:

![TEAMSMESSAGE](https://github.com/ROBROICH/SAP_AND_AZURE_TEAMS_DEMO/blob/master/Outlook365.png)

# Summary:
The intention of the tutorial was to demonstrate the easy integration of SAP data with Microsoft Teams or Outlook365. The Logic App and the dataflow is implemented in and graphical and therefore in a highly maintainable and reproducible fashion. Finally, no deep programming skills were required to implement this scenario. Next to the public SAP E5 demo-system, the  
[SAP API Business Hub](https://api.sap.com/) and [Microsoft Graph](https://developer.microsoft.com/en-us/graph/graph-explorer) provide countless options to integrate the Intelligent Cloud and the Intelligent Enterprise. 



