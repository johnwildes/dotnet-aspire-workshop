---
mode: agent
description: Testing procedure for workshop completion. This includes verifying code correctness, documentation clarity, and overall user experience using the new per-unit directory structure.
---

You are an intelligent developer validating this .NET Aspire workshop. You will step through modules 1–9 only, using the documentation under the `workshop/` folder and the `start` and `complete` solutions. You will test code builds, run integration tests, and review the docs for clarity and flow. If any requirements are unclear, you will ask clarifying questions and document assumptions.

You will create a markdown file documenting your progress and any issues encountered. Include a section for recommended documentation improvements for any modules that were unclear or required assumptions.

Scope note (important): This prompt intentionally excludes modules 10–15, including GitHub Models integration and Docker integration. Do not test or modify AI provider setup, MCP servers, or GitHub Models. Only validate modules 1–9.

## Purposes of this testing procedure

There are three main objectives:

1. Validate modules 1–9 independently using the provided docs and code.
2. Identify and document any issues or challenges encountered during the workshop.
3. Verify that the complete solution builds, tests pass, and deployment guidance (module 9) is actionable.

## Workshop Structure (modules 1–9)

The workshop is organized as 15 modules; this testing procedure focuses strictly on the first nine modules:

1. Setup & Installation (`workshop/1-setup.md`)
2. Service Defaults (`workshop/2-servicedefaults.md`)
3. Developer Dashboard & Orchestration (`workshop/3-dashboard-apphost.md`)
4. Service Discovery (`workshop/4-servicediscovery.md`)
5. Integrations (`workshop/5-integrations.md`)
6. Telemetry (`workshop/6-telemetry.md`)
7. Database (`workshop/7-database.md`)
8. Integration Testing (`workshop/8-integration-testing.md`)
9. Deployment (`workshop/9-deployment.md`)

Supporting code snapshots:
- `start/` — starting project used during the workshop
- `complete/` — a completed reference implementation (includes `IntegrationTests/`)

## Environment and Credentials

- Modules 1–8 require no secrets or external credentials.
- Module 9 (Deployment) requires an Azure subscription and the Azure Developer CLI (`azd`). Authentication and parameter collection are typically handled interactively by `azd`. Environment variables are optional; you can pass subscription and location on the command line when running `azd`.

Optional convenience environment variables (Windows PowerShell):

```powershell
# Optional, for non-interactive provisioning in Module 9
$env:AZURE_SUBSCRIPTION_ID = "<your-subscription-id>"
$env:AZURE_LOCATION = "<your-azure-region>"  # e.g. eastus, westus3
```

## Important Notes for Testing (modules 1–9)

- Use the `start/` solution as the baseline when following modules 1–8 instructions; consult `complete/` as a known-good reference.
- Ensure the `complete/` solution builds cleanly and all integration tests pass.
- For deployment (module 9):
  - Run `azd` commands from the correct directory (where `azure.yaml` resides or will be generated).
  - If `azure.yaml` is not present yet, run `azd init` first (as described in the docs) and follow the prompts.
  - Confirm the web project is exposed with external endpoints (look for `.WithExternalHttpEndpoints()` in `complete/AppHost/Program.cs`).

## Known Checks and Common Pitfalls (relevant to this repo)

- Directory context for `azd`: Provisioning often fails if run from the wrong directory. Confirm you are in the app root (for this repo, typically `complete/`) or use `--cwd`.
- External HTTP endpoints: The web app should call `.WithExternalHttpEndpoints()` in AppHost to be reachable post-deployment.
- NuGet restore/build: If encountering restore issues, run a clean build sequence: `dotnet clean`, `dotnet restore`, then `dotnet build`.

## Procedure

- **Preparation (Module 1 – Setup & Installation)**
  - Follow `workshop/1-setup.md` to install prerequisites (including .NET SDK 9, `azd` for Module 9, and any recommended tooling).
  - Verify the repository builds locally using the commands below.

- **Modules 2–8**
  - Follow `workshop/2-servicedefaults.md` through `workshop/8-integration-testing.md` using the `start/` solution as your working project.
  - At each module boundary, note any unclear instructions and deviations needed to get things working.
  - Use `complete/` as a reference if you get blocked; document the differences.

- **Module 9 – Deployment**
  - Follow `workshop/9-deployment.md`.
  - If `azure.yaml` is not present, initialize with `azd init` and confirm it detects the AppHost correctly. Then provision/deploy as appropriate.
  - If you cannot deploy to Azure in your environment, document the exact step where you stop and why (e.g., lack of subscription access), but still validate that `azd init` runs and generates expected files.
## Evaluation

- At the end of each module, verify that the described functionality is present and works as intended (builds succeed, pages load, services communicate, etc.).
- For the `complete/` solution, ensure that all integration tests pass and that AppHost defines external HTTP endpoints for the web project.
- Document any issues, missing prerequisites, or unclear steps, and propose concise improvements to the corresponding workshop module docs.
- Note any assumptions you made to progress, especially where the docs could be clarified.

## Reproducible Commands (Windows PowerShell)

Build and quick verification:

```powershell
# Build start solution
cd start
dotnet build .\MyWeatherHub.sln --verbosity minimal
cd ..

# Build complete solution
cd complete
dotnet build .\MyWeatherHub.sln --verbosity minimal
cd ..
```

Run integration tests (complete solution):

```powershell
cd complete
# Run all tests (includes IntegrationTests)
dotnet test .\MyWeatherHub.sln --verbosity minimal
cd ..
```

Sanity checks in AppHost (complete solution):

```powershell
# Confirm external HTTP endpoints are enabled for the web app
Select-String -Path "complete\AppHost\Program.cs" -Pattern "WithExternalHttpEndpoints"
```

Deployment (Module 9) — optional if you have Azure access:

```powershell
cd complete

# Ensure Azure Developer CLI is installed and up to date (optional)
winget upgrade Microsoft.Azd

# Initialize for deployment (if azure.yaml doesn't exist yet)
azd init
# Follow prompts: Use code in current directory -> Confirm -> Choose an environment name

# Provision (use explicit parameters or interactive prompts)
# Option A: Interactive
azd provision

# Option B: With explicit parameters
azd provision --subscription "$env:AZURE_SUBSCRIPTION_ID" --location "$env:AZURE_LOCATION"

# Deploy the application
azd deploy

# Show the deployed endpoints and resource info
azd show

# Optional cleanup
# azd down --purge --force
cd ..
```

## Deliverable

- Produce a markdown report (e.g., `TESTING-NOTES.md`) capturing:
  - Module-by-module status (1–9)
  - Build/test results and any failures encountered
  - Exact steps/commands you ran when deviating from docs
  - Recommended documentation improvements (concise, actionable)
  - Any environmental constraints or assumptions

## Out-of-scope for this prompt

- GitHub Models integration (workshop module 14)
- Docker integration (module 15)
- Any MCP server creation/publishing workflows
- AI chat templates, vector stores, Qdrant, or related JavaScript assets

Focus exclusively on modules 1–9 in this repository.

## Open questions and issues from modules 1–9

### Low-risk fixes (apply now)

- [x] Module 3 (Dashboard & App Host): The instruction says "Set Default Project"; in Visual Studio the label is "Set as Startup Project". Confirmed and updated wording.
- [x] Module 4 (Typos): "MyWeatheApp" appears in text; should be "MyWeatherHub". Fixed.
- [x] Module 5 (Redis admin UI naming): The text references "Redis Commander", but the instructions and screenshots refer to "Redis Insight". Standardized on "Redis Insight".
- [x] Module 5 (Output caching guidance): Clarified that removing the default in-memory policy avoids duplication; policies/tags still apply with Redis as the store when using `builder.AddRedisOutputCache("cache")`.
- [x] Module 5 (Container runtime prerequisite timing): The Redis container requires Docker/Podman. Added an explicit reminder to start Docker/Podman before launching the AppHost.
- [x] Module 6 (Telemetry duplication): Added a note that ServiceDefaults provides baseline OTEL and custom meters/sources are additive.
- [x] Module 7 (Persistent containers): Clarified that `WithLifetime(ContainerLifetime.Persistent)` and `WithInitFiles(...)` are optional enhancements.
- [x] Module 7 (EF usage in Blazor): The favorites feature compares `Zone` records with `FavoriteZones.Contains(context)`. Because `Zone` is a record, value equality applies, which is intended. Added a brief note to reduce confusion.
- [x] Module 8 (Package versions): The doc shows MSTest 3.8.2; `complete/IntegrationTests` uses 3.9.3. Added a note that any recent 3.x is fine.
- [x] Module 9 (Prereqs): Explicitly mentioned `azd login` and subscription selection; noted required subscription permissions.
- [x] Module 9 (Working directory): Reiterated to run `azd` from the directory containing the AppHost (typically `complete/`).
- [x] Cross-cutting (Health endpoints visibility): Added a note that `MapDefaultEndpoints()` only maps `/health` and `/alive` in Development.

### Recommended improvements

- [ ] Module 3 (VS Code): Provided `launch.json` snippet runs the AppHost; confirm it's included in the repo for `start/` (we only have compound configs for running Api and Web). Consider adding the AppHost launch snippet to `start/.vscode/launch.json` or clarify manual creation.
- [x] Module 3 (Error simulation): Docs mention an error after clicking ~5 different cities. Added a brief note explaining `NwsManager` simulates an exception roughly every 5th request.
- [x] Module 4 (Service discovery URL consistency): The sample changes for `NwsManager` now use `new Uri("https://weather-api")` (no trailing slash) and a `zoneUrl` with a leading slash, matching the complete code.
- [ ] Module 4 (External service modeling): Add a short note that this is development-time modeling and doesn't proxy traffic—requests still go to the actual external service.
- [x] Module 4 (External service modeling): Added a short note that this is development-time modeling and doesn't proxy traffic—requests still go to the actual external service.
- [x] Module 9 (Service names): Updated examples to use `myweatherhub`/`api` instead of `webfrontend`/`apiservice`.
- [ ] Cross-cutting (IT-Tools container): The AppHost includes an extra `it-tools` container with external endpoint. Either remove it from `complete` for focus or add a brief note so users aren't surprised by the extra endpoint.
- [x] Cross-cutting (IT-Tools container): Added a brief note that an optional `it-tools` container may appear and can be ignored for modules 1–9.

### Open questions (need answers before proceeding)

- [ ] Cross-cutting (AI dependency present in complete): The `complete` solution wires an AI chat client:
  - `complete/AppHost/Program.cs` defines `builder.AddGitHubModel("chat-model", "gpt-4o-mini")`.
  - `complete/MyWeatherHub/Program.cs` configures `AddAzureChatCompletionsClient("chat-model").AddChatClient()`.
  - `complete/MyWeatherHub/ForecastSummarizer.cs` uses `IChatClient` to summarize text.
  - `complete/MyWeatherHub/Components/Pages/Home.razor` calls the summarizer after retrieving a forecast.
  This likely requires credentials (GitHub Models or Azure AI) and conflicts with the statement that modules 1–8 require no secrets. Without credentials, selecting a zone will attempt to call the model and can fail. Proposals:
  - Add a guard/fallback in `ForecastSummarizer` (and/or caller) to gracefully skip summarization if the chat client isn't configured, or
  - Document optional environment variable setup for Module 14 (out of scope here) and instruct testers to avoid the summarization path when validating modules 1–9, or
  - Provide a feature flag to disable summarization for modules 1–9.
- [ ] Module 8 (Empty test file): `complete/IntegrationTests/WeatherBackgroundTests.cs` exists but is empty. Decide whether to remove it or provide a minimal test to avoid confusion.