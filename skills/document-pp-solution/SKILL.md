---
name: document-pp-solution
description: Guide for generating TOGAF-aligned architecture documentation for a Microsoft Power Platform solution using metadata retrieved from solution artifacts provided as inputs.
allowed-tools: shell
---

## 1. Purpose
This agent generates **TOGAF-aligned architecture documentation** for a **Microsoft Power Platform solution** using metadata retrieved from **solution artifacts provided as inputs** and any additional process documentation provided for context.

The agent MUST:
- Use `Expand-Archive` in PowerShell to extract the solution zip file to `/extracted-[solution-name]/`
- Analyze extracted files Power Platform solution by reading the extract XML, JSON and YAML files.
- If there are multiple apps in the extracted folder (recognizable as additional zip files) repeat the extraction step using the same naming convention for the output folder (e.g. `/[app-name]/`) and analyze each app separately.
- Map components to TOGAF architecture domains
- Produce a **structured but adaptable documentation set**
- Generate documentation suitable for **architecture governance**

If the user has not provided a solution name, the agent MUST explicitly state this as a blocker and request the solution name before proceeding.

The exported solution is the **single source of truth** for the code and architecture, while any additional documentation provided is there for business context understanding and is not mandatory if the user does not provide it.

Expected Folder Structure in the extracted solution

```
<SolutionRoot>/
  solution.xml                          — component manifest
  customizations.xml         — solution metadata, components and full desktop flow definitions
  CanvasApps/
    <AppName>/src/
      App.fx.yaml                       — named formulas, app-level settings
      <ScreenName>.fx.yaml              — per-screen controls and formulas
  Workflows/
    <FlowName>-<GUID>.json              — cloud flow definitions
  Entities/
    <TableName>/
      Entity.xml
      BusinessRules/                    — Dataverse business rule XML
  connectionreferences/                 — connector references
  environmentvariabledefinitions/       — env var definitions
  environmentvariablevalues/
  desktopflowbinaries/                            — Power Automate Desktop flow binary definitions
```

---

## 2. Architecture Framework
- TOGAF 9.x / 10
- Architecture Development Method (ADM)
- Covered phases:
  - Preliminary
  - Phase A – Architecture Vision
  - Phase B – Business Architecture
  - Phase C – Application & Data Architecture
  - Phase D – Technology Architecture

---

## 3. Condensed Documentation Structure (MANDATORY)

### 3.1 Root Structure

```text
/docs-[solution-name]/
  00_Overview.md
  01_Context_and_Vision.md
  02_Business_Architecture.md
  03_Application_Architecture.md
  04_Data_Architecture.md
  05_Technology_Architecture.md
  06_Security_and_Integration.md
  07_Deployment_and_Governance.md
  08_Appendix.md
```

This structure MUST be created exactly as shown.

---

## 3.2 Mandatory Chapter Structure per File

Except the 00_Overview.md file, each document MUST use the following internal structure. The overview must contain references to the other documents,
Sections may be empty if not applicable, but MUST be present.

```markdown
# <Document Title>

## 1. Purpose
## 2. Scope
## 3. TOGAF Phase Mapping
## 4. Architecture Principles
## 5. Current State (Baseline)
## 6. Key Decisions and Rationale
## 7. Risks and Assumptions
## 8. Open Questions
## 9. References
```

---

## 4. File-by-File Content Requirements

### 01_Context_and_Vision.md
**TOGAF:** Preliminary, Phase A

Include:
- Business drivers
- Stakeholders
- Architecture vision
- High-level solution overview

For any missing information, explicitly state: "⚠️ Information not available in solution metadata."
For any assumptions made, explicitly state: "⚠️ Assumption: [description]."

---

### 02_Business_Architecture.md
**TOGAF:** Phase B

Include:
- Business capabilities
- Business processes
- Mapping between Power Platform components (such as Canvas Apps, Power Automate flows, etc.)

---

### 03_Application_Architecture.md
**TOGAF:** Phase C (Application)

Include:
- Power Apps overview
- Power Automate flows (both cloud and desktop)
- Copilot Studio workflows and agents
- Custom connectors
- Application interaction diagrams (Mermaid required)

For each component, except for diagrams, provide the name, type (Canvas App, Model-Driven App, Cloud Flow, Desktop Flow, Agent, Agent Flow, etc.), purpose, relevant dependencies, major steps/screens and logic.

**Important:** Power Automate Desktop flow definitions are stored directly in the customizations.xml file as Robin code in a single XML node per desktop flow. The agent **MUST** parse those. The JSON files for workflows only include cloud flow definitions. Desktop flow JSON files are blank and do not contain their definitions. The desktop flow binary files contain some information on dependencies and objects used by those desktop flows, such as screenshots of UI elements, connectors used, etc. but they do not contain the actual logic of the desktop flows, which is only available in the customizations.xml file.

If any objects (especially flows and custom connectors) include any code, specify the language and purpose of each code sample. Provide the code inside Markdown code blocks with explanatory inline comments.

---

### 04_Data_Architecture.md
**TOGAF:** Phase C (Data)

Include:
- Dataverse usage
- Core tables
- Relationships
- Data ownership

For default Dataverse tables, explicitly state: "Default Dataverse table '[table name]' used. No custom metadata available."

---

### 05_Technology_Architecture.md
**TOGAF:** Phase D

Include:
- Power Platform environments
- Azure dependencies
- Identity and access (Microsoft Entra ID)

---

### 06_Security_and_Integration.md
**TOGAF:** Cross-cutting

Include:
- Security roles and model
- Compliance considerations
- Integration patterns

Do not include:
- Accounts or credentials
- Sensitive information not available in solution metadata

---

### 07_Deployment_and_Governance.md
**TOGAF:** Phase G, H (Governance & Change)

Include:
- ALM strategy
- CI/CD approach
- Architecture governance

**Important**: Unless otherwise stated in either the user prompt or additional documentation, assume that Power Platform Pipelines via a custom host are used for deployments. Assume there are separate environments for at least DEV and PROD. Ask the user if there are additional environments, if the user prompt does not include details. 

---

### 08_Appendix.md
Include:
- Glossary
- Abbreviations
- External references

---

## 5. Diagram Rules

- Use **Mermaid** for all diagrams
- Diagrams MUST be embedded in the relevant section
- Every diagram MUST include a title and description

Example:
```mermaid
graph TD
  User --> PowerApp
  PowerApp --> Dataverse
  Dataverse --> Flow
```

---

## 6. Content Rules (STRICT)

- No undocumented assumptions
- Any assumption MUST be explicitly stated with "⚠️ Assumption: [description]."
- List risks and assumptions in a table that has consistent formatting across all documents, including the following columns: ID (RA-[number]), Type (Risk/Assumption), Description, Impact (Low/Medium/High/Critical), Mitigation Strategy.
- Use continuous numbering for risks and assumptions across all documents (e.g. if 3 risks are listed in the Business Architecture document, the first risk in the Application Architecture document should start with RA-4).
- Consider all documents as a single set, and do not repeat risks or assumptions across documents. If a risk or assumption is relevant to multiple domains, list it in the most relevant document and reference it in the others with "See RA-[number] in [Document Name]."
- For any missing information, explicitly state: "⚠️ Information not available in solution metadata."
- Clearly distinguish baseline vs best practices
- Use formal, neutral language

---

## 7. Quality Criteria

Documentation MUST be:
- TOGAF-traceable
- Consistent across files
- Suitable for architecture review boards
- Easy to extend to a full ADM deliverable set
- Clear and concise
- For code, use high verbosity with self-explanatory naming and inline comments

---

## 8. Constraints

- Read-only analysis
- No deployment or configuration changes
- No inferred business intent beyond solution metadata

---

## 9. Versioning

Each file MUST include:
- Version
- Date
- Author (use the name of the user who invoked the agent)
- Change summary

---

## 10. Stop Conditions

- Complete and return the documents when every applicable section is thoroughly addressed. 
- Mark any non-applicable sections as 'Not Applicable.' 
- If any part is unclear or out of scope, seek clarification as needed.

---