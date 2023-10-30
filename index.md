---
duration: PT1H00M0S
description: Learn how to migrate your APIs to Oracle Cloud Infrastructure API Gateway (OCI API Gateway) with Oracle Integration.
level: Advanced
roles: DevOps Engineer;Developer
products: en/cloud/oracle-cloud-infrastructure/oci
keywords: APIs REST/SOAP;Data Integration;Migration
inject-note: true
---

# Migrate APIs to Oracle Cloud Infrastructure API Gateway with Oracle Integration

## Introduction

The Oracle Cloud Infrastructure API Gateway (OCI API Gateway) service allows you to publish APIs with accessible private endpoints on your network and that you can expose with public IP addresses if you want them to accept internet traffic. Endpoints support API validation, request and response transformation, CORS, authentication and authorization, and request limitation.

Using the OCI API Gateway Service, you create one or more API Gateways on a regional subnet to process API client traffic and route it to back-end services. You can use a single API Gateway to link multiple backend services (such as load balancers, compute instances, and Oracle Cloud Infrastructure Functions into a single consolidated API endpoint.

You can access the OCI API Gateway Service to define API Gateways and API deployments using the OCI Console and the REST API.

The OCI API Gateway service is integrated with Oracle Cloud Infrastructure Identity and Access Management (OCI IAM), which provides easy authentication with native Oracle Cloud Infrastructure (OCI) identity functionality.

The OCI API Gateway allows API deployment to be done by importing a JSON structure. Here you can see the format of this structure [Creating an API Deployment Specification](https://docs.oracle.com/en-us/iaas/Content/APIGateway/Tasks/apigatewaycreatingspecification.htm#Creating_an_API_Deployment_Specification).

```
{
  "requestPolicies": {},
  "routes": [
    {
      "path": "<api-route-path>",
      "methods": ["<method-list>"],
      "backend": {
        "type": "<backend-type>",
        "<backend-target>": "<identifier>"
      },
      "requestPolicies": {}
    }
  ]
}
```

In addition to the console, it is possible to deploy the API using the Oracle Cloud Infrastructure command line interface (OCI CLI) and also the OCI REST service for the OCI API Gateway. Here is the documentation to understand how to deploy:

- [Deploy an API using OCI CLI API in Python](https://docs.oracle.com/en-us/iaas/tools/python-sdk-examples/2.112.4/apigateway/create_deployment.py.html)

- [Deploy an API on an OCI API Gateway by creating an API Deployment](https://docs.oracle.com/en-us/iaas/Content/APIGateway/Tasks/apigatewaycreatingdeployment.htm)

If you already have your API structure to deploy in the standard OCI API Gateway format, it is easy to deploy it using the console (import), OCI CLI or by making a REST call. By having this structure ready, it is easy to create code to process multiple JSON files. This example below demonstrates how to process files using bash script code:

```
#!/bin/bash
# Folder containing JSON files
folder="/<your path to the JSON Files>"
# Loop through the JSON files in the folder
for file in "$folder"/*.json; do
    service="${file##*/}"; service="${service%.*}"
    echo "Service: $service"        # Read and display the contents of the JSON file
    oci api-gateway deployment create --compartment-id ocid1.compartment.oc1..xxxxxxxxxxxxxxxxxxxxxxxxxx --display-name $service --gateway-id ocid1.apigateway.oc1.iad.xxxxxxxxxxxxxxxxxxxxxxxxxxxx --path-prefix "/$service" --specification file://$file --debug
    echo "-------------------"
done
```

In some situations, API metadata information will need to be handled. We could do this through code using Python, Java or another language, but the purpose of this tutorial is to use Oracle Integration to automate it. There are several advantages to doing it this way. You can easily:

- Execute REST calls from OCI API Gateway to deploy your APIs.

- Map attributes from source to destination, including making the appropriate transformations.

- Implement flow to process all settings graphically.

### Objectives

- Explain how to import your Legacy APIs specification to the OCI API Gateway.

- Explain the known formats and what format OCI API Gateway can import natively.

- Create an Oracle Integration process to migrate a source API deployment definition to the OCI API Gateway.

- Process multiple source definitions.

- Differentiate REST and SOAP definitions.

- Map the source attributes to the target OCI API Gateway.

- Deploy into the correct environment (QA, DEV).

### Prerequisites

- An Oracle Integration instance.

- Two OCI API Gateway instances (in this tutorial, we have created one for QA and other for DEV environments).

- The Oracle Integration instance needs to reach the OCI API Gateway instances (attention to private networks).

## Task 1: Understand your Source API data structure

We can start from an API metadata structure. The best known definitions would be an OpenAPI or an Swagger.

If you have an SWAGGER or an OpenAPI, you can import your API data to OCI API Gateway by importing in your OCI Console, OCI CLI or OCI API Gateway REST Service, see [Creating an API Resource with an API Description](https://docs.oracle.com/en-us/iaas/Content/APIGateway/Tasks/apigatewaycreatingapiobject.htm).

If you don't have these formats (Swagger/OpenAPI), you need to structure your API data definitions in another format. In this task, we will work with an Excel structure and, from this structure, transform it into JSON.

![Excel_api_migration_source.png](images%2FExcel_api_migration_source.png "image1")

So, there is a JSON structure to work with.

```
[ {
  "API_NAME" : "cep",
  "TYPE" : "REST",
  "METHOD" : "GET",
  "PATH_PREFIX" : "/okecep",
  "PATH" : "/cep",
  "ENDPOINT" : "http://x.x.x.x/cep",
  "QUERY_PARAMETERS" : "cep",
  "GROOVY_SCRIPT" : "",
  "AUTHENTICATION_TYPE" : "BASIC",
  "ENVIRONMENT" : "QA",
  "HEADER" : "",
  "HEADER_VALUE" : ""
}, {
  "API_NAME" : "calculator",
  "TYPE" : "SOAP",
  "METHOD" : "POST",
  "PATH_PREFIX" : "/dneonline",
  "PATH" : "/calculator",
  "ENDPOINT" : "http://www.example.com/calculator.asmx",
  "QUERY_PARAMETERS" : "",
  "GROOVY_SCRIPT" : "",
  "AUTHENTICATION_TYPE" : "BASIC",
  "ENVIRONMENT" : "DEV",
  "HEADER" : "",
  "HEADER_VALUE" : ""
} ]
```

These API data structures can be found here: [source_apis.json](files%2Fsource_apis.json). You can structure your API definitions data in any format, this is just an example.

## Task 2: Understand the OCI API Gateway Deployment Data

In this task, you can see the native format to import your API definitions into the OCI API Gateway. Often, a simple structure is enough to implement an API, but as details such as security, header rules, parameters or other details appear, the JSON structure becomes larger. This would be the complete JSON structure for any type of deployment. We will use it in our Oracle Integration flow. You can find the file here: [apigw_structure.json](files%2Fapigw_structure.json).

```
{
  "requestPolicies": {
    "authentication": {
      "type": "CUSTOM_AUTHENTICATION",
      "tokenHeader": "",
      "tokenQueryParam": "",
      "tokenAuthScheme": "authentication-scheme",
      "isAnonymousAccessAllowed": false,
      "functionId": "ocid1.fnfunc.oc1.phx.aaaaaaaaac2______kg6fq",
      "maxClockSkewInSeconds": 0,
      "validationPolicy": {
        "type": "REMOTE_DISCOVERY",
        "clientDetails": {
          "type": "CUSTOM",
          "clientId": "client-id",
          "clientSecretId": "secret-ocid",
          "clientSecretVersionNumber": 0
        },
        "sourceUriDetails": {
          "type": "DISCOVERY_URI",
          "uri": "well-known-uri"
        },
        "isSslVerifyDisabled": true,
        "maxCacheDurationInHours": 0,
        "additionalValidationPolicy": {
          "issuers": ["issuer-url", "issuer-url"],
          "audiences": ["intended-audience"],
          "verifyClaims": [{
            "key": "claim-name",
            "values": ["acceptable-value", "acceptable-value"],
            "isRequired": true
          }]
        }
      },
      "tokenHeader": "Authorization",
      "validationPolicy": {
        "type": "REMOTE_DISCOVERY",
        "clientDetails": {
          "type": "CUSTOM",
          "clientId": "5hsti38yhy5j2a4tas455rsu6ru8yui3wrst4n1",
          "clientSecretId": "ocid1.vaultsecret.oc1.iad.amaaaaaa______cggit3q",
          "clientSecretVersionNumber": 1
        },
        "sourceUriDetails": {
          "type": "DISCOVERY_URI",
          "uri": "https://my-idp/oauth2/default/.well-known/openid-configuration"
        },
        "isSslVerifyDisabled": false,
        "maxCacheDurationInHours": 2,
        "additionalValidationPolicy": {
          "issuers": ["https://identity.oraclecloud.com/"],
          "audiences": ["api.dev.io"],
          "verifyClaims": [{
            "key": "is_admin",
            "values": ["service:app", "read:hello"],
            "isRequired": true
          }]
        }
      }
    },
    "mutualTls":{
      "isVerifiedCertificateRequired": true,
      "allowedSans": ["api.weather.com"]
    }
  },
  "routes": [
    {
      "path": "/weather",
      "methods": ["GET"],
      "backend": {
        "type": "HTTP_BACKEND",
        "url": "https://api.weather.gov/${request.auth[region]}"
      },
      "requestPolicies": {
        "authorization": {
          "type": "ANY_OF",
          "allowedScope": [ "weatherwatcher" ]
        },
        "headerValidations": {
          "headers": {
            "name": "header-name",
            "required": true
          },
          "validationMode": "ENFORCING|PERMISSIVE|DISABLED"
        },
        "queryParameterValidations": {
          "parameters": {
            "name": "query-parameter-name",
            "required": true
          },
          "validationMode": "ENFORCING|PERMISSIVE|DISABLED"
        },
        "bodyValidation": {
          "required": true,
          "content": {
            "media-type-1": {
              "validationType": "NONE"
            },
            "media-type-2": {
              "validationType": "NONE"
            }
          },
          "validationMode": "ENFORCING|PERMISSIVE|DISABLED"
        },
        "headerTransformations": {
          "renameHeaders": {
            "items": [
              {
                "from": "original-name",
                "to": "new-name"
              }
            ]
          },
          "setHeaders": {
            "items": [
              {
                "name": "header-name",
                "values": ["header-value"],
                "ifExists": "OVERWRITE|APPEND|SKIP"
              }
            ]
          }
        },
        "queryParameterTransformations": {
          "filterQueryParameters": {
            "type": "BLOCK|ALLOW",
            "items": [
              {
                "name": "query-parameter-name"
              }
            ]
          },
          "renameQueryParameters": {
            "items": [
              {
                "from": "original-name",
                "to": "new-name"
              }
            ]
          },
          "setQueryParameters": {
            "items": [
              {
                "name": "query-parameter-name",
                "values": ["query-parameter-value"],
                "ifExists": "OVERWRITE|APPEND|SKIP"
              }
            ]
          }
        }
      },
      "responsePolicies": {
        "headerTransformations": {
          "filterHeaders": {
            "type": "BLOCK|ALLOW",
            "items": [
              {
                "name": "header-name"
              }
            ]
          },
          "renameHeaders": {
            "items": [
              {
                "from": "original-name",
                "to": "new-name"
              }
            ]
          },
          "setHeaders": {
            "items": [
              {
                "name": "header-name",
                "values": ["header-value"],
                "ifExists": "OVERWRITE|APPEND|SKIP"
              }
            ]
          }
        }
      }
    }
  ]
}
```

## Task 3: Create Connections

There are 2 connections we need to create. The first connection is the Trigger REST Connection. This connection will be used to expose an endpoint in Oracle Integration. With this endpoint, you can call the migration process as a REST service, passing the source data of your APIs.

The second connection will be used to call the OCI API Gateway REST service to create your APIs deployments.
The OCI API Gateway REST documentation can be read here: [Deploying APIs via REST](https://docs.oracle.com/en-us/iaas/api/#/en/api-gateway/20190501/Deployment/CreateDeployment).

Create an Oracle Integration Connection, see [Create a REST Connection](https://docs.oracle.com/en/cloud/paas/integration-cloud/rest-adapter/create-connection.html#GUID-07B2A588-6DB8-47A9-A206-51B4132B0DA1).

1. Search the REST Adapter.

   ![connection_rest_1.png](images%2Fconnection_rest_1.png "image2")

2. Give a name and identifier, for example, give the name "REST_EXPOSE". This is the first connection. Select the Trigger Role. Trigger Role means you will expose the connection as a REST endpoint, click **Create**.

   ![Connection_Rest_2.png](images%2FConnection_Rest_2.png "image3")

3. For your Trigger Endpoint, you can execute a request with the Oracle Integration Basic Authentication. So, you can use an username and password.

   ![Connect_Rest_3.png](images%2FConnect_Rest_3.png "image4")

4. In the second connection, as you did in the first connection, give a name and identifier (for example, "APIGW_REST_API"). Remember to select the Invoke type unlike the first configuration. Select the proper OCI API Gateway REST for your region (in the example, the region is Ashburn). See here to view the proper endpoint for your region [OCI API Gateway endpoints](https://docs.oracle.com/en-us/iaas/api/#/en/api-gateway/20190501/). Obtain your OCI Credentials to provide access to the OCI API Gateway service.

   ![Connection_Apigateway_Rest_Api.png](images%2FConnection_Apigateway_Rest_Api.png "image5")

## Task 4: Create the Integration

The process of migrating your API source metadata to the OCI API Gateway is:

- Expose the Trigger as a REST Service in Oracle Integration.

- Initialize the Target OCI API Gateways (environments like QA and DEV).

- Initialize the Compartments for the targets.

- Execute a Loop to process all APIs in the source metadata.

- Translate the source metadata to the OCI API Gateway metadata.

- Execute a Request REST Service to the OCI API Gateway.

  ![Create_Integration_1.png](images%2FCreate_Integration_1.png "image6")

1. Expose your Integration with your Trigger REST Connection created previously.

   ![Integration_1.png](images%2FIntegration_1.png "image7")

2. Give a name to your request endpoint.

   ![Integration_1a.png](images%2FIntegration_1a.png "image8")

3. Give a path (for example, `/convert`) and select the POST method. Remember to establish a request payload.

   ![Integration_1b.png](images%2FIntegration_1b.png "image9")

4. Now you can select the "JSON Sample" and paste your JSON source structure.

   ![Integration_1c.png](images%2FIntegration_1c.png "image10")

5. Paste the JSON here.

   ![Integration_1d.png](images%2FIntegration_1d.png "image11")

6. Now let's initialize the variables for your API Gateway instances (each environment is a new API Gateway instance). Create a variable for the METHOD. Remember you can use several methods in REST (GET, POST, PUT, DELETE, ANY) and need to use POST in SOAP Services.

   ![Integration_2.png](images%2FIntegration_2.png "image12")

   ![Integration_2a.png](images%2FIntegration_2a.png "image13")

7. Create a FOR EACH Loop. This will process the source JSON list.

   ![Integration_3.png](images%2FIntegration_3.png "image14")

8. Edit the For Each Action and drag and drop your Loop Level Array to the **Repeating Element** Attribute.

   ![Integration_3a.png](images%2FIntegration_3a.png "image15")

9. Now let's configure the environment redirection. Create a Switch Action and put the IFs and the assignments for each condition.

   ![Integration_4.png](images%2FIntegration_4.png "image16")

10. Create the first condition for the QA environment.

    ![Integration_4a.png](images%2FIntegration_4a.png "image17")

11. Verify in the source metadata ENVIRONMENT attribute and test if it has "QA" value.

    ![Integration_4b.png](images%2FIntegration_4b.png "image18")

12. If the condition is true, you need to assign the correct environment. A GatewayID and CompartmentID is an OCID value. You need to assign the correct IDs to the variables.

    ![Integration_4c.png](images%2FIntegration_4c.png "image19")

13. Do the same for the other environments.

    ![Integration_5.png](images%2FIntegration_5.png "image20")

14. Now let's view if your API is a REST or SOAP service. If your API is a SOAP service, you must configure the method as POST.

    ![Integration_5a.png](images%2FIntegration_5a.png "image21")

15. Create an IF condition to test the service type.

    ![Integration_5b.png](images%2FIntegration_5b.png "image22")

16. Assign the POST value to the method variable.

    ![Integration_5c.png](images%2FIntegration_5c.png "image23")

    ![Integration_5d.png](images%2FIntegration_5d.png "image24")

17. In other situations, the source attribute gives the correct method.

    ![Integration_5e.png](images%2FIntegration_5e.png "image25")

    ![Integration_5f.png](images%2FIntegration_5f.png "image26")

18. Let's map the source metadata attributes to the OCI API Gateway parameters. Before the mapping action, you need to configure the OCI API Gateway REST Adapter.

    ![Integration_6.png](images%2FIntegration_6.png "image27")

19. Configure the connection.

    ![Integration_6a.png](images%2FIntegration_6a.png "image28")

20. Give a name for this service. The path must be `/deployments` and the method must be POST. Remember to configure the request payload.

    ![Integration_6b.png](images%2FIntegration_6b.png "image29")

21. You cannot paste the JSON OCI API Gateway structure because of the large size. So you need to upload a file with the content. The complete JSON Structure is here [apigw_structure.json](files%2Fapigw_structure.json).

    ![Integration_6c.png](images%2FIntegration_6c.png "image30")

22. Map the target metadata.

    ![Integration_6d.png](images%2FIntegration_6d.png "image31")

>**Note**: You can see the Oracle Integration artifact here: [Oracle Integration: Migrate your Source API to OCI API Gateway](files%2FMIGRATE_TO_APIGW_01.00.0001.iar). This example of artifact is not intended to be used as a final process for your migration and has only the objective to illustrate the process. Use this artifact to guide your real implementation.

## Task 5: Test the migration

For more information, see the following links and follow the steps below.

- [Activate and Deactivate Integrations](https://docs.oracle.com/en/cloud/paas/integration-cloud/integrations-user/activate-and-deactivate-integrations.html#GUID-4D78EC6F-BC59-4A2C-8C2B-213AD7A09556)

- [Testing REST trigger-based Integrations in Oracle Integration Console](https://blogs.oracle.com/integration/post/testing-rest-trigger-based-integrations-in-oic-console)

  ![Test_1aa.png](images%2FTest_1aa.png "image32")

1. Add the source JSON data.

   ![Test_1.png](images%2FTest_1.png "image33")

2. Click **Test** and wait till finish.

   ![Test_1a.png](images%2FTest_1a.png "image34")

3. Go to the OCI API Gateway instance deployments and see the creation of your APIs.

   ![Test_2.png](images%2FTest_2.png "image35")

## Related Links

- [Creating an API Resource with an API Description](https://docs.oracle.com/en-us/iaas/Content/APIGateway/Tasks/apigatewaycreatingapiobject.htm)

- [Creating an API Deployment Specification](https://docs.oracle.com/en-us/iaas/Content/APIGateway/Tasks/apigatewaycreatingspecification.htm#Creating_an_API_Deployment_Specification)

- [Deploy an API using OCI CLI API in Python](https://docs.oracle.com/en-us/iaas/tools/python-sdk-examples/2.112.4/apigateway/create_deployment.py.html)

- [Deploying an API on an API Gateway by Creating an API Deployment](https://docs.oracle.com/en-us/iaas/Content/APIGateway/Tasks/apigatewaycreatingdeployment.htm)

- [Deploying APIs via REST](https://docs.oracle.com/en-us/iaas/api/#/en/api-gateway/20190501/Deployment/CreateDeployment)

- [Create a REST Connection](https://docs.oracle.com/en/cloud/paas/integration-cloud/rest-adapter/create-connection.html#GUID-07B2A588-6DB8-47A9-A206-51B4132B0DA1)

- [OCI API Gateway endpoints](https://docs.oracle.com/en-us/iaas/api/#/en/api-gateway/20190501/)

- [Activate and Deactivate Integrations](https://docs.oracle.com/en/cloud/paas/integration-cloud/integrations-user/activate-and-deactivate-integrations.html#GUID-4D78EC6F-BC59-4A2C-8C2B-213AD7A09556)

- [Testing REST trigger-based Integrations in Oracle Integration Console](https://blogs.oracle.com/integration/post/testing-rest-trigger-based-integrations-in-oic-console)

## Acknowledgments

- **Author** - Cristiano Hoshikawa (Oracle LAD A-Team Solution Engineer)
