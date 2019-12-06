## Components ##

### autorest.core v3 ###

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

Output is json format.

### modelerfour ###

Repo

https://github.com/Azure/autorest.modelerfour

Input from autorest.core v3. Output is yaml format.

### autorest.java v4 ###

Repo

https://github.com/Azure/autorest.java/tree/v4, branch `v4`.

Input from modelerfour.

Build

`mvn package -Dlocal`

Run with single swagger file

`autorest-beta --use:"C:/github_fork/autorest.java" --java --input-file:"C:/github/autorest.testserver/swagger/body-string.json" --output-folder:"tests" --namespace:fixtures.bodystring --sync-methods=all`

Run with swagger for fluent (initial test, not final)

`autorest-beta --input-file="C:/github/azure-rest-api-specs/specification/resources/resource-manager/Microsoft.Resources/stable/2019-08-01/resources.json" --namespace="com.microsoft.azure.management.resources" --use="C:/github_fork/autorest.java" --java --azure-arm=true --fluent=true --sync-methods=all --license-header=MICROSOFT_MIT_NO_CODEGEN  --output-folder=v5`

## autorest pipeline ##

autorest.core v3 -> modelerfour -> autorest.java v4

## Legacy test server ##

Repo

https://github.com/Azure/autorest.testserver

Run under autorest.java

`node node_modules/@microsoft.azure/autorest.testserver`

Possible new test server https://github.com/Azure/autorest.test-server

## Detail on Autorest.java v4 ##

### Configuration ###

`readme.md` dependency and pipeline

### Flow within Javagen ###

codeModel -> clientModel -> JAVA code

### Modules ###

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

### Fluent ###

We suppose to add a `fluentgen` (parallel to `javagen`) to generate Fluent Lite/Premium JAVA code.

### Current Status ###

modelerfour is not finished and may not be stable yet.

Still working on vanila and azure-arm. I think short-term goal is generating comparable featured JAVA code as autorest.java v3.

Some minor feature in vanila is missing. LRO and Paging seems not finished.

Long-term goal is design for fluentgen, and implement it.
