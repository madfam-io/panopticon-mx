# Product Requirements Document (PRD)
**Project Code Name:** *Panopticon MX* (Internal) / **Public Name:** *NexoGob: El Observatorio Digital del Estado*
**Version:** 1.0
**Status:** Draft / Research Phase
**Date:** December 15, 2025

---

## 1. Brand & Positioning Statement

**Brand Name:** NexoGob
**Tagline:** "Visualizando la Estructura Real del Estado." (Visualizing the Real Structure of the State).

**Positioning Statement:**
For investigative journalists, policy analysts, and civic tech activists who struggle to navigate the fragmented and often opaque digital labyrinth of the Mexican government, **NexoGob** is a forensic data engine and interactive observatory. Unlike traditional transparency portals that offer static lists or isolated documents, NexoGob aggregates the entire topological structure of the Mexican public web—from federal secretariats to municipal servers—into a unified, navigable knowledge graph. We position ourselves not merely as a search engine, but as a **"Digital MRI" of the bureaucracy**, revealing hidden dependencies, structural silos, and patterns of corruption that text-based searches cannot detect. We stand for **Radical Structural Transparency**: the belief that the *connections* between institutions are as important as the institutions themselves.

---

## 2. Executive Summary

The Mexican government's digital presence is distributed across over 5,000 distinct domains, ranging from the centralized `gob.mx` ecosystem to independent servers for autonomous bodies (INE, Banxico) and thousands of municipal sites. This fragmentation creates "dark corners" where public accountability vanishes. NexoGob aims to build the first comprehensive **Mexican Government Web Graph (MGWG)**. By crawling, indexing, and linking every accessible URL related to the Mexican state, we will create a live, interactive map of the public administration. This tool will empower users to detect "zombie" infrastructure, visualize bid-rigging networks, and measure the bureaucratic "click-distance" between citizens and services.

## 3. Problem Definition

### 3.1 The Fragmentation Problem
While the Federal Executive mandates the use of `gob.mx`, "State Sovereignty" allows the 32 entities and ~2,460 municipalities to operate independent digital infrastructures. There is no single directory that maps the relationship between a Federal program (e.g., *Sembrando Vida*) and its municipal implementation sites.

### 3.2 The "Deep Web" of Bureaucracy
Valuable data (contracts, directories, technical reports) is often buried in "deep web" databases (SIPOT, CompraNet) or legacy subdomains that are not indexed by commercial search engines (Google/Bing) because they lack SEO optimization or proper sitemaps.

### 3.3 Lack of Relational Insight
Current transparency tools are document-centric (e.g., "Find this PDF"). They fail to answer relational questions, such as: *"Which agencies link to this specific private supplier?"* or *"Is there a circular link structure between the auditor and the audited entity?"*

---

## 4. Target Audience & User Personas

1.  **The Forensic Journalist (Primary):** Needs to find hidden links between a shell company and multiple government agencies. Uses the graph to detect "cliques" of corruption.
2.  **The Policy Analyst:** Wants to evaluate the "Digital Cohesion" of the government. Uses the tool to see if environmental agencies are actually linking to energy agencies, or if they operate in silos.
3.  **The Tech-Savvy Citizen:** Trying to navigate bureaucracy. Uses the "Pathfinder" feature to find the shortest route to a specific procedure without hitting broken links (404s).
4.  **The Government CIO (Internal Customer):** Needs to audit their own digital estate to identify broken links, security vulnerabilities, and outdated legacy domains.

---

## 5. Functional Requirements

### 5.1 The Aggregation Engine ("The Spider")
The core backend system responsible for discovering and archiving content.
*   **FR-01 Seed Generation:** System must auto-generate seed lists using the INAI *Padrón de Sujetos Obligados*, the `adela` API, and recursive DNS permutations for municipal domains (e.g., `*.gob.mx`, `*.mx`).
*   **FR-02 Distributed Crawling:** Utilization of **Apache Nutch** for high-volume static crawling and **Crawlee** (Headless Browsers) for dynamic SPA (Single Page Application) sites.
*   **FR-03 Politeness Enforcement:** Implementation of **Kubernetes APF** (API Priority and Fairness) to throttle request rates per domain, preventing inadvertent DDoS attacks on fragile municipal servers.
*   **FR-04 Archive Integration:** Ability to query and ingest historical link data from **Common Crawl** to visualize "Digital Rot" (links that existed in 2018 but are gone in 2024).

### 5.2 The Knowledge Graph ("The Brain")
The database schema designed to model bureaucracy.
*   **FR-05 Graph Storage:** Implementation of a Property Graph Database (**Neo4j**) to store nodes (`Agency`, `URL`, `Document`, `Person`) and edges (`LINKS_TO`, `BELONGS_TO`, `MENTIONS`).
*   **FR-06 Entity Resolution:** NLP pipeline to extract named entities (politicians, companies) from page text and map them to unique nodes, enabling cross-domain linkage.
*   **FR-07 Metadata Enrichment:** Automatic classification of nodes based on content (e.g., "Procurement Portal," "News," "Open Data Repository").

### 5.3 The Interactive Interface ("The Lens")
*   **FR-08 Macro Visualization:** A "Galaxy View" powered by **Cosmograph** (WebGL) capable of rendering 1M+ nodes in the browser, color-coded by branch of government (Federal, State, Autonomous).
*   **FR-09 Semantic Drill-Down:** Search bar accepting natural language queries (e.g., *"Show me connections between SEDENA and construction companies"*), returning a filtered subgraph via **Sigma.js**.
*   **FR-10 The Pathfinder:** A navigation tool where users select a Start Node and End Node; the system highlights the shortest path of hyperlinks, flagging redirects and broken chains.
*   **FR-11 Anomaly Dashboard:** Automated reporting of graph metrics:
    *   **High Betweenness Centrality:** Identifying "Bridge Nodes" that control information flow.
    *   **High Modularity:** Identifying isolated clusters (Silos).
    *   **Closed Triangles:** Potential collusion patterns.

---

## 6. Technical Architecture

### 6.1 Stack Overview
*   **Infrastructure:** Kubernetes Cluster (Google GKE or Azure AKS) for elastic scaling of crawlers.
*   **Ingestion:** Apache Nutch (Java) + Crawlee (Node.js/Playwright).
*   **Storage:** Neo4j (Graph Data) + Elasticsearch (Full-text Search).
*   **Frontend:** React + Cosmograph (GPU Rendering) + Sigma.js (Interaction).
*   **API Layer:** GraphQL middleware to translate frontend queries into Cypher/Gremlin.

### 6.2 Data Flow
1.  **Seed:** `adela` API -> JSON List -> Kafka Queue.
2.  **Crawl:** Workers fetch URLs -> Parse HTML -> Extract Links & Entities.
3.  **Process:** Links -> Neo4j (Edges); Text -> Elasticsearch (Index).
4.  **Analyze:** Graph Data Science Lib (PageRank, Louvain) runs nightly batches to update node scores.
5.  **Serve:** React App queries GraphQL -> Returns JSON graph -> WebGL Renderer.

---

## 7. Legal, Ethical, and Compliance Framework

### 7.1 Data Access & Scraping
*   **Legal Basis:** Operations are grounded in the *Ley General de Transparencia y Acceso a la Información Pública* (LGTAIP), treating scraping as an automated exercise of the right to know.
*   **Robots.txt:** The crawler will respect standard exclusion protocols but will flag "blocked" public interest sections for manual review/FOIA requests.

### 7.2 Privacy (LGPDPPSO)
*   **PII Redaction:** The ingestion pipeline must include regex filters to detect and hash/redact sensitive personal data (CURP, private emails, home addresses) inadvertently exposed on government servers before it hits the public graph.

### 7.3 Copyright (LFDA)
*   **Public Domain:** Official texts (laws, decrees, regulations) are treated as public domain.
*   **Fair Use:** The reproduction of snippets and link structures is justified under exceptions for news reporting and research. The app will *link* to content, not host full copyrighted binaries (images/videos) unless necessary for archival evidence.

---

## 8. Strategic Roadmap

*   **Phase 1: The Skeleton (Months 1-3):** Crawl `gob.mx` core. Setup Neo4j. Basic visualization of Federal Executive branch.
*   **Phase 2: The Deep Dive (Months 4-6):** Expand to States and Autonomous bodies (INE, Banxico). Integrate NLP for entity extraction.
*   **Phase 3: The Intelligence (Months 7-9):** Implement corruption detection algorithms (cycle detection, bipartite projections).
*   **Phase 4: Public Launch (Month 10):** Release "NexoGob" beta to select journalists and NGOs. Publish "State of the Digital Union" report.

---

## 9. Success Metrics (KPIs)

*   **Graph Density:** Number of verified edges (links) between distinct government entities.
*   **Silo Identification:** Number of "orphaned" clusters identified and reported.
*   **Query Performance:** Ability to render a 100k node subgraph in <2 seconds on a standard laptop.
*   **Impact:** Number of investigative stories or policy papers citing NexoGob data.
