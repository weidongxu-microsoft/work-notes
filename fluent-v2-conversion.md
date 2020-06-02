# Prepare develop environment

### Install autorest v3

`(sudo) npm install -g autorest

Update autorest.core

`autorest --version=latest`

### Build autorest.java v4 fluentgen

Checkout branch https://github.com/weidongxu-microsoft/autorest.java/tree/v4_fluentgen

Build

`mvn package -P local`

### Build autorest core

https://github.com/weidongxu-microsoft/work-notes/blob/master/autorest-java-v4.md#azure-core

### Check out Fluent Java

Checkout branch https://github.com/Azure/azure-sdk-for-java

Under `/sdk/management`

# Migrate SDK in Fluent

### 1. Update api-spec.json

Change `package` to e.g. `com.azure.management.resources` (without microsoft in namespace).

Optionally, add to `args`.
`--add-inner=Class` if you want some class be named as FooInner.

### 2. Regenerate code

Generate code

`gulp codegen --projects {SERVICE} --spec-root={SPEC_ROOT} --autorest-java={AUROREST_JAVA_ROOT}`

# Migrate notes

### Async types

#### Reactor does not allow `null` in stream

`Observable` -> `Mono`

`Observable` (result of list) -> `PagedFlux`

`PagedList` -> `PagedIterable`

Async to sync: `.block()` for single or `new PagedIterable<>(...)` for collection.

#### Common Reactor method

```
map
flatMap
then
switchIfEmpty
onErrorResume

Mono.error
Mono.just
Mono.fromCallable
Mono.defer
```

#### Example

```
getByIdAsync(id).map(this::WrapModel)

createOrUpdateAsync(param).then(Mono.fromCallable(() -> { cleanUp(); return this; }))

getByNameAsync(name).onErrorResume(...).switchIfEmpty(Mono.error(...))
```

Read [Reactor Core Features](https://projectreactor.io/docs/core/release/reference/#core-features)

Optionally, also [Advanced Features and Concepts](https://projectreactor.io/docs/core/release/reference/#advanced)

# Reference

https://github.com/weidongxu-microsoft/work-notes/blob/master/autorest-java-v4.md
