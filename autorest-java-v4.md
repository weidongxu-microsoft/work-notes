## Autorest Components

### autorest v3

Repo

https://github.com/Azure/autorest/

Install

`(sudo) npm install -g autorest`

Run

`autorest --info`

Useful parameters

`--reset` drop all extensions.

`--java.debugger` pause for Java debugger be attached.

`--version=` fall back to looking at github for specific version.

Doc

https://github.com/Azure/autorest/tree/master/docs/generate

### autorest.core

Repo

https://github.com/Azure/autorest/tree/master/packages/extensions/core

Output is json format.

### modelerfour

Repo

https://github.com/Azure/autorest/tree/master/packages/extensions/modelerfour

Input from autorest.core v3. Output is yaml format.

### autorest.java v4

#### Repo

https://github.com/Azure/autorest.java/

Input from modelerfour.

`fluentnamer` reads input from modelerfour, and output to `fluentgen` module.
`fluentgen` module output to files.

#### Build

`mvn clean package -P local`

Build and integration test (need autorest installed, currently run on Windows)

`mvn clean package -P local,testFluent`

Build and check code coverage (report generated in `fluentgen/target/site/test-coverage`)

`mvn clean verify -P local`

#### Release

1. Incremental version in `package.json`
2. Build.
3. Run `npm pack`, and upload the tgz to Github Release asset.

Github Action for release https://github.com/Azure/autorest.java/actions/workflows/pre-release.yml

After release, consider updating [configure](https://github.com/Azure/azure-sdk-for-java/blob/master/eng/mgmt/automation/parameters.py) for the [Fluent Lite automation on azure-sdk-for-java](https://github.com/weidongxu-microsoft/work-notes/blob/master/fluent-v2-generator-usage.md#automation) as well as the Swagger automation.

## autorest pipeline

autorest.core v3 -> modelerfour -> autorest.java v4

## Test server

Repo

https://github.com/Azure/autorest.testserver

Run under autorest.java

`node node_modules/@microsoft.azure/autorest.testserver`

Dashboard http://azure.github.io/autorest/dashboard.html

Possible new test server https://github.com/Azure/autorest.test-server

## Azure Core

Repo

https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/core

Build core (not needed unless Azure Core Management uses unpublished Azure Core)

`mvn -Dgpg.skip -Dcheckstyle.skip -Dspotbugs.skip -Djacoco.skip -Drevapi.skip -Dmaven.javadoc.skip -pl com.azure:azure-core,com.azure:azure-core-test,com.azure:azure-core-http-netty -am clean install`

Build other SDK: change `com.azure:azure-core` to other artifacts

`eng/code-quality-reports` is required to be installed, if any check is required (i.e. checkstyle, spotbugs, revapi).

## Azure Core Management

Same repo as [Core](#azure-core).

Build

`mvn -Dgpg.skip -Dcheckstyle.skip -Dspotbugs.skip -Djacoco.skip -Drevapi.skip -Dmaven.javadoc.skip -pl com.azure:azure-core-management -am clean install`

(Optional) Purge local repository of previous build

`mvn dependency:purge-local-repository -DmanualInclude=com.azure:azure-core-management`

## Detail on autorest.java v4

### Configuration

`readme.md` dependency and pipeline

### Flow within Javagen

codeModel -> clientModel -> JAVA code

### Modules

`extension-base`
- `jsonrpc` JSON-RPC, the communication pipeline with autorest.core. `ListInputs`, `ReadValue`, `Message`, `WriteFile`, `ReadFile`.
- `model` basically JAVA representation of **codeModel**, equavalent to the yaml of modelerfour.

`preprocessor`
- `transformer` transform codeModel into itself.

`javagen`
- `model.clientmodel` **clientModel**, supposed to be representation-neutral model for various JAVA code structure.
- `model.mapper` map codeModel to clientModel.
- `model.javamodel` util for coding and generating JAVA code.
- `template` template (using javamodel) for generating JAVA code; map codeModel to JAVA code.
- `package.json` npm config.
- `readme.md` readme, and config for autorest pipeline.

`tests` test cases.

### Fluent 

`fluentnamer` (parallel to `preprocessor`)

`fluentgen` (parallel to `javagen`) to generate Fluent Lite/Premium Java code.

`fluent-tests` to generate Fluent Java code based on genuine management swagger spec, test for compilation and conformity to expected interface.

### Samples

`generate.bat` for vanilla (data-plane).
`fluent-tests/prepare-tests.bat` for fluent (management-plane).

### Internal generated code for selected services

https://github.com/ChenTanyi/autorest.java/tree/ci/test
