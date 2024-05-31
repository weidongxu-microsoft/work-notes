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

```
gulp codegen --projects changes
gulp codegen --projects locks --preserve
gulp codegen --projects features --preserve
gulp codegen --projects policy --preserve
gulp codegen --projects subscriptions --preserve
gulp codegen --projects resources --preserve
gulp codegen --projects deploymentstacks --preserve

gulp codegen --projects storage

gulp codegen --projects keyvault

gulp codegen --projects msgraph
gulp codegen --projects authorization --preserve

gulp codegen --projects msi

gulp codegen --projects network
gulp codegen --projects compute

gulp codegen --projects appservice
gulp codegen --projects containerregistry
gulp codegen --projects containerservice
gulp codegen --projects containerinstance
gulp codegen --projects cosmos
gulp codegen --projects dns
gulp codegen --projects privatedns
gulp codegen --projects monitor
gulp codegen --projects sql
gulp codegen --projects appplatform
gulp codegen --projects redis
gulp codegen --projects eventhubs
gulp codegen --projects trafficmanager
gulp codegen --projects servicebus
gulp codegen --projects cdn
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
