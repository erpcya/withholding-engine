# Knowledge Contract — withholding-engine

## 1. Identity

| Field | Value |
|---|---|
| Name | withholding-engine |
| Repository Type | Library |
| Classification basis | `.ai/repository.yml` declares `type: Library`; owner and entity type are read from the same file. |
| Standards | knowledge-contract-v1, repository-classification-v1 |
| Component type | ADempiere Java library / engine |
| Language and target | Java 17 (`sourceCompatibility = 1.17`, `targetCompatibility = 1.17`) |
| Build / runtime | Gradle wrapper 7.3.3; CI builds with Temurin JDK 17 |
| Published artifact | `io.github.adempiere:withholding-engine:<version>` |
| Version | ADempiere base version `3.9.4`; tags: `adempiere-3.9.4-1.3.8`, `adempiere-3.9.4-1.3.7`, `adempiere-3.9.4-1.3.6`, `adempiere-3.9.3-1.3.6` |
| License | Source headers and README: GNU GPL v2 or later; `build.gradle` POM declares GNU GPL v3.0 |
| Root package or module | `org.spin.model`, `org.spin.process`, `org.spin.util`, `org.spin.withholding.setup` |
| Owner | ERP Consultores y Asociados |
| Entity type | WHH |
| Upstream | `https://github.com/adempiere/withholding-engine` |
| Default branch | `erpya` |

---

## 2. Responsibility

The repository owns a reusable ADempiere withholding engine. It provides dictionary structures, a model validator, document and process validation, withholding generation, declaration, reprocessing, and sending, together with the extension point for location-specific calculations.

What it owns:

- Withholding dictionary model: `WH_Definition`, `WH_DefinitionLine`, `WH_Type`, `WH_Setting`, `WH_Log`, `WH_Withholding`.
- ADempiere model validator `org.spin.model.Withholding`, which registers the engine in ADempiere document and model validation.
- Process classes for generating withholding documents, declaring withholding, reprocessing documents, and sending withholding documents by mail or notification queue.
- The Java engine singleton `org.spin.util.WithholdingEngine`.
- The extension point `org.spin.util.AbstractWithholdingSetting`, with `GenericWithholdingSetting` as the fallback.
- Dictionary migrations under `xml/migration/`.
- The Maven publication producing `io.github.adempiere:withholding-engine`.
- Platform workflows under `.github/workflows/` including `jarvis-release-result.yml` (managed by Jarvis), `knowledge-on-release.yml`, `pull-request-review.yml`, and `release-candidate.yml`.

What it does not own:

- Customer-specific withholding rules or location-specific legal implementations. The README states this is only an engine; consumers implement a custom class such as their external `APInvoiceIVA` example.
- Customer customizations belonging in `PatchCustomer`.
- The ADempiere core `Base` implementation; it depends on `io.github.adempiere:base:3.9.4`.
- Standalone Swing/Web/Mobile presentation code.
- Secrets or deployment credentials; CI obtains those from runner secrets or GitHub run tokens.
- The Jarvis service itself; the workflow only sends a notification to a configured webhook.

---

## 3. Architecture

```text
.ai/knowledge.md
.ai/repository.yml
.github/workflows/
  ci.yml
  jarvis-release-result.yml
  knowledge-on-release.yml
  publish.yml
  pull-request-review.yml
  release-candidate.yml
.gitignore
build.gradle
settings.gradle
gradle/wrapper/
src/main/java/org/spin/model/
  I_WH_*.java                         generated dictionary interfaces
  X_WH_*.java                         generated dictionary model classes
  MWHDefinition.java
  MWHDefinitionLine.java
  MWHLog.java
  MWHSetting.java
  MWHType.java
  MWHWithholding.java
  Withholding.java                    model validator
src/main/java/org/spin/process/
  WithholdingDeclaration*.java
  WithholdingGenerate*.java
  WithholdingProcess*.java
  WithholdingReProcess*.java
  WithholdingSend*.java
src/main/java/org/spin/util/
  AbstractWithholdingSetting.java
  GenericWithholdingSetting.java
  WithholdingEngine.java
src/main/java/org/spin/withholding/setup/
  Deploy.java
xml/migration/001_*_*.xml … 059_*_*.xml
docs/Withholding.erm
docs/Withholding.png
LICENSE
README.md
```

- `I_WH_*` and `X_WH_*` classes are generated from the ADempiere dictionary. The `X_WH_*` files carry `DO NOT CHANGE` in their headers.
- `MWH*` classes add cached lookup methods and application behavior over the generated classes.
- `WithholdingEngine` resolves applicable `MWHDefinition`/`MWHSetting` entries for a document type, table, model event, or process event and invokes the configured withholding setting implementation.
- Custom calculation logic is implemented by subclassing `AbstractWithholdingSetting` and is configured through `WH_Setting.WithholdingClassName`.
- `MWHSetting.getSettingInstance()` falls back to `GenericWithholdingSetting` when no or no valid custom class is configured.
- `Withholding` is the ADempiere model validator entry point; it registers document validation for invoices and model changes for tables configured in the dictionary.
- `Deploy` implements `ISetupDefinition` and creates the ADempiere model validator record for `org.spin.model.Withholding` when not already present.
- Migrations under `xml/migration/` are numbered sequence files that create dictionary records, windows, processes, smart browsers, and related metadata. Sequence numbers range from `500010` (001) to `500440` (059).
- `.github/workflows/jarvis-release-result.yml` is declared `Managed by Jarvis. Do not edit this file manually.` It notifies the Jarvis service when a release is published.
- `.github/workflows/knowledge-on-release.yml` asks the knowledge service to evaluate release impact against the contract.
- `.github/workflows/pull-request-review.yml` forwards `erp-ai:reanalyse` / `erp-ai:rework` label events to the knowledge service.
- `.github/workflows/release-candidate.yml` builds and publishes release candidates (`pr<number>-<sha7>`) when the `erp-ai:candidate` label is applied, and records `erp-ai:verified` events.

### Pre-existing records this repository modifies

| Record | Table | Columns changed | Effect | Migration |
|---|---|---|---|---|
| None | — | — | — | — |

The provided migration analysis records `UPDATES TO PRE-EXISTING RECORDS: None`.

---

## 4. Dependencies

| Dependency | Scope | Purpose |
|---|---|---|
| `io.github.adempiere:base:3.9.4` | build/runtime API | ADempiere core classes: model, process, document engine, model validator, persistence. |
| `lib/*.jar` via `fileTree` | build API | Optional local jar classpath; no tracked `lib/` directory is present in the repository evidence. |
| Gradle wrapper `7.3.3` | build | Build tooling declared in `gradle/wrapper/gradle-wrapper.properties`. |
| Java 17 | build/runtime | Compilation and execution target declared in `build.gradle` and CI. |
| Gradle plugins `maven-publish`, `signing` | build/publish | Maven publication and artifact signing. |
| `org.spin.queue.notification.DefaultNotifier`, `org.spin.queue.util.QueueLoader` | runtime | Used by `WithholdingSend` for the notification-queue path; source is not present in this repository and is expected from the consuming classpath. |
| Jarvis webhook endpoint | CI integration | `jarvis-release-result.yml` posts release results to Jarvis; the endpoint URL is hardcoded as an internal hostname in the workflow file. |
| GitHub Actions secrets/run tokens (`ORG_GRADLE_PROJECT_deploy*`, `GITHUB_TOKEN`, `DEPLOY_TOKEN`) | CI/publish | Publication authentication; values belong in GitHub secrets, not in this contract. |

---

## 5. Consumers

- ADempiere installations that register `org.spin.model.Withholding` as the client model validator.
- Other ADempiere customization modules that depend on the Maven artifact `io.github.adempiere:withholding-engine`.
- Location-specific implementations that subclass `AbstractWithholdingSetting` and configure a `WithholdingClassName`.
- ADempiere users interacting with the repository-created windows, processes, and smart browsers referenced by generated dictionary records.
- The Jarvis service consumes release events via the webhook defined in `jarvis-release-result.yml`.
- The ERP AI platform release and knowledge workflows that use `.ai/repository.yml`, migration files, and publication metadata.

Break-sensitive surfaces:

- Published Maven coordinates `io.github.adempiere:withholding-engine`.
- Java class names and packages: `org.spin.model.Withholding`, `org.spin.util.WithholdingEngine`, `org.spin.util.AbstractWithholdingSetting`, `org.spin.util.GenericWithholdingSetting`.
- Process values and dictionary entries: `WithholdingGenerate`, `WithholdingProcess`, `WithholdingReProcess`, `SBP_WithholdingDeclaration`, `WithholdingSend (Process)`.
- Migration sequence and EntityType `WHH`.
- Model validator registration class `org.spin.model.Withholding`.
- The Jarvis webhook endpoint URL in `jarvis-release-result.yml`.

---

## 6. Allowed changes

- Add or extend reusable withholding engine behavior in `org.spin.*`, provided the existing public class names and artifact coordinates remain stable.
- Add new ADempiere process classes by extending the generated abstract process classes and following the existing naming pattern.
- Add new dictionary migration files under `xml/migration/` with the next numeric sequence prefix.
- Add new `AbstractWithholdingSetting` implementations or improve the engine resolution logic without changing the extension contract.
- Improve build, CI, publish, and knowledge workflows as long as Java 17 and Gradle 7.3.3 building continue to work and the publication remains `io.github.adempiere:withholding-engine`.
- Correct documented facts in README or workflows.
- Regenerate `I_WH_*`, `X_WH_*`, and abstract generated process/model classes from the dictionary rather than hand-editing them.

---

## 7. Prohibited changes

- Do not introduce customer-only withholding rules or integrations into this repository; those belong in `PatchCustomer` or a consumer implementation.
- Do not depend on `PatchCustomer`.
- Do not hand-edit generated classes such as `X_WH_*`, `I_WH_*`, or generated `*Abstract` classes except by regeneration.
- Do not change the published artifact coordinates `io.github.adempiere:withholding-engine`, root package, or model validator class name without an explicit compatibility decision.
- Do not remove or rename `org.spin.model.Withholding`, `org.spin.util.WithholdingEngine`, or the `AbstractWithholdingSetting` extension point.
- Do not renumber, delete, or merge released migration files under `xml/migration/`; installations may have applied the sequence already.
- Do not add secrets, credentials, tokens, or personal data to the tree.
- Do not treat upstream merges as local release changes or merge them silently into the platform-tracked line.
- Do not hand-edit `.github/workflows/jarvis-release-result.yml`; it is declared `Managed by Jarvis. Do not edit this file manually.`

---

## 8. Architectural rules

1. New dictionary records must be introduced through ordered `xml/migration/*.xml` files and must keep the existing sequence direction.
2. The model validator entry point remains `org.spin.model.Withholding`; rule-selection logic belongs in `WithholdingEngine`.
3. Custom withholding calculation is an `AbstractWithholdingSetting` implementation loaded through `WH_Setting.WithholdingClassName`, never inline one-off behavior in the generic fallback.
4. The Maven publication must keep `groupId=io.github.adempiere` and `artifactId=withholding-engine`.
5. Java 17 and the Gradle wrapper must remain the build baseline unless changed through an explicit repository decision.
6. If a future migration modifies a dictionary record that previous installations already had, that modification must be recorded in section 3 as a pre-existing record modification.
7. Generated code remains generated; manual fixes must happen in the dictionary source and be regenerated.
8. `jarvis-release-result.yml` is managed by Jarvis and is not modified manually.

---

## 9. Risks

| Check | Finding | Impact | Precaution |
|---|---|---|---|
| Identifiers outside the allowed allocation range | No allowed allocation range is declared in this repository's evidence. Migration `Record_ID` values observed range from `0` to `104218`; the evidence reports no seven-digit-or-larger identifiers but asserts nothing about validity. | Agents cannot verify whether an identifier is outside allocation; wrong assumptions may block or accept changes incorrectly. | Declare the repository's allowed allocation range before using identifier magnitude as a gate. |
| Build output or IDE metadata under version control | `.classpath`, `.project`, and `.settings/` are tracked by Git. | Machine-specific IDE state creates noisy diffs and dirty checkouts. | Remove these from version control and add them to `.gitignore`. |
| Secrets in the tree or recoverable from history | None found. The masked CI/build lines name credential-bearing keys; no actual secret value or historical leak is shown in the evidence. | N/A | Continue keeping real credentials in GitHub secrets/runner tokens. |
| Absent verification mechanism | No test sources or test dependencies are present; CI runs `./gradlew build` only. | Changes are compile-verified but not behavior-verified. | Use the repository's release-candidate workflow or functional verification before relying on changes. |
| Pre-existing records modified (cross-reference section 3) | None found. | N/A | Re-check on future migrations that update records not inserted by this repository. |

| Risk | Impact | Precaution |
|---|---|---|
| Forked from `adempiere/withholding-engine`; local changes create recurring upstream merge costs. | Every hand-merged upstream update can conflict with local differences. | Minimize edits to the fork, keep organization work on `erpya`, and prefer extension over local patch. |
| Entity type anomalies in migrations 055 and 056: AD_Column `100066`, AD_Field `102244`, AD_Column `100068` are inserted with `EntityType=ECA12` rather than `WHH`. | Dictionary records are attributed to another entity type, which can affect export, ownership, and release analysis. | Confirm whether the `ECA12` references are deliberate owner decisions or typos. |
| License mismatch: source headers/README say GPLv2 or later, while `build.gradle` POM says GPLv3.0. | Legal ambiguity about redistribution terms. | Confirm intended license with the owner and align the files. |
| `build.gradle` depends on `lib/*.jar` but no `lib/` directory is tracked. | Builds could behave differently if environment-specific jars exist. | Document whether local jars are ever required, or remove the unneeded fileTree dependency. |
| `.ai/repository.yml` header still says the classification is “Declarative, not authoritative” while the current classification standard says the file's `type` is authoritative. | Pre-contract agents may derive a classification instead of using the declared one. | Update the header comment to match the current standard. |
| Publish workflow publishes to two targets (Sonatype and GitHub Packages) with distinct credential and permission requirements. | Missing or misconfigured credentials, or missing `packages: write` permission, can fail publish with a 401 that does not name the missing piece. | Validate target-specific credentials and permissions before release; keep values out of the repository. |
| `jarvis-release-result.yml` hardcodes the Jarvis webhook endpoint URL as an internal hostname in a tracked file. | Internal infrastructure address is exposed in version control; endpoint changes break the notification silently. | Move the endpoint URL to a repository/organization variable or secret. |
| `jarvis-release-result.yml` is declared `Managed by Jarvis` but is checked into the repository. | Manual edits to a managed file can be overwritten or conflict with Jarvis updates. | Hono the file's annotation; do not edit manually; treat changes as Jarvis-managed. |

---

## 10. Current state

The repository is an ADempiere withholding library forked from `adempiere/withholding-engine`, with its own default branch `erpya`. It builds with Java 17 and Gradle 7.3.3 against `io.github.adempiere:base:3.9.4`.

It provides the `WH_Definition`, `WH_DefinitionLine`, `WH_Type`, `WH_Setting`, `WH_Log`, and `WH_Withholding` dictionary models, together with `org.spin.model.Withholding` as the ADempiere model validator.

Implemented processes include:

- `WithholdingGenerate`
- `WithholdingProcess`
- `WithholdingReProcess`
- `SBP_WithholdingDeclaration`
- `WithholdingSend (Process)`

The `WithholdingSend` process supports both direct mail and notification-queue delivery.

The migration set comprises `001_Add_WHH_EntityType.xml` through `059_Add_Support_to_Notification_to_Queue.xml`. Later migrations include support for manual withholding, multi-currency generation/declaration, source tax, sales order document types, payment method withholding type/exempt flags, sign signature/print format on withholding type, and sending to the notification queue.

CI builds on push and pull requests. Release publication exists for Sonatype and GitHub Packages. Platform workflows cover Jarvis release notification (`jarvis-release-result.yml`, managed by Jarvis), knowledge-on-release, pull-request review, and release candidates.

A knowledge contract is present at `.ai/knowledge.md`. Tags currently include `adempiere-3.9.4-1.3.8`, `adempiere-3.9.4-1.3.7`, `adempiere-3.9.4-1.3.6`, and `adempiere-3.9.3-1.3.6`.

Known gaps and issues:

- No automated test suite is present.
- `.classpath`, `.project`, and `.settings/` are tracked.
- `lib/` dependency is declared but the directory is not tracked.
- Migrations 055 and 056 contain `ECA12` entity type anomalies.
- License declarations are inconsistent between source headers/README and the Maven POM.
- `.ai/repository.yml` header comment is out of date with respect to the current classification standard.
- The Jarvis webhook endpoint URL is hardcoded in a tracked workflow file.

---

## 11. UNKNOWN

- The repository's allowed dictionary identifier allocation range is not declared in the evidence.
- Whether a local `lib/` directory is expected or required for some build/runtime classpath is not documented.
- The provider of the runtime `org.spin.queue.notification.DefaultNotifier` and `org.spin.queue.util.QueueLoader` classes is not visible in this repository; it is expected from the consuming ADempiere environment.
- Whether the `ECA12` entity type records in migrations 055 and 056 are deliberate references to another module or accidental typos has not been confirmed.
- The intended final license — GPLv2-or-later versus GPLv3.0 — could not be confirmed from the inconsistent repository files alone.
- Whether the hardcoded Jarvis webhook endpoint URL in `jarvis-release-result.yml` is intended to remain in version control or should be moved to a repository/organization variable has not been confirmed.