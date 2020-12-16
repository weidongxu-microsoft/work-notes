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
export SPEC_ROOT=<path_to_swagger_spec>
export AUTOREST_JAVA=<path_to_autorest_java_v4>
```

E.g.
```
export SPEC_ROOT=/c/github/azure-rest-api-specs
export AUTOREST_JAVA=/c/github_fork/autorest.java
```

Generate Java code (for all services)

```
gulp codegen --projects features --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA
gulp codegen --projects policy --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA --preserve
gulp codegen --projects subscriptions --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA --preserve
gulp codegen --projects resources --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA --preserve

gulp codegen --projects storage --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA

gulp codegen --projects keyvault --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA

gulp codegen --projects graphrbac --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA
gulp codegen --projects authorization --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA --preserve

gulp codegen --projects msi --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA

gulp codegen --projects network --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA
gulp codegen --projects compute --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA

gulp codegen --projects appservice --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA
gulp codegen --projects containerregistry --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA
gulp codegen --projects containerservice --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA
gulp codegen --projects containerinstance --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA
gulp codegen --projects cosmos --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA
gulp codegen --projects dns --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA
gulp codegen --projects privatedns --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA
gulp codegen --projects monitor --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA
gulp codegen --projects sql --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA
gulp codegen --projects appplatform --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA
gulp codegen --projects redis --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA
gulp codegen --projects eventhubs --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA
gulp codegen --projects trafficmanager --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA
gulp codegen --projects servicebus --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA
gulp codegen --projects cdn --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA
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

For new service, to create corresponding CI, comment in PR:

`/azp run prepare-pipelines`

After it complete, trigger the CI:

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
