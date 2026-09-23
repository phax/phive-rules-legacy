# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**phive-rules-legacy** is a Maven multi-module project providing the *outdated* validation rules for [PHIVE](https://github.com/phax/phive) (Philip Helger Integrative Validation Engine): rule sets for document format versions that are no longer current, but that must remain available for validating older documents.

It was extracted from [phive-rules](https://github.com/phax/phive-rules) in 2026 because the deployed JARs had grown too large to release comfortably — the historic rule sets dominated the payload. It is versioned in lockstep with `phive-rules`, **starting at `4.6.0`**.

Part of the Peppol solution stack: https://github.com/phax/peppol

The repository contains 3 modules:
- `phive-rules-oioubl-legacy` — Danish OIOUBL: the deprecated 1.12.3 … 1.17.1 rule sets (VES group `dk.oioubl`, class `OIOUBLValidationOlder`) and the ancient 2.0.2 plus government-deprecated 3.0.1 rule sets (VES group `dk.oioubl.legacy`, class `OIOUBLLegacyValidation`)
- `phive-rules-peppol-legacy` — older Peppol rule sets
- `phive-rules-all-legacy` — the aggregator, `PhiveRulesLegacyValidation.initPhiveRulesLegacy`

## Build Commands

```bash
mvn clean install                                  # Full build (all modules)
mvn clean install -pl phive-rules-peppol-legacy    # Single module
mvn test -pl phive-rules-oioubl-legacy             # Tests for one module
mvn test -pl phive-rules-peppol-legacy -Dtest=PeppolLegacyValidationBisEuropeTest
```

CI runs on Java 17, 21, 25. Target is Java 17.

### Relationship to `phive-rules`

- Maven coordinates (`com.helger.phive.rules:phive-rules-*`) and VES/DVR coordinates of the moved rule sets are **unchanged** from their previous home in `phive-rules` — only the git repository differs.
- This project depends on artefacts built by `phive-rules`, so **`phive-rules` must be installed/released before this project builds**. Locally: run `mvn install` in `../phive-rules` first.
  - `phive-rules-en16931` — the legacy Peppol 2025-03 and legacy OIOUBL 3.0.1 sets build on the EN 16931 UBL 1.3.13 VES
  - `phive-rules-oioubl` — provides the `external/schemas/OIOUBL_v2.1-b` UtilityStatement XSDs that the OIOUBL 1.x rule sets load at runtime
  - `phive-rules-all` (test scope) — used by `ValidationRulesRegistrarFuncTest` to verify current + legacy coexistence
- The dependency direction is strictly `phive-rules-legacy` → `phive-rules`. Never introduce a dependency the other way round.
- The `phive-rules.version` property in the root `pom.xml` pins which `phive-rules` version is used. During development it points at the `-SNAPSHOT`; switch it to the final version once that `phive-rules` version is released.
- `phive-rules` in turn depends on `phive-rules-foundations`, which depends on `phive-rules-shared`. Full release order: `phive-rules-shared` → `phive-rules-foundations` → `phive-rules` → `phive-rules-legacy`.

## Architecture

Every module follows the same pattern as in `phive-rules`:

```
phive-rules-{format}-legacy/
├── src/main/java/.../
│   ├── {Format}…Validation.java             # Registers validation rule sets (init… methods)
│   └── {Format}LegacyValidationSPI.java     # SPI impl (IValidationRulesRegistrarSPI)
├── src/main/resources/
│   ├── META-INF/services/com.helger.phive.rules.shared.IValidationRulesRegistrarSPI
│   └── external/schematron/                 # Pre-compiled XSLT rules
├── src/test/java/.../
│   ├── {Format}LegacyValidationTest.java
│   ├── ValidationExecutionManagerFuncTest.java
│   ├── SPITest.java
│   └── mock/CTestFiles.java                 # Test file loading utility
└── src/test/resources/external/
    ├── rule-source/                         # Original .sch Schematron files
    └── test-files/{version}/                # Sample XML documents
```

The shared base — `IValidationRulesRegistrarSPI`, `ValidationRulesRegistrar`, `DVRHelper`, `PhiveRulesHelper`, `PhiveRulesUBLHelper`, `PhiveRulesCIIHelper`, `PhiveRulesTestHelper` — lives in the standalone `phive-rules-shared` project (package `com.helger.phive.rules.shared`) and is consumed as an external dependency. Do **not** use the deprecated `com.helger.phive.rules.api` package.

### Adding a rule set to this repository

Rule sets arrive here by being *retired* from `phive-rules`, not by being authored here. The move is:
1. Move the registration code (constants + the `VesXmlBuilder` blocks) into the matching `…Validation` class here, keeping the VES coordinates identical.
2. Move `src/main/resources/external/schematron/<version>/`, `src/test/resources/external/test-files/<version>/` and `src/test/resources/external/rule-source/<version>/` across. **Check for classpath resources left behind** — the registration class and the XSLTs it loads must end up in the same repository (this was the cause of the `openpeppol` 2024.x breakage during the initial split).
3. Wire it into the module's SPI and into `PhiveRulesLegacyValidation.initPhiveRulesLegacy`.
4. Update `CTestFiles` on both sides.
5. Add a bullet to the `# News and noteworthy` section of `README.md` in **both** repositories.

## Imports & Annotations

Same as `phive-rules`: **ph-commons 12.x** and **JSpecify** nullness annotations.
- Nullness: `org.jspecify.annotations.{NonNull,Nullable}` — never `javax.annotation.*` or `jakarta.annotation.*`.
- Core utilities live under `com.helger.base.*` and `com.helger.annotation.*` — not the old monolithic `com.helger.commons.*`.

## Schematron Rules

Validation rules are pre-compiled: `.sch` → `.xslt` via `ph-schematron-maven-plugin`. The compiled XSLT files are committed under `src/main/resources/external/schematron/`. The plugin executions are commented out in the module POMs; they exist for reference only — legacy rule sets are frozen and should not need regeneration.

## Testing

- **Framework:** JUnit 4
- **Test logging:** SLF4J Simple (`simplelogger.properties` in test resources)
- `ValidationRulesRegistrarFuncTest` in `phive-rules-all-legacy` asserts that SPI-based discovery yields exactly the same set of VES IDs as `PhiveRulesValidation.initPhiveRules` + `PhiveRulesLegacyValidation.initPhiveRulesLegacy`. It breaks whenever a module is registered by one path but not the other — a useful guard when moving rule sets between the repositories.
- `{Format}LegacyValidationTest.testFilesExist` walks every registered rule resource and asserts it exists. This is the fastest way to catch a rule set whose XSLTs were left behind in the other repository.

## Packaging

All modules produce plain JARs (`<packaging>jar</packaging>`).
