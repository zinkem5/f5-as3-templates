# f5-as3-templates
Mustache templates for use with AS3 with type annotations.

This is a repository of template sets to use in f5 template solutions.

Compatible with [F5 Cisco ACI Service Center](https://github.com/F5Networks/f5-aci-servicecenter)

[Built on AS3](https://github.com/F5Networks/f5-appsvcs-extension)

[mustache logic-less templates](https://mustache.github.io/)

## Contents of this repository

```
./templates             # directory containing template sets
./templates/simple      # simple starter templates for AS3
```

## How to use these Templates

These templates can be imported into any compatible F5 templating solution.

**Supported Solutions**
* [F5 Cisco ACI Service Center](https://github.com/F5Networks/f5-aci-servicecenter);

## How does this work?

At scale, configuring BIG-IP by writing AS3 by hand for every application and service deployed may get tedious. Many will resort to copy pasting a few basic Application objects and tweaking them to fit their needs.

The copy-paste workflow can be streamlined without much effort by using a templating system. This repository contains [mustache](https://mustache.github.io) templates of AS3 declarations. The templates are AS3 declarations with variables that need to be filled in. They can be 'rendered' into a concrete declaration by providing a view. The view is a dictionary that holds a value for each variable.

These templates can be parsed by mustache to auto-generate a simplified and streamlined font-end for configuring BIG-IP. An administrator can then provision BIG-IP using only the information needed by the template. Templated configurations can be easier to manage and are less prone to error than manual or imperative configuration methods.

A template can be filled in, and the Application class can be inserted into an existing Tenant declaration and POSTed to AS3. This example template will deploy an HTTP virtual address load balancing to HTTP servers on port 80. It also includes a parameter to enable/disable the virtual address.

```mustache
{
  "class": "AS3",
  "action": "deploy",
  "persist": true,
  "declaration": {
    "class": "ADC",
    "schemaVersion": "3.0.0",
    "id": "0123-4567-8910",
    "label": "Sample 1",
    "remark": "Basic HTTP with Monitor",
    "{{tenant_name}}": {
      "class": "Tenant",
      "{{application_name}}": {
        "class": "Application",
        "template": "http",
        "serviceMain": {
          "class": "Service_HTTP",
          "virtualAddresses": ["{{virtual_address}}"],
          "pool": "web_pool",
          "enable": {{enable::boolean}}
        },
        "web_pool": {
          "class": "Pool",
          "monitors": [
            "http"
          ],
          "members": [
            {
              "servicePort": 80,
              "serverAddresses": {{server_addresses::array}}
            }
          ]
        }
      }
    }
  }
}
```

Variables are encoded by surrounding them with curly braces, `{{` and `}}`. This declaration could be copied into Postman, the default variable syntax is mustache compatible. This enables template creation through Postman. Contact the author of this document for information on auto-importing postman collections into a templating environment.

In this template some variables are annotated with a type, following the double colon. By default, all variables are treated as strings. It is useful in the schema generation phase to have types specified. This enables the schema to provide richer information for auto-generating front end html forms.

The above template can be used to auto-generate the following schema:
```json
{
  "properties": {
    "tenant_name" : {
      "type": "string"
    },
    "application_name" : {
      "type": "string"
    },
    "virtual_address" : {
      "type": "string"
    },
    "server_addresses" : {
      "type": "array"
    },
    "enable" : {
      "type": "boolean"
    },
  }
}
```

In the example, the template would be loaded into the renderer and when the user invokes it, they need to provide a `tenant_name`, `application_name`, `virtual_address`, ... This is called a 'view'. The view is passed to the renderer, and the renderer outputs a valid AS3 declarations using the values in the view.

The example schema can validate the view the admin provides:

```json
{
  "tenant_name" : "myTenant",
  "application_name" : "simple_http_1",
  "virtual_address" : "10.0.0.1",
  "server_addresses" : [ "10.0.1.1", "10.0.2.2" ],
  "enable" : true
}
```

which will get translated into this:

```json
{
  "tenant_name" : "myTenant",
  "application_name" : "simple_http_1",
  "virtual_address" : "10.0.0.1",
  "server_addresses::array" : [ "10.0.1.1", "10.0.2.2" ],
  "enable::boolean" : true
}
```

This final view can be used with mustache and the template to render a valid AS3 declaration and apply it:

```json
{
  "class": "AS3",
  "action": "deploy",
  "persist": true,
  "declaration": {
    "class": "ADC",
    "schemaVersion": "3.0.0",
    "id": "0123-4567-8910",
    "label": "Sample 1",
    "remark": "Basic HTTP with Monitor",
    "myTenant": {
      "class": "Tenant",
      "simple_http_1": {
        "class": "Application",
        "template": "http",
        "serviceMain": {
          "class": "Service_HTTP",
          "virtualAddresses": ["10.0.0.1"],
          "pool": "web_pool",
          "enable": true
        },
        "web_pool": {
          "class": "Pool",
          "monitors": [
            "http"
          ],
          "members": [
            {
              "servicePort": 80,
              "serverAddresses": [ "10.0.1.1", "10.0.2.2" ]
            }
          ]
        }
      }
    }
  }
}
```
