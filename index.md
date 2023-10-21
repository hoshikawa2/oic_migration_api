# Migrate your APIs to Oracle Cloud API Gateway

## Introduction

## Objectives

## Prerequisites

## Task 1: Understand your Source API data structure

![Excel_api_migration_source.png](images%2FExcel_api_migration_source.png)

    [ {
      "API_NAME" : "cep",
      "TYPE" : "REST",
      "METHOD" : "GET",
      "PATH_PREFIX" : "/okecep",
      "PATH" : "/cep",
      "ENDPOINT" : "http://129.153.138.208/cep",
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
      "ENDPOINT" : "http://www.dneonline.com/calculator.asmx",
      "QUERY_PARAMETERS" : "",
      "GROOVY_SCRIPT" : "",
      "AUTHENTICATION_TYPE" : "BASIC",
      "ENVIRONMENT" : "DEV",
      "HEADER" : "",
      "HEADER_VALUE" : ""
    } ]


## Task : Understand the OCI API Gateway Deployment Data

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

## Task 2: Create Connections

![connection_rest_1.png](images%2Fconnection_rest_1.png)
![Connection_Rest_2.png](images%2FConnection_Rest_2.png)
![Connect_Rest_3.png](images%2FConnect_Rest_3.png)

![Connection_Apigateway_Rest_Api.png](images%2FConnection_Apigateway_Rest_Api.png)

## Task 3: Create the Integration

![Create_Integration_1.png](images%2FCreate_Integration_1.png)

![Integration_1.png](images%2FIntegration_1.png)

![Integration_1a.png](images%2FIntegration_1a.png)

![Integration_1b.png](images%2FIntegration_1b.png)

![Integration_1c.png](images%2FIntegration_1c.png)

![Integration_1d.png](images%2FIntegration_1d.png)

![Integration_2.png](images%2FIntegration_2.png)

![Integration_2a.png](images%2FIntegration_2a.png)

![Integration_3.png](images%2FIntegration_3.png)

![Integration_3a.png](images%2FIntegration_3a.png)

![Integration_4.png](images%2FIntegration_4.png)

![Integration_4a.png](images%2FIntegration_4a.png)

![Integration_4b.png](images%2FIntegration_4b.png)

![Integration_4c.png](images%2FIntegration_4c.png)

![Integration_5.png](images%2FIntegration_5.png)

![Integration_5a.png](images%2FIntegration_5a.png)

![Integration_5b.png](images%2FIntegration_5b.png)

![Integration_5c.png](images%2FIntegration_5c.png)

![Integration_5d.png](images%2FIntegration_5d.png)

![Integration_5e.png](images%2FIntegration_5e.png)

![Integration_5f.png](images%2FIntegration_5f.png)

![Integration_6.png](images%2FIntegration_6.png)

![Integration_6a.png](images%2FIntegration_6a.png)

![Integration_6b.png](images%2FIntegration_6b.png)

![Integration_6c.png](images%2FIntegration_6c.png)

![Integration_6d.png](images%2FIntegration_6d.png)

## Task : Test the migration

![Test_1aa.png](images%2FTest_1aa.png)

![Test_1.png](images%2FTest_1.png)

![Test_1a.png](images%2FTest_1a.png)

![Test_2.png](images%2FTest_2.png)

