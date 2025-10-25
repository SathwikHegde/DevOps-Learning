### Section 9: Pipeline Performance Optimizations - Maximizing CI/CD Efficiency ⏱️

Welcome to Section 9! Having built a feature-rich CI/CD pipeline, the next step in achieving DevOps maturity is focusing on **performance**. A slow pipeline is a productivity bottleneck, delaying feedback, increasing costs, and frustrating developers. This section explores advanced techniques and best practices to ensure your GitLab CI/CD pipeline runs as fast and lean as possible.

---

### Why Pipeline Performance is Critical

* **Faster Feedback Loops:** Quicker pipelines mean developers know sooner if their code is good, leading to faster bug fixes and higher-quality code integration.
* **Reduced Costs:** Shorter job execution times directly translate into lower usage of Runner minutes and reduced cloud resource consumption.
* **Increased Developer Productivity:** Eliminates "waiting time," allowing engineers to stay in their flow state and complete tasks more efficiently.
* **Scalability:** Efficient pipelines scale better with a growing codebase and team size.

---

### Key Areas for Performance Optimization

We will focus on four main pillars to boost your pipeline's speed and efficiency:

#### 1. Smart Execution Control (Reducing Unnecessary Runs)

| Technique | Description | GitLab Implementation |
| :--- | :--- | :--- |
| **Conditional Jobs** | Ensure resource-intensive jobs (like E2E tests or large builds) run only when the related files have changed. | Use the **`rules:changes`** keyword to specify file patterns. |
| **Targeted Pipelines** | Restrict long-running jobs to specific branches or pipeline types (e.g., run full security scans only on the `main` branch or MRs). | Refine **`rules:if`** logic using `$CI_COMMIT_BRANCH` or `$CI_PIPELINE_SOURCE`. |
| **Pipeline Skip** | Completely skip an entire pipeline run based on commit messages (e.g., for documentation-only changes). | Add **`[ci skip]`** or **`[skip ci]`** to the commit message. |

#### 2. Artifact and Dependency Management (Maximizing Speed)

| Technique | Description | GitLab Implementation |
| :--- | :--- | :--- |
| **C H Caching** | **Cache** external dependencies (like `node_modules` or Python `pip` packages) to avoid downloading them in every job. | Use the **`cache`** keyword with a unique `key` and appropriate `paths`. |
| **Docker Layer Caching** | Reuse layers from previous Docker builds to significantly accelerate the image build process. | Implement **`cache_from`** in your Docker build job and tag images consistently. |
| **Artifact Optimization** | Only share the necessary artifacts between jobs. Large artifacts slow down both upload and download times. | Fine-tune the **`artifacts:paths`** to include only required folders (e.g., just the compiled binary, not the source code). |

#### 3. Image and Environment Optimization

| Technique | Description | GitLab Implementation |
| :--- | :--- | :--- |
| **Lightweight Images** | Use minimal base images (like **Alpine** or **Slim** variants) for CI jobs and application containers. | Use tags like **`node:20-alpine`** or **`python:3.9-slim`** in the `image` keyword. |
| **Multi-Stage Builds** | Separate the build environment (compiler, development tools) from the runtime environment (minimal OS). | Define multiple **`FROM`** stages in your Dockerfile. |
| **Dedicated Runners** | Direct jobs to specialized, performant Runners optimized for speed (e.g., high-CPU machines). | Use the **`tags`** keyword to select high-performance executors. |

#### 4. Test Execution Efficiency

| Technique | Description | GitLab Implementation |
| :--- | :--- | :--- |
| **Parallel Execution** | Split your test suite into multiple jobs that run concurrently across different Runners. | Use the **`parallel`** keyword at the job level. |
| **Test Splitting/Sharding** | Intelligently distribute tests among parallel jobs based on test file or historical duration. | Use specific test runners (like Playwright/Jest sharding) combined with GitLab's parallel variables (`$CI_NODE_INDEX`, `$CI_NODE_TOTAL`). |
| **Quick Feedback First** | Schedule fast jobs (linting, unit tests) earlier in the pipeline to provide rapid failure feedback, saving time on subsequent stages. | Organize the job order within the `stages` section. |

---

### Learning Outcomes

By the end of this section, you will be able to:

* Analyze your existing pipeline to identify performance bottlenecks.
* Implement strategic caching policies to maximize speed.
* Use `rules:changes` to create **smarter, more targeted pipelines** for monorepos and large projects.
* Build faster Docker images using effective layer caching.
* Leverage parallelism to reduce the wall-clock time of your testing stages.

Let's begin optimizing your pipeline's speed and cost-efficiency!

---

### Skipping Cloning the Git Repository: The `GIT_STRATEGY` Keyword 🏃

By default, every GitLab CI job begins by cloning the entire Git repository. For pipelines that run frequently or work with very large repositories (monorepos), this cloning process can be a significant time sink. The **`GIT_STRATEGY`** predefined variable allows you to control how the GitLab Runner handles the repository, giving you options to speed up pipeline execution by skipping the clone entirely or using shallow clones.

-----

### The `GIT_STRATEGY` Options

You set the `GIT_STRATEGY` either globally in the `variables:` section or at the job level. It accepts three possible values:

| Strategy | Description | When to Use |
| :---: | :--- | :--- |
| **`fetch` (Default)** | The Runner ensures the repository is present. If it already exists from a previous job, it performs a **`git fetch`** to update it. This is faster than `clone` but requires the work tree to be present. | Standard practice; avoids full clone but maintains a clean repository state between jobs. |
| **`clone`** | The Runner performs a full **`git clone`** every time, ensuring the repository is always a fresh copy. | When jobs are very sensitive to a clean environment (rarely needed). This is the slowest option. |
| **`none`** | The Runner **skips all Git operations** (no `clone`, no `fetch`). The repository directory may not exist or may contain files from a previous run. | **Highest efficiency.** Use this when your job only needs artifacts from previous stages or operates on a tiny subset of files. |

-----

### Practical Example: Maximizing Speed with `none`

Using `GIT_STRATEGY: none` is the fastest way to start a job, as it completely skips network operations. This is highly effective for jobs that only consume artifacts created by a previous job.

```yaml
# .gitlab-ci.yml

stages:
  - build
  - security
  - notify

build_image:
  stage: build
  image: docker:latest
  script:
    - docker build -t my-app:latest .
    - docker push my-app:latest
    - echo "DOCKER_IMAGE_TAG=my-app:latest" > build.env
  artifacts:
    reports:
      dotenv: build.env
    paths:
      - ./build_output/ # Create artifacts needed by later jobs

# This security scan only needs the build artifacts (and not the source code)
# so we skip the Git clone operation entirely.
security_scan_artifact:
  stage: security
  image: registry.gitlab.com/gitlab-org/security-products/dind/klar:latest
  # Use the fastest strategy: skip the Git repository clone
  variables:
    GIT_STRATEGY: none
  needs:
    - build_image # Ensures the build_image artifacts are downloaded
  script:
    # The artifact output is available in the current directory, no cloning needed
    - klar --output-file security_report.json .
  artifacts:
    paths:
      - security_report.json

# This job only needs the variables from the build job and the security report
notify_slack:
  stage: notify
  image: curlimages/curl
  variables:
    GIT_STRATEGY: none # Skip the clone for maximum speed
  needs:
    - build_image
    - security_scan_artifact
  script:
    - echo "Notifying Slack..."
    # Logic to send notification, using variables from previous jobs
    - curl -X POST -H 'Content-type: application/json' --data '{"text":"Build $DOCKER_IMAGE_TAG success!"}' $SLACK_WEBHOOK_URL
```

### Best Practices for `GIT_STRATEGY`

  * **Use `fetch` (Default) as the Baseline:** Only override `GIT_STRATEGY` if you have a specific performance reason. The default `fetch` is often the best balance between speed and reliability.
  * **Test `none` Carefully:** When setting `GIT_STRATEGY: none`, make sure your job truly *does not* need the source code. If it fails, check if you need to use the `fetch` strategy instead.
  * **Combine with `artifacts` and `needs`:** The `GIT_STRATEGY: none` is most effective when the job is consuming artifacts generated by a preceding job (`artifacts: true` in the `needs` list), as the necessary files are provided via the network instead of Git.
  * **For Large Monorepos:** Use a shallow clone (set by `GIT_DEPTH`) in combination with `fetch` to significantly reduce the size of the repository data the Runner pulls down, even if you can't skip the clone entirely.

-----
### Disabling Artifact Download from Previous Stages: The `dependencies` Keyword 📦

By default, in GitLab CI/CD, a job automatically downloads and extracts **all artifacts** generated by jobs in **all previous stages** of the pipeline. While this is convenient for sharing files, it often leads to unnecessary downloads, slows down the job startup time, and wastes storage resources, especially in large pipelines with many jobs and big artifacts.

The **`dependencies`** keyword gives you precise control over which, if any, artifacts from preceding jobs should be downloaded and made available to the current job.

-----

### How the `dependencies` Keyword Works

The `dependencies` keyword is defined at the job level and accepts an array of strings (job names).

1.  **`dependencies: []` (Empty Array):** This is the **most common optimization**. It instructs the Runner **not to download any artifacts** from any previous jobs. The current job will start much faster.
2.  **`dependencies: [<job_name>]`:** This specifies a list of **only the necessary jobs** whose artifacts should be downloaded. All other artifacts from preceding stages are ignored.
3.  **Default Behavior (Dependencies not specified):** The job downloads artifacts from *all* previous jobs in *all* previous stages.

-----

### Practical Example: Optimizing a Deployment Job

Consider a pipeline where the `build_app` job creates a large Docker image artifact, and the `unit_tests` job creates a small report artifact. The `deploy` job only needs the small report, not the large Docker image.

```yaml
# .gitlab-ci.yml

stages:
  - build
  - test
  - deploy

build_app:
  stage: build
  script:
    - echo "Building large artifact..."
    - mkdir large_artifact
    - echo "Big file" > large_artifact/data.txt
  artifacts:
    paths: [large_artifact]
    expire_in: 1 day

unit_tests:
  stage: test
  script:
    - echo "Creating small report..."
    - echo "PASS" > test_report.txt
  artifacts:
    paths: [test_report.txt]
    expire_in: 1 hour

# 1. Without 'dependencies': This job downloads BOTH large_artifact and test_report.txt.
deploy_job_slow:
  stage: deploy
  script:
    - cat large_artifact/data.txt # Accesses the unnecessary large artifact
    - cat test_report.txt        # Accesses the needed report

# 2. With 'dependencies': This job only downloads the needed artifact from unit_tests.
deploy_job_fast:
  stage: deploy
  dependencies:
    - unit_tests # Only download artifacts from the 'unit_tests' job
  script:
    - cat test_report.txt # Accesses the needed report
    - ls -a # 'large_artifact' will NOT be present in this directory
```

### Best Practices for `dependencies`

  * **Use `dependencies: []` as a Default:** If your job relies solely on its own code and public dependencies (e.g., a simple linting job or a final notification job), set `dependencies: []` to achieve the fastest possible startup.
  * **Be Explicit:** If you need artifacts, explicitly list only the jobs that create them. This ensures you only pull the required files.
  * **Combine with `needs`:** When using the `needs` keyword to run jobs out of stage order, `dependencies` is often used in the `needs` definition as well. If you define a job in `needs`, you still need to use `dependencies: []` to disable all *other* artifacts, but the artifacts from the jobs specified in `needs` will be downloaded by default unless explicitly excluded there.

By intelligently managing artifact dependencies, you significantly cut down on network transfer time, making your pipeline more cost-effective and much faster.

-----
### Stageless Pipelines: Running Jobs Out of Order with `needs` 🔗

By default, GitLab CI/CD pipelines are strictly sequential: jobs in a later stage cannot start until *all* jobs in the preceding stage have finished successfully. While this stage-based model is logical, it can introduce unnecessary delays, especially when jobs have no actual data dependency.

The **`needs`** keyword allows you to break this strict sequential stage ordering, enabling a **stageless or directed acyclic graph (DAG)** workflow. This is a crucial optimization for maximizing parallelization and achieving the fastest possible wall-clock time for your pipeline.

-----

### Why Use `needs`?

  * **Maximize Parallelization:** Run jobs in later stages as soon as their direct dependencies in earlier stages are complete, without waiting for the entire preceding stage to finish.
  * **Reduce Wall-Clock Time:** By eliminating unnecessary waiting time, `needs` significantly reduces the total time it takes for your pipeline to run from start to finish.
  * **Targeted Artifacts:** When combined with `dependencies`, `needs` ensures that a job only downloads artifacts from the specific jobs it requires, further speeding up execution.
  * **Explicit Dependencies:** Clearly maps the flow of data and control, making complex pipelines easier to visualize and understand.

-----

### How the `needs` Keyword Works

The `needs` keyword is defined at the job level and accepts an array of job names.

1.  **Job Execution:** A job specified in `needs` will start as soon as all jobs listed in its `needs` array have successfully completed, regardless of which stage those preceding jobs belong to.
2.  **Skipping Stages:** A stage will automatically be skipped if all jobs within it are run via the `needs` keyword and have no other dependencies within that stage.
3.  **Artifacts:** When you use `needs`, the artifacts from the jobs listed in the array are downloaded by default.

#### Example 1: Bypassing Sequential Stages

Consider a scenario where `deploy_to_staging` only requires the output of `build_frontend`, but typically waits for all `test` jobs to finish.

```yaml
# .gitlab-ci.yml

stages:
  - build
  - test
  - deploy

build_frontend: # Stage: build
  script: ["echo Built frontend artifact"]
  artifacts: { paths: ["frontend.zip"] }

unit_tests: # Stage: test (Takes 10 min)
  script: ["echo Running unit tests"]

deploy_to_staging: # Stage: deploy
  # This job normally waits for build_frontend AND unit_tests.
  # Using 'needs' tells it to *only* wait for build_frontend.
  needs: ["build_frontend"]
  script: ["echo Deploying using frontend.zip artifact..."]
```

In this example, `deploy_to_staging` starts immediately after `build_frontend` finishes (in the `build` stage), while `unit_tests` (in the `test` stage) runs in parallel, saving approximately 10 minutes of waiting time.

#### Example 2: Targeted Artifacts with `dependencies`

You can refine artifact downloads using the `dependencies` keyword within the `needs` array.

```yaml
e2e_tests:
  stage: test
  needs:
    # Wait for build_api to finish, and download its artifacts
    - job: build_api
      artifacts: true 
    # Wait for setup_database to finish, but DO NOT download its artifacts
    - job: setup_database
      artifacts: false 
  script:
    - echo "Running E2E tests against the API artifact..."
```

This is a powerful combination: you force the dependency (`needs`), but explicitly control the data transfer (`artifacts: false`), minimizing I/O overhead.

-----

### Best Practices for `needs`

  * **Use for Efficiency, Not Structure:** While `needs` breaks strict stage ordering, you should still use the `stages` keyword to define the *logical* flow of your application (Build -\> Test -\> Deploy).
  * **Identify Independent Paths:** Look for jobs that belong to different functional paths (e.g., the security path, the frontend path, the database path). These are ideal candidates for parallelization using `needs`.
  * **Combine with `dependencies: []`:** If a job requires only one artifact from one preceding job, explicitly set `dependencies: []` to prevent the job from downloading *other* unnecessary artifacts from the jobs specified in `needs`.
  * **Use the Visualization Tab:** GitLab's pipeline visualization graph (DAG) makes it easy to see the dependency connections created by the `needs` keyword, ensuring you haven't introduced any unintended circular dependencies.

By leveraging the `needs` keyword, you move your pipeline from a rigid, waterfall-style sequence to a dynamic, high-performance graph that optimizes every minute of execution time.

-----
### Stageless Pipelines: Running Jobs Out of Order with `needs` 🔗

By default, GitLab CI/CD pipelines are strictly sequential: jobs in a later stage cannot start until *all* jobs in the preceding stage have finished successfully. While this stage-based model is logical, it can introduce unnecessary delays, especially when jobs have no actual data dependency.

The **`needs`** keyword allows you to break this strict sequential stage ordering, enabling a **stageless or directed acyclic graph (DAG)** workflow. This is a crucial optimization for maximizing parallelization and achieving the fastest possible wall-clock time for your pipeline.

-----

### Why Use `needs`?

  * **Maximize Parallelization:** Run jobs in later stages as soon as their direct dependencies in earlier stages are complete, without waiting for the entire preceding stage to finish.
  * **Reduce Wall-Clock Time:** By eliminating unnecessary waiting time, `needs` significantly reduces the total time it takes for your pipeline to run from start to finish.
  * **Targeted Artifacts:** When combined with `dependencies`, `needs` ensures that a job only downloads artifacts from the specific jobs it requires, further speeding up execution.
  * **Explicit Dependencies:** Clearly maps the flow of data and control, making complex pipelines easier to visualize and understand.

-----

### How the `needs` Keyword Works

The `needs` keyword is defined at the job level and accepts an array of job names.

1.  **Job Execution:** A job specified in `needs` will start as soon as all jobs listed in its `needs` array have successfully completed, regardless of which stage those preceding jobs belong to.
2.  **Skipping Stages:** A stage will automatically be skipped if all jobs within it are run via the `needs` keyword and have no other dependencies within that stage.
3.  **Artifacts:** When you use `needs`, the artifacts from the jobs listed in the array are downloaded by default.

#### Example 1: Bypassing Sequential Stages

Consider a scenario where `deploy_to_staging` only requires the output of `build_frontend`, but typically waits for all `test` jobs to finish.

```yaml
# .gitlab-ci.yml

stages:
  - build
  - test
  - deploy

build_frontend: # Stage: build
  script: ["echo Built frontend artifact"]
  artifacts: { paths: ["frontend.zip"] }

unit_tests: # Stage: test (Takes 10 min)
  script: ["echo Running unit tests"]

deploy_to_staging: # Stage: deploy
  # This job normally waits for build_frontend AND unit_tests.
  # Using 'needs' tells it to *only* wait for build_frontend.
  needs: ["build_frontend"]
  script: ["echo Deploying using frontend.zip artifact..."]
```

In this example, `deploy_to_staging` starts immediately after `build_frontend` finishes (in the `build` stage), while `unit_tests` (in the `test` stage) runs in parallel, saving approximately 10 minutes of waiting time.

#### Example 2: Targeted Artifacts with `dependencies`

You can refine artifact downloads using the `dependencies` keyword within the `needs` array.

```yaml
e2e_tests:
  stage: test
  needs:
    # Wait for build_api to finish, and download its artifacts
    - job: build_api
      artifacts: true 
    # Wait for setup_database to finish, but DO NOT download its artifacts
    - job: setup_database
      artifacts: false 
  script:
    - echo "Running E2E tests against the API artifact..."
```

This is a powerful combination: you force the dependency (`needs`), but explicitly control the data transfer (`artifacts: false`), minimizing I/O overhead.

-----

### Best Practices for `needs`

  * **Use for Efficiency, Not Structure:** While `needs` breaks strict stage ordering, you should still use the `stages` keyword to define the *logical* flow of your application (Build -\> Test -\> Deploy).
  * **Identify Independent Paths:** Look for jobs that belong to different functional paths (e.g., the security path, the frontend path, the database path). These are ideal candidates for parallelization using `needs`.
  * **Combine with `dependencies: []`:** If a job requires only one artifact from one preceding job, explicitly set `dependencies: []` to prevent the job from downloading *other* unnecessary artifacts from the jobs specified in `needs`.
  * **Use the Visualization Tab:** GitLab's pipeline visualization graph (DAG) makes it easy to see the dependency connections created by the `needs` keyword, ensuring you haven't introduced any unintended circular dependencies.

By leveraging the `needs` keyword, you move your pipeline from a rigid, waterfall-style sequence to a dynamic, high-performance graph that optimizes every minute of execution time.
-----
