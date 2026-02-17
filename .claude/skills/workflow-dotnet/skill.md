# Dapr Workflow .NET Application

## Overview

This skill describes how to create a Dapr Workflow application using .NET.

## Prerequisites

- .NET 10 SDK
- Dapr CLI
- Docker (required for running Dapr locally with Redis)
- NuGet package: `Dapr.Workflow` version `1.16.1`

## Project Setup

Create the project root folder, then create a new ASP.NET Core web application inside it:

```shell
mkdir <ProjectRoot>
cd <ProjectRoot>
dotnet new web -n <ProjectName>
dotnet add <ProjectName> package Dapr.Workflow --version 1.16.1
```

### Folder structure

```
<ProjectRoot>/
├── dapr.yaml
├── resources/
│   └── statestore.yaml
└── <ProjectName>/
    ├── <ProjectName>.csproj
    ├── Program.cs
    ├── Properties/
    │   └── launchSettings.json
    ├── <WorkflowName>.cs
    └── Activities/
        └── <ActivityName>.cs
```

### dapr.yaml

Create a `dapr.yaml` multi-app run file in the project root. This file configures the Dapr sidecar and points to the resources folder.

```yaml
version: 1
common:
  resourcesPath: ./resources
apps:
  - appID: <app-id>
    appDirPath: <ProjectName>
    appPort: 5255
    daprHTTPPort: 3555
    command: ["dotnet", "run"]
    appLogDestination: console
    daprdLogDestination: console
```

- `resourcesPath` points to the folder containing Dapr component definitions.
- `appDirPath` is the folder containing the .NET project.
- Use `dapr run -f dapr.yaml` to start the application with the Dapr sidecar.

### resources/statestore.yaml

Dapr Workflow requires a state store component. Create a `resources` folder in the project root with a `statestore.yaml` file:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: statestore
spec:
  type: state.redis
  version: v1
  initTimeout: 1m
  metadata:
  - name: redisHost
    value: localhost:6379
  - name: redisPassword
    value: ""
  - name: actorStateStore
    value: "true"
```

- The state store is required for workflow state persistence.
- `actorStateStore` must be set to `"true"` because Dapr Workflow uses the actor framework internally.
- This example uses Redis. Ensure Redis is running locally on port 6379 (e.g., `docker run -d -p 6379:6379 redis`).

### Properties/launchSettings.json

The application port must match the `appPort` specified in `dapr.yaml`. Create a `Properties/launchSettings.json` file in the project folder:

```json
{
  "$schema": "https://json.schemastore.org/launchsettings.json",
  "profiles": {
    "http": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "applicationUrl": "http://localhost:5255",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

- The `applicationUrl` port (5255) must match the `appPort` in `dapr.yaml` so the Dapr sidecar can communicate with the application.

### .csproj

The `.csproj` file should look like this:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Dapr.Workflow" Version="1.16.1" />
  </ItemGroup>

</Project>
```

## Program.cs

Use `AddDaprWorkflow` to register workflow and activity types. Use `DaprWorkflowClient` to schedule workflow instances and query their status via HTTP endpoints.

```csharp
using Dapr.Workflow;
using <ProjectNamespace>;
using <ProjectNamespace>.Activities;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddDaprWorkflow(options =>
{
    options.RegisterWorkflow<MyWorkflow>();
    options.RegisterActivity<MyActivity>();
});
var app = builder.Build();

app.MapPost("/start", async (DaprWorkflowClient workflowClient) =>
{
    var instanceId = await workflowClient.ScheduleNewWorkflowAsync(
        name: nameof(MyWorkflow),
        input: "Hello");

    return Results.Accepted(instanceId);
});

app.MapGet("/status", async (DaprWorkflowClient workflowClient, string instanceId) =>
{
    var state = await workflowClient.GetWorkflowStateAsync(instanceId);
    if (!state.Exists)
    {
        return Results.NotFound($"Workflow instance '{instanceId}' not found.");
    }

    return Results.Ok(state);
});

app.Run();
```

### Key points

- `AddDaprWorkflow` registers the Dapr Workflow services and the workflow/activity types with the DI container.
- `RegisterWorkflow<T>()` registers a workflow class.
- `RegisterActivity<T>()` registers an activity class. Register each activity separately.
- `DaprWorkflowClient` is injected via DI and used to schedule new workflow instances.
- `ScheduleNewWorkflowAsync` starts a new workflow instance and returns the instance ID.
- `GetWorkflowStateAsync` retrieves the current status of a workflow instance by its instance ID. Check `state.Exists` to verify the instance was found.
- Add `using` directives for the namespaces containing the workflow and activity classes.

## Workflow Class

A workflow class inherits from `Workflow<TInput, TOutput>` and overrides the `RunAsync` method. The workflow orchestrates one or more activities by calling `context.CallActivityAsync`.

```csharp
using Dapr.Workflow;

namespace <ProjectNamespace>;

internal sealed class MyWorkflow : Workflow<string, string>
{
    public override async Task<string> RunAsync(WorkflowContext context, string input)
    {
        var result = await context.CallActivityAsync<string>(
            nameof(MyActivity),
            input);

        return result;
    }
}
```

### Key points

- The first generic type parameter (`TInput`) is the workflow input type.
- The second generic type parameter (`TOutput`) is the workflow output type.
- Use `context.CallActivityAsync<TOutput>(activityName, input)` to call an activity.
- Use `nameof()` to reference activity names to avoid magic strings.
- Activities can be chained by passing the output of one activity as the input to the next.
- The workflow class should be `internal sealed`.

### Workflow determinism

Workflow code must be deterministic because the runtime may replay the `RunAsync` method multiple times. Avoid the following inside a workflow:

- `DateTime.Now` or `DateTime.UtcNow` — use `context.CurrentUtcDateTime` instead.
- `Guid.NewGuid()` — use `context.NewGuid()` instead.
- Random number generation.
- Direct I/O operations (HTTP calls, file access, database queries) — perform these in activities instead.
- `Thread.Sleep` or `Task.Delay` — use `context.CreateTimer()` instead.

### Using custom input/output types

Workflows and activities can use custom types for input and output instead of `string`. Define record or class types that are serializable:

```csharp
public record OrderRequest(string ProductId, int Quantity);
public record OrderResult(string OrderId, string Status);

internal sealed class OrderWorkflow : Workflow<OrderRequest, OrderResult>
{
    public override async Task<OrderResult> RunAsync(WorkflowContext context, OrderRequest input)
    {
        var result = await context.CallActivityAsync<OrderResult>(
            nameof(ProcessOrderActivity),
            input);

        return result;
    }
}
```

### Workflow patterns

#### Task chaining

Chain multiple activities by passing each result to the next activity:

```csharp
using Dapr.Workflow;

namespace <ProjectNamespace>;

internal sealed class ChainingWorkflow : Workflow<string, string>
{
    public override async Task<string> RunAsync(WorkflowContext context, string input)
    {
        var result1 = await context.CallActivityAsync<string>(
            nameof(Step1Activity),
            input);
        var result2 = await context.CallActivityAsync<string>(
            nameof(Step2Activity),
            result1);
        var result3 = await context.CallActivityAsync<string>(
            nameof(Step3Activity),
            result2);

        return result3;
    }
}
```

#### Fan-out/fan-in

Execute multiple activities in parallel and wait for all of them to complete:

```csharp
using Dapr.Workflow;

namespace <ProjectNamespace>;

internal sealed class FanOutFanInWorkflow : Workflow<string[], string[]>
{
    public override async Task<string[]> RunAsync(WorkflowContext context, string[] inputs)
    {
        var tasks = inputs.Select(input =>
            context.CallActivityAsync<string>(nameof(ProcessActivity), input));

        var results = await Task.WhenAll(tasks);

        return results;
    }
}
```

#### Sub-workflows

Call another workflow from within a workflow using `context.CallChildWorkflowAsync`:

```csharp
using Dapr.Workflow;

namespace <ProjectNamespace>;

internal sealed class ParentWorkflow : Workflow<string, string>
{
    public override async Task<string> RunAsync(WorkflowContext context, string input)
    {
        var result = await context.CallChildWorkflowAsync<string>(
            nameof(ChildWorkflow),
            input);

        return result;
    }
}
```

## Activity Class

An activity class inherits from `WorkflowActivity<TInput, TOutput>` and overrides the `RunAsync` method. Activities contain the actual business logic.

```csharp
using Dapr.Workflow;

namespace <ProjectNamespace>.Activities;

internal sealed class MyActivity : WorkflowActivity<string, string>
{
    public override Task<string> RunAsync(WorkflowActivityContext context, string input)
    {
        Console.WriteLine($"{nameof(MyActivity)}: Received input: {input}.");
        return Task.FromResult($"Processed: {input}");
    }
}
```

### Key points

- The first generic type parameter (`TInput`) is the activity input type.
- The second generic type parameter (`TOutput`) is the activity output type.
- The `RunAsync` method receives a `WorkflowActivityContext` and the input.
- Activities should be `internal sealed`.
- Place activity classes in an `Activities` folder/namespace for organization.
- If the activity method body is synchronous, return `Task.FromResult()` instead of marking the method `async`.
- Activities are where non-deterministic and I/O operations should be performed (HTTP calls, database queries, file access, etc.).
