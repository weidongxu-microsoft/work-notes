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

`mvn package -P local`

Run with single swagger file

`autorest-beta --use:"C:/github_fork/autorest.java" --java --input-file:"C:/github/autorest.testserver/swagger/body-string.json" --output-folder:"tests" --namespace:fixtures.bodystring --sync-methods=all`

Temporary Fluent Repo

https://github.com/weidongxu-microsoft/autorest.java/tree/v4_fluentgen

With `fluentgen` module.

Build and integration test (need autorest installed, currently run on Windows)

`mvn package -P local,integration`

[Azure DevOps CI](https://dev.azure.com/weidxu/public/_build?definitionId=4)

## autorest pipeline

autorest.core v3 -> modelerfour -> autorest.java v4

## Legacy test server

Repo

https://github.com/Azure/autorest.testserver

Run under autorest.java

`node node_modules/@microsoft.azure/autorest.testserver`

Possible new test server https://github.com/Azure/autorest.test-server

## Azure Core

Repo (use Azure Core Managemen branch for now)

https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/core

Build

Under `./eng/code-quality-reports`

`mvn install`

Build core

Under `./sdk/core`

`mvn -DskipTests -Dgpg.skip -Dcheckstyle.skip -Dspotbugs.skip -Drevapi.skip -f pom.service.xml -pl azure-core,azure-core-http-netty -am install`

Build identity

Under `./sdk/identity`

`mvn -DskipTests -Dgpg.skip -Dcheckstyle.skip -Dspotbugs.skip -Drevapi.skip -f pom.service.xml install`

Build keyvault (not needed for now)

Under `./sdk/keyvault`

`mvn -DskipTests -Dgpg.skip -Dcheckstyle.skip -Dspotbugs.skip -Drevapi.skip -f pom.service.xml install`

## Azure Core Management

Checkout https://github.com/yaohaizh/azure-sdk-for-java/tree/mgmt-poller

Build

Under `./sdk/core/azure-core-management`

`mvn -DskipTests -Dgpg.skip -Dcheckstyle.skip -Dspotbugs.skip -Drevapi.skip install`

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
