# Parity of TypeSpec and Swagger

Mostly with `x-ms-` extensions.
Ref:
- https://azure.github.io/autorest/extensions/
- https://github.com/Azure/autorest/blob/main/docs/extensions/readme.md

## Common

| Feature | Swagger | TypeSpec | State | Comments |
| --- | --- | --- | --- | --- |
| Operation group | OperationId=`Group_Method` | Interface or InterfaceTemplate e.g. `TrackedResourceOperations` (without `@operationGroup`) | Not solved | Should we treat Interface with `@armResourceOperations` as "operation group"? |
| LRO | `x-ms-long-running-operation` | [Issue 3594](https://github.com/Azure/typespec-azure/issues/3594) | Not solved |
| URL encoding | `x-ms-skip-url-encoding` | `url` type | Solved | See [Issue 851](https://github.com/microsoft/typespec/issues/851) |
| Extensible enum | `x-ms-enum` | `Enum` type | Solved |
| Naming | `x-ms-client-name` | `@projectedName` | Solved |
| Discriminator | `x-ms-discriminator-value` | `@discriminator` | Solved | `@discriminator` current does not support deep subclass |
| Mutability | `x-ms-mutability` | `@visibility` | Question | Note there be difference of meaning in Patch. In data-plane it is "create" and "update"; in mgmt it is "update". |
| Error response | `default` or `x-ms-error-response` | `@error` | Solved |
| Pageable | `x-ms-pageable` | `@pagedResult` | Solved |
| Nullable | `x-nullable` | `p?: Type` or `p: Type \| null` | Solved |
| ARM resource model | `x-ms-azure-resource` | `@armResourceInternal`? | Question | Can it be dropped from codegen? |
| ARM id | `x-ms-arm-id-details` | | Question | Do we have equivalent in TypeSpec? Currently seems only .NET codegen uses it. |
| Secret | `x-ms-secret` | `@visibility` without "read" | Solved |

## May not need to support in Ge

| Feature | Swagger | Comments |
| --- | --- | --- |
| Client parameter | `x-ms-parameter-location` | Known to have `subscriptionId` as client parameter (it does not apply to "subscriptions.json" `/subscriptions/{subscriptionId}`) |
| API on same path | `x-ms-paths` | `@sharedRoute` |
| Model flatten | `x-ms-client-flatten` | Avoid flatten in new TypeSpec |

## May not going to support

| Feature | Swagger | Comments |
| --- | --- | --- |
| Parameter grouping | `x-ms-parameter-grouping` or `x-ms-header-collection-prefix` |
| External model | `x-ms-external` | There is design discussion in TypeSpec of external package |
| Odata | `x-ms-odata` |
