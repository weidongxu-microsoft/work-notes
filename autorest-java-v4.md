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

`autorest-beta --use:"C:\github_fork\autorest.java" --java --input-file:"C:\github\autorest.testserver\swagger\body-string.json" --output-folder:"tests" --namespace:fixtures.bodystring`

Run with swagger for fluent lite (initial test, not final)

Run under e.g. `azure-rest-api-specs/specification/redis/resource-manager`

`autorest-beta --use="C:\github_fork\autorest.java" --java --azure-arm=true --fluent=true --license-header=MICROSOFT_MIT_NO_CODEGEN --multiapi=true --output-folder=v4`

## autorest pipeline ##

autorest.core v3 -> modelerfour -> autorest.java v4

## Legacy test server ##

Repo

https://github.com/Azure/autorest.testserver

Run under autorest.java

`node node_modules/@microsoft.azure/autorest.testserver`

Possible new test server

https://github.com/Azure/autorest.test-server
