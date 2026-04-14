---
name: project-analyze
description: Deep project analysis to extract architecture design, tech stack, core modules, and business semantics, then generate documentation.
---

# Goal
Deeply analyze the project in the current working directory, accurately extracting its architecture design, tech stack, core modules, and business semantics. Upon completion of the analysis, generate a `PROJECT_STATE` folder and a structured set of Markdown documents in the project root directory, following strict specifications.

# Principles
- **Observe before inferring**: At each step, first state the facts that can be directly observed from the code, then provide inferences. If the reason cannot be inferred, state only the facts — do not fabricate "why"
- **Stop and ask when uncertain**: If business semantics cannot be inferred from the code, mark as "To be confirmed" rather than guessing. If multiple interpretations are equally reasonable, present them to the user for selection
- **Treat simple projects simply**: If the project has no more than 20 source files, merge Phase 1/2 into a single step — no need to over-subdivide

# Analysis Scope Control
- **Must exclude**: node_modules, dist, build, out, .git, vendor, __pycache__, target, .next, coverage, and other generated artifact directories
- **Must exclude**: Lock files (package-lock.json, yarn.lock, pnpm-lock.yaml, etc.)
- **Focus on core**: Only analyze source code, dependency declarations, configuration files, and test code — do not analyze the internal implementation of third-party dependencies

# Execution Flow

## Phase 1: Top-Level Reconnaissance and Project Characterization

**Completion Criteria**: Can answer "What technology does this project use, what architectural paradigm, and where is the core entry point".

### 1.1 Configuration File Scan (Check in priority order)
Look for the following key configuration files (analyze if found, skip if not):
| Category | Typical Files | Inferable Information |
|----------|--------------|----------------------|
| Package Management | package.json, pom.xml, build.gradle, Cargo.toml, go.mod, requirements.txt, pyproject.toml | Language, framework, dependencies |
| Build Tools | webpack.config.*, vite.config.*, tsconfig.json, rollup.config.*, .babelrc, Makefile, Dockerfile | Build approach, compilation config |
| Code Standards | .eslintrc.*, .prettierrc, checkstyle.xml, .editorconfig | Code style constraints |
| CI/CD | .github/workflows/*, Jenkinsfile, .gitlab-ci.yml, .circleci/* | Deployment pipeline |
| Monorepo | lerna.json, nx.json, pnpm-workspace.yaml, turbo.json, rush.json | Multi-project structure |

### 1.2 Engineering Paradigm Identification
Determine from directory structure and dependency declarations:
- **Monorepo**: Multi-project directories like packages/*, apps/*, libs/* exist + workspace configuration files
- **Multi-module Project**: Multiple subdirectories each with independent build configurations
- **Single Application**: Single entry point, unified build
- **Library/SDK**: Primarily exports modules, no standalone runtime entry point
- **Hybrid**: A combination of the above patterns

### 1.3 Tech Stack Extraction
From the dependency manifest, identify:
- **Core Framework**: e.g., React, Vue, Spring Boot, Express, FastAPI
- **Build/Bundling Tools**: e.g., Vite, Webpack, Maven, Gradle
- **Key Dependency Libraries**: High-frequency or architecturally significant libraries (state management, ORM, UI frameworks, etc.)
- **Language and Version**: e.g., TypeScript 5.x, Java 17, Python 3.12

## Phase 2: Deep Topology and Business Deconstruction

**Completion Criteria**: Can answer "What does each core module do, what does it depend on, and what depends on it".

### 2.1 Entry Point Tracing
Find the project entry file (e.g., main.*, app.*, index.*, Application.*), and from the entry point, trace the core import/require chain to identify key nodes in the core dependency graph.

### 2.2 Architecture Layer Identification
Combining directory structure, import relationships, and data flow, determine the architectural pattern adopted by the project:
- **MVC/MVVM**: Clear separation of model, view, and controller layers
- **Clean Architecture / Hexagonal**: Separation of domain, infrastructure, and application layers
- **Feature-based (Feature Slices)**: Directories organized by business feature (e.g., src/features/order)
- **Layered Architecture**: e.g., controller → service → dao/repository
- **Microservices / Serverless**: Multiple independent services, each with its own entry point and database

For identified patterns, describe the architecture layer rules and data flow in a single sentence. If multiple patterns coexist, describe their respective scopes.

### 2.3 Business Semantics Mapping
For each core directory/module, you must answer:
- **What it is**: The directory/module name and physical location (e.g., `src/views/order`)
- **What it does**: Infer its business function from naming, type definitions (Interfaces/Types), and API routes (e.g., handles B2B order processing and refund logic)
- **What it depends on**: Other modules or external services this module depends on
- **What depends on it**: Which modules depend on this module

**When to stop and ask**: If the business semantics of a core module cannot be inferred from the code, do not guess — mark it as "To be confirmed" and ask the user. If inter-module dependency relationships are ambiguous, list the observed phenomena and ask the user to confirm.

## Phase 3: Based on Analysis Results and User Confirmation, Generate `PROJECT_STATE`

**Completion Criteria**: `PROJECT_STATE/` directory has been created, all documents have been generated and confirmed by the user.

Strictly follow these steps to generate `PROJECT_STATE`. Do not skip user confirmation and generate content directly.

### 3.1 Understand `PROJECT_STATE` Directory Structure Requirements

📁PROJECT_STATE/
 ├── index.md (Overall overview, entry file)
 ├── architecture.md (Architecture and tech stack design)
 └── 📁modules/ (Module details, organized by business/functional category)
      ├── module_A.md
      └── module_B.md

### 3.2 Content Specifications for Each File

#### PROJECT_STATE/index.md
`PROJECT_STATE/index.md` records the overall project evolution state, with the following format:
```
# <One-sentence description of the project's core value and engineering type, e.g., 'Vue3-based e-commerce site using Monorepo architecture'>

# Architecture Blueprint
<Concise Markdown description of key tech stack and architectural relationships, no more than ten lines>
See: PROJECT_STATE/architecture.md

# Business Modules
<Concise Markdown description of core business modules and capabilities, no more than ten lines>
See: Specific module documents under PROJECT_STATE/modules folder
```

#### PROJECT_STATE/architecture.md
Include the following content as appropriate for the specific project:
- **Detailed Tech Stack Inventory**: Categorized listing (frameworks, routing, state management, styling, tools, etc.).
- **Directory Tree Structure and Responsibilities**: Use tree text format to display core directories, with a one-sentence annotation of each directory's responsibility boundaries.
- **Architecture Pattern**: Describe the architectural pattern adopted by the project, including layering rules and data flow.
- **Key Design Decisions**: Only record decisions that can be clearly inferred from the code (e.g., "Using EventBus to decouple inter-module communication"). **Do not fabricate decision rationales that cannot be verified from the code**; if the rationale cannot be inferred, state only the fact (e.g., "Using Zustand for state management") without making up the "why". If the reason for a decision is uncertain, mark it as "To be confirmed".

#### PROJECT_STATE/modules/*.md
Module documents should be **consolidated to around 5, not exceeding 10**.
- **Business Module Partitioning**: Organize modules by business/functional category; one module may contain multiple sub-projects/folders.
- **Core Capabilities**: Describe in detail the specific business/functions this module handles.
- **Key Folders/Files**: List the 2-3 most important folder/file paths within this module, and describe the core logic implemented inside them (do not paste code or function names).
- **Dependency Relationships**: Briefly describe the key modules this module depends on / is depended upon by.

### 3.3 Confirm Module Partitioning with User (Mandatory)
Since module classification under `PROJECT_STATE/modules` depends heavily on business understanding, you must first confirm with the user:
- Present 2-3 module classification options for the user to choose from, with the recommended option listed first. Common classification dimensions to consider:
  - **By business domain**: e.g., User Module, Order Module, Payment Module (recommended, clearest business semantics)
  - **By technical layer**: e.g., API Layer, Service Layer, Data Layer (suitable for technology-oriented projects)
  - **By feature**: e.g., Authentication & Authorization, Data Export, Notifications (suitable for cross-cutting features)
- **Ask only one question at a time**, with each question limited to 200 characters
- Use multiple-choice questions only, presented conversationally with options, also allowing the user to provide additional input
- Only proceed to the document generation phase after user confirmation

### 3.4 Generate Specific Documents
Strictly based on the `PROJECT_STATE` specification and the user-confirmed classification scheme, generate:
1. `PROJECT_STATE/index.md`
2. `PROJECT_STATE/architecture.md`
3. Documents under `PROJECT_STATE/modules/` for each module
