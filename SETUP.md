# Install, Start and Remove an MCP Through Copilot

Use these prompts in a local GitHub Copilot chat in VS Code. Copilot performs the filesystem and terminal work; you review permissions and complete any UI steps it identifies. Start with [Robust MCP](https://github.com/spkane/freecad-addon-robust-mcp-server), the server selected after this experiment. [Neka](https://github.com/neka-nat/freecad-mcp) is the alternative tested successfully in chat. Blwfish remains in the results as a client-compatibility finding.

## VS Code Settings

Open a dedicated local folder in VS Code, sign in to Copilot and select **Agent**. Choose a model that supports tool calls. Enable the built-in file editing, terminal and web tools needed for installation; enable the selected FreeCAD MCP tools once the server is registered. Review the repository and generated commands before granting Workspace Trust or any server trust prompt. Your organization's policy may control which servers and permissions are available.

The chat input has controls for permissions and agent mode. Labels vary between VS Code versions: the UI used during this experiment shows **Default permissions**, **Allow all** and **Autopilot (Preview)** in the permissions menu; newer Agent Host sessions expose Autopilot in the mode picker.

| Choice | Use in this workflow |
| --- | --- |
| Default / Manual permissions | A suitable starting point. Copilot carries out the steps and asks you to approve actions according to your settings. |
| Allow all | Auto-approves tool calls for the session. It reduces approval prompts. |
| Autopilot (Preview) | Adds autonomous continuation and retries. It can answer blocking questions itself and automate tool approvals; use it only for a well-defined task in a trusted workspace. |

**Autopilot is optional, not an MCP requirement.** In versions that include it in the permissions menu, open that menu at the bottom of the chat and select **Autopilot (Preview)**, then review the warning. Use the mode picker if your version places it there. With normal permissions, approve the requested steps or ask Copilot to continue when it pauses. Autopilot consumes Copilot usage like other agent activity and can also approve destructive operations. Prefer session-scoped permissions, keep the task limited to the chosen server, and return to Default / Manual permissions when finished.

Terminal sandboxing is a separate control. It can restrict installation paths or network access; ask Copilot to explain a blocked operation and the narrow access it needs rather than broadly disabling protection. It also does not make Python executed inside FreeCAD a sandboxed CAD operation. OS elevation and sign-in prompts still need your direct action. Enter any credentials only in their trusted UI or terminal prompt.

## Install Through Copilot

Paste this into the chat. To choose neka instead, replace the repository URL on the first line with `https://github.com/neka-nat/freecad-mcp`.

```text
Install and configure https://github.com/spkane/freecad-addon-robust-mcp-server so that this local GitHub Copilot chat can control FreeCAD on my computer.

First inspect the installed FreeCAD version, its Python/Qt versions and the machine architecture. Read the selected project's current installation documentation. Briefly show the components and locations you will use, then perform the setup within this workspace. Use a suitable stable release with matching addon/server components and record the exact versions and source revisions.

Keep the external MCP dependencies in a dedicated virtual environment. Preserve the normal FreeCAD installation, bundled Python and existing user profile. Install the addon in an isolated FreeCAD profile, and create a launcher for that profile. Register only this chosen MCP in the workspace configuration, using explicit executable paths and the supported stdio transport. Preserve unrelated settings and MCP entries. Bind the FreeCAD bridge to loopback only and match its host/ports with the external server.

Have the launcher activate the correct FreeCAD workbench and start its bridge through the addon's supported command, or document the exact toolbar clicks and optional auto-start setting. Keep only the selected FreeCAD candidate active. Ask me before closing any FreeCAD document with unsaved changes.

Verify dependency installation, server startup, MCP tool-schema acceptance and an actual connection from this Copilot chat to the isolated FreeCAD profile. List the open documents, create a small test box in a new document, verify its dimensions and show its view. Distinguish a protocol-only check from actual tool availability in this chat. Record any errors; do not patch the upstream server to hide a compatibility failure.

Write a local installation record listing owned files, environments, profile, configuration entries and launch/stop/remove commands. Keep generated environments, profiles, caches, logs and private installation records out of version control with scoped ignore rules. Keep credentials and private paths out of any public report. Prepare removal as a dry run by default, preserving FreeCAD and all model files. Group any UI actions I must perform into one short checklist. If a UI approval is needed before you can finish the live check, stop there and resume the check after I confirm it.
```

For a new setup, a workspace-root `.mcp.json` can hold the server entry under `mcpServers`. VS Code also supports `.vscode/mcp.json` with a `servers` object. Ask Copilot to use one supported configuration surface and check for duplicate registrations. The process command must point to the installed server or its launcher. The bridge's XML-RPC port is not itself an MCP HTTP endpoint.

Installation often ends with a short UI handoff:

1. Run **Developer: Reload Window** if VS Code has not picked up the new configuration. This reloads VS Code, not FreeCAD.
2. Open **MCP: List Servers**, select the chosen server and start it if necessary. Review any trust prompt.
3. Enable its tools in the chat tool picker and ask Copilot to complete the connection check.

The prompt asks Copilot to discover version-specific paths, including FreeCAD's versioned addon directories. Its generated launcher and installation record are the instructions for your machine.

## Daily Startup

Start FreeCAD with the launch command or shortcut generated during installation. An isolated addon is loaded by that profile, so use its launcher instead of an unrelated FreeCAD shortcut. If the launcher already starts the bridge, check the running status and proceed to VS Code.

For manual bridge startup:

| Server | FreeCAD workbench | Start action | Optional automatic startup |
| --- | --- | --- | --- |
| Robust | Robust MCP Bridge | Start Bridge toolbar button (command `Start_MCP_Bridge`) | Edit > Preferences > Robust MCP Bridge > Auto-start bridge |
| neka | MCP Addon | Start RPC Server toolbar button (command `Start_RPC_Server`) | FreeCAD MCP menu > Auto-Start Server |

Then use **MCP: List Servers > your server > Start** in VS Code if its external process is stopped. FreeCAD must stay open with the bridge running while the chat works. For a quick check, send:

```text
Check the selected FreeCAD MCP connection, confirm the active document and profile, and list the open documents without changing or closing anything.
```

If the connection is refused, inspect FreeCAD's **View > Panels > Report view** and the server's **Show Output** action in **MCP: List Servers**. Both successful candidates use XML-RPC port 9875 by default. One active bridge at a time avoids connecting to the wrong instance; any custom port must match on both sides. After restarting FreeCAD, start its bridge again unless auto-start is enabled. Ordinary drawing and subsequent dimension changes use the same running instance.

The startup script used in this experiment performed the workbench activation and start command automatically. The installation prompt above asks Copilot to create that convenience for the reader's own profile too.

## Uninstall Through Copilot

Removing unused candidates is optional. Keeping their MCP entries disabled and using only the selected profile is sufficient to avoid mixing them during drawing. Removal reclaims disk space and reduces clutter. Save your models first, and use the installation record to distinguish owned files from shared runtimes and application files.

Use Default / Manual permissions for this review, then paste the prompt below. Name the exact project URL to remove, especially if more than one package exposes an executable called `freecad-mcp`.

```text
Prepare to uninstall the FreeCAD MCP server identified by this repository URL: <repository URL to remove>.

Read its local installation record and inspect the actual workspace configuration. Show a dry-run list of the exact server entry, addon/profile, virtual environment, checkout, launcher and candidate-owned cache files that would be removed. Identify any shared files and any saved models inside those locations. Preserve all model files, exports, benchmark results, the FreeCAD application, bundled Python, shared external Python installations and other MCP servers. Preserve an existing normal FreeCAD profile.

If ownership is unclear, report it instead of deleting. Do not terminate a FreeCAD instance with unsaved documents. Stop after presenting the removal plan and wait for my explicit confirmation. Keep a record of the approved paths so the removal can be checked afterwards.
```

After reviewing that list and saving any open work, send:

```text
Remove only the files and configuration entries in the approved plan, preserving the models, exports, shared runtimes and other servers identified there. Stop only the matching owned processes. Ask me to stop the corresponding MCP in VS Code if necessary, and resume after confirmation. Verify afterwards that the selected server entry and its owned files are gone, the remaining MCP setup is intact, and no matching listener or process is left running.
```

## References

- [VS Code MCP configuration and startup](https://code.visualstudio.com/docs/agent-customization/mcp-servers)
- [VS Code permissions and Autopilot](https://code.visualstudio.com/docs/agents/run/approvals)
- [VS Code security guidance](https://code.visualstudio.com/docs/agents/run/security)
- [Robust bridge usage and preferences](https://spkane.github.io/freecad-addon-robust-mcp-server/latest/guide/workbench/)
- [Neka installation](https://github.com/neka-nat/freecad-mcp/blob/main/docs/installation.md) and [automatic startup](https://github.com/neka-nat/freecad-mcp/blob/main/docs/configuration.md)
