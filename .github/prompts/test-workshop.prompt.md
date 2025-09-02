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

- [ ] Module 3 (Dashboard & App Host): The instruction says “Set Default Project”; in Visual Studio the label is “Set as Startup Project”. Confirm and update wording for accuracy.
- [ ] Module 3 (VS Code): Provided `launch.json` snippet runs the AppHost; confirm it’s included in the repo for `start/` (we only have compound configs for running Api and Web). Consider adding the AppHost launch snippet to `start/.vscode/launch.json` or clarify manual creation.
- [ ] Module 3 (Error simulation): Docs mention an error after clicking ~5 different cities. This relies on simulated exceptions in `complete/Api/Data/NwsManager.cs` (every 5th request). Consider briefly mentioning that mechanism for clarity.
- [ ] Module 4 (Typos): “MyWeatheApp” appears in text; should be “MyWeatherHub”.
- [ ] Module 4 (Service discovery URL consistency): The sample changes for `NwsManager` show `new Uri("https://weather-api/")` and a `zoneUrl` without a leading slash. In `complete/Api/Data/NwsManager.cs` the base address is `https://weather-api` (no trailing slash) and `zoneUrl` begins with `/zones/...`. Either approach works, but the doc and code should use one consistent pattern to avoid confusion.
- [ ] Module 4 (External service modeling): The doc adds an external `weather-api` resource. Consider adding a short note that this is purely for development-time modeling and doesn’t proxy traffic—requests still go to the actual external service.
- [ ] Module 5 (Redis admin UI naming): The text references “Redis Commander”, but the instructions and screenshots refer to “Redis Insight”. Standardize on “Redis Insight”.
- [ ] Module 5 (Output caching guidance): The doc says to delete default Output Caching code; in `complete` the API uses `builder.AddRedisOutputCache("cache")` and still configures `services.AddOutputCache(...)` to add tags/policies. Clarify that the intent is to remove the default in-memory policy, not prevent using Output Caching; with Redis configured, `AddOutputCache` policies still apply but store is Redis.
- [ ] Module 5 (Container runtime prerequisite timing): The Redis container requires Docker/Podman. The doc calls this out—good. Consider explicitly reminding users to start Docker before launching the AppHost to avoid first-run surprises.
- [ ] Module 6 (Telemetry duplication): `ServiceDefaults` already enables baseline OpenTelemetry for ASP.NET Core and HttpClient. The module adds custom meters and an activity source (good). Consider adding a note that duplication is expected and that the custom registrations are additive.
- [ ] Module 7 (Persistent containers): The doc suggests `WithLifetime(ContainerLifetime.Persistent)` and `WithInitFiles(...)` as options; the `complete` sample doesn’t use them. Clarify they’re optional enhancements.
- [ ] Module 7 (EF usage in Blazor): The favorites feature compares `Zone` records with `FavoriteZones.Contains(context)`. Because `Zone` is a record, value equality applies, which is intended. Consider a brief doc note to reduce confusion.
- [ ] Module 8 (Package versions): The doc shows MSTest 3.8.2; `complete/IntegrationTests` uses 3.9.3. Align the version or note that any recent 3.x is fine.
- [ ] Module 8 (Empty test file): `complete/IntegrationTests/WeatherBackgroundTests.cs` exists but is empty. Decide whether to remove it or provide a minimal test to avoid confusion.
- [ ] Module 9 (Service names): The deployment examples show `webfrontend`/`apiservice`, while this repo uses `myweatherhub`/`api`. Update examples to match actual resource names to reduce friction during `azd init` prompts.
- [ ] Module 9 (Prereqs): Consider explicitly mentioning that `azd` will trigger interactive login if needed (`az login`) and that users may need appropriate Azure subscription permissions.
- [ ] Module 9 (Working directory): Reiterate that `azd init/provision/deploy` should be run from the `complete/` directory (or wherever the AppHost is) so `azure.yaml` is generated in the right place.
- [ ] Cross-cutting (AI dependency present in complete): The `complete` solution wires an AI chat client:
  - `complete/AppHost/Program.cs` defines `builder.AddGitHubModel("chat-model", "gpt-4o-mini")`.
  - `complete/MyWeatherHub/Program.cs` configures `AddAzureChatCompletionsClient("chat-model").AddChatClient()`.
  - `complete/MyWeatherHub/ForecastSummarizer.cs` uses `IChatClient` to summarize text.
  - `complete/MyWeatherHub/Components/Pages/Home.razor` calls the summarizer after retrieving a forecast.
  This likely requires credentials (GitHub Models or Azure AI) and conflicts with the statement that modules 1–8 require no secrets. Without credentials, selecting a zone will attempt to call the model and can fail. Proposals:
  - Add a guard/fallback in `ForecastSummarizer` (and/or caller) to gracefully skip summarization if the chat client isn’t configured, or
  - Document optional environment variable setup for Module 14 (out of scope here) and instruct testers to avoid the summarization path when validating modules 1–9, or
  - Provide a feature flag to disable summarization for modules 1–9.
- [ ] Cross-cutting (Health endpoints visibility): `MapDefaultEndpoints()` only maps `/health` and `/alive` in Development. Ensure docs/screenshots assume Development; otherwise mention required environment config to see those endpoints in non-dev.
- [ ] Cross-cutting (IT-Tools container): The AppHost includes an extra `it-tools` container with external endpoint. It’s not mentioned in modules 1–9 docs. Either remove it from `complete` for focus or add a brief note so users aren’t surprised by the extra endpoint.