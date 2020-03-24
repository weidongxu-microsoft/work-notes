Source repo: https://github.com/Azure/azure-libraries-for-java

Target repo: https://github.com/Azure/azure-sdk-for-java

1.

For SDK `azure-mgmt-*`, we can move it to e.g.

`azure-mgmt-storage -> sdk/storage/mgmt`

Question: Should we use a better name than "mgmt"? Do we need to save a name for track1?

2.

Name could change for some SDK.

`cosmosdb -> cosmos`

`datalake-analytics -> datalakeanalytics`

`datalake-store -> datalakestore`

`graph-rbac -> authorization`

3.

For aggregate SDK and samples, we can move it to `sdk/management`.

`azure -> management/azure`

`azure-samples -> management/samples`

4.

For CI

`.azure-pipelines -> management`

Do we use them? `ci & tools`

5.

`notes`, and other files in project root would in principle go to `sdk/management` (need some clean up).
