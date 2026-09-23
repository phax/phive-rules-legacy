# phive-rules-legacy

<!-- ph-badge-start -->
[![Sonatype Central](https://maven-badges.sml.io/sonatype-central/com.helger.phive.rules/phive-rules-legacy-parent-pom/badge.svg)](https://maven-badges.sml.io/sonatype-central/com.helger.phive.rules/phive-rules-legacy-parent-pom/)
[![javadoc](https://javadoc.io/badge2/com.helger.phive.rules/phive-rules-all-legacy/javadoc.svg)](https://javadoc.io/doc/com.helger.phive.rules/phive-rules-all-legacy)

> If this project saved you some time or made your day a little easier, a star would mean a lot — it helps others find it too.
<!-- ph-badge-end -->

A set of preconfigured **legacy** validation rules for PHIVE (Philip Helger Integrative Validation Engine) - pronounced `[ˈfaɪv]`.

This project holds the **outdated rule sets** - validation rules for document format versions that are no longer current, but that still need to be available for validating older documents. It was extracted from [phive-rules](https://github.com/phax/phive-rules) in 2026, so that the current rules can be built and released without carrying the accumulated weight of all historic rule sets. It is versioned in lockstep with `phive-rules`, starting at `4.6.0`.

This project is part of my Peppol solution stack. See https://github.com/phax/peppol for other components and libraries in that area.

All projects found in here rely on the PHIVE validation engine provided by https://github.com/phax/phive

The shared API used by all rule modules - the validation rules registration SPI (`IValidationRulesRegistrarSPI`), the `ValidationRulesRegistrar`, and the core helper classes - lives in the separate [phive-rules-shared](https://github.com/phax/phive-rules-shared) project (Maven artifact `com.helger.phive.rules:phive-rules-shared`).

This project is divided into the following sub-projects:
* phive-rules-oioubl-legacy - Legacy validation rules for Danish OIOUBL: the deprecated 1.12.3 up to 1.17.1 rule sets (VES group `dk.oioubl`) as well as the ancient 2.0.2 and the government-deprecated 3.0.1 rule sets (VES group `dk.oioubl.legacy`)
* phive-rules-peppol-legacy - Older Peppol specific rules that are out of date (since v2.0.5, in this repository since v4.6.0)

Aggregator module:
* phive-rules-all-legacy - Aggregator depending on all legacy modules with `PhiveRulesLegacyValidation.initPhiveRulesLegacy` to register them all at once (since v4.4.0, in this repository since v4.6.0)

The Maven coordinates (`com.helger.phive.rules:phive-rules-*`) and the VES coordinates of the moved rule sets are **unchanged** compared to their previous home in `phive-rules` - only the git repository differs.

The Java code in this project is licensed under the Apache 2 license.
The code of the validation artefacts used may use a different license.

# Relationship to `phive-rules`

This project depends on artefacts built by [phive-rules](https://github.com/phax/phive-rules), so **`phive-rules` must be released before this project builds**:
* `phive-rules-en16931` - the legacy Peppol 2025-03 and the legacy OIOUBL 3.0.1 rule sets build on the EN 16931 UBL 1.3.13 rule sets
* `phive-rules-oioubl` - provides the OIOUBL UtilityStatement XML Schemas referenced by the OIOUBL 1.x rule sets
* `phive-rules-all` (test scope only) - used to verify that the current and the legacy rule sets can coexist in a single registry

The dependency direction is strictly `phive-rules-legacy` → `phive-rules`. Do not introduce a dependency the other way round.

# Maven usage

Add the following to your `pom.xml` to use this artifact, replacing `x.y.z` with the latest version:

```xml
<dependency>
  <groupId>com.helger.phive.rules</groupId>
  <artifactId>phive-rules-oioubl-legacy</artifactId>
  <version>x.y.z</version>
</dependency>

<dependency>
  <groupId>com.helger.phive.rules</groupId>
  <artifactId>phive-rules-peppol-legacy</artifactId>
  <version>x.y.z</version>
</dependency>
```

Alternatively depend on the aggregator only:

```xml
<dependency>
  <groupId>com.helger.phive.rules</groupId>
  <artifactId>phive-rules-all-legacy</artifactId>
  <version>x.y.z</version>
</dependency>
```

and register all legacy rule sets in a single call. The legacy rule sets build on the EN 16931 rule sets, so when combining them with the current rules, register the current rules first:

```java
final ValidationExecutorSetRegistry <IValidationSourceXML> aRegistry = new ValidationExecutorSetRegistry <> ();
PhiveRulesValidation.initPhiveRules (aRegistry);
PhiveRulesLegacyValidation.initPhiveRulesLegacy (aRegistry);
```

`initPhiveRulesLegacy` can also be used standalone - it registers the EN 16931 rule sets itself if they are not present in the registry yet.

Alternatively, every module on the classpath that ships an `IValidationRulesRegistrarSPI` implementation (that is, all rule modules) can be discovered and registered automatically - including the correct ordering of modules that depend on each other - via a single call:

```java
final ValidationExecutorSetRegistry <IValidationSourceXML> aRegistry = new ValidationExecutorSetRegistry <> ();
ValidationRulesRegistrar.registerAllValidationRules (aRegistry);
```

# News and noteworthy

v4.6.0 - work in progress
* Initial release after extraction from [phive-rules](https://github.com/phax/phive-rules) v4.6.0
* Contains `phive-rules-peppol-legacy` and `phive-rules-all-legacy`, moved unchanged from `phive-rules`
* Added the new module `phive-rules-oioubl-legacy` holding the legacy Danish OIOUBL rule sets that were previously part of `phive-rules-oioubl`:
    * The deprecated OIOUBL 1.12.3, 1.13.0, 1.13.2, 1.14.2, 1.15.0-rc, 1.15.1, 1.15.2, 1.16.1, 1.17.0-rc and 1.17.1 rule sets (VES group `dk.oioubl`) are now registered by the new class `OIOUBLValidationOlder` in package `com.helger.phive.oioubl.legacy`. `phive-rules-oioubl` retains only the current 1.17.2 rule set
    * `OIOUBLLegacyValidation` (the ancient OIOUBL 2.0.2 and the government-deprecated 3.0.1 rule sets, VES group `dk.oioubl.legacy`) moved from package `com.helger.phive.oioubl` to `com.helger.phive.oioubl.legacy`
    * All VES coordinates are unchanged
* The Peppol `openpeppol` 2024.5 and 2024.11 Schematron XSLTs moved from `phive-rules-peppol` to `phive-rules-peppol-legacy` - they were only referenced by the legacy rule sets

---

My personal [Coding Styleguide](https://github.com/phax/meta/blob/master/CodingStyleguide.md) |
It is appreciated if you star the GitHub project if you like it.
