# FNSR Bridge Ontology (FBO)

## Project Specification — Version 1.1

**Status:** Draft
**Author:** Aaron
**Date:** 2026-03-02
**Scope:** Bridge ontology connecting FNSR services (SNP, BIBSS, SAS) to BFO/IAO/CCO/STATO
**Tooling:** Protégé, OWL 2 DL, SPARQL for validation
**Namespace:** `https://ontology.fnsr.dev/fbo/`
**Prefix:** `fbo:`
**Supersedes:** FBO v1.0

---

## Changelog from v1.0

| ID | Section(s) | Severity | Change |
|---|---|---|---|
| F-001 | 5.9, 8.1, 4.1, 13.4 | **Critical** | `fbo:TemporalVariable` moved from `SubClassOf: fbo:QuantitativeVariable` to `SubClassOf: fbo:VariableType` (direct child). Resolves fatal OWL 2 DL unsatisfiability: v1.0 declared TemporalVariable both as a subclass of QuantitativeVariable and disjoint with QuantitativeVariable, making TemporalVariable equivalent to `owl:Nothing`. The 5-way disjointness axiom is now logically valid because all five variable types are siblings under VariableType. This also removes the incorrect assumption that temporal data is always continuous — ordinal time (fiscal quarters, date buckets) is temporal but not continuous. |
| F-002 | 5.10, 7.1, 8.2, 11.4, 12, 13.3 | **High** | Added `fbo:correspondsTo` object property linking semantic-phase `fbo:DataField` individuals to their structural-phase counterparts. v1.0 relied on string matching of `fbo:hasFieldName` to establish cross-phase field identity — an anti-pattern in semantic web design. The new property enables graph traversal from a semantic field back to its structural TypeDistribution without copying string literals onto assertions. New CQ-13 validates this traversal. CQ-07 rewritten to use `fbo:correspondsTo` as primary path. |
| F-003 | 16.1 | **Medium** | Design decision §16.1 rewritten. TemporalVariable is now type-agnostic regarding the continuous/categorical distinction: ISO 8601 timestamps are continuous, fiscal quarters are ordinal categorical. FBO does not force a mathematical commitment that future SAS versions might contradict. |
| F-004 | 16.5, 16.6 | **Clarification** | Added §16.5 (meta-ICE documentation note: SemanticSchema is an ICE *about* another ICE) and §16.6 (TypeDistribution null accounting: sum of all type counts including null must equal hasTotalObservations). |

---

## 1. Purpose

The FNSR Bridge Ontology (FBO) is a thin formal ontology that grounds the FNSR pipeline's operational vocabulary — the `viz:`, `sas:`, and `snp:` namespace terms used in SNP manifests, BIBSS CISMs, and SAS DatasetSchemas — in established upper ontologies. FBO does not invent new conceptual territory. It places existing FNSR concepts into the BFO/IAO/CCO/STATO class hierarchy, enabling formal reasoning about what the pipeline produces, what processes it performs, and what epistemic commitments its outputs carry.

### 1.1 What FBO Is

FBO is a bridge. It defines approximately 40–60 classes and 20–30 object/data properties that formalize the relationships between:

- **Data artifacts** (raw CSV files, cleaned CSV files, CISMs, DatasetSchemas) as information content entities
- **Processes** (normalization, structural inference, semantic alignment) as BFO planned processes
- **Assertions** (type assignments, consensus scores, provenance claims) as information quality assertions
- **Configurations** (thresholds, vocabularies, patterns) as directive information entities

FBO imports and extends:

| Ontology | Role in FBO | Version |
|----------|-------------|---------|
| BFO (Basic Formal Ontology) | Upper ontology. All FBO classes are BFO descendants. | BFO 2020 (ISO/IEC 21838-2) |
| IAO (Information Artifact Ontology) | Mid-level. Data items, documents, assertions. | IAO 2024-01 or latest |
| CCO (Common Core Ontologies) | Mid-level. Measurement, information entities, agents. | CCO 1.5+ |
| STATO (Statistical Methods Ontology) | Domain. Variable types, statistical summaries. | STATO 2024 or latest |

### 1.2 What FBO Is Not

- FBO is not a domain ontology for customs, trade, or border protection. It is domain-agnostic.
- FBO is not a replacement for the JSON-LD `@context` in SAS output. The `@context` is a serialization concern; FBO is an ontological grounding.
- FBO does not define runtime behavior. SNP, BIBSS, and SAS operate on JSON structures, not OWL axioms. FBO describes what those structures *are* in ontological terms.
- FBO is not a full formalization of data quality or statistical methodology. It imports what it needs from STATO and CCO and extends only where gaps exist.

### 1.3 Success Criteria (Project-Level)

FBO is complete when:

1. Every `viz:`, `sas:`, and `snp:` term used in FNSR pipeline output has a formal OWL class or property mapping in FBO
2. Every FBO class is a subclass of exactly one BFO category (continuant or occurrent)
3. The ontology passes OWL 2 DL consistency checking (HermiT or Pellet) with zero unsatisfiable classes
4. The ontology passes all 14 competency question SPARQL queries defined in Section 12
5. A worked example covering the full SNP → BIBSS → SAS pipeline can be represented as FBO individuals — including `fbo:correspondsTo` links between semantic and structural DataFields — and passes all invariant checks
6. No FBO class duplicates a class already defined in IAO, CCO, or STATO — if a mapping exists, FBO uses `owl:equivalentClass` or `rdfs:subClassOf`, not redefinition
7. All five fbo:VariableType subclasses are individually satisfiable and mutually disjoint (F-001 regression gate)

---

## 2. Governing Principles

### 2.1 Ontological Realism

FBO follows the BFO commitment to ontological realism. Classes represent kinds of entities that exist (or could exist) in reality, not conceptual categories or database schema patterns. A `fbo:DatasetSchema` is not a "type of output" — it is an information content entity that *denotes* the structural properties of a data collection. The distinction matters: information content entities are about something, and FBO must specify what each entity is about.

### 2.2 Single Inheritance from BFO

Every FBO class must trace to exactly one BFO top-level category. The two relevant categories are:

- **BFO:continuant** → **BFO:specifically dependent continuant** → **BFO:generically dependent continuant** → IAO:information content entity (for all data artifacts, schemas, assertions, configurations)
- **BFO:occurrent** → **BFO:process** (for all pipeline operations: normalization, inference, alignment)

No FBO class may be both a continuant and an occurrent. No FBO class may be a BFO:material entity (the pipeline operates on information, not physical objects).

### 2.3 Minimal Commitment

FBO defines a class only when:

1. No class in IAO, CCO, or STATO adequately captures the concept, AND
2. The concept is required to answer at least one competency question (Section 12), AND
3. The concept represents a genuine ontological distinction, not a software engineering convenience

When an existing class is close but not exact, FBO subclasses it rather than creating a parallel hierarchy.

### 2.4 Import Discipline

FBO imports full ontology modules, not cherry-picked axioms. Import chain:

```
fbo.owl
  ├── imports: bfo.owl (BFO 2020)
  ├── imports: iao.owl (IAO core)
  ├── imports: cco-information-entity.owl (CCO Information Entity Ontology)
  ├── imports: cco-measurement.owl (CCO Measurement Information Ontology)
  └── imports: stato.owl (STATO core)
```

If an import creates reasoning performance problems, FBO may use MIREOT (Minimum Information to Reference an External Ontology Term) to extract only needed terms with their complete lineage.

---

## 3. BFO Category Mapping

This section establishes which BFO top-level category each FNSR concept belongs to. This mapping is the foundation — every class definition in Sections 5–8 must be consistent with this table.

### 3.1 Information Content Entities (GDC → ICE)

All data artifacts in the FNSR pipeline are **generically dependent continuants** (BFO:GDC) and more specifically **information content entities** (IAO:ICE). They are about something, they can be copied without loss of identity, and they depend on some material bearer (a disk, a memory register) but are not identical to that bearer.

| FNSR Concept | Current Namespace | BFO Path | IAO/CCO/STATO Parent |
|---|---|---|---|
| Raw input dataset | (implicit) | GDC → ICE | IAO:data item |
| SNP-cleaned dataset | (implicit) | GDC → ICE | IAO:data item |
| SNP normalization manifest | `snp:manifest` | GDC → ICE | IAO:data item |
| BIBSS CISM | `bibss:cism` | GDC → ICE | IAO:data item |
| BIBSS SchemaNode | (internal) | GDC → ICE | IAO:information content entity |
| BIBSS typeDistribution | (internal) | GDC → ICE | CCO:measurement information content entity |
| SAS DatasetSchema | `viz:DatasetSchema` | GDC → ICE | IAO:data item |
| SAS DataField | `viz:DataField` | GDC → ICE | IAO:information content entity |
| SAS DataType assertion | `viz:hasDataType` | GDC → ICE | IAO:data quality assertion (proposed subclass) |
| SAS consensus score | `viz:consensusScore` | GDC → ICE | CCO:measurement information content entity |
| SAS configuration | `sas:SASConfig` | GDC → ICE | IAO:directive information entity |
| Consensus threshold | `sas:consensusThreshold` | GDC → ICE | IAO:directive information entity |
| Null vocabulary | `sas:nullVocabulary` | GDC → ICE | IAO:directive information entity |
| Temporal name pattern | `sas:temporalNamePattern` | GDC → ICE | IAO:directive information entity |
| Fandaws concept resolution | `fandaws:concept` | GDC → ICE | IAO:information content entity |

### 3.2 Processes (Occurrents)

All pipeline operations are **BFO:processes** — they unfold in time, have temporal parts, and transform information content entities.

| FNSR Concept | Current Namespace | BFO Path | IAO/CCO Parent |
|---|---|---|---|
| SNP normalization | (implicit) | process → planned process | CCO:act of information processing |
| BIBSS structural inference | (implicit) | process → planned process | CCO:act of information processing |
| SAS semantic alignment | (implicit) | process → planned process | CCO:act of information processing |
| Consensus promotion | (internal) | process → planned process | CCO:act of measurement |
| Temporal detection | (internal) | process → planned process | CCO:act of information processing |
| Fandaws concept resolution | (future) | process → planned process | CCO:act of information processing |

### 3.3 Roles and Dispositions

| FNSR Concept | BFO Category | Rationale |
|---|---|---|
| "This column is Quantitative" | role | The column *plays the role of* quantitative variable in a dataset. The same bytes could play a different role in a different analytical context. |
| "This field is required" | disposition | The field has the disposition to be present in every record of the population. This is an observation about regularity, not an assigned function. |
| "This field is nullable" | disposition | The field has the disposition to bear null values in some records. |

### 3.4 Qualities

| FNSR Concept | BFO Category | Rationale |
|---|---|---|
| Consensus score (0.0–1.0) | quality | A measurable attribute of a type assertion — the degree to which observed evidence supports the assignment. Inheres in the assertion. |
| Occurrences count | quality | A measurable attribute of a schema node — the count of times the structural pattern was observed. |

---

## 4. Upper Ontology Alignments

This section defines the specific mappings between FNSR's five `viz:DataType` values and existing classes in STATO and CCO. These are the most important alignments in FBO because they ground the semantic types that SAS assigns.

### 4.1 Variable Type Alignment (STATO)

STATO defines a hierarchy of statistical variable types under `STATO:0000258` (statistical variable). FBO maps each `viz:DataType` to the most specific applicable STATO class.

| viz:DataType | STATO Class | STATO ID | Relationship | Justification |
|---|---|---|---|---|
| `viz:QuantitativeType` | continuous variable | STATO:0000251 | `rdfs:subClassOf` | SAS QuantitativeType includes both integer and float. STATO's continuous variable is the parent of both ratio and interval variables. FBO subclasses STATO's continuous variable rather than using `owl:equivalentClass` because SAS does not distinguish ratio from interval scale. |
| `viz:NominalType` | categorical variable | STATO:0000252 | `owl:equivalentClass` | Direct equivalence. SAS NominalType is the assignment given when values are unordered categories — exactly STATO's categorical variable. |
| `viz:TemporalType` | (no direct STATO class) | — | `rdfs:subClassOf` STATO:0000258 (via fbo:VariableType) | STATO does not define a temporal variable type. FBO defines `fbo:TemporalVariable` as a direct subclass of `fbo:VariableType` — a sibling of Quantitative, Nominal, Boolean, and Indeterminate. It is not placed under STATO continuous variable because temporal data can be either continuous (ISO 8601 timestamps) or ordinal categorical (fiscal quarters, date buckets). See §16.1. |
| `viz:BooleanType` | binary variable | STATO:0000255 | `owl:equivalentClass` | Direct equivalence. Binary variable in STATO is a categorical variable with exactly two levels, which matches SAS's BooleanType semantics. |
| `viz:UnknownType` | (no STATO class) | — | (FBO-defined) | FBO defines `fbo:IndeterminateVariableType` as a subclass of STATO statistical variable. It represents the case where structural evidence is insufficient for classification. |

### 4.2 Measurement Type Alignment (CCO)

CCO's Measurement Information Ontology provides classes for representing measurements and their results. FBO maps the FNSR measurement concepts:

| FNSR Concept | CCO Class | Relationship | Justification |
|---|---|---|---|
| `viz:consensusScore` | CCO:MeasurementInformationContentEntity | `rdf:type` | The consensus score is the result of a measurement process (counting type occurrences and computing a ratio). |
| `typeDistribution` | CCO:MeasurementInformationContentEntity | `rdf:type` | The distribution is a structured measurement result recording observation counts per type. |
| `occurrences` | CCO:MeasurementInformationContentEntity | `rdf:type` | The count of observations. |
| Consensus computation | CCO:ActOfMeasurement | `rdf:type` | The process of computing consensus from typeDistribution is an act of measurement. |
| SAS alignment | CCO:ActOfInformationProcessing | `rdf:type` | The overall alignment process transforms a CISM into a DatasetSchema. |

### 4.3 Information Entity Alignment (IAO)

| FNSR Concept | IAO Class | IAO ID | Relationship |
|---|---|---|---|
| Raw input dataset | IAO:data item | IAO:0000027 | `rdf:type` |
| CISM | IAO:data item | IAO:0000027 | `rdf:type` |
| DatasetSchema | IAO:data item | IAO:0000027 | `rdf:type` |
| SAS diagnostic | IAO:report | IAO:0000088 | `rdf:type` |
| SASConfig | IAO:directive information entity | IAO:0000033 | `rdf:type` |
| SAS type assertion | IAO:data quality assertion (proposed) | — | See §4.4 |
| SNP manifest entry | IAO:report | IAO:0000088 | `rdf:type` |

### 4.4 The Data Quality Assertion Gap

IAO does not define a "data quality assertion" class. The closest is `IAO:assertion` (IAO:0000034), which is "a sentence that is intended to be interpreted as true or false." SAS's type assignments are assertions: "This field has data type QuantitativeType" is a claim intended to be evaluated as true or false.

However, SAS assertions carry epistemic weight — they have a consensus score, a provenance chain, and a configurable confidence threshold. IAO's bare `assertion` class does not distinguish between well-evidenced and poorly-evidenced assertions.

**FBO resolution:** Define `fbo:EvidencedTypeAssertion` as a subclass of `IAO:assertion` with the additional axiom:

```
fbo:EvidencedTypeAssertion ⊑ IAO:assertion
  ⊓ ∃ fbo:hasEvidenceStrength xsd:decimal
  ⊓ ∃ fbo:hasEvidenceSource fbo:TypeDistribution
  ⊓ ∃ fbo:assertsVariableType STATO:statistical_variable
```

This is the most conceptually significant class FBO introduces. It formalizes the idea that SAS's type assignments are not arbitrary labels — they are assertions backed by statistical evidence (the typeDistribution) with a quantified confidence (the consensus score).

---

## 5. FBO Class Definitions — Information Content Entities

Each class below is defined with: BFO parent lineage, natural language definition, formal axioms (OWL Manchester Syntax), and the FNSR artifact it grounds.

### 5.1 fbo:RawDataset

**Parent:** IAO:data item
**Definition:** An information content entity that is a collection of data records in a tabular or structured format, prior to any normalization or processing.
**Grounds:** The input to the FNSR pipeline (CSV or JSON file).

```
Class: fbo:RawDataset
  SubClassOf: IAO:data_item
  SubClassOf: fbo:hasFormat some fbo:DataFormat
  SubClassOf: IAO:is_about some owl:Thing
```

**Acceptance Criteria:**
- [ ] Classified as IAO:data item by reasoner
- [ ] Not classified as IAO:document (a dataset is not a document — it lacks narrative structure)
- [ ] Disjoint with fbo:NormalizedDataset, fbo:StructuralSchema, fbo:SemanticSchema

### 5.2 fbo:NormalizedDataset

**Parent:** IAO:data item
**Definition:** An information content entity that is a collection of data records that has undergone format-level normalization (whitespace trimming, empty-to-null conversion, currency symbol stripping, date format standardization) but retains the same record structure as the source raw dataset.
**Grounds:** The output of SNP (the `cleaned` field).

```
Class: fbo:NormalizedDataset
  SubClassOf: IAO:data_item
  SubClassOf: fbo:isOutputOf some fbo:NormalizationProcess
  SubClassOf: fbo:derivedFrom some fbo:RawDataset
```

**Acceptance Criteria:**
- [ ] Classified as IAO:data item by reasoner
- [ ] Disjoint with fbo:RawDataset (a dataset cannot be both raw and normalized)
- [ ] Entails: if `x` is a fbo:NormalizedDataset, then there exists some fbo:RawDataset `y` such that `x fbo:derivedFrom y`

### 5.3 fbo:NormalizationManifest

**Parent:** IAO:report
**Definition:** An information content entity that is a structured record of every transformation applied during normalization, including the rule invoked, the data path affected, and the before/after values.
**Grounds:** The SNP manifest (the `manifest` array in SNP output).

```
Class: fbo:NormalizationManifest
  SubClassOf: IAO:report
  SubClassOf: fbo:isOutputOf some fbo:NormalizationProcess
  SubClassOf: IAO:is_about some fbo:NormalizationProcess
```

**Acceptance Criteria:**
- [ ] Classified as IAO:report by reasoner
- [ ] Every fbo:NormalizationManifest is about exactly one fbo:NormalizationProcess

### 5.4 fbo:ManifestEntry

**Parent:** IAO:information content entity
**Definition:** An atomic unit within a normalization manifest recording a single transformation: the rule applied, the field or row affected, and the transformation detail.
**Grounds:** Individual entries in the SNP manifest array.

```
Class: fbo:ManifestEntry
  SubClassOf: IAO:information_content_entity
  SubClassOf: fbo:hasRule some xsd:string
  SubClassOf: fbo:hasDataPath some xsd:string
  SubClassOf: IAO:is_part_of some fbo:NormalizationManifest
```

### 5.5 fbo:StructuralSchema

**Parent:** IAO:data item
**Definition:** An information content entity that describes the structural properties of a dataset — the fields present, their observed primitive types, their occurrence frequencies, and their nesting relationships — without interpreting the semantic meaning of field names or values.
**Grounds:** The BIBSS CISM (Canonical Internal Schema Model).

```
Class: fbo:StructuralSchema
  SubClassOf: IAO:data_item
  SubClassOf: fbo:isOutputOf some fbo:StructuralInferenceProcess
  SubClassOf: IAO:is_about some fbo:RawDataset or fbo:NormalizedDataset
  SubClassOf: fbo:hasSchemaVersion some xsd:string
```

**Acceptance Criteria:**
- [ ] Classified as IAO:data item by reasoner
- [ ] Disjoint with fbo:SemanticSchema
- [ ] A fbo:StructuralSchema makes no claims about semantic variable types — it reports only structural types (string, integer, number, boolean, null)

### 5.6 fbo:TypeDistribution

**Parent:** CCO:MeasurementInformationContentEntity
**Definition:** A measurement information content entity that records the count of each observed primitive type for a data field across a population of records, prior to type widening or semantic interpretation. The distribution is the pre-widening evidence that downstream services use for consensus-based type promotion.
**Grounds:** The `typeDistribution` field on BIBSS SchemaNode.

```
Class: fbo:TypeDistribution
  SubClassOf: CCO:MeasurementInformationContentEntity
  SubClassOf: IAO:is_about some fbo:StructuralSchemaNode
  SubClassOf: fbo:hasTotalObservations some xsd:nonNegativeInteger
```

**Acceptance Criteria:**
- [ ] Classified as CCO:MeasurementInformationContentEntity by reasoner
- [ ] Entails: the sum of all type counts in a TypeDistribution equals its hasTotalObservations value
- [ ] Disjoint with fbo:ConsensusScore (distribution is evidence; score is a derived measure)

### 5.7 fbo:SemanticSchema

**Parent:** IAO:data item
**Definition:** An information content entity that describes the semantic properties of a dataset — the variable type (Quantitative, Nominal, Temporal, Boolean, Unknown) assigned to each field, the confidence of each assignment, and the provenance chain connecting the assignment to its structural evidence.
**Grounds:** The SAS DatasetSchema (`viz:DatasetSchema`).

```
Class: fbo:SemanticSchema
  SubClassOf: IAO:data_item
  SubClassOf: fbo:isOutputOf some fbo:SemanticAlignmentProcess
  SubClassOf: fbo:derivedFrom some fbo:StructuralSchema
  SubClassOf: IAO:is_about some (fbo:RawDataset or fbo:NormalizedDataset)
```

**Acceptance Criteria:**
- [ ] Classified as IAO:data item by reasoner
- [ ] Disjoint with fbo:StructuralSchema
- [ ] Every fbo:SemanticSchema is derived from exactly one fbo:StructuralSchema
- [ ] Every fbo:SemanticSchema contains at least one fbo:EvidencedTypeAssertion

### 5.8 fbo:EvidencedTypeAssertion

**Parent:** IAO:assertion
**Definition:** An assertion that a data field has a specific statistical variable type, supported by quantified structural evidence (a type distribution and consensus score) and traceable to a specific inference process and configuration.
**Grounds:** Each `viz:DataField` in SAS output is an evidenced type assertion.

```
Class: fbo:EvidencedTypeAssertion
  SubClassOf: IAO:assertion
  SubClassOf: fbo:assertsVariableType some fbo:VariableType
  SubClassOf: fbo:hasEvidenceStrength some xsd:decimal
  SubClassOf: fbo:hasEvidenceSource some fbo:TypeDistribution
  SubClassOf: fbo:producedByRule some xsd:string
  SubClassOf: IAO:is_about some fbo:DataField
```

**Acceptance Criteria:**
- [ ] Classified as IAO:assertion by reasoner
- [ ] Every fbo:EvidencedTypeAssertion has exactly one fbo:assertsVariableType
- [ ] Every fbo:EvidencedTypeAssertion has exactly one fbo:hasEvidenceStrength with value in [0.0, 1.0]
- [ ] The hasEvidenceStrength value is traceable to a fbo:TypeDistribution via fbo:hasEvidenceSource

### 5.9 fbo:VariableType (Bridge to STATO)

**Parent:** STATO:statistical variable (STATO:0000258)
**Definition:** The kind of statistical variable a data field represents. FBO defines this as an intermediate class that holds the five FNSR types as named individuals or subclasses.

```
Class: fbo:VariableType
  EquivalentTo: STATO:0000258
```

**Named Subclasses:**

```
Class: fbo:QuantitativeVariable
  SubClassOf: fbo:VariableType
  SubClassOf: STATO:0000251  (continuous variable)

Class: fbo:NominalVariable
  EquivalentTo: STATO:0000252  (categorical variable)

Class: fbo:TemporalVariable
  SubClassOf: fbo:VariableType
  Annotations: rdfs:comment "Time-valued variable. FBO does not place this under
    QuantitativeVariable (STATO continuous) or NominalVariable (STATO categorical)
    because temporal data can be either: ISO 8601 timestamps are continuous points
    on a timeline, while fiscal quarters or date buckets are ordinal categories.
    FBO remains agnostic on this distinction. STATO does not define a temporal
    variable type; this is FBO's primary extension. See §16.1."

Class: fbo:BooleanVariable
  EquivalentTo: STATO:0000255  (binary variable)

Class: fbo:IndeterminateVariable
  SubClassOf: fbo:VariableType
  Annotations: rdfs:comment "Variable type assignment where structural evidence was
    insufficient for classification. Not a 'missing' type — it is a positive assertion
    that the evidence does not support any specific type."
```

**Acceptance Criteria:**
- [ ] fbo:QuantitativeVariable classified as subclass of STATO continuous variable
- [ ] fbo:NominalVariable classified as equivalent to STATO categorical variable
- [ ] fbo:BooleanVariable classified as equivalent to STATO binary variable
- [ ] fbo:TemporalVariable classified as direct subclass of fbo:VariableType (NOT under QuantitativeVariable)
- [ ] fbo:TemporalVariable is NOT classified as subclass of STATO:0000251 (continuous variable) or STATO:0000252 (categorical variable)
- [ ] fbo:IndeterminateVariable is NOT classified as any STATO typed variable — it is disjoint with QuantitativeVariable, NominalVariable, TemporalVariable, BooleanVariable
- [ ] All five subclasses are mutually disjoint (a field cannot be both Quantitative and Temporal)
- [ ] Reasoner can infer: if a field is fbo:BooleanVariable, it is also a STATO:categorical variable (via STATO's own hierarchy: binary ⊑ categorical)

### 5.10 fbo:DataField

**Parent:** IAO:information content entity
**Definition:** An information content entity that denotes a named column or property within a dataset. A DataField is the subject of type assertions and carries structural metadata (occurrences, nullability, required status). DataField individuals exist at both the structural phase (within a CISM) and the semantic phase (within a DatasetSchema). Cross-phase identity is established via `fbo:correspondsTo`, not by matching field name strings.
**Grounds:** `viz:DataField` in SAS output, `SchemaEdge` in BIBSS CISM.

```
Class: fbo:DataField
  SubClassOf: IAO:information_content_entity
  SubClassOf: fbo:hasFieldName some xsd:string
  SubClassOf: IAO:is_part_of some (fbo:StructuralSchema or fbo:SemanticSchema)
```

**Acceptance Criteria:**
- [ ] Classified as IAO:information content entity by reasoner
- [ ] A semantic-phase DataField linked via `fbo:correspondsTo` to a structural-phase DataField enables graph traversal to the structural TypeDistribution without string matching

### 5.11 fbo:AlignmentConfiguration

**Parent:** IAO:directive information entity (IAO:0000033)
**Definition:** A directive information entity that specifies the parameters governing a semantic alignment process — consensus thresholds, observation minimums, temporal patterns, null vocabularies, and boolean field designations.
**Grounds:** `SASConfig` in SAS.

```
Class: fbo:AlignmentConfiguration
  SubClassOf: IAO:directive_information_entity
  SubClassOf: fbo:hasConsensusThreshold some xsd:decimal
  SubClassOf: fbo:hasMinObservationThreshold some xsd:nonNegativeInteger
```

**Acceptance Criteria:**
- [ ] Classified as IAO:directive information entity by reasoner
- [ ] A directive information entity is "about" a planned process — it prescribes how the process should be carried out
- [ ] The configuration does not perform alignment; it governs an alignment process

### 5.12 fbo:DataFormat

**Parent:** IAO:information content entity
**Definition:** An information content entity that denotes the syntactic format of a dataset.

```
Class: fbo:DataFormat
  SubClassOf: IAO:information_content_entity

Individual: fbo:CSV
  Types: fbo:DataFormat

Individual: fbo:JSON
  Types: fbo:DataFormat
```

---

## 6. FBO Class Definitions — Processes

### 6.1 fbo:FNSRProcess (Abstract)

**Parent:** BFO:planned process → CCO:ActOfInformationProcessing
**Definition:** A planned process in which information content entities are transformed according to a specification. All FNSR pipeline operations are subclasses.

```
Class: fbo:FNSRProcess
  SubClassOf: CCO:ActOfInformationProcessing
  SubClassOf: BFO:has_participant some IAO:information_content_entity
```

### 6.2 fbo:NormalizationProcess

**Parent:** fbo:FNSRProcess
**Definition:** A planned process in which a raw dataset is transformed into a normalized dataset by applying format-level cleaning rules (whitespace trimming, empty-to-null conversion, currency stripping, date format standardization). The process produces both a cleaned dataset and a manifest recording every transformation.
**Grounds:** SNP `normalize()` invocation.

```
Class: fbo:NormalizationProcess
  SubClassOf: fbo:FNSRProcess
  SubClassOf: BFO:has_input some fbo:RawDataset
  SubClassOf: BFO:has_output some fbo:NormalizedDataset
  SubClassOf: BFO:has_output some fbo:NormalizationManifest
```

**Acceptance Criteria:**
- [ ] Every fbo:NormalizationProcess has exactly one input (fbo:RawDataset)
- [ ] Every fbo:NormalizationProcess has exactly two outputs (NormalizedDataset + NormalizationManifest)
- [ ] Classified as CCO:ActOfInformationProcessing by reasoner

### 6.3 fbo:StructuralInferenceProcess

**Parent:** fbo:FNSRProcess
**Definition:** A planned process in which a dataset (raw or normalized) is analyzed to produce a structural schema describing the observed types, nesting, optionality, and frequency of fields. No semantic interpretation is performed.
**Grounds:** BIBSS `infer()` invocation.

```
Class: fbo:StructuralInferenceProcess
  SubClassOf: fbo:FNSRProcess
  SubClassOf: BFO:has_input some (fbo:RawDataset or fbo:NormalizedDataset)
  SubClassOf: BFO:has_output some fbo:StructuralSchema
```

**Acceptance Criteria:**
- [ ] Every fbo:StructuralInferenceProcess produces exactly one fbo:StructuralSchema
- [ ] The output fbo:StructuralSchema is about the same dataset that was the input

### 6.4 fbo:SemanticAlignmentProcess

**Parent:** fbo:FNSRProcess
**Definition:** A planned process in which a structural schema is transformed into a semantic schema by applying rule cascades (consensus promotion, temporal detection, boolean pair detection, null vocabulary reclassification) governed by an alignment configuration. The process produces evidenced type assertions for each field.
**Grounds:** SAS `align()` invocation.

```
Class: fbo:SemanticAlignmentProcess
  SubClassOf: fbo:FNSRProcess
  SubClassOf: BFO:has_input some fbo:StructuralSchema
  SubClassOf: BFO:has_input some fbo:AlignmentConfiguration
  SubClassOf: BFO:has_output some fbo:SemanticSchema
  SubClassOf: fbo:optionallyConsults some fbo:NormalizationManifest
```

**Acceptance Criteria:**
- [ ] Every fbo:SemanticAlignmentProcess has at least two inputs (StructuralSchema + AlignmentConfiguration)
- [ ] Every fbo:SemanticAlignmentProcess produces exactly one fbo:SemanticSchema
- [ ] NormalizationManifest is optional input (standalone mode has no manifest)

### 6.5 fbo:ConsensusComputation

**Parent:** CCO:ActOfMeasurement
**Definition:** A planned process in which the type distribution of a data field is evaluated against a consensus threshold to determine whether a dominant type exists with sufficient confidence. The process produces a consensus score and, if threshold is met, a type assignment.
**Grounds:** SAS consensus promotion (§6.1).

```
Class: fbo:ConsensusComputation
  SubClassOf: CCO:ActOfMeasurement
  SubClassOf: BFO:has_input some fbo:TypeDistribution
  SubClassOf: BFO:has_output some fbo:EvidencedTypeAssertion
  SubClassOf: fbo:governedBy some fbo:AlignmentConfiguration
```

### 6.6 fbo:ConceptResolutionProcess

**Parent:** fbo:FNSRProcess
**Definition:** A planned process in which a field name is resolved against a concept knowledge base to find semantic enrichment information (type overrides, metric hints, epistemic annotations). This process is mediated by the Fandaws service.
**Grounds:** SAS Fandaws enrichment (§6.7, future Phase 1v2).

```
Class: fbo:ConceptResolutionProcess
  SubClassOf: fbo:FNSRProcess
  SubClassOf: BFO:has_input some fbo:DataField
  SubClassOf: BFO:has_output some fbo:EvidencedTypeAssertion
```

---

## 7. FBO Property Definitions

### 7.1 Object Properties

| Property | Domain | Range | Characteristics | Grounds |
|---|---|---|---|---|
| `fbo:derivedFrom` | fbo:InformationEntity | fbo:InformationEntity | Transitive | Provenance chain: NormalizedDataset derivedFrom RawDataset; SemanticSchema derivedFrom StructuralSchema |
| `fbo:correspondsTo` | fbo:DataField | fbo:DataField | — | Links a semantic-phase DataField to its structural-phase counterpart. Establishes cross-phase field identity via graph structure rather than string matching on fbo:hasFieldName. Enables traversal from a semantic assertion back to the structural TypeDistribution. |
| `fbo:isOutputOf` | IAO:ICE | fbo:FNSRProcess | Functional | Links output artifact to producing process |
| `fbo:isInputTo` | IAO:ICE | fbo:FNSRProcess | — | Inverse of has_input participant |
| `fbo:assertsVariableType` | fbo:EvidencedTypeAssertion | fbo:VariableType | Functional | Each assertion assigns exactly one variable type |
| `fbo:hasEvidenceSource` | fbo:EvidencedTypeAssertion | fbo:TypeDistribution | Functional | Links assertion to its evidential basis |
| `fbo:governedBy` | fbo:FNSRProcess | fbo:AlignmentConfiguration | Functional | Links process to its governing configuration |
| `fbo:optionallyConsults` | fbo:SemanticAlignmentProcess | fbo:NormalizationManifest | — | Optional provenance input |
| `fbo:hasFormat` | fbo:RawDataset | fbo:DataFormat | Functional | CSV or JSON |
| `fbo:hasField` | fbo:StructuralSchema or fbo:SemanticSchema | fbo:DataField | — | Links schema to its constituent fields |

### 7.2 Data Properties

| Property | Domain | Range | Grounds |
|---|---|---|---|
| `fbo:hasFieldName` | fbo:DataField | xsd:string | `viz:fieldName` |
| `fbo:hasEvidenceStrength` | fbo:EvidencedTypeAssertion | xsd:decimal [0.0, 1.0] | `viz:consensusScore` |
| `fbo:hasConsensusNumerator` | fbo:EvidencedTypeAssertion | xsd:nonNegativeInteger | `sas:consensusNumerator` |
| `fbo:hasConsensusDenominator` | fbo:EvidencedTypeAssertion | xsd:nonNegativeInteger | `sas:consensusDenominator` |
| `fbo:hasTotalObservations` | fbo:TypeDistribution | xsd:nonNegativeInteger | `occurrences` |
| `fbo:hasSchemaVersion` | fbo:StructuralSchema | xsd:string | CISM version |
| `fbo:hasRawInputHash` | fbo:SemanticSchema | xsd:string | `viz:rawInputHash` |
| `fbo:producedByRule` | fbo:EvidencedTypeAssertion | xsd:string | `sas:alignmentRule` |
| `fbo:hasStructuralType` | fbo:EvidencedTypeAssertion | xsd:string | `sas:structuralType` |
| `fbo:hasConsensusThreshold` | fbo:AlignmentConfiguration | xsd:decimal | SASConfig.consensusThreshold |
| `fbo:hasMinObservationThreshold` | fbo:AlignmentConfiguration | xsd:nonNegativeInteger | SASConfig.minObservationThreshold |
| `fbo:hasTotalRows` | fbo:SemanticSchema | xsd:nonNegativeInteger | `viz:totalRows` |
| `fbo:hasRowsInspected` | fbo:SemanticSchema | xsd:nonNegativeInteger | `viz:rowsInspected` |

---

## 8. FBO Axioms and Constraints

### 8.1 Disjointness Axioms

```
DisjointClasses: fbo:RawDataset, fbo:NormalizedDataset, fbo:StructuralSchema, fbo:SemanticSchema
DisjointClasses: fbo:QuantitativeVariable, fbo:NominalVariable, fbo:TemporalVariable,
                 fbo:BooleanVariable, fbo:IndeterminateVariable
DisjointClasses: fbo:NormalizationProcess, fbo:StructuralInferenceProcess,
                 fbo:SemanticAlignmentProcess
DisjointClasses: fbo:CSV, fbo:JSON
```

**Note on variable type disjointness (v1.1 fix):** The 5-way disjointness axiom is logically valid because all five variable type classes are direct children of `fbo:VariableType` — none is a subclass of another. In v1.0, `fbo:TemporalVariable` was declared as `SubClassOf: fbo:QuantitativeVariable` while also being disjoint with it, which made TemporalVariable unsatisfiable (equivalent to `owl:Nothing`). The fix in v1.1 moves TemporalVariable to be a sibling of QuantitativeVariable, eliminating the contradiction. See changelog F-001.

### 8.2 Cardinality Constraints

```
fbo:EvidencedTypeAssertion SubClassOf: fbo:assertsVariableType exactly 1 fbo:VariableType
fbo:EvidencedTypeAssertion SubClassOf: fbo:hasEvidenceStrength exactly 1 xsd:decimal
fbo:EvidencedTypeAssertion SubClassOf: fbo:hasEvidenceSource exactly 1 fbo:TypeDistribution
fbo:NormalizationProcess SubClassOf: BFO:has_input exactly 1 fbo:RawDataset
fbo:NormalizationProcess SubClassOf: BFO:has_output exactly 1 fbo:NormalizedDataset
fbo:NormalizationProcess SubClassOf: BFO:has_output exactly 1 fbo:NormalizationManifest
fbo:StructuralInferenceProcess SubClassOf: BFO:has_output exactly 1 fbo:StructuralSchema
fbo:SemanticAlignmentProcess SubClassOf: BFO:has_output exactly 1 fbo:SemanticSchema
fbo:AlignmentConfiguration SubClassOf: fbo:hasConsensusThreshold exactly 1 xsd:decimal
```

**v1.1 note on `fbo:correspondsTo` cardinality:** A semantic-phase DataField has at most one `correspondsTo` link (it traces back to exactly one structural-phase field). However, this is NOT modeled as a class-level cardinality restriction in OWL because not all DataField individuals are semantic-phase — structural-phase DataField individuals have no `correspondsTo` link. The constraint is enforced at the instance level via CQ-13 (every semantic-phase field must have a link) and is validated procedurally, not axiomatically.

### 8.3 Value Constraints

```
fbo:hasEvidenceStrength Range: xsd:decimal[>= 0.0, <= 1.0]
fbo:hasConsensusThreshold Range: xsd:decimal[> 0.0, <= 1.0]
fbo:hasMinObservationThreshold Range: xsd:nonNegativeInteger[>= 0]
```

### 8.4 Provenance Chain Axiom

The full FNSR pipeline forms a provenance chain. FBO formalizes this:

```
fbo:SemanticSchema SubClassOf:
  fbo:derivedFrom some (fbo:StructuralSchema
    and (fbo:derivedFrom some (fbo:NormalizedDataset
      and (fbo:derivedFrom some fbo:RawDataset))))
```

This axiom states: every semantic schema is derived from a structural schema, which is derived from a normalized dataset, which is derived from a raw dataset. The transitivity of `fbo:derivedFrom` enables the inference: the semantic schema is (transitively) derived from the raw dataset.

**Acceptance Criteria:**
- [ ] Given individuals `raw`, `norm`, `cism`, `schema` with the appropriate `derivedFrom` links, reasoner infers `schema fbo:derivedFrom raw` via transitivity
- [ ] Breaking any link in the chain (e.g., omitting `cism fbo:derivedFrom norm`) prevents the transitive inference

---

## 9. Pipeline Provenance Model

### 9.1 The Full Chain

```
fbo:RawDataset ──[input to]──→ fbo:NormalizationProcess
                                  │
                         ┌────────┴─────────┐
                         ▼                   ▼
              fbo:NormalizedDataset    fbo:NormalizationManifest
                         │                   │
                   [input to]          [optionally consults]
                         ▼                   │
              fbo:StructuralInferenceProcess  │
                         │                   │
                         ▼                   │
              fbo:StructuralSchema           │
                         │                   │
                   [input to]                │
                         ▼                   │
              fbo:SemanticAlignmentProcess ◄─┘
                         │
                         ▼
              fbo:SemanticSchema
                   │
                   ├── fbo:EvidencedTypeAssertion (field 1)
                   ├── fbo:EvidencedTypeAssertion (field 2)
                   └── fbo:EvidencedTypeAssertion (field N)
```

### 9.2 Standalone Mode (No SNP)

When SAS operates without SNP preprocessing (standalone mode), the provenance chain is shorter:

```
fbo:RawDataset ──[input to]──→ fbo:StructuralInferenceProcess
                                  │
                                  ▼
                       fbo:StructuralSchema
                                  │
                            [input to]
                                  ▼
                       fbo:SemanticAlignmentProcess (no manifest)
                                  │
                                  ▼
                       fbo:SemanticSchema
```

FBO handles this by making `fbo:optionallyConsults` non-mandatory. The SemanticAlignmentProcess can operate with or without a NormalizationManifest. The provenance chain axiom (§8.4) requires the full chain; standalone mode would use a relaxed version where the StructuralSchema is derived directly from a RawDataset (no intermediate NormalizedDataset).

---

## 10. Namespace Alignment Table

This table maps every namespace term currently used in FNSR pipeline output to its FBO grounding.

### 10.1 viz: Namespace

| Current Term | Used In | FBO Class/Property | Notes |
|---|---|---|---|
| `viz:DatasetSchema` | SAS output | `fbo:SemanticSchema` | |
| `viz:DataField` | SAS output | `fbo:DataField` + `fbo:EvidencedTypeAssertion` | SAS conflates field and assertion; FBO separates them |
| `viz:fieldName` | SAS output | `fbo:hasFieldName` | |
| `viz:hasDataType` | SAS output | `fbo:assertsVariableType` | |
| `viz:QuantitativeType` | SAS output | `fbo:QuantitativeVariable` | STATO:0000251 subclass |
| `viz:NominalType` | SAS output | `fbo:NominalVariable` | STATO:0000252 equivalent |
| `viz:TemporalType` | SAS output | `fbo:TemporalVariable` | FBO extension to STATO |
| `viz:BooleanType` | SAS output | `fbo:BooleanVariable` | STATO:0000255 equivalent |
| `viz:UnknownType` | SAS output | `fbo:IndeterminateVariable` | FBO-defined |
| `viz:consensusScore` | SAS output | `fbo:hasEvidenceStrength` | |
| `viz:rawInputHash` | SAS output | `fbo:hasRawInputHash` | |
| `viz:totalRows` | SAS output | `fbo:hasTotalRows` | |
| `viz:rowsInspected` | SAS output | `fbo:hasRowsInspected` | |
| `viz:hasField` | SAS output | `fbo:hasField` | |
| `viz:numericPrecision` | SAS output | `fbo:hasNumericPrecision` (data property, range `{"integer", "float"}`) | |
| `viz:wasNormalized` | SAS output | `fbo:derivedFrom some fbo:NormalizationProcess` | FBO replaces boolean flag with ontological provenance |
| `viz:wasPercentage` | SAS output | `fbo:hasNormalizationDetail` (annotation on manifest entry) | |

### 10.2 sas: Namespace

| Current Term | Used In | FBO Class/Property | Notes |
|---|---|---|---|
| `sas:consensusNumerator` | SAS output | `fbo:hasConsensusNumerator` | |
| `sas:consensusDenominator` | SAS output | `fbo:hasConsensusDenominator` | |
| `sas:alignmentRule` | SAS output | `fbo:producedByRule` | |
| `sas:structuralType` | SAS output | `fbo:hasStructuralType` | |
| `sas:fandawsAvailable` | SAS output | (annotation on fbo:SemanticAlignmentProcess) | |
| `sas:alignmentMode` | SAS output | (annotation on fbo:SemanticAlignmentProcess) | |
| `sas:fandawsConsulted` | SAS output | (annotation on fbo:EvidencedTypeAssertion) | |

### 10.3 Terms with No Direct FBO Mapping

| Current Term | Reason | Resolution |
|---|---|---|
| `@context` | JSON-LD serialization artifact, not ontological | No FBO class. Context is a serialization concern. |
| `@type` | JSON-LD serialization artifact | Maps to `rdf:type` in RDF — no FBO intervention needed. |
| `@id` | JSON-LD serialization artifact | Maps to the IRI of the individual. |
| `$schema`, `$id`, `$comment` | JSON Schema metadata | Not ontological. Adapter output only. |

---

## 11. Worked Example — Full Pipeline

This section traces a concrete dataset through the entire FNSR pipeline and represents each stage as FBO individuals.

### 11.1 Input

A 4-row CSV file:

```csv
name,revenue,created_at
Alice,1234.56,2026-03-04
Bob,5678.90,2026-03-05
Charlie,,2026-03-06
Diana,9999.99,
```

### 11.2 SNP Output

SNP produces (assuming no format issues in this clean dataset):
- Cleaned CSV (identical to input — nothing to clean)
- Manifest: 2 entries (empty-to-null for `revenue` row 3 and `created_at` row 4)

**FBO Individuals:**

```turtle
:raw_dataset_1 a fbo:RawDataset ;
  fbo:hasFormat fbo:CSV .

:snp_process_1 a fbo:NormalizationProcess ;
  BFO:has_input :raw_dataset_1 ;
  BFO:has_output :normalized_dataset_1 ;
  BFO:has_output :manifest_1 .

:normalized_dataset_1 a fbo:NormalizedDataset ;
  fbo:derivedFrom :raw_dataset_1 .

:manifest_1 a fbo:NormalizationManifest ;
  IAO:is_about :snp_process_1 .

:manifest_entry_1a a fbo:ManifestEntry ;
  fbo:hasRule "empty-to-null" ;
  fbo:hasDataPath "revenue" ;
  IAO:is_part_of :manifest_1 .

:manifest_entry_1b a fbo:ManifestEntry ;
  fbo:hasRule "empty-to-null" ;
  fbo:hasDataPath "created_at" ;
  IAO:is_part_of :manifest_1 .
```

### 11.3 BIBSS Output

BIBSS infers CISM from the cleaned CSV. Post-narrowing types:

| Field | Row 1 | Row 2 | Row 3 | Row 4 | typeDistribution | Widened |
|---|---|---|---|---|---|---|
| name | string | string | string | string | `{ "string": 4 }` | string |
| revenue | number | number | null | number | `{ "null": 1, "number": 3 }` | number |
| created_at | string | string | string | null | `{ "null": 1, "string": 3 }` | string |

**FBO Individuals:**

```turtle
:bibss_process_1 a fbo:StructuralInferenceProcess ;
  BFO:has_input :normalized_dataset_1 ;
  BFO:has_output :cism_1 .

:cism_1 a fbo:StructuralSchema ;
  fbo:derivedFrom :normalized_dataset_1 ;
  fbo:hasSchemaVersion "1.3" ;
  fbo:hasField :field_name_structural, :field_revenue_structural, :field_created_structural .

:field_name_structural a fbo:DataField ;
  fbo:hasFieldName "name" ;
  IAO:is_part_of :cism_1 .

:field_revenue_structural a fbo:DataField ;
  fbo:hasFieldName "revenue" ;
  IAO:is_part_of :cism_1 .

:field_created_structural a fbo:DataField ;
  fbo:hasFieldName "created_at" ;
  IAO:is_part_of :cism_1 .

:typedist_name a fbo:TypeDistribution ;
  IAO:is_about :field_name_structural ;
  fbo:hasTotalObservations 4 .
  # Distribution: { "string": 4 }. Sum = 4 = hasTotalObservations. ✓

:typedist_revenue a fbo:TypeDistribution ;
  IAO:is_about :field_revenue_structural ;
  fbo:hasTotalObservations 4 .
  # Distribution: { "null": 1, "number": 3 }. Sum = 4 = hasTotalObservations. ✓
  # Note: null counts ARE included in the sum. See §16.6.

:typedist_created a fbo:TypeDistribution ;
  IAO:is_about :field_created_structural ;
  fbo:hasTotalObservations 4 .
  # Distribution: { "null": 1, "string": 3 }. Sum = 4 = hasTotalObservations. ✓
```

### 11.4 SAS Output

SAS aligns with default config (threshold 0.95):

| Field | typeDistribution | nonNullTotal | Consensus | Assigned Type | Rule |
|---|---|---|---|---|---|
| name | `{ "string": 4 }` | 4 | 1.000000 | NominalType → no temporal match → **NominalType** | consensus-promotion |
| revenue | `{ "null": 1, "number": 3 }` | 3 | 1.000000 | **QuantitativeType** | consensus-promotion |
| created_at | `{ "null": 1, "string": 3 }` | 3 | 1.000000 | NominalType → temporal name match → **TemporalType** | temporal-detection |

**FBO Individuals:**

```turtle
:sas_process_1 a fbo:SemanticAlignmentProcess ;
  BFO:has_input :cism_1 ;
  BFO:has_input :sas_config_default ;
  fbo:optionallyConsults :manifest_1 ;
  BFO:has_output :schema_1 .

:sas_config_default a fbo:AlignmentConfiguration ;
  fbo:hasConsensusThreshold 0.95 ;
  fbo:hasMinObservationThreshold 5 .

:schema_1 a fbo:SemanticSchema ;
  fbo:derivedFrom :cism_1 ;
  fbo:hasRawInputHash "abc123..." ;
  fbo:hasTotalRows 4 ;
  fbo:hasField :field_name_semantic, :field_revenue_semantic, :field_created_semantic .

# Semantic-phase fields with correspondsTo links to structural-phase fields
:field_name_semantic a fbo:DataField ;
  fbo:hasFieldName "name" ;
  fbo:correspondsTo :field_name_structural ;
  IAO:is_part_of :schema_1 .

:field_revenue_semantic a fbo:DataField ;
  fbo:hasFieldName "revenue" ;
  fbo:correspondsTo :field_revenue_structural ;
  IAO:is_part_of :schema_1 .

:field_created_semantic a fbo:DataField ;
  fbo:hasFieldName "created_at" ;
  fbo:correspondsTo :field_created_structural ;
  IAO:is_part_of :schema_1 .

:assertion_name a fbo:EvidencedTypeAssertion ;
  IAO:is_about :field_name_semantic ;
  fbo:assertsVariableType fbo:NominalVariable ;
  fbo:hasEvidenceStrength 1.000000 ;
  fbo:hasConsensusNumerator 4 ;
  fbo:hasConsensusDenominator 4 ;
  fbo:hasEvidenceSource :typedist_name ;
  fbo:producedByRule "consensus-promotion" ;
  fbo:hasStructuralType "string" .

:assertion_revenue a fbo:EvidencedTypeAssertion ;
  IAO:is_about :field_revenue_semantic ;
  fbo:assertsVariableType fbo:QuantitativeVariable ;
  fbo:hasEvidenceStrength 1.000000 ;
  fbo:hasConsensusNumerator 3 ;
  fbo:hasConsensusDenominator 3 ;
  fbo:hasEvidenceSource :typedist_revenue ;
  fbo:producedByRule "consensus-promotion" ;
  fbo:hasStructuralType "number" .

:assertion_created a fbo:EvidencedTypeAssertion ;
  IAO:is_about :field_created_semantic ;
  fbo:assertsVariableType fbo:TemporalVariable ;
  fbo:hasEvidenceStrength 1.000000 ;
  fbo:hasConsensusNumerator 3 ;
  fbo:hasConsensusDenominator 3 ;
  fbo:hasEvidenceSource :typedist_created ;
  fbo:producedByRule "temporal-detection" ;
  fbo:hasStructuralType "string" .
```

**Note on `fbo:correspondsTo` (v1.1):** Each semantic-phase DataField is linked to its structural-phase counterpart via `fbo:correspondsTo`. This enables graph traversal from any EvidencedTypeAssertion back to the originating TypeDistribution without relying on string matching:

```
:assertion_revenue → IAO:is_about → :field_revenue_semantic
  → fbo:correspondsTo → :field_revenue_structural
    ← IAO:is_about ← :typedist_revenue
```

The `fbo:hasStructuralType` data property on the assertion is retained as a denormalized convenience for flat queries, but the authoritative structural provenance path is through `correspondsTo`.

### 11.5 Provenance Chain Verification

With the individuals above, the reasoner should infer:

```sparql
# Transitive derivation
:schema_1 fbo:derivedFrom :cism_1 .        # direct
:schema_1 fbo:derivedFrom :normalized_dataset_1 .  # transitive (via cism_1)
:schema_1 fbo:derivedFrom :raw_dataset_1 .  # transitive (via normalized_dataset_1)
```

---

## 12. Competency Questions

These SPARQL queries define what FBO must be able to answer. Each query is an acceptance test. FBO is not complete until all queries return correct results against the worked example (Section 11) and at least two additional test datasets.

### CQ-01: What variable type was assigned to a field?

```sparql
SELECT ?field ?type WHERE {
  ?assertion a fbo:EvidencedTypeAssertion ;
    IAO:is_about ?field ;
    fbo:assertsVariableType ?type .
  ?field fbo:hasFieldName "revenue" .
}
# Expected: ?type = fbo:QuantitativeVariable
```

**Acceptance:** Returns exactly one result with the correct variable type.

### CQ-02: What is the confidence of a type assertion?

```sparql
SELECT ?field ?score WHERE {
  ?assertion a fbo:EvidencedTypeAssertion ;
    IAO:is_about ?field ;
    fbo:hasEvidenceStrength ?score .
  ?field fbo:hasFieldName "created_at" .
}
# Expected: ?score = 1.000000
```

**Acceptance:** Returns exactly one decimal value in [0.0, 1.0].

### CQ-03: What rule produced a type assertion?

```sparql
SELECT ?field ?rule WHERE {
  ?assertion a fbo:EvidencedTypeAssertion ;
    IAO:is_about ?field ;
    fbo:producedByRule ?rule .
  ?field fbo:hasFieldName "created_at" .
}
# Expected: ?rule = "temporal-detection"
```

**Acceptance:** Returns exactly one rule string.

### CQ-04: Trace provenance from semantic schema to raw dataset

```sparql
SELECT ?raw WHERE {
  :schema_1 fbo:derivedFrom+ ?raw .
  ?raw a fbo:RawDataset .
}
# Expected: ?raw = :raw_dataset_1
```

**Acceptance:** Returns exactly one raw dataset. Property path `+` traverses transitive derivedFrom.

### CQ-05: Which fields were classified as a specific STATO variable type?

```sparql
SELECT ?fieldName WHERE {
  ?assertion fbo:assertsVariableType ?type .
  ?type rdfs:subClassOf* STATO:0000251 .  # continuous variable
  ?assertion IAO:is_about ?field .
  ?field fbo:hasFieldName ?fieldName .
}
# Expected: "revenue" (QuantitativeVariable ⊑ STATO continuous)
# NOTE (v1.1): "created_at" is NO LONGER returned here. TemporalVariable is a direct
# child of fbo:VariableType, not a subclass of QuantitativeVariable, so it is not
# a STATO continuous variable. This is correct: temporal data may be continuous
# (timestamps) or ordinal categorical (fiscal quarters), and FBO does not force
# the mathematical commitment.
```

**Acceptance:** Returns only "revenue". Does NOT return "created_at".

### CQ-05b: Which fields were classified as Temporal? (v1.1 addition)

```sparql
SELECT ?fieldName WHERE {
  ?assertion fbo:assertsVariableType fbo:TemporalVariable .
  ?assertion IAO:is_about ?field .
  ?field fbo:hasFieldName ?fieldName .
}
# Expected: "created_at"
```

**Acceptance:** Returns "created_at". This query is necessary because TemporalVariable is not reachable via STATO subsumption — it must be queried directly.

### CQ-06: Which processes contributed to producing a semantic schema?

```sparql
SELECT ?process ?processType WHERE {
  { :schema_1 fbo:isOutputOf ?process . }
  UNION
  { :schema_1 fbo:derivedFrom ?intermediate .
    ?intermediate fbo:isOutputOf ?process . }
  ?process a ?processType .
  ?processType rdfs:subClassOf* fbo:FNSRProcess .
}
# Expected: sas_process_1 (SemanticAlignment), bibss_process_1 (StructuralInference),
#           snp_process_1 (Normalization)
```

**Acceptance:** Returns all three pipeline processes.

### CQ-07: What structural type did BIBSS observe before SAS applied semantic judgment?

```sparql
SELECT ?fieldName ?structuralType ?semanticType WHERE {
  ?assertion a fbo:EvidencedTypeAssertion ;
    IAO:is_about ?semField ;
    fbo:assertsVariableType ?semanticType .
  ?semField fbo:hasFieldName ?fieldName ;
    fbo:correspondsTo ?strField .
  ?typeDist IAO:is_about ?strField .
  # Structural type is available via the denormalized shortcut:
  ?assertion fbo:hasStructuralType ?structuralType .
}
# Expected:
#   "name",       "string",  fbo:NominalVariable
#   "revenue",    "number",  fbo:QuantitativeVariable
#   "created_at", "string",  fbo:TemporalVariable
```

**Acceptance:** Returns three rows showing the structural-to-semantic promotion for each field. The query uses `fbo:correspondsTo` to verify cross-phase field identity via graph traversal, not string matching.

### CQ-07b: Traverse from semantic assertion to structural TypeDistribution via graph (v1.1 addition)

```sparql
SELECT ?fieldName ?totalObs WHERE {
  ?assertion a fbo:EvidencedTypeAssertion ;
    IAO:is_about ?semField .
  ?semField fbo:hasFieldName ?fieldName ;
    fbo:correspondsTo ?strField .
  ?typeDist a fbo:TypeDistribution ;
    IAO:is_about ?strField ;
    fbo:hasTotalObservations ?totalObs .
}
# Expected:
#   "name",       4
#   "revenue",    4
#   "created_at", 4
```

**Acceptance:** Returns the total observation count from the structural TypeDistribution for each field, reached entirely through object property traversal (no string matching). This validates that `fbo:correspondsTo` correctly bridges the semantic/structural phases.

### CQ-08: Which fields have type assertions with evidence below a given threshold?

```sparql
SELECT ?fieldName ?score WHERE {
  ?assertion a fbo:EvidencedTypeAssertion ;
    IAO:is_about ?field ;
    fbo:hasEvidenceStrength ?score .
  ?field fbo:hasFieldName ?fieldName .
  FILTER (?score < 0.95)
}
# Expected: empty (all fields in the example have consensus 1.0)
```

**Acceptance:** Returns only fields below the specified threshold. Test with a dataset containing mixed-type fields.

### CQ-09: Which normalization transformations affected a given field?

```sparql
SELECT ?rule ?path WHERE {
  ?entry a fbo:ManifestEntry ;
    fbo:hasRule ?rule ;
    fbo:hasDataPath ?path ;
    IAO:is_part_of ?manifest .
  ?manifest a fbo:NormalizationManifest .
  FILTER (?path = "revenue")
}
# Expected: "empty-to-null", "revenue"
```

**Acceptance:** Returns all manifest entries for the specified field.

### CQ-10: Are all variable type assignments mutually exclusive?

```sparql
ASK {
  ?assertion1 a fbo:EvidencedTypeAssertion ;
    IAO:is_about ?field ;
    fbo:assertsVariableType ?type1 .
  ?assertion2 a fbo:EvidencedTypeAssertion ;
    IAO:is_about ?field ;
    fbo:assertsVariableType ?type2 .
  FILTER (?assertion1 != ?assertion2)
}
# Expected: false (no field has two type assertions)
```

**Acceptance:** Returns false. If it returns true, there is a data integrity violation.

### CQ-11: Is the evidence strength consistent with numerator/denominator?

```sparql
ASK {
  ?assertion a fbo:EvidencedTypeAssertion ;
    fbo:hasEvidenceStrength ?score ;
    fbo:hasConsensusNumerator ?num ;
    fbo:hasConsensusDenominator ?denom .
  FILTER (?denom > 0 && abs(?score - ?num/?denom) > 0.000001)
}
# Expected: false (score always equals num/denom within precision)
```

**Acceptance:** Returns false.

### CQ-12: Which fields were assigned IndeterminateVariable and why?

```sparql
SELECT ?fieldName ?rule WHERE {
  ?assertion a fbo:EvidencedTypeAssertion ;
    IAO:is_about ?field ;
    fbo:assertsVariableType fbo:IndeterminateVariable ;
    fbo:producedByRule ?rule .
  ?field fbo:hasFieldName ?fieldName .
}
# Expected: empty for the simple example; test with all-null or low-observation fields
```

**Acceptance:** Returns fields with their assignment rules (e.g., "unknown-assignment" for all-null, or the diagnostic code for insufficient observations).

### CQ-13: Are all semantic-phase fields linked to their structural counterparts? (v1.1 addition)

```sparql
ASK {
  ?semField a fbo:DataField ;
    IAO:is_part_of ?schema .
  ?schema a fbo:SemanticSchema .
  FILTER NOT EXISTS { ?semField fbo:correspondsTo ?strField }
}
# Expected: false (every semantic-phase field has a correspondsTo link)
```

**Acceptance:** Returns false. Every DataField that is part of a SemanticSchema must have a `fbo:correspondsTo` link to a DataField that is part of a StructuralSchema. If true, a semantic field was created without tracing it back to its structural origin — a provenance integrity violation.

### CQ-14: Does correspondsTo preserve field name consistency? (v1.1 addition)

```sparql
ASK {
  ?semField fbo:correspondsTo ?strField ;
    fbo:hasFieldName ?semName .
  ?strField fbo:hasFieldName ?strName .
  FILTER (?semName != ?strName)
}
# Expected: false (correspondsTo-linked fields must share the same field name)
```

**Acceptance:** Returns false. This is a consistency check: if two fields are linked by `correspondsTo`, their `hasFieldName` values must match. A mismatch indicates a wiring error in the SAS → BIBSS field mapping.

---

## 13. Test Coverage Requirements

### 13.1 Ontology-Level Tests

| Test | Tool | Acceptance |
|---|---|---|
| OWL 2 DL consistency | HermiT / Pellet | No unsatisfiable classes |
| Import resolution | Protégé | All 5 imports (BFO, IAO, CCO-IE, CCO-Measurement, STATO) resolve |
| Class satisfiability | HermiT | Every FBO class is satisfiable (has at least one possible instance) |
| Disjointness verification | HermiT | No individual can be classified under two disjoint classes |
| Property domain/range | SPARQL | No property usage violates declared domain/range |

### 13.2 Competency Question Tests

All 14 competency questions (Section 12, including v1.1 additions CQ-05b, CQ-07b, CQ-13, CQ-14) must return correct results against:

1. The worked example (Section 11) — 3-field CSV with clean data
2. A mixed-type dataset — CSV with integer/string/null mix that triggers consensus below threshold
3. A standalone-mode dataset — no SNP manifest, direct BIBSS → SAS
4. A Fandaws-enriched dataset (when Phase 1v2 is complete) — concept resolution overrides consensus

### 13.3 Structural Invariant Tests

| Invariant | SPARQL Check |
|---|---|
| Every EvidencedTypeAssertion has exactly 1 assertsVariableType | `ASK { ?a a fbo:ETA . FILTER NOT EXISTS { ?a fbo:assertsVariableType ?t } }` → false |
| Every EvidencedTypeAssertion has exactly 1 hasEvidenceStrength | same pattern |
| Every EvidencedTypeAssertion has exactly 1 hasEvidenceSource | same pattern |
| No SemanticSchema exists without at least 1 EvidencedTypeAssertion | `ASK { ?s a fbo:SemanticSchema . FILTER NOT EXISTS { ?s fbo:hasField ?f } }` → false |
| Every SemanticSchema derivedFrom exactly 1 StructuralSchema | cardinality check |
| Every NormalizationProcess has exactly 1 RawDataset input | cardinality check |
| Every NormalizationProcess has exactly 2 outputs (dataset + manifest) | count check |
| Evidence strength in [0.0, 1.0] | `ASK { ?a fbo:hasEvidenceStrength ?s . FILTER (?s < 0 \|\| ?s > 1) }` → false |
| Numerator ≤ denominator | `ASK { ?a fbo:hasConsensusNumerator ?n ; fbo:hasConsensusDenominator ?d . FILTER (?n > ?d) }` → false |
| Every semantic-phase DataField has correspondsTo link (v1.1) | CQ-13 — `ASK { ?f a fbo:DataField ; IAO:is_part_of ?s . ?s a fbo:SemanticSchema . FILTER NOT EXISTS { ?f fbo:correspondsTo ?sf } }` → false |
| correspondsTo-linked fields share field name (v1.1) | CQ-14 — `ASK { ?sf fbo:correspondsTo ?stf ; fbo:hasFieldName ?n1 . ?stf fbo:hasFieldName ?n2 . FILTER (?n1 != ?n2) }` → false |
| TypeDistribution total includes nulls (v1.1, §16.6) | `ASK { ?td a fbo:TypeDistribution ; fbo:hasTotalObservations ?total . FILTER (?total < 1) }` → false (and manual review that sum of all type counts in distribution equals hasTotalObservations) |

### 13.4 STATO Integration Tests

| Test | Expected | Notes |
|---|---|---|
| `fbo:QuantitativeVariable rdfs:subClassOf* STATO:0000251` | true | |
| `fbo:NominalVariable owl:equivalentClass STATO:0000252` | true | |
| `fbo:BooleanVariable owl:equivalentClass STATO:0000255` | true | |
| `fbo:BooleanVariable rdfs:subClassOf* STATO:0000252` | true (binary ⊑ categorical) | |
| `fbo:TemporalVariable rdfs:subClassOf* STATO:0000251` | **false** | v1.1 change: TemporalVariable is a direct child of fbo:VariableType, not under QuantitativeVariable. See F-001. |
| `fbo:TemporalVariable rdfs:subClassOf* STATO:0000252` | **false** | TemporalVariable is not categorical either. |
| `fbo:TemporalVariable rdfs:subClassOf fbo:VariableType` | **true** | v1.1 addition: direct parent is VariableType. |
| `fbo:IndeterminateVariable rdfs:subClassOf* STATO:0000252` | false (not categorical) | |
| `fbo:IndeterminateVariable rdfs:subClassOf* STATO:0000251` | false (not continuous) | |
| All 5 variable types pairwise disjoint | true | v1.1 critical test: HermiT must NOT flag any of the 5 as unsatisfiable. If TemporalVariable is unsatisfiable, the F-001 fix was not applied correctly. |

---

## 14. Implementation Phases

### Phase 1: Core Classes and STATO Alignment

**Scope:** Define all classes in Sections 5 and 6, all properties in Section 7, all axioms in Section 8. Establish STATO alignments (§4.1). Pass ontology-level tests (§13.1) and STATO integration tests (§13.4).

**Deliverables:**
- [ ] `fbo.owl` file with all class, property, and axiom definitions, including `fbo:correspondsTo` object property
- [ ] Import chain verified (BFO, IAO, CCO-IE, CCO-Measurement, STATO)
- [ ] HermiT consistency check passes — specifically verify all 5 variable type subclasses are satisfiable (not equivalent to owl:Nothing)
- [ ] All 5 variable type subclasses satisfiable and mutually disjoint (F-001 regression test)
- [ ] fbo:TemporalVariable is NOT classified under STATO:0000251 or STATO:0000252
- [ ] All STATO integration tests (§13.4) pass

### Phase 2: Worked Example and Competency Questions

**Scope:** Instantiate the worked example (Section 11) as OWL individuals. Implement and validate all 14 competency questions as SPARQL queries.

**Deliverables:**
- [ ] `fbo-example-pipeline.owl` with all individuals from Section 11, including `fbo:correspondsTo` links between semantic and structural DataField individuals
- [ ] SPARQL query file with all 14 competency questions (including v1.1 additions CQ-05b, CQ-07b, CQ-13, CQ-14)
- [ ] All 14 CQ tests pass against the worked example
- [ ] Provenance chain transitivity verified (CQ-04)
- [ ] STATO subsumption reasoning verified (CQ-05 returns "revenue" only; CQ-05b returns "created_at")
- [ ] Field-level cross-phase traversal verified (CQ-07b returns TypeDistribution data via `correspondsTo`)

### Phase 3: Additional Test Datasets and Hardening

**Scope:** Create 3 additional test datasets (§13.2 items 2–4). Run all competency questions against all 4 datasets. Run structural invariant tests (§13.3).

**Deliverables:**
- [ ] Mixed-type dataset individuals (consensus below threshold scenario)
- [ ] Standalone-mode dataset individuals (no SNP manifest)
- [ ] All CQ tests (14 total) pass against all datasets
- [ ] All structural invariant tests (§13.3, including v1.1 additions) pass
- [ ] No unsatisfiable classes across all test data

### Phase 4: Namespace Migration Plan

**Scope:** Document the migration path from current `viz:`/`sas:` namespace terms to FBO-grounded IRIs. This does not change the runtime output — it produces a mapping document and a recommended `@context` update for SAS v3.0.

**Deliverables:**
- [ ] Mapping document: current term → FBO IRI for every term in Section 10
- [ ] Draft `@context` for SAS that uses FBO IRIs instead of placeholder `viz:`/`sas:` terms
- [ ] Backward compatibility analysis: which consumers would break if IRIs change
- [ ] Recommendation: whether to do a hard migration or maintain `viz:` as aliases via `owl:equivalentProperty`

---

## 15. Non-Goals (v1.0)

- Formalizing the Fandaws concept hierarchy (that's Fandaws's own ontology, not FBO)
- Defining domain-specific variable types beyond the 5 FNSR types (e.g., ordinal, interval, ratio)
- Representing the internal data structures of SNP, BIBSS, or SAS (FBO describes inputs and outputs, not implementation)
- Runtime reasoning (FBO is a design-time ontology for grounding and validation, not a runtime inference engine)
- RDF serialization of FNSR pipeline output (the pipeline produces JSON-LD; FBO grounds the terms used in that JSON-LD)
- SHACL shape generation (deferred to v2.0 — shapes would validate that JSON-LD output conforms to FBO constraints)

---

## 16. Known Design Decisions

### 16.1 Why fbo:TemporalVariable Is a Sibling, Not a Child of Quantitative

**v1.0 design:** TemporalVariable was a subclass of QuantitativeVariable (STATO continuous) on the reasoning that ISO 8601 dates represent continuous points on a timeline.

**v1.0 bug:** This created a fatal OWL 2 DL contradiction. Section 8.1 declared all five variable types mutually disjoint: `DisjointClasses: QuantitativeVariable, NominalVariable, TemporalVariable, BooleanVariable, IndeterminateVariable`. In OWL, if B ⊑ A and B ∩ A = ∅, then B ≡ owl:Nothing (B can have no instances). HermiT would immediately flag TemporalVariable as unsatisfiable.

**v1.1 fix:** TemporalVariable is now a direct child of fbo:VariableType — a sibling of the other four types, not a child of any of them. The 5-way disjointness axiom is now logically valid because none of the five types is a subclass of another.

**Beyond the bug fix — the conceptual argument is also stronger this way.** Time is not always continuous. ISO 8601 timestamps (`2026-03-04T14:30:00Z`) are continuous: they represent precise points on a real-valued timeline, and arithmetic operations (duration, interval) are meaningful. But fiscal quarters (`2025-Q1`), year-month buckets (`2026-03`), and epoch labels (`"Renaissance"`) are temporal but ordinal-categorical: they represent ordered bins, not continuous quantities. The SAS pipeline's `TemporalType` applies to all of these — it signals "this field's values represent time" without committing to a mathematical scale. Placing TemporalVariable under QuantitativeVariable would force the reasoner to treat every temporal field as continuous, which misrepresents ordinal temporal data. Placing it under NominalVariable would deny that temporal data has intrinsic ordering, which misrepresents timestamps. The correct ontological position is: TemporalVariable is its own kind of variable type, orthogonal to the continuous/categorical distinction. Future FBO versions may introduce subclasses (e.g., `fbo:ContinuousTemporalVariable` under both TemporalVariable and QuantitativeVariable, `fbo:OrdinalTemporalVariable` under both TemporalVariable and NominalVariable) if the SAS engine gains the ability to distinguish them.

### 16.2 Why DataField and EvidencedTypeAssertion Are Separate

In SAS output, `viz:DataField` contains both the field identity (name, IRI) and the type assertion (hasDataType, consensusScore). FBO separates these because ontologically they are different kinds of things: the field is an information content entity (it *exists* as part of a dataset), while the type assertion is a claim *about* the field (it may be wrong, it has evidence, it was produced by a process). Conflating them makes it impossible to represent disagreements: two different SAS runs with different configurations could produce different type assertions about the same field.

### 16.3 Why IndeterminateVariable Is Not owl:Nothing

Unknown/Indeterminate is a positive assertion: "the evidence is insufficient to classify this field." It is not the absence of an assertion. Modeling it as owl:Nothing (no type) would make it invisible to queries — you couldn't ask "which fields were indeterminate?" FBO models it as a concrete subclass of VariableType, disjoint with all typed subclasses.

### 16.4 Why fbo:derivedFrom Is Transitive

The provenance chain (raw → normalized → structural → semantic) is the core audit trail. Making `derivedFrom` transitive allows CQ-04 ("trace from semantic schema to raw dataset") to work without enumerating every intermediate link. The cost is that transitive properties can cause reasoning performance issues in large ABoxes — but FNSR provenance chains are short (4 links maximum), so this is not a practical concern.

### 16.5 The Meta-ICE Relationship (v1.1 clarification)

In BFO/IAO, an Information Content Entity (ICE) is *about* something in reality. A dataset about border crossings is an ICE about border crossings. A schema is an ICE about a dataset — it describes the structure of data, not the world the data represents. This means `fbo:SemanticSchema IAO:is_about fbo:RawDataset` is an ICE-about-ICE relationship: information about information.

This is ontologically valid in IAO — nothing prohibits an ICE from being about another ICE. But it is worth documenting because downstream ontologists may assume `IAO:is_about` always targets a BFO:material entity or BFO:process in the physical world. In FBO, the `is_about` chain has two levels: the SemanticSchema is about the RawDataset, and the RawDataset is about some domain reality (trade transactions, sensor readings, etc.). FBO formalizes only the first level. The second level (what the raw data is about) is outside FBO's scope — it belongs to domain ontologies.

### 16.6 TypeDistribution Null Accounting (v1.1 clarification)

The `fbo:hasTotalObservations` value on a `fbo:TypeDistribution` must equal the sum of **all** type counts in the distribution, **including null**. In the worked example (§11.3), the `revenue` field has distribution `{ "null": 1, "number": 3 }`. The sum is 1 + 3 = 4, which must equal `hasTotalObservations`.

This matters because SAS computes consensus using the non-null denominator: `consensusNumerator / consensusDenominator` where `consensusDenominator` excludes null observations. If `hasTotalObservations` excluded nulls, it would equal `consensusDenominator`, making the two properties redundant and losing the ability to distinguish "3 out of 3 non-null values are numbers" from "3 out of 4 total values are numbers (one is null)." The null count is load-bearing information: it tells you how much data was missing, which is itself a data quality signal.

The structural invariant test in §13.3 verifies this: the sum of all distribution entries must equal `hasTotalObservations`. If it doesn't, either the BIBSS CISM was generated incorrectly or the FBO individual was instantiated incorrectly.

---

## 17. Glossary

| Term | Definition |
|---|---|
| **ABox** | Assertion box — the set of individuals (instances) in an ontology |
| **TBox** | Terminological box — the set of classes and axioms in an ontology |
| **ICE** | Information Content Entity — an IAO class for entities that are about something and can be copied |
| **GDC** | Generically Dependent Continuant — a BFO category for entities that depend on a bearer but can migrate between bearers |
| **MIREOT** | Minimum Information to Reference an External Ontology Term — a strategy for importing only needed terms |
| **CISM** | Canonical Internal Schema Model — BIBSS's output format |
| **FBO** | FNSR Bridge Ontology — this specification |
| **Meta-ICE** | An Information Content Entity whose `IAO:is_about` target is another ICE rather than a material entity or process. See §16.5. |
| **correspondsTo** | FBO object property linking a semantic-phase DataField to its structural-phase counterpart. Establishes cross-phase field identity via graph structure. See F-002. |

---

*End of specification.*
