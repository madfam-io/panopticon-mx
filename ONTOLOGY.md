# 📐 NexoGob: Graph Schema & Ontology Specification

**Version:** 1.0.0
**Engine:** Neo4j (Property Graph) / RDF Compatible
**Status:** Engineering Freeze

-----

## 1\. Executive Summary

This document defines the **NexoGob Ontology**, the structural logic governing the Knowledge Graph. It dictates how the chaotic reality of the Mexican Public Administration is normalized into a queryable network. It strictly separates **Digital Infrastructure** (domains, servers) from **Bureaucratic Structure** (agencies, budgets) to allow for forensic analysis of the gap between "what exists online" and "what exists in law."

## 2\. Naming Conventions & ID Strategy

To ensure global uniqueness across millions of nodes, we utilize a strictly typed ID system.

  * **Node Labels:** `PascalCase` (e.g., `GobEntity`, `PublicContract`).
  * **Relationship Types:** `SCREAMING_SNAKE_CASE` (e.g., `:ADMINISTERS`, `:LINKS_TO`).
  * **Property Keys:** `snake_case` (e.g., `budget_cycle`, `rfc_hash`).

### 2.1 The Universal ID (UID)

Every node must have a `uid` property derived deterministically where possible to prevent duplication during distributed crawling.

| Node Type | UID Strategy | Example |
| :--- | :--- | :--- |
| **Agency** | `MX--` | `MX-FED-SEGOB` |
| **Domain** | SHA-256 of normalized Root URL | `a1b2...` (hash of `gob.mx`) |
| **Person** | SHA-256 of RFC or CURP (Salted) | `f9e3...` (Redacted PII) |
| **Contract** | `-` | `MX-IMSS-059002` |

-----

## 3\. Node Definitions (Vertices)

### 🏛️ Class: `GobEntity` (The Bureaucracy)

Represents a legal body recognized by the *Padrón de Sujetos Obligados*.

  * **Labels:** `:GobEntity`, `:SujetoObligado`
  * **Properties:**
      * `official_name`: (String) e.g., "Secretaría de la Defensa Nacional".
      * `acronym`: (String) e.g., "SEDENA".
      * `level`: (Enum) \`\`.
      * `budget_ramo`: (Integer) The federal budget branch ID (e.g., `07` for Defense).
      * `status`: (Enum) \`\`.

### 🌐 Class: `DigitalResource` (The Web)

Represents a digital asset found by the crawler.

  * **Labels:** `:Domain`, `:Page`, `:API`
  * **Properties:**
      * `url`: (String) Normalized absolute URL.
      * `http_status`: (Integer) Last known status (200, 404, 500).
      * `cms_fingerprint`: (String) e.g., "WordPress 5.8", "Drupal", "React".
      * `last_crawled`: (Datetime) ISO 8601.
      * `is_zombie`: (Boolean) True if content is outdated (\>2 years) but server is active.

### 👤 Class: `FiscalPerson` (The Actors)

Represents an individual or company interacting with the state. *Strict Privacy Rules Apply.*

  * **Labels:** `:Person`, `:Company`, `:PublicOfficial`
  * **Properties:**
      * `name_redacted`: (String) First Name + Last Initial (if private citizen).
      * `rfc_hash`: (String) Anonymized tax ID.
      * `is_pep`: (Boolean) "Politically Exposed Person" flag (Public Officials).
      * `salary_grade`: (String) For public officials (e.g., "G-1").

### 📄 Class: `LegalInstrument` (The Evidence)

Represents a document that creates a legal bond.

  * **Labels:** `:Contract`, `:Decree`, `:Law`
  * **Properties:**
      * `contract_amount`: (Float) Value in MXN.
      * `contract_type`: (Enum) \`\`.
      * `signed_date`: (Date).
      * `url_pdf`: (String) Link to the evidence file.

-----

## 4\. Edge Definitions (Relationships)

### 🔗 Structural Edges (The Web Graph)

  * `(:Page)-->(:Page)`
      * *Usage:* Captures the hyperlink structure. The `anchor_text` is vital for NLP analysis (e.g., identifying deceptive links).
  * `(:Domain)-->(:Page)`
      * *Usage:* Hierarchical parent-child relationship.

### 🏛️ Bureaucratic Edges (The Power Graph)

  * `(:GobEntity)-->(:GobEntity)`
      * *Example:* `SEMARNAT` -\> `ASEA`.
  * `(:GobEntity)-->(:DigitalResource)`
      * *Example:* `INE` -\> `ine.mx`. Crucial for identifying "Orphaned Domains" (domains with no clear owner).

### 💸 Forensic Edges (The Corruption Graph)

  * `(:GobEntity)-->(:Contract)`
  * `(:Company)-->(:Contract)`
  * `(:Person)-->(:Contract)`
  * `(:Person)-->(:Company)`
      * *Analysis:* A circular path like `(Official)-->(Company)-->(Contract)<--(Agency)` triggers a **Conflict of Interest Alert**.

-----

## 5\. Cypher Queries for Analysis (The "Red Flags")

### 🚩 Red Flag 1: The "Ghost" Satellite

*Find government domains that are active but not linked to by the central `gob.mx` portal (Shadow IT).*

```cypher
MATCH (d:Domain {tld_suffix: ".gob.mx"})
WHERE NOT (:Domain {name: "gob.mx"})-->(d)
RETURN d.url, d.cms_fingerprint
```

### 🚩 Red Flag 2: The "Revolving Door" (Closed Cycle)

*Find companies that win contracts from agencies regulated by their own shareholders.*

```cypher
MATCH (official:PublicOfficial)-->(agency:GobEntity)
MATCH (official)-->(company:Company)
MATCH (company)-->(c:Contract)<--(agency)
RETURN official.name, agency.acronym, c.amount
ORDER BY c.amount DESC
```

### 🚩 Red Flag 3: Bid Rigging "Cliques"

*Find groups of companies that only bid on contracts where a specific other company wins (Rotation Cartel).*

```cypher
MATCH (c1:Company)-->(con:Contract)<--(c2:Company)
WITH c1, c2, count(con) as interaction_count
WHERE interaction_count > 10
RETURN c1.name, c2.name, interaction_count
```

-----

## 6\. Privacy & Redaction Protocol (LGPDPPSO Compliance)

This schema strictly enforces the *Ley General de Protección de Datos Personales*.

1.  **Ingestion Middleware:** Before any node is written to Neo4j, it passes through a PII scrubber.
2.  **The "Public Official" Exception:** If a person is listed in the *Nómina Transparente* (Public Payroll), their name is stored in cleartext with the `:PublicOfficial` label.
3.  **The "Private Citizen" Rule:** If a person appears only as a beneficiary or witness without public office, their data is hashed (`SHA-256`) and labelled `:PrivateCitizen`.
4.  **Sensitive Fields:** `home_address`, `personal_email`, and `mobile_phone` are **never** stored in the graph. Only `office_address` and `institutional_email` are persisted.
