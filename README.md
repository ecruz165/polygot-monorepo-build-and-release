## 🔹 PRD: Monorepo Build + Deployment Setup

### **Goal**

* Provide a unified developer + CI/CD workflow for a mixed **Node.js** + **Java** monorepo.
* Ensure **change-aware builds** (only affected modules rebuilt).
* Maintain **consistent commands** (`make build`, `make run`, `make stop`, `make deploy`) across modules and root.
* Use **AWS CodeBuild** as the CI/CD runtime.
* Use **Changesets** to manage **versioning & release notes**.

---

### **Architecture Overview**

1. **Modules**

   * **Node.js modules** → `pnpm` + `changeset` for version bumps.
   * **Java modules** → `maven` for local dev/build.

2. **Root**

   * **Makefile** provides developer-friendly commands (`make build`, `make run`, etc.).
   * **Changesets** configured in root (`.changeset/`) for cross-module versioning.

3. **CI/CD**

   * **Gradle** orchestrates pipelines in AWS CodeBuild by delegating to `make`.
   * Change detection via a **TypeScript CLI tool**:

     * Queries `git diff` against mainline branch.
     * Produces `changes.json` with impacted modules.
     * Updates a `results.json` artifact for Gradle & CodeBuild.

---

### **Workflow**

#### **Local Development**

* Developers work in module scope (`pnpm dev`, `mvn spring-boot:run`).
* At root, use `make` for consistent commands:

  ```sh
  make build   # builds only changed modules
  make run     # starts services
  make stop    # stops all running processes
  make deploy  # local deploy/test script
  ```

#### **Versioning & Releases**

* Developer runs:

  ```sh
  pnpm changeset
  ```

  → Creates a changeset describing version bumps.
* Root handles publishing (Node.js to npm, Java artifacts to internal registry).

#### **CI/CD (AWS CodeBuild)**

1. **Detect Changes**

   * Run TS CLI:

     ```sh
     pnpm ts-node scripts/detectChanges.ts --base origin/main --head HEAD
     ```
   * Writes `changes.json`:

     ```json
     {
       "changedModules": ["service-a", "service-b"]
     }
     ```

2. **Build Phase**

   ```sh
   ./gradlew build
   ```

   → Gradle calls `make build`, which uses `changes.json` to only build affected modules.

3. **Deploy Phase**

   ```sh
   ./gradlew deploy
   ```

   → Runs module-specific deploy scripts (ECS, Lambda, or EKS).

---

### **Optional Comparison: Nx / Turborepo**

* **Nx / Turborepo give you:**

  * Built-in change detection.
  * Distributed caching across CI/CD runners.
  * Dependency graph visualization.

* **Why we might skip them here:**

  * We already have **custom change detection**.
  * AWS CodeBuild has limited benefit from distributed caching (no shared cache by default).
  * Adds another abstraction devs must learn.

* **Verdict:**
  Stick with `make + gradle + changesets`. Add Nx/Turborepo only if:

  * You want **visual dependency graphing**.
  * Or caching is a bottleneck and you configure a **remote cache backend** (like S3 or Redis).

---

### **Deliverables**

* [ ] Root `Makefile` with consistent commands.
* [ ] Gradle build scripts wrapping `make`.
* [ ] TS CLI tool for git-based change detection.
* [ ] Changesets configuration (`.changeset/config.json`).
* [ ] AWS CodeBuild spec (`buildspec.yml`) wired to Gradle.

---

👉 Do you want me to draft the actual **`buildspec.yml` for AWS CodeBuild** showing how Gradle + make + changesets plug in together? That’d make the PRD more implementation-ready.
