---
name: create-workflow-dotnet
description: This skill creates a Dapr workflow application in .NET. Use this skill when the user asks to "create a workflow in .NET", "write a .NET workflow application" or "build a workflow app in C#".
---

# Create Dapr Workflow .NET Application

## Overview

This skill describes how to create a Dapr Workflow application using .NET.

## Prerequisites

This skill requires the C# LSP plugin that can be installed with  `claude plugin install csharp-lsp@claude-plugins-official`

To build and run the .NET Dapr Workflow applications the following is required: 
- [.NET 10 SDK](https://dotnet.microsoft.com/en-us/download)
- [Dapr CLI](https://docs.dapr.io/getting-started/install-dapr-cli/)
- Docker or Podman (required for running Dapr locally with Redis)
- NuGet package: `Dapr.Workflow` version `1.16.1`
- [Diagrid Dev Dashboard](https://www.diagrid.io/blog/improving-the-local-dapr-workflow-experience-diagrid-dashboard)

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
├── local.http
├── resources/
│   └── statestore.yaml
└── <ProjectName>/
    ├── <ProjectName>.csproj
    ├── Program.cs
    ├── Properties/
    │   └── launchSettings.json
    ├── Models/
    │   └── <ModelName>.cs
    ├── Workflows/
    │   └── <WorkflowName>.cs
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

Inherits from `Workflow<TInput, TOutput>`, overrides `RunAsync`, and orchestrates activities via `context.CallActivityAsync`. Must be `internal sealed`. Place in a `Workflows` folder/namespace. See `reference.md` for full example, key points, determinism rules, and workflow patterns (chaining, fan-out/fan-in, sub-workflows).

## Activity Class

Inherits from `WorkflowActivity<TInput, TOutput>`, overrides `RunAsync`, and contains the actual business logic. Must be `internal sealed`. Place in an `Activities` folder/namespace. See `reference.md` for full example and key points.

## local.http

HTTP request file for testing the workflow endpoints. Contains a `start` request (POST) to schedule a new workflow instance and a `status` request (GET) to query the workflow state. Uses the `<app-port>` from `launchSettings.json`. See `reference.md` for full example.

## Running Locally

Start the application with the Dapr sidecar from the project root:

```shell
dapr run -f .
```

This reads the `dapr.yaml` multi-app run file and launches the app with its Dapr sidecar.

To inspect workflow executions, run the Diagrid Dev Dashboard:

```shell
docker run -p 8080:8080 ghcr.io/diagridio/diagrid-dashboard:latest
```

Then open `http://localhost:8080` in a browser to view workflow instances, their status, and execution history.

## Running with Diagrid Catalyst

[Diagrid Catalyst](https://catalyst.diagrid.io/) provides managed Dapr APIs with better workflow execution insights and easier workflow management through the Catalyst web UI.

1. Sign up at [catalyst.diagrid.io](https://catalyst.diagrid.io/) and install the Diagrid CLI.
2. Run the application from the project root:

```shell
diagrid dev run -p <ProjectName> -f dapr.yaml
```

This uses the same `dapr.yaml` multi-app run file but connects to Catalyst instead of a local Dapr sidecar, giving access to the Catalyst dashboard for monitoring and managing workflow executions.

