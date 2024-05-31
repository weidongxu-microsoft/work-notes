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
```

### Script to generate code for Fluent Premium

```
npx gulp codegen --projects changes
npx gulp codegen --projects locks --preserve
npx gulp codegen --projects features --preserve
npx gulp codegen --projects policy --preserve
npx gulp codegen --projects subscriptions --preserve
npx gulp codegen --projects deploymentstacks --preserve
npx gulp codegen --projects resources --preserve

npx gulp codegen --projects storage

npx gulp codegen --projects keyvault

npx gulp codegen --projects msgraph
npx gulp codegen --projects authorization --preserve

npx gulp codegen --projects msi

npx gulp codegen --projects network
npx gulp codegen --projects compute

npx gulp codegen --projects appservice
npx gulp codegen --projects containerregistry
npx gulp codegen --projects containerservice
npx gulp codegen --projects containerinstance
npx gulp codegen --projects cosmos
npx gulp codegen --projects dns
npx gulp codegen --projects privatedns
npx gulp codegen --projects monitor
npx gulp codegen --projects sql
npx gulp codegen --projects appplatform
npx gulp codegen --projects redis
npx gulp codegen --projects eventhubs
npx gulp codegen --projects trafficmanager
npx gulp codegen --projects servicebus
npx gulp codegen --projects cdn
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
