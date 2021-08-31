# Automation to collect SDK examples to central repository

## Goal for the repository that will be pulled by MSDocs team

Currently, the repo name will be "Azure/azure-rest-api-specs-examples".

1. Same folder structure as azure-rest-api-specs
2. Only examples of the released SDK will be uploaded
3. Keep it minimal

Therefore, we might need another (private) repo for automation (configuration and tooling, etc.).

## Design on the process to publish and collect SDK examples

### Option 1 - Directly pull from SDK repo

Step 1, when SDK team generates package with new version, it also generates the examples and upload to SDK repo as well.
E.g. [an aggregated sample that composed by multiple samples](https://github.com/Azure/azure-sdk-for-java/blob/main/sdk/datafactory/azure-resourcemanager-datafactory/src/samples/java/com/azure/resourcemanager/datafactory/PipelinesCreateOrUpdateSamples.java)

For each example, a certain inline metadata is required. E.g.,
```
// operationId: VirtualMachineScaleSets_CreateOrUpdate
// api-version: 2021-04-01
// x-ms-examples: Create a custom-image scale set from an unmanaged generalized os image.
```
or
```
// x-ms-original-file: specification/compute/resource-manager/Microsoft.Compute/stable/2021-04-01/examples/CreateACustomImageVmFromAnUnmanagedGeneralizedOsImage.json
```

After modelerfour support (tracked via [modelerfour feature](https://github.com/Azure/autorest/issues/4251)), we only need the single line of `x-ms-original-file` as it points us to the location of the source JSON example.
E.g. the [JSON example](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/compute/resource-manager/Microsoft.Compute/stable/2021-04-01/examples/CreateACustomImageVmFromAnUnmanagedGeneralizedOsImage.json)

Therefore, the automation is able to locate the JSON example file, and potentially break the aggregated sample and post its components to corresponding location to examples repo.

This step is done by SDK. Only `x-ms-original-file` inline metadata is required.

Step 2, when SDK team releases the new package, release pipeline will automatically create a new release tag.

For instance, Java release tag is "azure-resourcemanager_2.7.0". Golang release tag is "storage/armstorage/v0.1.1".
Generally the release tag contains the package name and version.

This step should be fully automated.

Documentation reference is something like below, so that customer can get to the SDK doc and start using it.
```
Read the [SDK documentation](https://github.com/Azure/azure-sdk-for-java/blob/azure-resourcemanager_2.7.0/sdk/resourcemanager/README.md) on how to add the SDK to your project and authenticate.
```

Step 3, automation to collect the examples.

Potentially it includes compiler and formatter, if there is requirement to break the aggregated SDK sample to multiple SDK examples.

So that after break-down and re-construction of the examples, automation will verify compile pass, with correct formatting.

The final result could be like [this compute example in markdown](https://raw.githubusercontent.com/weidongxu-microsoft/azure-rest-api-specs-examples/8544933b1852081db8b26c9b1b44651029b757b0/specification/compute/resource-manager/Microsoft.Compute/stable/2021-04-01/examples-java/CreateACustomImageScaleSetFromAnUnmanagedGeneralizedOsImage.md).

This step is done by the automation, with possible plugin from language to help convert aggregated sample to the final markdown.

### Option 2 - SDK push to intermediate repo

Step 1, when SDK team generates the package, it generates a specific version of the examples that matching the JSON example, and upload it to an intermediate repo.

But flag it as "not released".

Step 2, when SDK team releases the package, flag it as "released".

Documentation reference is still required.

Step 3, automation copy the "released" ones to the central repo.

## Design on "Step 3" of "Option 1"

The plan is to follow the design for [SDK automation in swagger specs](https://github.com/Azure/azure-rest-api-specs/tree/main/documentation/sdkautomation).

We will have central functionality implemented once.
1. Find candidate release tags from SDK repo.
2. Prepare input, call script, parse output.
3. If success, commit the output to example repo.

In (2), language plugin as script will process the examples in the release tag, convert them to the final markdown, put them to target path.
These plugins can be implemented by different language, consume customized configuration, and run customized logic, as long as it can have the desired output.

A few thoughts on the plugin.
1. Verify that the package is indeed released (for some SDK, it could happen that a release tag is created, but the follow-up package publish failed).
2. Find the examples in the release tag.
3. Break-down the aggregated sample and re-construction to the SDK example (one to one map of this example to JSON example).
4. Verify that the SDK example can pass compilation.
5. Prepare the documentation reference.
6. Make the final markdown by composing of the documentation reference and the SDK example.
7. Output the markdown to its target location.

## TODO

### Issue 1

The case that the SDK example is outdated. For example, service might ping the REST API to 2020-02-01, but their service and SDK is on 2021-06-01.
Even if they change/fix swagger for api-version 2020-02-01, SDK might not ever re-release for 2020-02-01 again.

For now, we probably do not have good solution, except try to ask service to update their REST API doc configuration to move to later api-version.

One mitigation is to update the document reference part (periodically), to notify reader that a new SDK is available.
E.g.
```
Read the [SDK documentation](https://github.com/Azure/azure-sdk-for-java/blob/azure-resourcemanager_2.6.0/sdk/resourcemanager/README.md) on how to add the SDK to your project and authenticate.
The latest SDK is [2.7.0](https://github.com/Azure/azure-sdk-for-java/blob/azure-resourcemanager_2.7.0/sdk/resourcemanager/README.md).
```

### Issue 2

The case that service only update JSON example, but did not touch swagger specs.

Ideal solution would to have another automation to catch this and re-generate SDK examples.
For this, we will need to ping down all the parameter (of autorest) to create that SDK.
If possible we should also ping down the swagger JSON (in case the tag in README is modified).

Currently, this is available in some SDK but not all of them.
