# Task completed, document out-of-date

Source repo: https://github.com/Azure/azure-libraries-for-java/tree/vnext

Target repo: https://github.com/Azure/azure-sdk-for-java

### Folder structure

1.

For SDK `azure-mgmt-*`, we can move it to e.g.

`azure-mgmt-storage -> sdk/storage/mgmt`

2.

Name could change for some SDK.

`cosmosdb -> cosmos`

`datalake-analytics -> datalakeanalytics`

`datalake-store -> datalakestore`

`graph-rbac -> authorization`

3.

For aggregate SDK and samples, we can move it to `sdk/management`.

`azure -> sdk/management/azure`

`azure-samples -> sdk/management/samples`

4.

For CI

`.azure-pipelines` moves into `sdk/management`

Do we use them? `ci & tools`

5.

`notes`, and other files in project root would in principle moves into `sdk/management` (need some clean up).

### POM structure

Currently `sdk/management/pom.xml` works as both parent pom and aggregate pom. It aggregates all POMs from `sdk/management/azure`, `sdk/management/sample`, `sdk/resources/mgmt`, `sdk/storage/mgmt`, etc.

POM in root could aggregate this `sdk/management/pom.xml`, to include latest Fluent management SDKs.

We can keep it this way at first, then try to see if we need to break it to 2 separate POMs as parent and as aggregate.

As parent POM, it needs lots of rework to conform it to similar functions as pom.client.xml (checkstyle, spotbugs, javadoc, surefire, jacoco, revapi).
