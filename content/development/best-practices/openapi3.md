---
title: "OpenAPI 3.0 Specification Best Practices"
date: 2018-03-17T05:36:42-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

OpenAPI 3.0 specification is a very loose specification and it gives designer too many options to write the spec. Most 
of the cases, developers write the code with annotations and generate the specification afterward. With enterprise 
scale in mind, we encourage [design first][] approach. The outcome is not just a document but a specification that 
can be used to scaffold a new project and loaded during runtime to verify JWT scopes for security and validate requests 
to protect the business layer against attacks. 

To enable [light-codegen][] to generate meaningful code and utilize the full potential of the [light-rest-4j][] framework, 
the author of the OpenAPI 3.0 specification should follow the best practices below. 

### Guidelines

- clear and easily readable by architects, analysts, developers
- well documented, with explanations provided in description tags
- adhere to OpenAPI specifications - v3.0 at the time of this writing
- use a design which lends itself to a clean and easily consumable object model

### Security First

Often API designers focus on functionalities and add security later on. We would encourage to follow security first 
approach so that security is considered for every endpoint during the design. 
 
Before working on a new specification, you can copy from an existing one and [petstore][] is a good starting point. 
There are also other OpenAPI specifications in the [model-config][] repository to help you in learning the design style.   

Within petstore specification you can find the following block that defines the security under the component. This section
is optional in the specification, but here it is mandatory. [light-codegen][] throws an error if securitySchemes are not
found.  

```yaml
components:
  securitySchemes:
    petstore_auth:
      type: oauth2
      description: This API uses OAuth 2 with the client credential grant flow.
      flows:
        clientCredentials:
          tokenUrl: 'https://localhost:6882/token'
          scopes:
            'write:pets': modify pets in your account
            'read:pets': read your pets
```

Once the securitySchemes is defined, you can specify security scopes for each endpoint. When you add a new endpoint you
might ask yourselves a question. Do I need to create a brand new scope or pick one or more existing scopes from the scopes
defined in securitySchemes?

The endpoint/operation definition with security definition looks like this. 

```yaml
  '/pets/{petId}':
    get:
      summary: Info for a specific pet
      operationId: showPetById
      tags:
        - pets
      parameters:
        - name: petId
          in: path
          required: true
          description: The id of the pet to retrieve
          schema:
            type: string
      security:
        - petstore_auth:
            - 'read:pets'
      responses:
        '200':
          description: Expected response to a valid request
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Pet'
              example:
                id: 1
                name: Jessica Right
                tag: pet
``` 

Above design ensures that the [light-rest-4j][] framework compares the scopes from JWT token against the scopes defined 
for each endpoint at runtime. It gives you the flexibility to grant permissions based on endpoints to consumers. 
 
### Schemas

In the specification, models define what would be the request/response body for each endpoint. There are two different 
places to define models. Flattened/scattered in each endpoint or extracted into the schemas in the components. 

When the model definition scattered in each endpoint, there is no name and the same model might be duplicated in several 
endpoints. The generator does not generate POJO classes as it does not make sense to generate some classes named body0, 
body1, body2, etc. Chances are body0 is for get request, and body2 is for put request. As both of them are dealing a 
same set of attributes, they are the same. How would developers remember that in your get handler, you should use body0 
but in your put handler, you should use body2? They would be shocked when they found that these two classes have the 
same fields and methods except the class names are different. 

To generate POJO classes with proper names and to avoid duplications, it is best to extract common data objects into the 
schemas section under the components. 

Here is an example in the petstore specification.

```yaml
  schemas:
    Pet:
      required:
        - id
        - name
      properties:
        id:
          type: integer
          format: int64
        name:
          type: string
        tag:
          type: string
```

The above Pet definition is used to generate a Java class called Pet in [light-codegen][]. With Pet is defined, you
can put $ref in each point for schema definition. Here is an example response that utilizes the Pet schema definition.

```yaml
      responses:
        '200':
          description: Expected response to a valid request
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Pet'
              examples:
                response:
                  value:
                    id: 1
                    name: Jessica Right
                    tag: pet
``` 

Here is another response example with an array. You can see both endpoints are sharing the same Pet schema and in the
generated code, the corresponding handlers share the same POJO class called Pet.  

```yaml
      responses:
        '200':
          description: An paged array of pets
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Pet'
              example:
                - id: 1
                  name: catten
                  tag: cat
                - id: 2
                  name: doggy
                  tag: dog
```

### Examples

OpenAPI specification gives you an opportunity to define an example response for each endpoint so that your API consumer 
can easily understand what would be expected when the endpoint is accessed. Also, [light-codegen][] has a feature to 
generate mock responses based on the examples defined in the specification. Once you have examples defined, the generated 
project can be built and started with mock responses for consumers to start their work immediately without waiting for 
the provider to complete the API implementation. 

For more details on this topic, please refer to [OpenAPI 3.0 Mock][].


### Naming Convention

As schema name is translated into Java class name, it is better to follow the naming convention of Java. 

- Elements should always start with an upper-case letter, to respect class definitions in generated code, which always start with an upper-case letter.

```
  properties:
     error:
       $ref: 'common/XERR0001/1.0.1/errors.yaml#/error'
  ...
  vs
  ...
  properties:
    error:
      $ref: 'common/XERR0001/1.0.1/errors.yaml#/Error'
```

- Elements should use only alpha-numeric characters and avoid underscores, @ signs or others. OpenAPI general guidelines recommend alpha-numeric only, and, while these would generate correct programming language code, it would break accepted programming guidelines.

```
  "@nameID":
    $ref: "common/XPAR0001/0.0.1/elementDefs.yaml#/NameID"
  "@resourceItemIntegrityCode":
    $ref: "common/XPAR0001/0.0.1/elementDefs.yaml#/ResourceItemIntegrityCode"
...
  filterList:
     type: array
     items:
       $ref: "common/XPAR0001/0.0.1/elementDefs.yaml#/FilterID_Type"
...
vs
...
  nameID:
    $ref: "common/XPAR0001/0.0.2/elementDefs.yaml#/NameID"
  resourceItemIntegrityCode:
    $ref: "common/XPAR0001/0.0.2/elementDefs.yaml#/ResourceItemIntegrityCode"
...
  filterList:
    type: array
    items:
      $ref: "common/XPAR0001/0.0.2/elementDefs.yaml#/FilterIDType"    
```

### Object definitions are to be avoided from the declaration of Body elements

The objects should be moved to the __*components/schemas*__ section of the specification or in an external document, for __*shared definitions*__.

The Body should not contain any declarations as in "type: object".

__Ex.: *selection element*__:

__Original version:__

```
  ...
  selection:
    description:  Identifies the selection to retrieve information for. Only one of the child elements of this structure are to be provided.
    type: object
    properties:
      id:
        $ref: "common/XPAR0001/0.0.1/elementDefs.yaml#/ID"
      card:
        $ref: "common/XPAR0001/0.0.1/elementDefs.yaml#/Card"
  ...      
```

__Updated version:__
```
...
  selection:
    $ref: "#/components/schemas/SelectionEA"
  ...
  schemas:
  # Party Selection structures.
    SelectionEA:
      type: object
      description:  Identifies the selection to retrieve information for. Only one of the child elements of this structure are to be provided.
      properties:
        id:
          $ref: "common/XPAR0001/0.0.2/elementDefs.yaml#/ID"
        card:
          $ref: "common/XPAR0001/0.0.2/elementDefs.yaml#/Card"
  ...
```

### Implicit object definitions are to be avoided from the declaration of Body elements when used from collection declarations

An object should be defined in the __*components/schemas*__ section of the specification or an external document, for __*shared definitions*__, and be referenced from the collection in the Body, instead of the declaration of an implicit object.

The Body should not contain any collection declarations with implicit object definitions; the equivalent of Java inline declarations.

__Ex.: *names element*__:

__Original version:__
```
  ...
  names:
    description: Contact information that can be used in contacting the client. Can be used for either mailing, greeting, or for a third party reference.
    type: array
    items:
      properties:
        "id":
          $ref: "common/XPAR0001/0.0.1/elementDefs.yaml#/ID"
        name:
          description: A name can contain up to two lines of name information.
          type: array
          items:
            $ref: "common/XPAR0001/0.0.1/elementDefs.yaml#/Name"
  ...          
```

__Updated version:__
```
...
  names:
    type: array
    items:
      $ref: "#/components/schemas/GetNames"
  ...
  schemas:
  GetNames:
    description: Contact information that can be used in contacting the client. Can be used for either mailing, greeting, or for a third party reference.
    type: object
    properties:
      id:
        $ref: "common/XPAR0001/0.0.2/elementDefs.yaml#/ID"
      name:
        description: A name can contain up to two lines of name information.
        type: array
        items:
          $ref: "common/XPAR0001/0.0.2/elementDefs.yaml#/Name"
  ...
```



[design first]: /design/design-first/
[light-codegen]: /tool/light-codegen/
[light-rest-4j]: /style/light-rest-4j/
[petstore]: https://github.com/networknt/model-config/tree/master/rest/openapi/petstore/1.0.0
[model-config]: https://github.com/networknt/model-config/tree/master/rest/openapi
[OpenAPI 3.0 Mock]: /tutorial/generator/openapi-mock/