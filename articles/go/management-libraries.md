---
title: Working with the Azure SDK for Go management libraries 
description: In this article, you learn the basic tasks of working with the Azure SDK for Go management libraries.
ms.date: 12/13/2021
ms.topic: conceptual
ms.custom: devx-track-go
---

# Working with the Azure SDK for Go management libraries

As explained in the article [What is the Azure SDK for Go?](overview.md), the Azure SDK for Go contains a set of management and client libraries.
The management libraries share many features such as Azure Identity support, HTTP pipeline, and error-handling.
You can find the full list of the management libraries on the [Azure SDK for Go module page](https://azure.github.io/azure-sdk/releases/latest/mgmt/go.html).

In this article, you'll learn the basic steps of how to use the management libraries to interact with Azure resources.

## Installing Go packages

In most projects, you'll install the Go packages for versioning and dependency management.

To install a Go package, use the `go get` command.

For example, to install the `armcompute` package, you run the following at the command line:

```azurecli
go get github.com/Azure/azure-sdk-for-go/sdk/resourcemanager/compute/armcompute
```

In most Go apps, you'll install the following packages for authentication and core functionality:

- github.com/Azure/azure-sdk-for-go/sdk/resoucemanager/compute/armcompute
- github.com/Azure/azure-sdk-for-go/sdk/azcore
- github.com/Azure/azure-sdk-for-go/sdk/azidentity
- github.com/Azure/azure-sdk-for-go/sdk/azcore/to

## Importing packages into your Go code

Once downloaded, the package are imported into your app via the `import` statement:

```go
import (
    "github.com/Azure/azure-sdk-for-go/sdk/armcore"
    "github.com/Azure/azure-sdk-for-go/sdk/resoucemanager/compute/armcompute"
    "github.com/Azure/azure-sdk-for-go/sdk/azidentity"
    "github.com/Azure/azure-sdk-for-go/sdk/to"
)
```

## Authenticating to Azure

To run code against an Azure subscription, you need to authenticate to Azure. The [azidentity](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/sdk/azidentity) package supports multiple options to authenticate to Azure. These options include client/secret, certificate, and managed identity.

The default authentication option is **DefaultAzureCredential**, which uses the environment variables set earlier in this article. In your Go code, you'll create an `azidentity` object as follows:

```go
cred, err := azidentity.NewDefaultAzureCredential(nil)
```

## Creating a Resource Management client

<!--  -->
Once you have a credential from Azure Identity, create a client to connect to the target Azure service.

For example, let's say you want to connect to the [Azure Compute](https://azure.microsoft.com/product-categories/compute/) service. The [Compute package](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/sdk/resourcemanager/compute/armcompute) consist of one or more clients. A client groups a set of related APIs, providing access to its functionality within the specified subscription. You create one or more clients to access the APIs you require using an `armcore.Connection` object.

In the following code snippet, the [armcompute.NewVirtualMachinesClient type](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/sdk/resourcemanager/compute/armcompute#VirtualMachinesClient) is used to create a client to manage virtual machines:

```go
client := armcompute.NewVirtualMachinesClient("<subscription ID>",cred, nil)
```

The same pattern is used to connect with other Azure services. For example, install the [armnetwork](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/sdk/resourcemanager/network/armnetwork) package and create a [VirtualNetwork](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/sdk/resourcemanager/network/armnetwork#VirtualNetworksClient) client to manage virtual network (VNET) resources.

```go
client := armnetwork.NewVirtualNetworksClient("<subscription ID>",cred, nil)
```

## Using the Azure SDK for Go reference documentation

Once instantiated, clients are used to make API calls against your Azure resources. For resource management scenarios, most of the use-cases are CRUD (Create/Read/Update/Delete) operations.

To look up the operations for a specific type, do the following steps:

1. Browse to the [Azure SDK for Go reference documentation](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/sdk).
1. Search the page for the type. (Pressing **&lt;F>** automatically expands all nodes on the page for searching.)
1. Select the package.
1. Search the package's page for the type.
1. Read the type's description and information about its usage in your Go code.

You can also manually build the URL by appending the name of the package to `github.com/Azure/azure-sdk-for-go/sdk/`.

For example, if you're looking for the `compute/armcompute` reference documentation, the URL is `github.com/Azure/azure-sdk-for-go/sdk/compute/armcompute`.

The following example shows how to find the reference documentation for Azure resource group operations:

1. Browse to the main [Azure SDK for Go reference](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go) on pkg.go.dev.
1. Expand the `arm` directory
1. Select `armresources`
1. Click **&lt;F>** and enter `resourcegroup`. As you type the search term, you see a close match with the [resource/armresources](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/sdk/resources/armresources) package.
1. Select the appropriate package for your application.
1. Read through "Getting Started" sections or search for a specific operation. For example, searching for the term "create" (if you want to create a resource group) leads you to the [CreateOrUpdate function](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/sdk/resources/armresources#ResourceGroupsClient.CreateOrUpdate).
1. At this point, you can read how to make the API call to create an Azure resource group.

## Long-running operations

As some operations can take a long time to complete, the management libraries contain functions to support long-running operations (LRO) via asynchronous calls. These function names start with `Begin`. Examples of this pattern are `BeginCreate` and `BeginDelete`.

As these functions are asynchronous, your code doesn't block until the function finishes its task. Instead, the function returns a *poller* object immediately. Your code then calls a synchronous poller function that returns when the original asynchronous function has completed.

The following code snippet shows an example of this pattern.

```go
ctx := context.Background()
// Call an asynchronous function to create a client. The return value is a poller object.
poller, err := client.BeginCreate(ctx, "resource_identifier", "additonal_parameter")
if err != nil {
	// handle error...
}

// Call the poller object's PollUntilDone function that will block until the poller object
// has been updated to indicate the task has completed.
resp, err = poller.PollUntilDone(ctx, 5*time.Second)
if err != nil {
	// handle error...
}

// Print the fact that the LRO completed.
fmt.Printf("LRO done")

// Work with the response ("resp") object.
```

**Key points:**

- The `PollUntilDone` function requires a polling interval that specifies how often it should try to get the status.
- The interval is typically short. See the documentation for the specific Azure resource for recommended intervals.
- The [LRO section of the Go Azure SDK Design Guidelines page](https://azure.github.io/azure-sdk/golang_introduction.html#methods-invoking-long-running-operations) has a move advanced example and general guidelines for LRO.

## Next steps

> [!div class="nextstepaction"]
> [Manage Azure resource groups](manage-resource-groups.md)
