# Prepare develop environment #

### Install autorest v3 ###

`(sudo) npm install -g "@autorest/autorest"`

Update autorest.core

`autorest-beta –version=3.0.6182`

### Build autorest.java v4 fluentgen ###

Checkout branch https://github.com/weidongxu-microsoft/autorest.java/tree/v4_fluentgen

Build

`mvn package -Dlocal`

### Build autorest core ###

https://github.com/weidongxu-microsoft/work-notes/blob/master/autorest-java-v4.md#azure-core

### Check out Fluent Java v2 ###

Checkout branch https://github.com/Azure/azure-libraries-for-java/tree/vnext

Overall build might not work. But build/run single test case within a successfully migrated service should work.

# Migrate SDK in Fluent #

### 1. Update api-spec.json ###

Change `package` to e.g. `com.azure.management.resources` (without microsoft in namespace)

Optionally, add to `args`.
`--add-inner=Class` if you want some class be named as FooInner.
`--add-client-flatten=Class` or `--add-client-flatten=Class.properties` if there is x-ms-client-flatten not correctly processed.

### 2. Regenerate code ###

Generate code

`gulp codegen --projects {SERVICE} --spec-root={SPEC_ROOT} --autorest-java={AUROREST_JAVA_ROOT}/fluentgen`

Since there is local processing for README.md, SPEC_ROOT must be a local folder.

Note `./fluentgen` under AUROREST_JAVA_ROOT.

# General notes #

### Async types ###

`Observable` -> `Mono`

`Observable` (result of list) -> `PagedFlux`

`PagedList` -> `PagedIterable`

Async to sync: `.block()` for single or `new PagedIterable<>(...)` for collection.

### Properties ###

Use `getFoo` or `isFoo`.

# Reference #

https://github.com/weidongxu-microsoft/work-notes/blob/master/autorest-java-v4.md#autorestcore-v3