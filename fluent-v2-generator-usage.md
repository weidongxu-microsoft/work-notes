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
gulp codegen --projects monitor --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA
gulp codegen --projects sql --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA
gulp codegen --projects appplatform --spec-root $SPEC_ROOT --autorest-java $AUTOREST_JAVA
```

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
