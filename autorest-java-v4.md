## Autorest Components

### autorest.core v3

Repo

https://github.com/Azure/autorest/tree/v3, branch `v3`.

Install

`(sudo) npm install -g "@autorest/autorest"`

Run

`autorest-beta --info`

Useful parameters

`--reset` drop all extensions.

`--interactive` show UI with pipeline graph, and related input/output.

`--debugger` pause for debugger be attached.

`--version=` fall back to looking at github for specific version.

Output is json format.

### modelerfour

Repo

https://github.com/Azure/autorest.modelerfour

Input from autorest.core v3. Output is yaml format.

### autorest.java v4

Repo

https://github.com/Azure/autorest.java/tree/v4, branch `v4`.

Input from modelerfour.

Build

`mvn package -Dlocal`

Run with single swagger file

`autorest-beta --use:"C:/github_fork/autorest.java" --java --input-file:"C:/github/autorest.testserver/swagger/body-string.json" --output-folder:"tests" --namespace:fixtures.bodystring --sync-methods=all`

Temporary Fluent Repo

https://github.com/weidongxu-microsoft/autorest.java/tree/v4_fluentgen

With `fluentgen` module.

Run with swagger for fluent (initial test, not final)

`autorest-beta --input-file="C:/github/azure-rest-api-specs/specification/resources/resource-manager/Microsoft.Resources/stable/2019-08-01/resources.json" --namespace="com.azure.management.resources" --use="C:/github_fork/autorest.java/fluentgen" --java --azure-arm=true --fluent=true --generate-client-as-impl=true --sync-methods=all --license-header=MICROSOFT_MIT_NO_CODEGEN --output-folder=v4`

## autorest pipeline

autorest.core v3 -> modelerfour -> autorest.java v4

## Legacy test server

Repo

https://github.com/Azure/autorest.testserver

Run under autorest.java

`node node_modules/@microsoft.azure/autorest.testserver`

Possible new test server https://github.com/Azure/autorest.test-server

## Azure Core

Repo

https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/core

Build

Under `./eng/code-quality-reports`

`mvn install`

Under `./sdk/core`

`mvn -DskipTests=true -Dgpg.skip -Dcheckstyle.skip -Dspotbugs.skip -Drevapi.skip -f pom.service.xml -pl azure-core,azure-core-http-netty -am install`

Under `./sdk/identity`

`mvn -DskipTests=true -Dgpg.skip -Dcheckstyle.skip -Dspotbugs.skip -Drevapi.skip -f pom.service.xml install`

## Azure Core Management

Branch on Draft PR https://github.com/Azure/azure-sdk-for-java/pull/6303

Checkout https://github.com/anuchandy/azure-sdk-for-java/tree/mgmt-poller

Modify 2 files

```
diff --git a/sdk/core/azure-core-management/src/main/java/com/azure/core/management/implementation/polling/PollerFactory.java b/sdk/core/azure-core-management/src/main/java/com/azure/core/management/implementation/polling/PollerFactory.javaindex b4865caf14..9ad9116249 100644
--- a/sdk/core/azure-core-management/src/main/java/com/azure/core/management/implementation/polling/PollerFactory.java
+++ b/sdk/core/azure-core-management/src/main/java/com/azure/core/management/implementation/polling/PollerFactory.java
@@ -252,6 +252,7 @@ public final class PollerFactory {
      * @param <U> the type to decode to
      * @return decoded value
      */
+    @SuppressWarnings("unchecked")
     private static <U> U deserialize(SerializerAdapter serializerAdapter, String value, Type type) {
         if (value == null || value.equalsIgnoreCase("")) {
             logger.info("Ignoring decoding of null or empty value to:" + type.getTypeName());
diff --git a/sdk/core/azure-core-management/src/main/java/module-info.java b/sdk/core/azure-core-management/src/main/java/module-info.java
index 010f7492a8..3c43449258 100644
--- a/sdk/core/azure-core-management/src/main/java/module-info.java
+++ b/sdk/core/azure-core-management/src/main/java/module-info.java
@@ -6,7 +6,7 @@ module com.azure.core.management {
     requires org.reactivestreams;

     opens com.azure.core.management to com.fasterxml.jackson.databind;
-    opens com.azure.core.management.implementation to com.fasterxml.jackson.databind;
+    opens com.azure.core.management.implementation.polling to com.fasterxml.jackson.databind;

     uses com.azure.core.http.HttpClientProvider;
 }
```

Also check `sdk/core/azure-core-management/pom.xml` for beta version update on azure-core, azure-core-test, and azure-core-http-netty.

Build

Under `./sdk/core/azure-core-management`

`mvn -DskipTests=true -Dgpg.skip -Dcheckstyle.skip -Dspotbugs.skip install`

## Detail on autorest.java v4

### Configuration

`readme.md` dependency and pipeline

### Flow within Javagen

codeModel -> clientModel -> JAVA code

### Modules

`extension-base`
- `jsonrpc` JSON-RPC, the communication pipeline with autorest.core. `ListInputs`, `ReadValue`, `Message`, `WriteFile`, `ReadFile`.
- `model` basically JAVA representation of **codeModel**, equavalent to the yaml of modelerfour.

`javagen`
- `transformer` transform codeModel into itself.
- `model.clientmodel` **clientModel**, supposed to be representation-neutral model for various JAVA code structure.
- `model.mapper` map codeModel to clientModel.
- `model.javamodel` util for coding and generating JAVA code.
- `template` template (using javamodel) for generating JAVA code; map codeModel to JAVA code.

`tests` test cases.

`Javagen.java` entry point.

### Fluent 

`fluentgen` (parallel to `javagen`) to generate Fluent Lite/Premium JAVA code.
