# After Generate Client SDK via Data-Plane Generator

## Documentation

### README.md

Generator typically helps generate an outline of start-up guide as `README.md` in the output folder.
Upon SDK publish, the content of the `README.md` will be synchronized to [Microsoft Docs site](https://docs.microsoft.com/), and to respective language sites as well.

It is important to understand that automated generator does not have the domain knowledge to make the `README.md` easy to use for first-time user.
Therefore, it is highly recommended for developer to enhance the `README.md`.

How to:
- [Java](https://github.com/Azure/azure-sdk-for-java/wiki/Protocol-Methods-Quickstart-with-AutoRest#improve-sdk-documentation)

### Samples

Besides the start-up guide as `README.md`, user welcomes code samples of variety use cases, from code snippet of API call, to end-to-end champion scenario.
Upon SDK publish, a properly configured samples folder will be synchronized to [Samples Browse](https://docs.microsoft.com/samples/browse/).

TODO (weidxu): there would be a shared guideline on `samples/README.md` and its metadata, for publishing to Samples Browse.

How to:
- [Java](https://github.com/Azure/azure-sdk-for-java/wiki/Protocol-Methods-Quickstart-with-AutoRest#samples)

### API documentation

Generator generates API documentation as docstring in code.

Though all API documentation will be published to Microsoft Docs site, user of language SDK typically use API documentation from language sites.
Therefore, it is helpful for developer to review the API documentation, see it as user will see after SDK publish.

How to:
- [Java](https://github.com/Azure/azure-sdk-for-java/wiki/Protocol-Methods-Quickstart-with-AutoRest#improve-sdk-code)

## Authentication

Generator supports [OAuth2 and ApiKey authentication](https://github.com/Azure/autorest/blob/main/docs/generate/authentication.md).

Example Open API 2.0
```json
"securityDefinitions": {
  "AADToken": {
    "type": "oauth2",
    "flow": "accessCode"
  }
},
"security": [
  {
    "AADToken": ["https://myservice.azure.com/.default"]
  }
]
```

If service requires advanced solution (for example, JWT claim or HMAC-SHA256 authentication scheme), developer is required to customize the SDK.

How to:
- [Java](https://github.com/Azure/azure-sdk-for-java/wiki/Protocol-Methods-Quickstart-with-AutoRest#authentication-and-credential)

## API

### Long-running operation

SDK generated from generator typically supports [LRO in Azure guidelines](https://github.com/microsoft/api-guidelines/blob/vNext/azure/Guidelines.md#long-running-operations--jobs).

Example Open API 2.0
```json
"x-ms-long-running-operation": true
```

It is highly recommended for developer to add live tests for these LROs, and check below language guide on how LRO is handled by SDK.

How to:
- [Java](https://github.com/Azure/azure-sdk-for-java/wiki/Protocol-Methods-Quickstart-with-AutoRest#long-running-operation)

### File upload with multiple content-type

SDK generated from generator provides basic API for file upload.

Example Open API 2.0
```json
"consumes": [
  "application/octet-stream",
  "text/plain",
  "application/json"
],
"parameters": [
  {
    "in": "body",
    "name": "body",
    "description": "The payload body.",
    "required": true,
    "schema": {
      "format": "binary",
      "type": "string"
    }
  }
]
```

In the case of multiple content-type, it is recommended for developer to add convenience methods, which builds upon the basic API.

How to:
- [Java](https://github.com/Azure/azure-sdk-for-java/wiki/Protocol-Methods-Quickstart-with-AutoRest#file-upload)

Alternative to file upload, service can use [Bring Your Own Storage](https://github.com/microsoft/api-guidelines/blob/vNext/azure/Guidelines.md#bring-your-own-storage).

## API after evolution of Open API

### Addition of optional parameter

### Conversion of required parameter to optional parameter

### Addition of content-type
