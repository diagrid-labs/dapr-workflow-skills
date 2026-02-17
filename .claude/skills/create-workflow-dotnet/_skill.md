---
name: create-workflow-dotnet
description: This skill creates a Dapr workflow application in .NET. Use this skill when the user asks to "create a workflow in .NET", "write a .NET workflow application" or "build a workflow app in C#".
---

# Create Dapr Workflow .NET Application

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
    ├── Models/
    │   └── <ModelName>.cs
    ├── <WorkflowName>.cs
    └── Activities/
        └── <ActivityName>.cs
```

### dapr.yaml

Multi-app run file in the project root. Configures the Dapr sidecar and points to the resources folder. See `reference.md` for full example and key points.

### resources/statestore.yaml

Dapr Workflow requires a state store component (with `actorStateStore` set to `"true"`). See `reference.md` for full example and key points.

### Properties/launchSettings.json

Configures the application port, which must match `appPort` in `dapr.yaml`. See `reference.md` for full example.

### .csproj

Standard ASP.NET Core web project targeting `net10.0` with the `Dapr.Workflow` package. See `reference.md` for full example.

## Models

Record types for workflow and activity input/output, placed in a `Models` folder. Must be serializable since Dapr persists workflow state. See `reference.md` for full example and key points.

## Program.cs

Uses `AddDaprWorkflow` to register workflow and activity types. Uses `DaprWorkflowClient` to schedule workflow instances and query status via HTTP endpoints. See `reference.md` for full example and key points.

## Workflow Class

Inherits from `Workflow<TInput, TOutput>`, overrides `RunAsync`, and orchestrates activities via `context.CallActivityAsync`. Must be `internal sealed`. See `reference.md` for full example, key points, determinism rules, and workflow patterns (chaining, fan-out/fan-in, sub-workflows).

## Activity Class

Inherits from `WorkflowActivity<TInput, TOutput>`, overrides `RunAsync`, and contains the actual business logic. Must be `internal sealed`. Place in an `Activities` folder/namespace. See `reference.md` for full example and key points.
