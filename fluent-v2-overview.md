# Fluent Java SDK Overview

## Fluent interface

The Fluent interface can be deemed as an internal DSL which runs in Java.

Under the hood, it calls generated code for most REST APIs.

The functionality of creating storage account can be achieved by (compare with calling [generated code](#gen_storage_account) directly):

```java
StorageAccount storageAccount =
    client.storageAccounts().define(accountName)
        .withRegion(region)
        .withNewResourceGroup(resourceGroup)
        .withSku(StorageAccountSkuType.STANDARD_LRS)
        .withGeneralPurposeAccountKindV2()
        .withOnlyHttpsTraffic()
        .withTag("product", "javasdk")
        .withTag("cause", "automation")
        .create();
```

There are multiple reasons for adopting Fluent interface as recommended customer facing interface.

### 1. Guidance for customer

#### 1a. Configure management resource with help from IDE

![alt text](https://docs.microsoft.com/en-us/azure/developer/java/sdk/media/intellij-fluent-method-chain.gif "Fluent method chain")

Here is the documentation from [track1 Fluent SDK](https://docs.microsoft.com/en-us/azure/developer/java/sdk/java-sdk-azure-concepts).

#### 1b. Guidance for resource configuration

Fluent interface confines customer to relevant parameters for current resource configuration. Required parameters first, optional parameters last, non-relevant parameters excluded.

For example, a *simple* virtual machine resource has 20+ properties, most of these having nested properties.

So the overall properties could be over a hundred.

Some of the properties is only required for Windows VM, and some only for Linux VM.
These are not specified in OpenAPI.

```
class VirtualMachineInner extends Resource {
    private Plan plan;
    private List<VirtualMachineExtensionInner> resources;
    private VirtualMachineIdentity identity;
    private List<String> zones;
    private HardwareProfile hardwareProfile;
    private StorageProfile storageProfile;
    private AdditionalCapabilities additionalCapabilities;
    private OSProfile osProfile;
    private NetworkProfile networkProfile;
    private DiagnosticsProfile diagnosticsProfile;
    private SubResource availabilitySet;
    private SubResource virtualMachineScaleSet;
    private SubResource proximityPlacementGroup;
    private VirtualMachinePriorityTypes priority;
    private VirtualMachineEvictionPolicyTypes evictionPolicy;
    private BillingProfile billingProfile;
    private SubResource host;
    private String provisioningState;
    private VirtualMachineInstanceViewInner instanceView;
    private String licenseType;
    private String vmId;
}

class OSProfile {
    private String computerName;
    private String adminUsername;
    private String adminPassword;
    private String customData;
    private WindowsConfiguration windowsConfiguration;
    private LinuxConfiguration linuxConfiguration;
    private List<VaultSecretGroup> secrets;
    private Boolean allowExtensionOperations;
    private Boolean requireGuestProvisionSignal;
}
```

Fluent interface helps guide customer through the complexity of management resource configuration.

Here is the sample code to create a Windows virtual machine by Fluent interface.

```java
VirtualMachine windowsVM = 
    client.virtualMachines().define(windowsVMName)
        .withRegion(region)
        .withNewResourceGroup(rgName)
        .withNewPrimaryNetwork("10.0.0.0/28")
        .withPrimaryPrivateIPAddressDynamic()
        .withoutPrimaryPublicIPAddress()
        .withPopularWindowsImage(KnownWindowsVirtualMachineImage.WINDOWS_SERVER_2019_DATACENTER)
        .withAdminUsername(userName)
        .withAdminPassword(password)
        .withNewDataDisk(10)
        .withNewDataDisk(dataDiskCreatable)
        .withExistingDataDisk(dataDisk)
        .withSize(VirtualMachineSizeTypes.STANDARD_D3_V2)
        .create();
```

In above code, if customer type `withPopularWindowsImage`, IDE will prompt `withAdminUsername` as next method.
If customer choose `withPopularLinuxImage`, IDE will prompt `withRootUsername` as next method.

This helps customer focus on the properties that is valid for current configuration.

There would be solutions other than Fluent interface, e.g. define `WindowsVirtualMachineCreateParameter` and `LinuxVirtualMachineCreateParameter`, extends `SharedVirtualMachineCreateParameter`.
But still it is hard to distinguish required parameters and optional parameters in this approach.

#### 1c. Convenience for creating dependent resources

As in above sample, `withNewResourceGroup` specifies that a new resource group need to be created before the virtual machine; `withNewPrimaryNetwork` specifies that a new virtual network and NIC be created; `withPrimaryPrivateIPAddressDynamic` specifies that a new dynamic private IP address be created.

All of the 3 (or more) dependent resources need to be created beforehand, separately, if not for Fluent interface.

These convenience methods helps customer focus on the central task of configuring virtual machine, instead of being distracted by all these dependencies.

#### 1d. Grouping of nested properties

An example for creating a load balancer.

```java
LoadBalancer loadBalancer1 = 
    client.loadBalancers().define(loadBalancerName1)
        .withRegion(region)
        .withExistingResourceGroup(rgName)
        // Add two rules that uses above backend and probe
        .defineLoadBalancingRule(httpLoadBalancingRule)
            .withProtocol(TransportProtocol.TCP)
            .fromFrontend(frontendName)
            .fromFrontendPort(80)
            .toBackend(backendPoolName1)
            .withProbe(httpProbe)
            .attach()
        .defineLoadBalancingRule(httpsLoadBalancingRule)
            .withProtocol(TransportProtocol.TCP)
            .fromFrontend(frontendName)
            .fromFrontendPort(443)
            .toBackend(backendPoolName2)
            .withProbe(httpsProbe)
            .attach()
        // Add nat pools to enable direct VM connectivity for
        // SSH to port 22 and TELNET to port 23
        .defineInboundNatPool(natPool50XXto22)
            .withProtocol(TransportProtocol.TCP)
            .fromFrontend(frontendName)
            .fromFrontendPortRange(5000, 5099)
            .toBackendPort(22)
            .attach()
        .defineInboundNatPool(natPool60XXto23)
            .withProtocol(TransportProtocol.TCP)
            .fromFrontend(frontendName)
            .fromFrontendPortRange(6000, 6099)
            .toBackendPort(23)
            .attach()
        // Explicitly define the frontend
        .definePublicFrontend(frontendName)
            .withExistingPublicIPAddress(publicIPAddress)
            .attach()
        // Add two probes one per rule
        .defineHttpProbe(httpProbe)
            .withRequestPath("/")
            .withPort(80)
            .attach()
        .defineHttpProbe(httpsProbe)
            .withRequestPath("/")
            .withPort(443)
            .attach()
        .create();
```

After the first `define` for load balancer, each block of `define...` to `attach` specifies a nested property (load balancing rule, inbound NAT pool, public frontend, probe, respectively).

And proper choice of method name (`fromFrontend` instead of `new FrontendIPConfigurationInner().setName(...)`) makes certain properties easier to understand.

#### 1e. Grouping of nested resources

It is common for management resources to have multi-level nested resources.

One extreme example from Cosmos, resource nested 4 levels:

`/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DocumentDB/databaseAccounts/{accountName}/sqlDatabases/{databaseName}/containers/{containerName}/storedProcedures/{storedProcedureName}`

Similar to the effect of large number of properties and nested properties, nested resources is hard to comprehend.

Fluent interface groups related nested resources to a common root resource.

An example for creating a DNS zone and record set.

```java
DnsZone rootDnsZone = 
    client.dnsZones().define(domainName)
        .withExistingResourceGroup(resourceGroup)
        .defineCNameRecordSet("www", hostname)
            .withAlias(alias)
            .withETagCheck()
            .attach()
        .create();
```

This is much easier to write than calling generated code on 2 different service clients:
```java
implClient.zones().createOrUpdate(...);
implClient.recordSets().createOrUpdate(..., RecordType.CNAME, ...);
```

### 2. Integration with non-management client

Fluent SDK incorporates a few uncommon non-management client, for easy of use.

For example, [graphrbac](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/graphrbac/data-plane) for authorization, [kudu](https://github.com/projectkudu/kudu) for web app deployment and log streaming.

AFAIK, there is no replacement from other Java clients.
This is part of the reason that [azure-maven-plugin-lib](https://github.com/microsoft/azure-maven-plugins/blob/develop/azure-maven-plugin-lib/pom.xml#L125-L129) uses the Fluent SDK, for management of web app as well as for kudu client.

For example, deploy zip to web app.

```
webapp.zipDeploy(zipFile);
```

Customer can deploy zip to web app after creating web app resource, without aware that this REST actually sends to `userwebapp.azurewebsite.net`, instead of management endpoint.

### 3. Mitigation of frequent breaking changes on management API

Management API updates frequently.

For example, [network](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/network/resource-manager/Microsoft.Network/stable) service updates every few months, all with some breaking changes.

Stable versions since 2019:

```
2019-02-01
2019-04-01
2019-06-01
2019-07-01
2019-08-01
2019-09-01
2019-11-01
2019-12-01
2020-03-01
2020-04-01
```

A separate layer of Fluent interface mitigates the impact of the frequent breaking changes from API.

If service owner made non-critical change to a resource or resource property, Fluent interface can opt to insulate the change from breaking customer.

### 4. Simpler interface with domain knowledge

Linux function app can be created by:

```java
FunctionApp app = 
    client.functionApps().define(appName)
        .withRegion(region)
        .withNewResourceGroup(rgName)
        .withNewLinuxAppServicePlan(planName, PricingTier.STANDARD_S1)
        .withBuiltInImage(FunctionRuntimeStack.JAVA_8)
        .withNewStorageAccount(storage1Name, StorageAccountSkuType.STANDARD_RAGRS)
        .withHttpsOnly(true)
        .withAppSetting("WEBSITE_RUN_FROM_PACKAGE", FUNCTION_APP_PACKAGE_URL)
        .create();
```

By a single line of `withBuiltInImage(FunctionRuntimeStack.JAVA_8)`, Fluent SDK actually does:

```java
appSettings
    .properties()
    .putAll(Map.of("FUNCTIONS_WORKER_RUNTIME", "java",
        "FUNCTIONS_EXTENSION_VERSION", "~3"));

siteConfig
    .withLinuxFxVersion(isConsumptionPlan
        ? "java|8" 
        : "DOCKER|mcr.microsoft.com/azure-functions/java:3.0-java8-appservice");
```

This is an example of domain knowledge integrated with Fluent interface.

Without it, to figure out the setting, customer probably need to dig on docs and query on StackOverflow or GitHub.

### 5. Consistency of naming

There is hundreds of management services, maintained by different service teams.

As the result, there is huge discrepancy in naming from OpenAPI.

For example, management resource type could be called `Resource`, `TrackedResource`, `ARMResource`, etc. Fluent SDK would normalize them to `Resource`.

Another example, REST API would be named `list`, `listAll`, `listResource`, etc. Fluent SDK would normalize them to `listByResourceGroup`.

Further example on Exception, OpenAPI would specify `CloudError`, `DefaultErrorResponse`, `ErrorResponse`, etc. Fluent SDk would normalize them to `ManagementError`, or its customized subclass.

In future, etag handling and private link would expected to be normalized as well.

---

I do not intend to claim that item 2 to 5 is only solvable by Fluent interface, but rather they are currently integrated with Fluent interface.

---

## Drawback of Fluent interface

There is always tradeoff.
Some apparent shortcomings:

- Lots of interfaces clusters javadoc and make it harder to find reference to fundamental classes
- Certain resource with dynamic list of items is harder to configure
- Some detail (e.g. lower level: the exact REST response with status code and HTTP headers; higher level: result of creation of dependency resources) is hidden from customer, due to higher level of abstraction
- Higher manual efforts to maintain Fluent interface
- As internal DSL, it does not conform to common Java style

Possible enhancement for track2:
- `beginCreate` with first response (Success or Accept), then optionally continue the polling, or wait till later. Helpful in case that operation takes hours.
- Alternative for operation without resource object. Helpful in case that lots of operation on different resources is needed, and additional GET is to be avoided.

---

## Fundamental APIs

There are APIs that relates more to `azure-core` and `azure-identity` (instead of Fluent interface), which we are expecting to conform to Java client guideline.

### HttpClient, HttpPipeline and TokenCredential

Fluent SDK is fully integrated with `azure-core` and `azure-identity` on these aspects.

### Exceptions

As discussed with Srikanta, we will use `ManagementException` which ships `ManagementError`.

Question:
1. Is it a good choice to have parallel `ManagementResourceNotFoundException`, `ManagementAuthenticationException`, `ManagementResourceModifiedException`, `ManagementResourceExistsException`, `ManagementTooManyRedirectsException`, etc.; or it might be easier for customer to catch `ManagementException` then check status code directly?

### Build Fluent client

```
AzureProfile profile = new AzureProfile(AzureEnvironment.AZURE, true);
TokenCredential credential = new DefaultAzureCredentialBuilder()
    .authorityHost(profile.environment().getActiveDirectoryEndpoint())
    .build();

Azure client = Azure
    .configure()
    .withLogOptions(new HttpLogOptions().setLogLevel(HttpLogDetailLevel.BASIC))
    .authenticate(credential, profile)
    .withDefaultSubscription();
```

If customer wishes Fluent SDK to do most of the configure.

Or

```
Azure client = Azure
    .authenticate(httpPipeline, profile)
    .withDefaultSubscription();
```

If customer already have a configured `HttpPipeline`.

Because of non-global Azure AD, and subscriptionId required by most management services, Fluent SDK takes an additional `AzureProfile` object, providing necessary information about management cloud configure, either from environment variables, or from customer configure.

Question:
1. Is it better to use builder style for `AzureProfile`?
2. Do we keep the Fluent interface here, or try to conform to builder style for service client?

---

## Generated code

The generated code basically follows same style as data-plane, except naming convention.

And the code can be called by customer. For example, <a name="gen_storage_account">to create a storage account</a>.

```java
StorageManagementClientImpl implClient = 
    new StorageManagementClientBuilder()
        .pipeline(httpPipeline)
        .host(resourceManagerEndpoint)
        .subscriptionId(subscriptionId)
        .buildClient();

StorageAccountInner storageAccount = 
    implClient.storageAccounts().create(resourceGroupName, accountName, 
        new StorageAccountCreateParameters()
            .withLocation(location)
            .withKind(Kind.STORAGE_V2)
            .withSku(new Sku().withName(SkuName.STANDARD_LRS))
            .withEnableHttpsTrafficOnly(true)
            .withTags(Map.of("product", "javasdk",
                "cause", "automation"))));
```

However Fluent SDK does not recommend customer to use this directly.
The detail of the reasons is already elaborated in [Fluent Interface](#fluent-interface) (primary for easy of use, and secondrary to avoid frequent breaking change from service upgrade).

But if Fluent interface cannot meet certain requirement (when some interface is not implemented, or customer really want to access all the details of HTTP request/response), customer can still fallback to this.

---

## Management service APIs

### KeyVault

Fluent KeyVault SDK wraps data-plane `azure-security-keyvault-keys`, `azure-security-keyvault-secrets`, mainly for ease of use.

Is it preferable to stop doing this? Fluent KeyVault SDK would only handle vault resource, and let customer use keyvault data-plane client to manage key and secret.
