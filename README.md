# SAP Data Warehouse Cloud Sample Content for Partner Integration
You can study the PartnerApp sample content to understand the SAP Data Warehouse Cloud partner integration. The download contains an HTML file simulating an IFRAME that connects to the SAP Data Warehouse Cloud. It also describes the response expected by SAP Data Warehouse Cloud for validating the connections.

# Download
Download the ZIP file (PartnerApp.zip) from [release page] (https://github.wdf.sap.corp/I537933/PartnerApp). The ZIP file contains:
 
*	PartnerApp.html: an example of the API calls and responses to send to SAP Data Warehouse Cloud
*	README (present file)

# Partner Connection Tutorial

#### Overview
Partner Connection shows up as his own connection on Connection Wizard. 
Customer is redirected to the partner IFRAME to set up his own credentials/settings on the partner side. When ready, the partner sends the customer back to SAP Data Warehouse Cloud by postMessage() and the customer can specify the details of the partner connection. In the last step of the creation wizard, the connection is created accordingly:
*	Partner Connection is added to Repository
*	It creates a database schema ("Partner SQL Schema") - can be used in SAP HANA DB Client. It is only visible in the Space management for users with Administrator roles.
*	Credentials/host/port is to be returned to partner API 
* Partner sends approval
    *  In case of success, the connection gets created, so as the database schema 
    *  In case of error, the connection creation is rolled back and an error message is provided from the provider if given or from SAP Data Warehouse Cloud by default

#### Pre-requisites
The following information need to be provided in SAP Data Warehouse Cloud for setting up the environment:
* Partner logo: the logo will be shown in the list of connections available in SAP Data Warehouse Cloud
* IFRAME URL: The URL of the IFRAME should be provided to SAP Data Warehouse Cloud administration
* URL DOMAIN validation: As SAP Data Warehouse always checks the origin of the IFRAME URL for security reasons, it is important to communicate the URLs where the customers could be redirected.
* IP Whitelisting: to access the open SQL schema, the IP addresss of the server which is accessing the schema via Hana DB client needs to be white-listed in the SAP Data Warehouse Cloud database. To accomplish that, the partner must send the list of IP addresses used by customers and also during testing to SAP Data Warehouse Cloud administration, so the white-list entries can be created.

#### Step 0: Start
At first the customer gets redirected to the partner web site via the IFRAME URL. The "sessionId" is also sent to the partner via a hash parameter, it will be kept during the entire flow and checked by SAP Data Warehouse Cloud continously.  
Example: https://github.wdf.sap.corp/pages/i537933/PartnerApp/PartnerApp.html?DWCVersion=1#sessionid=abcdefgh

The API version supported by the partner is also sent via the query string in the IFRAME URL.

#### Step 1: Redirect from IFRAME to SAP Data Warehouse Cloud
At first the partner sends a first response to SAP Data Warehouse Cloud via the following PostMessage:
```
{
  sessionId: "sessionId", // actual session (mandatory)
  requestMethod: "ConnectionSet", // used by SAP Data Warehouse to retrieve event action (mandatory)
  partnerConnection: {
    name: "PartnerApp", // example of the connection name provided by the partner (optional) 
    description: "Description of the Partner App" // example of the connection description provided by the partner (optional)
  },
  status: {
    code: statusCode // "success" or "error" (mandatory)
  }
}
```
**Note:** regarding the connection name, no space or special characters are allowed.
The sessionId is validated by SAP Data Warehouse Cloud, so as the URL referrer. 

#### Step 2: Entering Connection details
After being redirected to SAP Data Warehouse Cloud, customers can decide which name, business name and description name they wants to set up for their connection. SAP Data Warehouse Cloud submits the result to the partner via the following request:
```
{
  sessionId: "sessionId", // actual session (mandatory)
  requestMethod: "ConnectionCreated", // used for the PartnerApp demo for retrieving the action to perform 
  status: {
    code: "success", // success code
  },
  connection: {
    APIVersion: "API_VERSION", // API Version supported by the partner in SAP Data Warehouse Cloud
    name: "CONNECTIONNAME", // Connection name given by partner or customer
    businessName: "CONNECTION_BUSINESS_NAME", // connection business name entered by the customer
    description: "CONNECTION_DESCRIPTION", // connection description entered by the customer
    schema: "SCHEMA_NAME", // schema name created by SAP Data Warehouse Cloud - can be used in SAP HANA DB Client
    user: {
      username: "USERNAME", // schema user name created by SAP Data Warehouse Cloud - can be used in SAP HANA DB Client
      password: "PASSWORD", // schema password created by SAP Data Warehouse Cloud - can be used in SAP HANA DB Client
    },
    host: "HOST_NAME", // host name
    port: "PORT", // port
  },
}
```
#### Step 3: Getting response form the partner
A response from the partner is expected via the following format:
```
{
  sessionId: "sessionId", // actual session (mandatory)
  requestMethod: "ConnectionReceived", // used by SAP Data Warehouse to retrieve event action (mandatory)
  status: {
    code: "success", // "success" or "error" (mandatory)
    message: "",  // Error message delivered by the provider in case of error (optional)
  }
}
```
Based on the status code, the connection will be created or not in SAP Data Warehouse Cloud.

# Versioning
API Versioning is the practice of managing changes to the partner API. It is about communicating changes to the API, so consumers know what to expect from it.
SAP Data Warehouse Cloud can support different versions and some others would be considered as deprecated. Conditional code is used for differentiating what the API is capable or not. If the partner is using a version lower that is not supported, the connection will not be visible in the connection list.

# Known Issues
None

# Support and Contribution
This project is provided "as-is"; there is no guarantee that raised issues will be answered or addressed in future releases. For more information, please visit the SAP Community or contact your SAP contact to obtain support.

# License
Copyright © 2019 SAP SE or an SAP affiliate company. All rights reserved. This project is licensed under the Apache Software License, version 2.0 except as noted otherwise in the [LICENSE file](/LICENSE).

