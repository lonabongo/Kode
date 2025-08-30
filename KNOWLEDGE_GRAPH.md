# Claude Code Knowledge Graph

This document provides a structured analysis of the entire codebase according to the specified ontology. It maps out the key strategic, tactical, and implementation components and their relationships, offering a comprehensive view of the project's architecture, features, and design principles.

---

## I. Core Node Types

### A. Strategic Nodes

#### 1. Vision

*   **ID:** `Vision:Agentic-Coding-Assistant`
    *   **Description:** The core vision of the project is to create an AI-powered coding assistant that can understand and operate within a software development project, using tools to perform tasks like a human developer. It is designed to be deeply integrated into the user's local development environment, acting as a true "agent" rather than a simple chatbot.
    *   **Relationships:**
        *   `Guides -> Policy:Local-First-Architecture`
        *   `Guides -> Policy:AI-Driven-Tool-Usage`
        *   `Guides -> Feature:Core-REPL`

#### 2. Policy

*   **ID:** `Policy:Local-First-Architecture`
    *   **Description:** The application is fundamentally designed to run on the user's local machine, with direct access to the filesystem, shell, and local development environment. This contrasts with cloud-based assistants and is central to the product's value proposition.
    *   **Relationships:**
        *   `GuidedBy -> Vision:Agentic-Coding-Assistant`
        *   `ImplementedBy -> Feature:Local-File-Access`
        *   `ImplementedBy -> Feature:Shell-Command-Execution`
        *   `Enables -> Feature:Project-Tool-Whitelist`

*   **ID:** `Policy:AI-Driven-Tool-Usage`
    *   **Description:** The AI agent is the primary actor. Instead of the application having hardcoded logic for every action, complex tasks are delegated to the AI, which then uses a defined set of "tools" (like `BashTool`, `FileEditTool`) to interact with the system. This is a core architectural principle.
    *   **Relationships:**
        *   `GuidedBy -> Vision:Agentic-Coding-Assistant`
        *   `ImplementedBy -> Abstraction:Tool-System`
        *   `RealizedIn -> Command:init`, `Command:pr-comments`, `Command:review`

*   **ID:** `Policy:AI-Driven-Configuration`
    *   **Description:** The principle of using the AI agent itself to generate its own configuration (`claude.json`) by inspecting the repository. This reduces the manual setup burden on the user and leverages the AI's analytical capabilities.
    *   **Relationships:**
        *   `DefinedBy -> Data:Init-Command-Prompt`
        *   `ImplementedBy -> Command:init`

*   **ID:** `Policy:AI-Driven-GitHub-Interaction`
    *   **Description:** The practice of using the AI as a script interpreter to perform complex, multi-step interactions with external services like GitHub via their CLI tools. This avoids writing and maintaining brittle, high-maintenance integration code.
    *   **Relationships:**
        *   `DefinedBy -> Data:PR-Comments-Prompt`, `Data:Review-Command-Prompt`
        *   `ImplementedBy -> Command:pr-comments`, `Command:review`

*   **ID:** `Policy:Structured-Summarization`
    *   **Description:** The policy that conversation summaries must be structured into a specific 8-part format to ensure all critical dimensions of a software development conversation are captured consistently when compacting context.
    *   **Relationships:**
        *   `DefinedBy -> Data:COMPRESSION_PROMPT`
        *   `ImplementedBy -> Command:compact`

*   **ID:** `Policy:Context-Injection-Via-Tags`
    *   **Description:** The system uses XML-like tags (`<context name="...">...</context>`) to dynamically inject contextual information (like file contents, git status, etc.) into the main system prompt before sending it to the LLM.
    *   **Relationships:**
        *   `RevealedBy -> Command:ctx-viz`
        *   `ParsedBy -> Function:getContextSections`

*   **ID:** `Policy:Agent-File-Storage`
    *   **Description:** Agent configurations are stored as markdown files with YAML frontmatter. They can be stored at the user-level (`~/.claude/agents/`) for personal use or at the project-level (`./.claude/agents/`) for team sharing.
    *   **Relationships:**
        *   `ImplementedBy -> Function:saveAgent`, `Function:getAgentDirectory`

#### 3. Constraint

*   **ID:** `Constraint:Agent-Naming-Convention`
    *   **Description:** Agent types (identifiers) must start with a letter, contain only letters, numbers, and hyphens, be between 3 and 50 characters, and not be a reserved name (e.g., 'help', 'task').
    *   **Relationships:**
        *   `EnforcedBy -> Function:validateAgentConfig`

*   **ID:** `Constraint:macOS-Only-And-Specific-Terminals`
    *   **Description:** Certain features, like native dictation (`/listen`) or terminal setup (`/terminal-setup`), are only available on the macOS operating system and may have further restrictions based on the terminal application being used.
    *   **Relationships:**
        *   `AppliesTo -> Command:listen`
        *   `AppliesTo -> Command:terminal-setup`

*   **ID:** `Constraint:Tool-Schema-Zod`
    *   **Description:** The input schema for all tools must be defined using the `zod` library. This allows for automatic runtime validation and JSON schema generation for the AI.
    *   **Relationships:**
        *   `EnforcedBy -> Abstraction:Tool-System`
        *   `AppliesTo -> Abstraction:Tool`

---

### B. Tactical Nodes

#### 1. Feature

*   **ID:** `Feature:Core-REPL`
    *   **Description:** The main read-eval-print loop interface where users interact with the AI.
    *   **Relationships:**
        *   `RealizedBy -> Screen:REPL`

*   **ID:** `Feature:Agent-Management`
    *   **Description:** Allows users to create, manage, and use specialized, reusable AI agents with custom prompts, tools, and models.
    *   **Relationships:**
        *   `RealizedBy -> Command:agents`

*   **ID:** `Feature:Shell-Command-Execution`
    *   **Description:** Provides the AI with the ability to execute arbitrary shell commands.
    *   **Relationships:**
        *   `RealizedBy -> Tool:BashTool`

*   **ID:** `Feature:Local-File-Access`
    *   **Description:** Provides the AI with tools to read, write, and edit files on the local filesystem.
    *   **Relationships:**
        *   `RealizedBy -> Tool:ReadFileTool`, `Tool:WriteFileTool`, `Tool:EditFileTool`

*   **ID:** `Feature:Context-Compaction`
    *   **Description:** Allows the user to reduce the token count of a long conversation by replacing the detailed history with an AI-generated, structured summary.
    *   **Relationships:**
        *   `RealizedBy -> Command:compact`

*   **ID:** `Feature:User-Authentication`
    *   **Description:** Handles user sign-in with an external provider (Anthropic) using a console-based OAuth flow.
    *   **Relationships:**
        *   `RealizedBy -> Command:login`

*   **ID:** `Feature:Model-Configuration-UI`
    *   **Description:** Provides a UI for users to select their preferred AI model, provider, and configure related settings.
    *   **Relationships:**
        *   `RealizedBy -> Command:model`

*   **ID:** `Feature:Custom-Command-Hot-Reload`
    *   **Description:** A developer experience feature that allows for the hot-reloading of custom commands without a restart.
    *   **Relationships:**
        *   `RealizedBy -> Command:refresh-commands`

*   **ID:** `Feature:AI-Code-Review`
    *   **Description:** Leverages the AI to automatically review a pull request.
    *   **Relationships:**
        *   `RealizedBy -> Command:review`

*   **ID:** `Feature:Project-Initialization`
    *   **Description:** Automatically creates a `claude.json` file by having the AI analyze the repository.
    *   **Relationships:**
        *   `RealizedBy -> Command:init`

*   **ID:** `Feature:Append-To-Context-Files`
    *   **Description:** A mechanism for users to add free-form notes to important context files (`AGENTS.md`, `CLAUDE.md`) via a hash command (`#`).
    *   **Relationships:**
        *   `RealizedBy -> Function:handleHashCommand`

#### 2. Process

*   **ID:** `Process:Release-Management`
    *   **Description:** The process for creating a new release, defined in `.github/workflows/release.yml`. It involves manual triggering, version bumping, building, and publishing to an S3 bucket.
    *   **Relationships:**
        *   `Orchestrates -> Action:Build`, `Action:Test`
        *   `ManagedBy -> Tool:GitHub-Actions`

*   **ID:** `Process:User-Onboarding-Flow`
    *   **Description:** A sequence of steps presented to a new user, including login, model configuration, and setting up the Shift+Enter key binding.
    *   **Relationships:**
        *   `RealizedBy -> Command:onboarding`
        *   `Includes -> Feature:User-Authentication`, `Feature:Model-Configuration`, `Feature:Shift-Enter-Key-Binding`

#### 3. Stakeholder

*   **ID:** `Stakeholder:End-User-Developer`
    *   **Description:** The primary user of the application; a software developer who uses the tool to assist in their coding tasks.
    *   **Relationships:**
        *   `InteractsWith -> Feature:Core-REPL`

*   **ID:** `Stakeholder:Core-Contributor`
    *   **Description:** A developer working on the application itself.
    *   **Relationships:**
        *   `Follows -> Document:CONTRIBUTING.md`

---

### C. Implementation Nodes

#### 1. Command

*   This application uses a command system with three types: `local` (runs a TS function), `local-jsx` (renders a React component), and `prompt` (sends a formatted prompt to the AI).
*   **ID:** `Command:agents` - Manages AI agent configurations.
*   **ID:** `Command:bug` - Submits user feedback.
*   **ID:** `Command:clear` - Clears the conversation history.
*   **ID:** `Command:compact` - Summarizes the conversation to save tokens.
*   **ID:** `Command:config` - Opens the configuration panel.
*   **ID:** `Command:cost` - Shows session cost and duration.
*   **ID:** `Command:ctx-viz` - Visualizes context token usage.
*   **ID:** `Command:doctor` - Checks installation health.
*   **ID:** `Command:help` - Shows available commands.
*   **ID:** `Command:init` - (`prompt` type) Initializes a `claude.json` file.
*   **ID:** `Command:listen` - Activates macOS speech-to-text.
*   **ID:** `Command:login` - Handles user authentication.
*   **ID:** `Command:logout` - Signs the user out.
*   **ID:** `Command:mcp` - Shows MCP server connection status.
*   **ID:** `Command:model` - Opens the model configuration UI.
*   **ID:** `Command:modelstatus` - Displays the current model status.
*   **ID:** `Command:onboarding` - Starts the new user setup flow.
*   **ID:** `Command:pr-comments` - (`prompt` type) Fetches PR comments from GitHub.
*   **ID:** `Command:refresh-commands` - Hot-reloads custom commands.
*   **ID:** `Command:release-notes` - Shows release notes (currently disabled).
*   **ID:** `Command:resume` - Resumes a previous conversation from logs.
*   **ID:** `Command:review` - (`prompt` type) Performs an AI-driven code review of a PR.
*   **ID:** `Command:terminal-setup` - Installs Shift+Enter key bindings for terminals.

#### 2. Service

*   **ID:** `Service:Claude-API`
    *   **Description:** The core service for interacting with the backend LLM. It's wrapped by a model-agnostic adapter.
    *   **Relationships:**
        *   `UsedBy -> Function:queryLLM`

*   **ID:** `Service:Model-Adapter-Factory`
    *   **Description:** A factory that provides a consistent interface for different LLM backends (e.g., Anthropic, OpenAI, Google), allowing the application to be model-agnostic.
    *   **Relationships:**
        *   `Produces -> Abstraction:ModelAdapter`

*   **ID:** `Service:MCP-Client`
    *   **Description:** Manages connections to external "Multi-Connect Protocol" servers to dynamically fetch additional tools.
    *   **Relationships:**
        *   `Manages -> Abstraction:MCP-Server`

*   **ID:** `Service:Custom-Commands-Loader`
    *   **Description:** Scans the filesystem for user-defined and project-defined custom commands and loads them into the application.
    *   **Relationships:**
        *   `InvokedBy -> Command:refresh-commands`

#### 3. Abstraction

*   **ID:** `Abstraction:Tool-System`
    *   **Description:** The core system that defines, manages, and executes tools. It's responsible for presenting tools to the AI and running them safely.
    *   **Relationships:**
        *   `Implements -> Policy:AI-Driven-Tool-Usage`
        *   `ComposedOf -> Abstraction:Tool`

*   **ID:** `Abstraction:Tool`
    *   **Description:** The base interface for any action the AI can perform, such as `BashTool` or `ReadFileTool`. Each tool must define a `zod` schema for its inputs.
    *   **Relationships:**
        *   `PartOf -> Abstraction:Tool-System`

*   **ID:** `Abstraction:Command-System`
    *   **Description:** The system responsible for registering and executing user-facing commands (e.g., `/help`, `/agents`).
    *   **Relationships:**
        *   `ComposedOf -> Command`

*   **ID:** `Abstraction:REPL-Screen`
    *   **Description:** The main React component that implements the REPL UI, including the input box, message history, and status bar.
    *   **Relationships:**
        *   `Realizes -> Feature:Core-REPL`

*   **ID:** `Abstraction:Model-Manager`
    *   **Description:** A global singleton responsible for managing all aspects of model configuration, including providers, API keys, and profiles.
    *   **Relationships:**
        *   `ModifiedBy -> Command:model`
        *   `ReloadedBy -> Function:reloadModelManager`

#### 4. Function

*   **ID:** `Function:queryLLM`
    *   **Description:** The central function for making requests to the configured LLM, handling prompt assembly, API calls, and response parsing.
    *   **Relationships:**
        *   `Uses -> Service:Model-Adapter-Factory`

*   **ID:** `Function:clearConversation`
    *   **Description:** The core logic for the `/clear` command, responsible for resetting numerous pieces of application state.
    *   **Relationships:**
        *   `InvokedBy -> Command:clear`, `Command:login`, `Command:onboarding`

*   **ID:** `Function:generateAgentWithClaude`
    *   **Description:** Uses the LLM to generate a new agent configuration from a natural language description.
    *   **Relationships:**
        *   `PartOf -> Command:agents`

#### 5. Data

*   **ID:** `Data:COMPRESSION_PROMPT`
    *   **Description:** The detailed, structured prompt used by the `/compact` command to summarize conversations.
    *   **Relationships:**
        *   `UsedBy -> Command:compact`
        *   `Defines -> Policy:Structured-Summarization`

*   **ID:** `Data:ProjectConfig` (`claude.json`)
    *   **Description:** A project-specific configuration file stored at the root of a repository.
    *   **Relationships:**
        *   `CreatedBy -> Command:init`

*   **ID:** `Data:GlobalConfig`
    *   **Description:** A user-specific configuration file stored in the home directory, containing authentication tokens and global settings.
    *   **Relationships:**
        *   `ModifiedBy -> Command:login`, `Command:logout`, `Command:terminal-setup`

*   **ID:** `Data:AGENTS.md`
    *   **Description:** A key file providing instructions and context for the AI agent when operating within a repository.
    *   **Relationships:**
        *   `ModifiedBy -> Function:handleHashCommand`

#### 6. Dependency

*   **ID:** `Dependency:React` & `Dependency:Ink` - Core libraries for building the interactive terminal UI.
*   **ID:** `Dependency:zod` - Used for defining and validating the input schemas for all Tools.
*   **ID:** `Dependency:chalk` - Used for coloring text output in the terminal.
*   **ID:** `Dependency:cli-table3` - Used by `/ctx-viz` to generate ASCII tables.
*   **ID:** `Dependency:gray-matter` - Used by `/agents` to parse the YAML frontmatter from agent markdown files.

#### 7. Quality

*   **ID:** `Quality:Unit-Tests` & `Quality:Integration-Tests`
    *   **Description:** The project uses Vitest for testing. Tests are located in the `test/` directory and cover components, utils, and services.
    *   **Relationships:**
        *   `VerifiedBy -> Action:Test`

*   **ID:** `Quality:Linting-Rules` & `Quality:Code-Formatting`
    *   **Description:** The project uses ESLint for linting and Prettier for code formatting to maintain code consistency.
    *   **Relationships:**
        *   `VerifiedBy -> Action:Lint`
