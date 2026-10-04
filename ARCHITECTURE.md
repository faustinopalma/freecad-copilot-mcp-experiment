# Architecture and Operating Flows

The [README diagram](README.md#architecture) shows the Robust connection used in this experiment. Neka uses the same broad split between an external MCP server and an addon inside FreeCAD. The component names and tool sets differ between projects.

## Components

**VS Code and Copilot** own the conversation and tool dispatch. The model receives the design request and relevant tool results, then proposes tool calls. VS Code applies the session's approval rules and sends approved calls to the selected MCP server. The [permissions settings](SETUP.md#vs-code-settings) determine how much of this needs confirmation.

**The external MCP server** is a Python process launched from the workspace MCP configuration. In this setup it communicates with VS Code using MCP over standard input/output. It exposes tool schemas and translates calls into operations for FreeCAD. The experiment's thin launcher wrapper supplied the candidate-specific environment while forwarding this stream unchanged.

**The bridge addon** runs inside FreeCAD. Robust's external server connects to it through XML-RPC on loopback port 9875. The bridge queues CAD work for FreeCAD's main thread, where GUI and document operations can run in the correct execution context. The same port must be configured on both sides. This XML-RPC connection is separate from the MCP transport used by VS Code.

**FreeCAD** owns the document, feature history, constraints, geometric computation, recomputation and viewport. Its Python APIs expose those operations to the bridge. Documents, STEP/STL exports and saved images are written locally. Tool responses can carry geometry measurements, operation results, errors or images back to Copilot.

**The profile launcher** starts FreeCAD with the chosen addon, settings and paths. In this experiment, its startup macro selected the workbench and invoked the bridge's Start command. The equivalent manual steps are described in [daily startup](SETUP.md#daily-startup). Keep one candidate active at a time so calls reach the intended FreeCAD instance.

## Installation Flow

1. The user sends the [installation prompt](SETUP.md#install-through-copilot) in Agent mode.
2. Copilot uses VS Code's existing web, file and terminal tools to inspect FreeCAD, read the project's installation instructions and prepare a compatible environment.
3. It installs the external MCP dependencies, places the addon in a dedicated FreeCAD profile, creates the profile launcher and registers the workspace MCP entry.
4. The user completes any required trust or UI actions. Copilot verifies the advertised tool schemas and an actual call from the chat to FreeCAD.
5. An installation record lists the versions, paths and owned files for subsequent startup and removal.

This setup flow uses VS Code's built-in tools before the FreeCAD MCP is available. Once the connection is verified, drawing uses the selected MCP tools.

## Startup Flow

```mermaid
sequenceDiagram
    actor User
    participant Launcher as Profile launcher
    participant FreeCAD as FreeCAD and bridge addon
    participant Client as VS Code MCP client
    participant Server as External MCP server
    User->>Launcher: Open the selected FreeCAD profile
    Launcher->>FreeCAD: Start FreeCAD, load addon, invoke Start Bridge
    FreeCAD-->>Launcher: Bridge ready on loopback
    User->>Client: Start the configured MCP server if needed
    Client->>Server: Launch process and initialize MCP over stdio
    Server-->>Client: Server capabilities
    Client->>Server: List tools
    Server-->>Client: Tool definitions and parameter schemas
    Client->>Client: Validate and expose tool definitions
    User->>Client: Ask Copilot to verify the connection
    Client->>Server: Connection or document-list tool call
    Server->>FreeCAD: XML-RPC request
    FreeCAD-->>Server: Live version, profile or document information
    Server-->>Client: MCP tool result
    Client-->>User: Connection confirmed
```

Opening a normal FreeCAD shortcut may load a different profile. Use the generated launcher when the addon lives in an isolated profile. With manual startup, select the appropriate workbench and start its bridge before checking the connection in Copilot. The addon can also be configured to auto-start. FreeCAD remains open for the rest of the drawing session.

## Modeling and Revision Flow

```mermaid
sequenceDiagram
    actor User
    participant Agent as VS Code and Copilot agent
    participant Model as Language model
    participant Server as MCP server
    participant FreeCAD as Bridge and FreeCAD document
    User->>Agent: Describe the part or request a change
    loop Inspect, build and validate
        Agent->>Model: Request, document context and tool results
        Model-->>Agent: Choose tool and arguments
        Agent->>Agent: Apply session approval rules
        Agent->>Server: MCP tool call over stdio
        Server->>FreeCAD: XML-RPC operation
        Note over FreeCAD: Queue on the main thread, execute and recompute
        FreeCAD-->>Server: Geometry, measurements, image or error
        Server-->>Agent: MCP response
    end
    Agent->>Server: Save, export and verify
    Server->>FreeCAD: Write files and check the result
    FreeCAD-->>Server: Artifact paths and validation results
    Server-->>Agent: Completion evidence
    Agent-->>User: Model view, saved files and measured result
```

A dedicated tool might create a sketch, pocket a profile or export a solid. For operations that the server does not expose directly, Copilot can use its Python execution tool; the code still travels through MCP and the bridge into FreeCAD. The experiment recorded this separately because it places more of the implementation work on the model.

Errors and measurements feed the next iteration. A returned success flag alone does not establish that a part is correct: dimensions, solid validity, constraints, clearances and reopened exports provide the evidence. That feedback loop caught the open STL mesh in the Robust trial and the arc-constraint problem during the neka revision.

A revision continues in the same FreeCAD document. Copilot changes the named parameters, FreeCAD recomputes dependent features, and the validation is repeated. Initial exports can be preserved alongside the revised files, making the change inspectable.

## Autopilot and Approvals

Autopilot changes how the agent continues and handles approvals; it does not change either transport in these diagrams or replace the bridge startup step. Default / Manual permissions supports the same MCP workflow with user confirmations. Autopilot can continue, retry and answer blocking questions automatically, with broader tool permissions. Use the setting intentionally and keep access scoped to the task. Details and UI differences are in [VS Code settings](SETUP.md#vs-code-settings).

## Removal Flow

1. Send the [removal planning prompt](SETUP.md#uninstall-through-copilot). Copilot reads the installation record and identifies candidate-owned files, processes and settings.
2. Review the dry run and save any open models. Confirm the exact removal scope.
3. Stop that MCP in VS Code and close only its owned FreeCAD instance when safe. Remove the matching entry, addon/profile, environment and launcher according to the approved plan.
4. Check that the removed server has no remaining process or listener and that the retained server and model files remain usable.

Unused candidates may instead stay installed and disabled. Our continued drawing setup selects Robust only; the other environments can be removed later to reclaim space.
