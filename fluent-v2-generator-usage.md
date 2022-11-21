### Repository of autorest.java

https://github.com/Azure/autorest/

### Target track2 Java SDK repository

https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/resourcemanager

### Swagger spec repostitory

https://github.com/Azure/azure-rest-api-specs

### Environment

Utility is based on [Node.js](https://nodejs.org/en/). Please use 12 LTS or later version.

Prepare nodejs

```
npm install
npm install -g autorest gulp
```

### Script to generate code for Fluent Premium

Prepare env

```
export AUTOREST_JAVA=<path_to_autorest_java_v4>
```

E.g.
```
export AUTOREST_JAVA=/c/github/autorest.java
```

Generate Java code (for all services)

```
gulp codegen --projects changes --autorest-java $AUTOREST_JAVA
gulp codegen --projects locks --autorest-java $AUTOREST_JAVA --preserve
gulp codegen --projects features --autorest-java $AUTOREST_JAVA --preserve
gulp codegen --projects policy --autorest-java $AUTOREST_JAVA --preserve
gulp codegen --projects subscriptions --autorest-java $AUTOREST_JAVA --preserve
gulp codegen --projects resources --autorest-java $AUTOREST_JAVA --preserve

gulp codegen --projects storage --autorest-java $AUTOREST_JAVA

gulp codegen --projects keyvault --autorest-java $AUTOREST_JAVA

gulp codegen --projects msgraph --autorest-java $AUTOREST_JAVA
gulp codegen --projects authorization --autorest-java $AUTOREST_JAVA --preserve

gulp codegen --projects msi --autorest-java $AUTOREST_JAVA

gulp codegen --projects network --autorest-java $AUTOREST_JAVA
gulp codegen --projects compute --autorest-java $AUTOREST_JAVA

gulp codegen --projects appservice --autorest-java $AUTOREST_JAVA
gulp codegen --projects containerregistry --autorest-java $AUTOREST_JAVA
gulp codegen --projects containerservice --autorest-java $AUTOREST_JAVA
gulp codegen --projects containerinstance --autorest-java $AUTOREST_JAVA
gulp codegen --projects cosmos --autorest-java $AUTOREST_JAVA
gulp codegen --projects dns --autorest-java $AUTOREST_JAVA
gulp codegen --projects privatedns --autorest-java $AUTOREST_JAVA
gulp codegen --projects monitor --autorest-java $AUTOREST_JAVA
gulp codegen --projects sql --autorest-java $AUTOREST_JAVA
gulp codegen --projects appplatform --autorest-java $AUTOREST_JAVA
gulp codegen --projects redis --autorest-java $AUTOREST_JAVA
gulp codegen --projects eventhubs --autorest-java $AUTOREST_JAVA
gulp codegen --projects trafficmanager --autorest-java $AUTOREST_JAVA
gulp codegen --projects servicebus --autorest-java $AUTOREST_JAVA
gulp codegen --projects cdn --autorest-java $AUTOREST_JAVA
```

### Script to generate code for Fluent Lite

Prepare env

```
export AUTOREST_JAVA=<path_to_autorest_java_v4>
export AZURE_JAVA=<path_to_azure_sdk_for_java>

export MODELERFOUR_ARGUMENTS="--pipeline.modelerfour.flatten-payloads=false"
export FLUENTLITE_ARGUMENTS="--java --use=$AUTOREST_JAVA --azure-libraries-for-java-folder=$AZURE_JAVA $MODELERFOUR_ARGUMENTS --azure-arm --fluent=lite --license-header=MICROSOFT_MIT_SMALL"

export RP=<resoruce_provider>
```

Generate Java code

```
autorest $FLUENTLITE_ARGUMENTS --output-folder=$AZURE_JAVA/sdk/$RP/azure-resourcemanager-$RP --sdk-integration --java.namespace=com.azure.resourcemanager.$RP https://raw.githubusercontent.com/Azure/azure-rest-api-specs/master/specification/$RP/resource-manager/readme.md
```

If PreCheck fails but still need to generate code for the RP, override this in `MODELERFOUR_ARGUMENTS` env.

```
export MODELERFOUR_ARGUMENTS="--pipeline.modelerfour.additional-checks=false --pipeline.modelerfour.lenient-model-deduplication=true --pipeline.modelerfour.flatten-payloads=false"
```

#### Automation

https://dev.azure.com/azure-sdk/internal/_build?definitionId=2238

For new service, to create corresponding CI, comment in PR

`/azp run prepare-pipelines`

After it complete, trigger the CI

`/azp run java - <service> - ci`

### Code

nodejs configure `package.json`

nodejs gulp `gulpfile.js`

config `api-specs.json`

### Other utilities

#### servcheck

Check the difference from current implemented api-version to lastest api-version in spec.

`npm run servcheck -- --spec-root=<path_to_swagger_spec>`

#### credcheck

Delete secrets in recorded sessions. (Currently include storage account connection string)

`npm run credcheck_all`

### Reference

[autorest.java v4](autorest-java-v4.md)
