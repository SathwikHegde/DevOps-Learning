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