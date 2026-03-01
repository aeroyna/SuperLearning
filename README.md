# SuperLearning Repository

[**➡️ Go to Learning Dashboard (SuperLearning.md)**](SuperLearning.md)

---

## 📖 About This Repository

**SuperLearning** is a comprehensive, structured knowledge base designed for mastering complex software engineering topics. It serves as a "Digital Garden" or a personal wiki, organized for deep learning and quick reference.

### 🧠 Core Domains
*   **Data Structures & Algorithms**: Extensive interview prep (Google, Microsoft, Amazon, Adobe), pattern recognition, and algorithm implementation.
*   **System Design**: Comprehensive HLD (High Level Design) and LLD (Low Level Design) for FAANG interviews, including distributed systems, design patterns, and case studies.
*   **Systems Programming**: Deep dives into C++ and Java internals.
*   **Mobile Development**: React Native architecture and performance.
*   **Version Control**: Advanced Git workflows.

Each section is meticulously curated to provide not just "how-to" guides, but deep "under-the-hood" explanations, making it ideal for:
*   Technical Interview Preparation (FAANG+)
*   Skill Upgrading for Senior Engineering Roles
*   University curriculum supplementation

---

## 🚀 How to Use with Obsidian (FIT Workflow)

This repository is optimized for use with **Obsidian**, a powerful knowledge base that works on top of a local folder of plain text Markdown files.

### Why Obsidian?
*   **Bi-directional Linking**: Navigate concepts naturally (e.g., jumping from "Hash Maps" to "LRU Cache").
*   **Local-First**: You own your data. No lock-in.
*   **Visualization**: See how your knowledge connects using the Graph View.

### 🛠️ Setup via FIT Extension (Git Integration)

To seamlessly sync this repository with your Obsidian vault across devices, we recommend using the **Obsidian Git** (often referred to as the "FIT" workflow for *Fast/Fluid Integrated Tracking* of changes).

1.  **Clone the Repository**:
    ```bash
    git clone https://github.com/your-username/SuperLearning.git
    ```
2.  **Open in Obsidian**:
    *   Launch Obsidian.
    *   Click "Open folder as vault".
    *   Select the cloned `SuperLearning` folder.
3.  **Install Obsidian Git Plugin**:
    *   Go to **Settings** -> **Community Plugins** -> **Browse**.
    *   Search for **"Obsidian Git"**.
    *   Install and **Enable** it.
4.  **Configure FIT (Auto-Sync)**:
    *   In Obsidian Git settings, enable **"Backup (commit and push) on interval"**.
    *   Set the interval to something frequent (e.g., 5-10 minutes) for a "FIT" experience where your notes are always up to date.
    *   Enable **"Pull updates on startup"** to ensure you always see the latest changes when you open the app.

### 📱 Mobile Access
*   **iOS/Android**: You can use the official Obsidian mobile app. Syncing can be handled via **Obsidian Git** (if using a compatible method like Working Copy on iOS or Termux on Android) or by simply syncing the folder via iCloud/Drive.

---

## 📂 Repository Structure

The repository follows a strict hierarchical structure to maintain order as it scales:

*   `cs_fundamentals/`: Core Computer Science concepts.
    *   `dsa/`: The massive Data Structures & Algorithms module.
    *   `system_design/`: System Design for FAANG interviews.
    *   `databases/`: A-Z guide to database mastery.
    *   `operating_systems/`: Operating Systems concepts.
*   `programming_languages/`:
    *   `cpp/`: Modern C++ and memory management.
    *   `java/`: JVM internals and enterprise Java.
    *   `python/`: Pythonic patterns and internals.
    *   `javascript/`: JS engine internals and patterns.
    *   `go/`: Golang concurrency and systems.
    *   `rust/`: Ownership, borrowing, and safety.
*   `development/`:
    *   `web/`:
        *   `fundamentals/`: Web fundamentals (HTTP, Browsers, etc.).
        *   `frameworks/`:
            *   `react/`: Frontend architecture.
    *   `mobile/`:
        *   `react_native/`: Mobile architecture and performance.
    *   `backend/`:
        *   `springboot/`: Enterprise Java web development.
        *   `django/`: Python web framework.
*   `cloud_and_devops/`:
    *   `aws/`: Cloud infrastructure at scale.
    *   `docker/`: Containerization fundamentals.
    *   `kubernetes/`: Container orchestration.
*   `tools/`: Developer tools and workflows.
    *   `git/`: Advanced Git workflows.
    *   `build_tools/`: CMake, Bazel, Maven, Gradle.
*   `daily_notes/`: A journal for tracking daily learning progress.

---

## 📝 Topic Structure Guidelines (Standard Course Specification)

To maintain consistency across all topics and courses, the following **Course Structure Specification** is strictly followed. This ensures a predictable, deep, and organized learning experience.

### 1. Directory Hierarchy
- **Root Directory:** Each major course or topic should reside in its own root directory (e.g., `[topic]_course` or `cs_fundamentals/[topic]`).
- **Logical Subdivisions:** Create high-level folders that logically group the curriculum (e.g., `Core_Concepts`, `Ecosystem`, `Deployment`). *Avoid defaulting to Basics/Intermediate/Advanced unless it specifically fits the topic.*
- **Chapter Folders:** Within these groups, create specific sub-directories for each chapter or granular topic (e.g., `Core_Concepts/Variables`).
- **Folder Notes (Critical):** Every chapter folder must contain a main note acting as the "Folder Page".
    - **Format:** `00_[folder_name_in_snake_case].md` (e.g., inside `Variables/`, the file is `00_variables.md`).
    - **Content:** This file must contain a strong conceptual overview of the chapter and **direct links to all sub-topic files** within that folder.

### 2. Naming Conventions
- **Folders:** Use **Pascal_Snake_Case** (e.g., `Memory_Management`, `Data_Structures`).
- **Files:** Use **snake_case** with a numerical prefix for ordering (e.g., `01_stack_vs_heap.md`).
- **Cleanliness:** Ensure no UUIDs, hashes, or special characters (other than underscores) appear in filenames.

### 3. Content Depth & Quality
- **Avoid "Cheat Sheet" Style:** Do not provide superficial, one-line explanations or just code snippets.
- **Deep Dive:** Focus on *how* things work under the hood, internal mechanics (e.g., memory layout, CPU interaction, architectural decisions).
- **Nuances & Pitfalls:** Explicitly highlight common misunderstandings, edge cases, and best practices.
- **Tone:** Authoritative, technical, and educational.
- **Code Examples:** Include code in **C++, Java, Python, and JavaScript** (in that order) where applicable.
- **Visuals:** Use Mermaid code blocks (` ```mermaid `) to visualize complex relationships or architectures. **Prefer Mermaid over ASCII diagrams.**

#### Diagram Types by Use Case

| Diagram Type | Best For | Example Use |
|--------------|----------|-------------|
| `flowchart TD` | System architecture, data flow | HLD components, request flow |
| `classDiagram` | Class hierarchies, OOP relationships | LLD design patterns, UML |
| `sequenceDiagram` | Service interactions, API calls | Saga patterns, auth flows |
| `stateDiagram-v2` | State machines, lifecycle | Circuit breaker, game states |
| `erDiagram` | Database schema | Entity relationships |

#### Mermaid Best Practices
- Keep diagrams focused; split large diagrams into multiple smaller ones.
- Use descriptive node IDs: `UserService[User Service]` not `A[User Service]`.
- **Do not put list items** (e.g. `1.`, `2.`) in the label text.

### 4. Overview Page (The Entry Point)
- **Topic Overview:** Each main topic has a `00_overview.md` file at its root.
- **Structure:** Present the content as a **"Learning Path"** (e.g., "Phase 1: Foundations", "Phase 2: Deep Dive").
- **Table of Contents Style:** Links must be **Standard English Title Case** with a **Number Prefix**.
    - *Format:* `[X.Y Title Name](path/to/file)`
    - *Example:* `[1.1 Stack vs Heap](Memory_Management/01_stack_vs_heap.md)`
    - **Note:** Chapter links should point to the folder note (e.g., `[1.0 Memory Management](Memory_Management/00_memory_management.md)`).

---

## 🏭 Company Pattern Update Guidelines

When overhauling the interview preparation section for a specific company (e.g., Microsoft, Amazon), follow this standardized process to ensure consistency and depth.

### Phase 1: Research & Analysis
1.  **Search for recent trends (Last 1-2 years):**
    *   Find the most frequently asked LeetCode questions.
    *   Identify shifts in interview style (e.g., "More graph problems?", "Is system design coding becoming common?").
    *   Look for specific "high signal" questions.
2.  **Rank the Patterns:**
    *   Group questions into patterns (e.g., Graphs, Trees, Arrays & Strings).
    *   Rank by frequency (Very High, High, Medium, Low/Niche).

### Phase 2: Create Pattern Sub-pages
*Target Directory:* `dsa/Interview_Prep/Company_Patterns/[COMPANY_NAME]/Patterns/`

Create a separate markdown file for **each** major pattern found (e.g., `01_graphs.md`). Each file must include:
*   **Header**: Pattern Name and Frequency status (e.g., 🔴 Very High).
*   **Key Concepts**: Bullet points explaining *why* this pattern matters.
*   **Phase 1: Must-Do (Foundation)**: A table of ~10 essential problems.
*   **Phase 2: Practice & Variants (Depth)**: A table of ~10 harder or niche variations.
*   **Links**: Every problem name MUST be a direct link to LeetCode.

### Phase 3: Create the 1-Month Plan
*Target File:* `dsa/Interview_Prep/Company_Patterns/[COMPANY_NAME]/[company_name]_1_month_plan.md`

Create a structured 30-day schedule:
*   **Daily Target**: 10 problems per day (Total 300 problems).
*   **Structure**: Group days by pattern (e.g., "Week 1: Arrays & Strings").
*   **Content**: Use the problems identified in Phase 2. Ensure every problem name is hyperlinked.
*   **Review Days**: Include days for "Speed Run" and "Mock Interview" sets.

### Phase 4: Update the Main Overview
*Target File:* `dsa/Interview_Prep/Company_Patterns/[COMPANY_NAME]/00_[company_name].md`

Rewrite the main company file to include:
1.  **Trends & Shift**: A summary of findings.
2.  **Top Patterns by Frequency**: A list linking to the sub-pages created in Phase 2.
3.  **Must-Do Top 10**: A curated list of the absolute most important questions.
4.  **1-Month Study Plan**: A link to the plan created in Phase 3.
5.  **Company-Specific Tips**: Advice on their specific interview culture.

---

## 🤝 Contribution

This is a living document. Contributions, corrections, and new topic requests are welcome!
1.  Fork the repo.
2.  Create a feature branch.
3.  Submit a Pull Request.

Happy Learning! 💡