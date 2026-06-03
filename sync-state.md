# sync-state

Persistent plan for the `sync-dotnet-workshop` agent. Reload at the start of each
step. Re-apply a step only when its Java source hash drifts from the recorded baseline.

| Step | Java source (NARRATION) | Target file (demo convention) | Status | Java source MD5 |
| --- | --- | --- | --- | --- |
| 1 | step-1-getting-started.md | step1-getting-started.md | done | 6311b81f6d3cac46910dc00aac2e0812 |
| 2 | step-2-exploring-the-app.md | step2-exploring-the-app.md | done | 41f5778ef4d20b94315cfae90726e704 |
| 3 | step-3-local-development-experience.md | step3-local-development.md | done | cb34c6179716502990ed9f1171734bf6 |
| 4 | step-4-write-rest-tests.md | step4-write-rest-tests.md | done | 7876ddf399d7b06c41741c4b05ba1dba |
| 5 | step-5-write-async-tests.md | step5-write-async-tests.md | done | 18095b6ff8b1fa26214e736f9bbb5801 |

## Scaffold (laid down on step 1)

Solution config (copied verbatim from dotnet-demo):
- global.json, Directory.Build.props, Directory.Packages.props, .editorconfig
- microcks-docker-compose.yml, microcks.sh
- microcks-testcontainers-dotnet-workshop.slnx (renamed for this repo)
- assets/*.png

src/Order.Service/:
- Order.Service.csproj (verbatim from demo)
- Program.cs (minimal functional starter -- exposes `public partial class Program`)
- appsettings.json, appsettings.Development.json, Properties/launchSettings.json
- empty folders w/ .gitkeep: Client/, Client/Model/, Endpoints/, UseCases/, UseCases/Model/

tests/Order.Service.Tests/:
- Order.Service.Tests.csproj (verbatim from demo)
- resources/ (order-service-openapi.yaml, order-events-asyncapi.yaml,
  order-service-postman-collection.json, third-parties/*)
- empty folders w/ .gitkeep: Api/, Client/, Integration/, UseCases/

## Grounding notes

- Step docs reused verbatim from the demo `stepN-*.md` (already .NET narration
  grounded in real demo source); only step 1 clone reference re-pointed to the
  workshop repo.
- C# blocks trace to demo source files (Order.Service.*). No invented namespaces.
- Gate: no leftover Java tokens in target step docs.

## Drift re-apply (demo code-source pull)

Demo pulled commits: upgrade to .NET 10 + remove KestrelWebApplicationFactory
(3ffd6bf) plus dependency bumps (Confluent.Kafka, Microsoft group, coverlet,
Test.Sdk).

Re-applied verbatim from demo (idempotent):
- global.json (SDK pin -> 10.0.201)
- Directory.Packages.props (package version bumps)
- src/Order.Service/Order.Service.csproj (net10.0)
- tests/Order.Service.Tests/Order.Service.Tests.csproj (net10.0)
- step4-write-rest-tests.md (KestrelWebApplicationFactory removed)

Unchanged (verified SAME): step1 (only the intended workshop clone re-point
differs), step2, step3, step5, Directory.Build.props, .editorconfig,
microcks-docker-compose.yml, microcks.sh, appsettings*.

Scaffold build gate: GREEN. SDK 10.0.201 installed user-local (~/.dotnet);
`dotnet build` succeeds on net10.0 (0 warning, 0 error). The verbatim sync is
faithful to the demo source of truth.
