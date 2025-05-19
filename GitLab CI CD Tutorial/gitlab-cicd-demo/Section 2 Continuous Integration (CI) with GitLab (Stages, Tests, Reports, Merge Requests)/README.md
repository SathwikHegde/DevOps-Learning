## Section 1: Understanding gitlab basics

## Section 2: Continuous Integration (CI) with GitLab

### Diving into Continuous Integration

We've explored the fundamental concepts and the crucial benefits of Continuous Integration. Now, let's put those principles into practice within the GitLab ecosystem.

### Setting the Stage: Forking the Repository

To get started, we forked the `learn-gitlab-app` repository: [https://gitlab.com/devops-projects9112313/learn-gitlab-app](https://gitlab.com/devops-projects9112313/learn-gitlab-app). This provided us with a working codebase to experiment with.

During this step, we also gained familiarity with:

  * **Dependencies:** Understanding how our application relies on external libraries.
  * **npm (Node Package Manager):** A powerful tool for managing JavaScript project dependencies.
  * `package.json`: The blueprint of our project, outlining its dependencies and scripts.
  * `package-lock.json`: Ensuring consistent dependency versions across different environments.

We then took the application for a spin using the following commands:

```bash
npm install -g serve
serve -s build/
```

This allowed us to serve the application locally and confirm its basic functionality.

### Leveling Up: Utilizing Docker for a Consistent Build Environment

As we aim for reliable and reproducible CI pipelines, Docker emerges as a powerful ally. By containerizing our build environment, we can eliminate inconsistencies that might arise from different local setups or build agents.

### Choosing an appropriate Docker image

The first crucial step in leveraging Docker for our builds is selecting a suitable Docker image as our base. This image will provide the necessary operating system and pre-installed tools for our project. For our learn-gitlab-app, which is a Node.js application, a logical choice would be an official Node.js image from Docker Hub.

These official images come in various flavors, often differentiated by the underlying Linux distribution and the specific Node.js version. For instance, you might encounter images based on Alpine Linux (known for its small size) or Debian. You'll also find tags specifying the Node.js version (e.g., node:18, node:18-alpine).

When making this decision, consider factors such as:

 1) **Node.js Version Compatibility:** Ensure the image has the Node.js version (or a compatible one) required by your application as specified in your package.json.
 2) **Image Size:** Smaller images, like those based on Alpine, can lead to faster build times and reduced storage. However, they might require installing additional packages if your build process has more dependencies.
 3) **Pre-installed Tools:** Some images might include common build tools, which can be convenient. However, it's often better to keep the image minimal and install only what you need in your CI pipeline for better control and security.
  
For our learn-gitlab-app, a good starting point might be one of the official Node.js images, such as node:lts-alpine (which provides a small Alpine Linux base with the current Long Term Support version of Node.js).

By starting with a well-chosen Docker image, we lay a solid foundation for a consistent and efficient CI process.

###  Choosing an appropriate Docker image

#### Choosing the Right Foundation: Exploring Lightweight Docker Images

As we discussed, selecting the initial Docker image is a critical step. For optimizing our build process, particularly in terms of speed and size, lightweight Docker images like those based on Alpine Linux or the "slim" variants of other distributions are often excellent choices.

####  Lightweight Docker images (alpine, slim)
#### The Benefits of Lightweight Images

* **Smaller Size:** Lightweight images inherently have a smaller footprint. This translates to faster download and upload times during CI pipeline execution, as well as reduced storage requirements for your Docker registry.
* **Faster Build Times:** Due to their minimal nature, fewer layers need to be processed when building your Docker image, potentially leading to quicker build times.
* **Reduced Security Footprint:** A smaller set of installed packages generally means a reduced attack surface, potentially enhancing the security of your build environment.

#### Alpine Linux: Minimal and Efficient

Alpine Linux is a security-oriented, lightweight Linux distribution based on musl libc and busybox. Its small size makes it a popular choice for base Docker images. Official Node.js Alpine-based images (e.g., `node:lts-alpine`) are readily available and provide a minimal environment to run Node.js applications.

**Considerations when using Alpine:**

* **Package Manager (apk):** Alpine uses the `apk` package manager, which is different from the `apt` (Debian/Ubuntu) or `yum`/`dnf` (CentOS/Fedora) package managers you might be more familiar with.
* **libc Implementation (musl vs. glibc):** While generally compatible, some native Node.js modules might have specific dependencies or build requirements related to the `glibc` library, which is more common in other distributions. This can sometimes lead to build issues requiring specific workarounds.

#### "Slim" Variants: Stripped-Down Alternatives

Many official Docker images, including those for Node.js (e.g., `node:lts-slim`), offer "slim" variants. These are typically based on more common distributions like Debian but are stripped down to contain only the essential packages required to run the specified software.

**Advantages of "slim" images:**

* **Familiarity:** Often based on widely used distributions, which can make troubleshooting and installing additional packages more straightforward.
* **Potentially Better Compatibility:** May have better compatibility with native modules that have `glibc` dependencies.

#### Making the Choice for `learn-gitlab-app`

For our `learn-gitlab-app`, starting with `node:lts-alpine` is a reasonable approach to benefit from its small size and efficiency. If we encounter any compatibility issues with native modules later on, we could then consider switching to a `node:lts-slim` image.

By carefully considering the trade-offs between size, familiarity, and potential compatibility, we can select the most appropriate lightweight Docker image for our CI pipeline.

