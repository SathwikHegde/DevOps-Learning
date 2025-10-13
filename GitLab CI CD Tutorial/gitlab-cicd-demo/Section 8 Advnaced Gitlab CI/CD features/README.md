### Section 8: Advanced GitLab CI - Pushing the Boundaries of Your Pipeline 🚀

Welcome to Section 8 of our journey! Having mastered the fundamentals of GitLab CI/CD, this section will elevate your skills to the next level. We'll move beyond basic builds and tests to explore advanced features and best practices that make your pipelines more efficient, secure, and maintainable.

This section focuses on the techniques used by professional DevOps teams to build scalable, flexible, and powerful CI/CD workflows.

***

### Why Advanced CI/CD Matters

A basic pipeline is great, but a mature pipeline is a strategic asset. Advanced CI/CD techniques enable you to:

* **Optimize Performance:** Reduce build times and lower costs by running jobs only when necessary and reusing results.
* **Enhance Security:** Proactively scan for vulnerabilities in both your code and your dependencies.
* **Increase Flexibility:** Build a pipeline that can adapt to different application types and deployment environments with a single, reusable configuration.
* **Improve Maintainability:** Keep your `.gitlab-ci.yml` file clean, dry, and easy to understand as your project grows.
* **Automate More:** Extend the reach of your pipeline beyond just building and deploying, into tasks like dynamic security testing and code analysis.

***

### Section Overview: What You'll Learn

In this section, we will dive into the following advanced topics:

* **Advanced Job Rules:** Go beyond simple `only/except` statements to use complex, conditional logic with `rules` to control when jobs run.
* **CI/CD Templates and Includes:** Learn how to create reusable job templates and share them across projects, making your pipeline configurations DRY (Don't Repeat Yourself).
* **Dynamic Pipeline Generation:** Master the use of keywords like `extends` and `include` to build modular, maintainable, and flexible pipelines.
* **Docker Multi-Stage Builds:** Learn how to create smaller, more secure Docker images by separating your build environment from your production runtime.
* **Pipeline for Monorepos:** Discover how to configure a pipeline that intelligently runs jobs only for the parts of a monorepo that have changed.
* **Vulnerability Scanning:** Integrate automated security scans (SAST, DAST, Dependency Scanning) to find and fix security issues early in the development cycle.
* **Auto DevOps:** Explore how GitLab's powerful Auto DevOps feature can automatically detect, build, test, and deploy your applications with zero configuration.

***

### Learning Outcomes: By the End of This Section, You Will Be Able To...

* Write highly efficient and conditional `.gitlab-ci.yml` files using advanced `rules`.
* Create reusable CI/CD templates that can be shared across multiple projects.
* Build optimized, multi-stage Docker images for production.
* Implement a smarter, more targeted pipeline for monorepos.
* Proactively find security vulnerabilities in your applications using GitLab's built-in scanning tools.
* Understand the full potential of GitLab CI/CD as an end-to-end DevOps platform.

***
### Retrying Failed Jobs Automatically: The `retry` Keyword 🔄

In a CI/CD pipeline, some jobs may fail intermittently due to temporary network issues, race conditions, or other transient problems. These are often called **flaky failures**. Manually restarting a job every time this happens is a waste of time and a source of frustration. GitLab's `retry` keyword provides a simple, yet powerful solution to this problem by automatically re-running failed jobs.

-----

### How the `retry` Keyword Works

The `retry` keyword is defined at the job level. It accepts an integer value that specifies the number of times GitLab should automatically retry the job before marking it as a final failure.

  * `retry: <number>`: Retries a job up to `<number>` times. For example, `retry: 2` means GitLab will try the job a total of three times (the initial run plus two retries).
  * By default, `retry` is set to `0` (no retries). The maximum value you can set is `2`.

-----

### The Problem of Flaky Tests

A common use case for `retry` is with E2E (end-to-end) tests. These tests can sometimes fail due to temporary network latency or a UI element not loading in time. A simple retry can often resolve these issues, allowing the pipeline to proceed without human intervention. This prevents your pipeline from being unnecessarily blocked by transient failures.

-----

### Step-by-Step Guide

#### 1\. Add the `retry` Keyword to a Job

You add the `retry` keyword directly to the job you want to automatically retry.

```yaml
# .gitlab-ci.yml

stages:
  - test
  - deploy

# A job that runs E2E tests, which can sometimes be flaky
e2e_tests:
  stage: test
  image: mcr.microsoft.com/playwright/node:lts-slim
  
  script:
    - echo "Running E2E tests..."
    # The command to run your tests
    - npx playwright test

  # The `retry` keyword is added here
  retry: 2

# A deployment job that depends on the tests
deploy_to_staging:
  stage: deploy
  # This job will only run if the 'e2e_tests' job succeeds (after all retries)
  script:
    - echo "Deploying to staging..."
```

#### 2\. Observe the Pipeline Behavior

When you run this pipeline, if the `e2e_tests` job fails on its first attempt, GitLab will:

1.  Automatically restart the job.
2.  If the job succeeds on the retry, the pipeline will continue as normal.
3.  If the job fails on its second attempt, GitLab will automatically restart it for a final time.
4.  If the job fails on the third attempt, it will be marked as a final failure, and the pipeline will stop.

In the pipeline view, retried jobs are clearly marked with a retry count, so you can easily identify them. This provides valuable insights into which jobs are flaky and might need to be improved.

-----

### What's Next?

While the `retry` keyword is a quick and effective solution for dealing with flaky jobs, it should not be a long-term fix for poorly written code or unstable tests. A best practice is to use `retry` as a temporary solution while you investigate and fix the root cause of the intermittent failures. It should be seen as a safety net, not a replacement for reliable code.

### Allowing Failures: The `allow_failure` Keyword ⚙️

In a GitLab CI/CD pipeline, not all job failures are critical. Some jobs, like code quality reports or performance checks, are intended to provide information and warnings rather than act as a strict gate for the pipeline. The `allow_failure` keyword provides a way to mark a job as non-critical. If a job with `allow_failure: true` fails, the pipeline will continue to run, but the job will be marked with a yellow warning icon.

-----

### Why Use `allow_failure`?

  * **Non-Critical Feedback:** It allows you to integrate jobs that provide valuable feedback (like a security scan or linter report) without blocking the entire pipeline.
  * **Pipeline Efficiency:** Your pipeline can continue running, and you can review the failures of non-critical jobs later.
  * **Flexibility:** It gives you granular control over which jobs are strict gates and which are for informational purposes.

-----

### Step-by-Step Guide

#### 1\. Add the `allow_failure` Keyword to a Job

You add the `allow_failure` keyword directly to the job you want to allow to fail.

```yaml
# .gitlab-ci.yml

stages:
  - build
  - test
  - review

build_job:
  stage: build
  script:
    - echo "Building app..."

e2e_tests:
  stage: test
  script:
    - echo "Running E2E tests..."
    - exit 1 # This job will fail

# This job is for code quality reports and is allowed to fail
code_quality_check:
  stage: test
  script:
    - echo "Running code quality check..."
    - exit 1 # This job will fail
  allow_failure: true

deploy_review_app:
  stage: review
  script:
    - echo "Deploying review app..."
```

In this example, even though `code_quality_check` fails, the `e2e_tests` and `deploy_review_app` jobs will still run.

-----

### What's Next?

The `allow_failure` keyword is often used in conjunction with other features like `rules` and `when`. For example, you might have a job that only runs for a merge request and is allowed to fail, but the same job is a strict gate when it runs on the main branch. This provides a balance between giving fast feedback and ensuring quality in your main branch.

### YAML Anchors and Aliases: Streamlining Your `.gitlab-ci.yml` ⚓

As your GitLab CI/CD pipeline grows, your `.gitlab-ci.yml` file can become long and repetitive, particularly when many jobs share identical configurations (like the `before_script`, `image`, or `rules`). **YAML Anchors (`&`) and Aliases (`*`)** provide a powerful mechanism to reuse blocks of configuration, making your pipeline definition significantly cleaner, more concise, and easier to maintain.

Using anchors and aliases is a core best practice for building mature and scalable CI/CD pipelines.

-----

### 1\. The Core Concept: DRY (Don't Repeat Yourself)

YAML anchors and aliases allow you to define a configuration block once (the anchor) and reference it many times (the alias). This prevents repetition, meaning you only have to update a configuration in one place, and all jobs referencing it are instantly updated.

| Symbol | Name | Purpose | Example |
| :---: | :---: | :--- | :---: |
| **`&`** | **Anchor** | **Defines** the reusable block of configuration. | `&test_setup` |
| **`*`** | **Alias** | **References** the anchored block, copying its content. | `*test_setup` |
| **`<<:`** | **Merge Key** | **Merges** the referenced block into the current mapping. | `<<: *test_setup` |

-----

### 2\. Step-by-Step Implementation

The most common and effective way to use anchors and aliases in GitLab CI/CD is to define reusable blocks at the **top-level** of your `.gitlab-ci.yml` file, often prefixed with a dot (`.`) so they are treated as abstract configurations and are not executed as jobs themselves.

#### Example: Reusing the `before_script` and `image`

Let's simplify a pipeline where both `unit_tests` and `integration_tests` use the same setup.

```yaml
# .gitlab-ci.yml

stages:
  - test
  - deploy

# 1. Define the reusable block (the ANCHOR)
.default_test_setup: &common_test_config
  image: node:18-alpine
  before_script:
    - npm install --silent
    - echo "Dependencies installed."
  tags:
    - docker

# -------------------------------------------------------------

unit_tests:
  stage: test
  # 2. Reference and merge the anchor (the ALIAS and MERGE KEY)
  <<: *common_test_config
  script:
    - npm run test:unit
  # This job now automatically includes the image and before_script from the anchor.

integration_tests:
  stage: test
  <<: *common_test_config
  script:
    - npm run test:integration
  # This job also includes the common setup.
```

-----

### 3\. Advanced Use: Overriding and Extending

The beauty of the YAML merge key (`<<:`) is that it intelligently merges configuration, allowing you to easily override specific keys from the anchored block.

#### Example: Overriding a Key

In the example below, the `deploy_production` job uses the shared `docker_setup` but overrides the `rules` to require a manual action.

```yaml
# .gitlab-ci.yml

# Define the common Docker setup
.default_docker_build_config: &docker_build_config
  image: docker:latest
  services:
    - docker:dind
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: on_success # Default: Automated deployment

# -------------------------------------------------------------

deploy_staging:
  stage: deploy
  <<: *docker_build_config # Merge all common Docker setup
  script:
    - echo "Deploying automatically to staging..."

deploy_production:
  stage: deploy
  <<: *docker_build_config # Merge the common Docker setup
  rules: # OVERRIDE the rules defined in the anchor
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual # Change from automated to manual
  script:
    - echo "Deployment requires manual approval..."
```

**Key Result:** The `deploy_production` job inherits the `image` and `services` but uses its local, customized `rules` block. The merge key (`<<:`) always prioritizes keys defined *locally* over those referenced in the anchor.

-----

### 4\. Best Practices for Anchors and Aliases

  * **Use Abstract Names:** Prefix reusable blocks with a dot (`.`) (e.g., `.common_settings`) to prevent GitLab from recognizing them as runnable jobs.
  * **Define Anchors at the Top:** Place all your anchor definitions at the beginning of your file for easy readability and maintenance.
  * **Keep Anchors Focused:** Create anchors that are small and focused on one specific area (e.g., `.docker_service_setup`, `.test_dependencies`). This makes them easier to combine and manage.
  * **Combine Anchors:** You can combine multiple anchors within a single job to build complex configurations from smaller, reusable parts. This significantly improves pipeline maintainability and reduces file size.

By implementing YAML anchors and aliases, you transform a sprawling configuration file into a clean, modular, and professional CI/CD pipeline definition.

-----

### Job Templates with YAML Anchors and Aliases: A Practical Example ⚓

YAML Anchors (`&`) and Aliases (`*`) are powerful tools for creating reusable configuration blocks, effectively turning them into job templates. This is a core practice for achieving **DRY (Don't Repeat Yourself)** principles in your `.gitlab-ci.yml` file, making your pipeline highly maintainable and readable.

This guide provides a practical example of creating and consuming a base test template to define multiple testing jobs quickly.

-----

### The Goal: Creating Reusable Job Templates

We want three testing jobs (`unit_tests`, `lint_check`, `e2e_tests`) that share the following common configuration:

1.  Use the same **base Docker image**.
2.  Run the same **dependency installation** steps.
3.  Are restricted to only run on the `main` branch or a **Merge Request pipeline**.

By using a YAML anchor, we define this common configuration only once.

-----

### Step 1: Define the Base Template (The Anchor)

Create an abstract configuration block at the top of your `.gitlab-ci.yml`. We name it with a leading dot (`.`) so GitLab does not recognize it as a runnable job.

```yaml
# .gitlab-ci.yml

stages:
  - test
  - security
  - deploy

# 1. THE ANCHOR: Define the reusable template block
.base_test_template: &test_definition
  # Configuration shared by all jobs:
  image: node:20-alpine
  before_script:
    - npm install --silent
    - echo "Dependencies ready."
  # Rules shared by all jobs: restrict to main branch or MR pipelines
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: on_success
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: on_success
```

  * **`&test_definition`**: This is the **Anchor** name. It marks this entire block as reusable.

-----

### Step 2: Consume the Template (The Alias and Merge Key)

Now, we define our specific jobs and use the **Merge Key** (`<<:`) to pull in the configuration from the anchor. We only need to define the specific `stage` and the unique `script` for each job.

```yaml
# .gitlab-ci.yml (Continued)

# --- Consuming the Template ---

unit_tests:
  stage: test
  <<: *test_definition # ALIAS: Merge all config from the 'test_definition' anchor
  script:
    - npm run test:unit # Unique command for this job
  # This job inherits image, before_script, and rules.

lint_check:
  stage: test
  <<: *test_definition # ALIAS: Merge all config from the 'test_definition' anchor
  script:
    - npm run lint # Unique command for this job
  allow_failure: true # Local OVERRIDE: This job is non-critical

e2e_tests:
  stage: test
  <<: *test_definition # ALIAS: Merge all config from the 'test_definition' anchor
  script:
    - npm run start &
    - sleep 10 # Wait for app to start
    - npx playwright test # Unique command for this job
  timeout: 15 minutes # Local ADDITION: Add a specific timeout for E2E tests

# -------------------------------------------------------------
```

-----

### Practical Advantages Demonstrated

1.  **Readability:** The definition of each job is now very concise, focusing only on its unique purpose (`script`).
2.  **Maintenance:** If you decide to change the base image from `node:20-alpine` to `node:22-alpine`, you only need to update the image tag in **one place** (`.base_test_template`). All three jobs automatically inherit the change.
3.  **Flexibility (Overriding):** The `lint_check` job demonstrates how to easily **override** or **add** local settings. It inherits all the common setup but locally sets `allow_failure: true`, showing how templates provide a strong baseline while still permitting customization.

This approach ensures that your GitLab CI configuration is modular, easy to scale, and professional.

-----