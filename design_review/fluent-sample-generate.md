# Design Review - Generate samples for Java Fluent Lite

# Summary

1. Design on autorest.java to support generate Java sample code from Swagger JSON example
2. Coordination to make the samples accessible to customer

# Problem Statement or Scenario Description

Swagger on management contains extensive [example JSONs](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/mysql/resource-manager/Microsoft.DBforMySQL/stable/2017-12-01/examples/ServerCreate.json), which is released to [MSDocs](https://docs.microsoft.com/en-us/rest/api/mysql/singleserver/servers(2017-12-01)/create#create-a-new-server).
And swagger teams continue working on improving the quality of these examples.

It is not straightforward for customer to write SDK code from the JSON example (especially for strongly-typed languages), and usually SDK docs and swagger docs do not link to each other and hence hard to find.

Therefore first-time user can feel lost on the collection of classes and methods.
Even experienced user could find it difficult to configure Azure resource, when they start on a new service.

As a solution, it would be very helpful to generate SDK sample, keep it compile with the SDKs, and make it accessible to customer.

# Scope: Goals and Non-Goals

## Goals

1. Generate Java sample code from Swagger JSON example
2. Keep sample code at same quality as that of Swagger JSON
3. Make the sample accessible to customer

Long-term candidate

4. Feedback for possible JSON issue (e.g. required property not provided in JSON example; extra property in JSON example but not in SDK/Swagger)
5. Smart classification and simplification on placeholder (e.g. resource group name, resource name, dependent resource ID)

## Non-Goals

1. Not directly usable by customer, as it contains placeholder that requires input from some other sources
2. Not serve as test cases or test frameworks
3. No smart fix if swagger or JSON is not correct

# Design Details

## Generate Java sample code

Example JSON (in YAML format) already exists in CodeModel (under `x-ms-examples`).

First step, `x-ms-examples` transform to multiple `ProxyMethodExample` under `ProxyMethod` (`ProxyMethod` one-to-one maps to REST API, i.e. operation in m4).
It is a simple parameter to value map on REST API.

For example, `ProxyMethodExample`
```yaml
databaseName: db1
serverName: testserver
resourceGroupName: TestGroup
api-version: 2017-12-01
subscriptionId: ffffffff-ffff-ffff-ffff-ffffffffffff
parameters: {"properties": {"charset": "utf8", "collation": "utf8_general_ci"}}
```
for `Databases.CreateOrUpdate` operation
```java
@Put("/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforMySQL/servers/{serverName}/databases/{databaseName}")
@ExpectedResponses({200, 201, 202})
@UnexpectedResponseExceptionType(ManagementException.class)
Mono<Response<Flux<ByteBuffer>>> createOrUpdate(
    @HostParam("$host") String endpoint,
    @QueryParam("api-version") String apiVersion,
    @PathParam("subscriptionId") String subscriptionId,
    @PathParam("resourceGroupName") String resourceGroupName,
    @PathParam("serverName") String serverName,
    @PathParam("databaseName") String databaseName,
    @BodyParam("application/json") DatabaseInner parameters,
    @HeaderParam("Accept") String accept,
    Context context);
```

This simple `ProxyMethodExample` can be potentially used by LLC.

Then autorest.java would do certain transformation on models and APIs. The final product is `FluentCollectionMethod`, `ResourceCreate`, and `ResourceUpdate`.
All of them still contains a `ProxyMethod` and hence the related `ProxyMethodExample`.

For reference, `FluentCollectionMethod >> ClientMethod >> ProxyMethod` and `ResourceCreate/ResourceUpdate >> FluentCollectionMethod >> ClientMethod >> ProxyMethod`.

Then, `ExampleParser` will transform the simple `ProxyMethodExample` to one of `FluentCollectionMethodExample`, `FluentResourceCreateExample`, `FluentResourceUpdateExample`.
All of them contains multiple trees of `ExampleNode`, which is constructed from the root of the JSON, and recursively to JSON literal, in reference to the swagger model.

Take an example on `FluentCollectionMethodExample`. Its `ExampleNode` tree is constructed from parameter value, with the knowledge of models (e.g. `DatabaseInner.charset` serialize to `properties.charset`):
```yaml
LiteralNode: "TestGroup"
LiteralNode: "testserver"
LiteralNode: "db1"
ModelNode: (model=DatabaseInner)
- LiteralNode: "utf8" (accesser=charset)
- LiteralNode: "utf8_general_ci" (accesser=collation)
```
and generated as:
```java
mySqlManager.databases().createOrUpdate(
    "TestGroup",
    "testserver",
    "db1",
    new DatabaseInner()
        .withCharset("utf8")
        .withCollation("utf8_general_ci"));
```
for `Databases.CreateOrUpdate` operation (for illustration only, this form does not exist in code)

`ModelNode` is internal node (as it require further elaboration on the properties/accessers), `LiteralNode` is external node (as boolean, int, or something that can be converted directly from string, e.g. enum, DateTime, or UUID).
There is a few other types of `ExampleNode`, e.g. `ListNode` and `MapNode` as internal, `ObjectNode` as external.

`FluentResourceCreateExample` is slight different, but basic structure is similar. It generates:
```java
mySqlManager
    .databases()
    .define("db1")
    .withExistingServer("TestGroup", "testserver")
    .withCharset("utf8")
    .withCollation("utf8_general_ci")
    .create();
```
for `Databases.CreateOrUpdate` operation.

Finally, `FluentExampleTemplate` generates the code we have seen above.

It convert each `ExampleNode` to code segment. For example:

- `ModelNode: (model=DatabaseInner)` generates `new DatabaseInner()`
- `LiteralNode: "utf8" (accesser=charset)` generates `.withCharset("utf8")`
- `ListNode` generates `Arrays.asList(...)`
- `ObjectNode` use primitive type if example value is JSON literal (string/number/boolean), otherwise code to deserialized JSON object/array

And include additional code, like import, class and methods, and optional utility methods.

Complications:

1. `flatten-models` in m4
1. `flatten-payloads` and `group-parameters` in m4, we try to avoid these options
1. Polymorphism of model, need to find the correct sub-model base on discriminator value
1. `object` in swagger with no properties, need to deserialized from JSON
1. `additionalProperties` in swagger
1. Parameter serialization, e.g. `collectionFormat=csv` and `collectionFormat=multi` in swagger
1. `anyOf` (in future)

## Technology Considerations

### No YAML output on `FluentCollectionMethodExample`, `FluentResourceCreateExample`, `FluentResourceUpdateExample`

Reason

1. Existing autorest.java classes except CodeModel is not designed to be serialized.
1. And these classes are for Java only, and hence not language-agnostic. Even if it is serialized to YAML, it is not possible to be reused by other languages.
1. Easier to handle in memory than via YAML. Input/output of YAML via SnakeYaml does not work well with the one in TS.

Con

1. It is not as modular as autorest plugins. If we target multiple types of output (i.e. samples, tests, test scenarios), the code will be in same plugin, switchable via configuration.

Reflection on this: we should keep it modular for future expansion.

### Sample on Client (instead of Fluent) for Fluent Premium

Reason

1. Fluent layer on Fluent Premium is hand-written, accompanied with hand-written end2end samples. It is not possible to have generated sample to cover this part.
1. Generate sample on the generated Client layer, e.g. `computeManager.serviceClient().getVirtualMachines().createOrUpdate(...)`.
1. Provide code comment or `documentation-link.md` to refer to the hand-written samples as well.

## Verify sample code quality

Auto verification during generation

1. Generated sample code passes compilation.
1. Warn if required parameter/property is not provided.
1. Warn if value in JSON example is not used in sample code (exclude constant value, and other client parameter e.g. subscription ID and api-version).

Unit test

1. Test cases for selective scenario on generated sample code.

Smoke test

1. Similar daily report as the current [autorest.java daily run on all RPs](https://github.com/Azure/autorest.java/blob/v4/fluent-tests/report.md)

Manual verification

1. For selective scenario, fill the placeholders and send live request.

## Make the sample accessible to customer

For Java, generate `SAMPLE.md` links to all the sample files, and have [`README.md`](https://github.com/Azure/azure-sdk-for-java/blob/main/sdk/mysql/azure-resourcemanager-mysql/README.md#examples) link to the `SAMPLE.md`.

The content of the `README.md` will be published to [MSDocs](https://docs.microsoft.com/en-us/java/api/overview/azure/resourcemanager-mysql-readme?view=azure-java-stable#examples) upon release.

## Technology Considerations

### Integrate with REST API MSDocs

We already have JSON examples on [azure-rest-api-specs](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/compute/resource-manager/Microsoft.Compute/stable/2021-03-01/examples).
And it will be released to [MSDocs](https://docs.microsoft.com/en-us/rest/api/compute/virtual-machines/create-or-update#create-a-custom-image-vm-from-an-unmanaged-generalized-os-image.).

We plan to commit sample SDK code to azure-rest-api-specs repository as well.

For example, we had JSON example at
```
Microsoft.Compute/stable/2021-03-01/examples/CreateAVmFromACustomImage.json
```

And we would create
```
Microsoft.Compute/stable/2021-03-01/examples-java/CreateAVmFromACustomImage.md
Microsoft.Compute/stable/2021-03-01/examples-python/CreateAVmFromACustomImage.md
Microsoft.Compute/stable/2021-03-01/examples-go/CreateAVmFromACustomImage.md
```
for java, python, golang respectively, upon SDK release for API version 2021-03-01.

This way, it will be straightforward for MSDocs team to gather all related code samples across SDKs when updating MSDocs for REST API.

The final MSDocs on REST API might look like the current one for [MSGraph](https://docs.microsoft.com/en-us/graph/api/user-post-events?view=graph-rest-1.0&tabs=http#examples).

There could be a bit of technical challenge to create the side-by-side sample file in azure-rest-api-specs, as autorest currently erased filename information on the JSON examples.

A candidate solution is to print the HTTP path, HTTP method, api-version, example name, operation group name, operation name (last 2 is likely redundant) in the comment block of the sample or md. And have a post-process to match it to Swagger.

For example, the above `CreateAVmFromACustomImage` case would have this in comment block
```yaml
path: /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}
method: put
api-version: '2021-03-01'
example: Create a vm from a custom image.
operationGroup: VirtualMachines
operation: CreateOrUpdate
```

And a post-process will search in `compute.json` to find that it matches `Microsoft.Compute/stable/2021-03-01/examples/CreateAVmFromACustomImage.json`.

Alternatively we can check with autorest whether they can carry the filename along the processing, though in current design the filename is likely discarded in early stages.

# Appendix

### Deep `ExampleNode`

[VirtualMachines_CreateOrUpdate](https://github.com/weidongxu-microsoft/share/blob/main/compute/src/samples/java/com/azure/resourcemanager/compute/VirtualMachinesCreateOrUpdateSamples.java)

### Polymorphism

[Servers_Create](https://github.com/Azure/azure-sdk-for-java/blob/main/sdk/mysql/azure-resourcemanager-mysql/src/samples/java/com/azure/resourcemanager/mysql/ServersCreateSamples.java)

At least 4 different sub-models for `ServerProperties`: `ServerPropertiesForReplica`, `ServerPropertiesForGeoRestore`, `ServerPropertiesForDefaultCreate`, `ServerPropertiesForRestore`.

### `object` with no properties

[PolicyDefinitions_CreateOrUpdate](https://github.com/weidongxu-microsoft/share/blob/main/policy/src/samples/java/com/azure/resourcemanager/policy/PolicyDefinitionsCreateOrUpdateSamples.java)

### Existing procedure to extract code from manual end2end test as sample

[ADF](https://github.com/Azure/azure-sdk-for-java/tree/main/sdk/datafactory/azure-resourcemanager-datafactory#examples)
