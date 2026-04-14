---
name: doc-update
description: Execute this skill after completing all code changes (not needed if no code was changed). Update project documentation based on the changes made and user conversations in this session.
---

# Documentation Update Skill

After completing all code changes, update project documentation based on the changes made and user conversations:
1. Record overall project evolution state (PROJECT_STATE/index.md)
2. Describe features and usage guide (README.md)
3. Document knowledge useful for future evolution (LEARNING/index.md)

# Principles
- **Only change what needs changing**: Only update documentation content that has become outdated due to this code change — do not take the opportunity to "incidentally" rewrite, reformat, or polish unaffected paragraphs
- **Match existing style**: When updating documentation, follow the existing writing style, format, and level of detail — do not change the presentation based on personal preference
- **Ask when uncertain**: If you cannot determine whether a document needs updating, present the reasons for and against updating to the user and ask for confirmation

## Step 1: Determine Change Scope

Before using this skill, you must first clarify the changes made through:
1. **Check conversation context**: Review what code changes were completed in this conversation
2. **Use git diff**: Run `git diff --stat` to see the list of changed files, combined with conversation context to determine the scope of changes
3. **Classify the changes**: Determine which type the changes fall under:
   - **New Feature**: Added new business functionality or module
   - **Architecture Adjustment**: Changed project structure, tech stack, or architectural pattern
   - **Feature Adjustment**: Modified behavior of existing features
   - **Bug Fix**: Fixed a bug
   - **Performance Optimization**: Optimized performance-related code
   - **Refactoring**: Code structure changes with no behavioral change
   - **Documentation/Configuration**: Documentation or configuration file changes only

## Step 2: Update Documents per Decision Matrix

Based on the change type, decide which documents need updating (Must update / As needed / Skip):

| Change Type | PROJECT_STATE/index.md Evolution Timeline | PROJECT_STATE/index.md Architecture Blueprint / Business Modules | PROJECT_STATE/architecture.md | PROJECT_STATE/modules/*.md | README.md | LEARNING/index.md |
|-------------|:---:|:---:|:---:|:---:|:---:|:---:|
| New Feature | Must | As needed | As needed | Must (new module) or As needed | As needed | As needed |
| Architecture Adjustment | Must | Must | Must | Must | As needed | As needed |
| Feature Adjustment | Must | As needed | Skip | Must (affected modules) | As needed | As needed |
| Bug Fix | Must | Skip | Skip | As needed | Skip | As needed |
| Performance Optimization | Must | Skip | As needed | As needed | Skip | As needed |
| Refactoring | Must | As needed | As needed | As needed | Skip | As needed |
| Documentation/Configuration | As needed | Skip | Skip | Skip | As needed | Skip |

**Legend**:
- "Must": Must update
- "As needed": Update only if the relevant content in that document has actually changed
- "Skip": No update needed
- **The evolution timeline must never be skipped** (except for documentation-only changes)

## Step 3: Execute Documentation Updates

### 3.1 PROJECT_STATE/index.md
Location: The `PROJECT_STATE` folder is in the project root directory, globally unique.
Update: Must update `PROJECT_STATE/index.md` after completing each feature development, testing, or maintenance task.
Content: Record the overall project evolution state, with the following format:
```
# <One-sentence description of the project's core value and engineering type, e.g., 'Vue3-based e-commerce site using Monorepo architecture'>

# Architecture Blueprint
<Concise Markdown description of key tech stack and architectural relationships, no more than ten lines>
See: PROJECT_STATE/architecture.md

# Business Modules
<Concise Markdown description of core business modules and capabilities, no more than ten lines>
See: Specific module documents under PROJECT_STATE/modules folder

# Evolution Timeline
- <yyyy-mm>: <Summarize last month's changes in three sentences or less>
- <yyyy-mm-dd-hh-mm>: <One-sentence description of the change (new feature / architecture adjustment / feature adjustment / bug fix / performance optimization)>
```

#### Update Rules
- **Architecture Blueprint**: Update only when tech stack or architectural relationships change
- **Business Modules**: Update only when business modules are added, removed, or significantly modified
- **Evolution Timeline**: Update on every code change, defaulting to yyyy-mm-dd-hh-mm format. When total entries exceed 20, automatically consolidate earlier entries into yyyy-mm format (keeping the most recent month's detailed entries when consolidating)

### 3.2 PROJECT_STATE/architecture.md
Update only when architecture or tech stack related changes occur. Content scope:
- **Detailed Tech Stack Inventory**: Categorized listing (frameworks, routing, state management, styling, tools, etc.).
- **Directory Tree Structure and Responsibilities**: Use tree text format to display core directories, with a one-sentence annotation of each directory's responsibility boundaries.
- **Architecture Pattern**: Describe the architectural pattern adopted by the project, including layering rules and data flow.
- **Key Design Decisions**: Only record decisions that can be clearly inferred from the code. Do not fabricate rationales.

### 3.3 PROJECT_STATE/modules/*.md
When changes involve a module, find and update the corresponding module document. How to find it:
1. Read the "Business Modules" section in `PROJECT_STATE/index.md` to understand module partitioning
2. Match changed file paths from git diff to the corresponding module document
3. If changed files do not belong to any existing module, consider whether a new module document is needed or whether module attribution should be adjusted

Module documents should be **consolidated to around 5, not exceeding 10**. Each module document contains:
- **Business Module Partitioning**: Organize modules by business/functional category; one module may contain multiple sub-projects/folders.
- **Core Capabilities**: Describe in detail the specific business/functions this module handles.
- **Key Folders/Files**: List the 2-3 most important folder/file paths within this module, and describe the core logic implemented inside them (do not paste code or function names).
- **Dependency Relationships**: Briefly describe the key modules this module depends on and is depended upon by.

### 3.4 README.md
Location: The README.md in the project root directory serves as the overall project introduction; each sub-project also has its own README.md.
Update conditions (update if any condition is met):
- A user-visible feature or command has been added
- The project's installation/startup/configuration method has changed
- The project introduction or feature description is outdated
Content: Should only contain project introduction and usage guide:
- **Project Introduction**: Background, goals, features, use cases of the project.
- **Usage Guide**: Main features and usage examples of the project.
README.md content and examples should be kept concise. **If changes are only internal code changes that do not affect how users use the project, do not update the README.**

### 3.5 LEARNING/index.md
Location: The `LEARNING` folder is in the project root directory, globally unique.
**Only update if any of the following situations occurred during this work session** (skip if none apply):
- Encountered and resolved anomalies/errors during execution (e.g., build failures, dependency conflicts, runtime errors)
- User corrected the Agent's working approach or provided improvement suggestions
- Discovered technical constraints or pitfalls worth documenting in the project

**Anti-patterns — what NOT to record**:
- Routine operation steps (e.g., "installed dependencies") — these are not learnings
- Naming conventions specific only to this project — the code itself is the documentation
- Unconfirmed personal speculations

Content format:
```
# Project Experience
The following are experiences from working on this project. Details should only be read on demand when necessary.

# Execution Anomalies and Resolutions
1. <Anomaly 1>: <One-sentence description of the anomaly and resolution>, See: <LEARNING/xxx.md>

# Corrections from User
1. <Correction 1>: <One-sentence description of what the user corrected>, See: <LEARNING/yyy.md>
```
Notes:
- Generate a corresponding detail document for each execution anomaly and resolution.
- Generate a corresponding detail document for each user correction (issue feedback, improvement suggestion, architecture requirement).
- Original errors/anomalies/code snippets should be placed in the corresponding detail documents under the `LEARNING` folder. `LEARNING/index.md` should concisely distill the key learnings.
