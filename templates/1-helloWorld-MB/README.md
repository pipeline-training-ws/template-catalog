## Overview

This directory contains a Jenkins pipeline template intended for **MultiBranch Pipeline Projects** (or Organization Folders). Unlike [`0-helloWorld`](../0-helloWorld), the entire pipeline — agent, stages, and steps — is externalized into the Shared Library. This `Jenkinsfile` only resolves and loads the library, then delegates to a single global pipeline step.

## Prerequisites

Before running this pipeline, ensure you have the following:

- A Jenkins MultiBranch Pipeline Project (or Organization Folder) with a marker file (`ci-config.yaml`) in each branch/repo
- The following Jenkins plugins installed:
  - **Pipeline: Shared Groovy Libraries**
  - **Git Plugin**
  - **Kubernetes Plugin** (the agent/pod definition lives inside the Shared Library, not in this Jenkinsfile)

## Loading the Shared Library

Instead of a fixed `library` declaration, this Jenkinsfile resolves the Shared Library coordinates from environment variables (with defaults), so they can be overridden at the folder or controller level without touching the Jenkinsfile:

```groovy
env.SHAREDLIB_GIT_SERVER = env.SHAREDLIB_GIT_SERVER ?: "https://github.com"
env.SHAREDLIB_GIT_ORG = env.SHAREDLIB_GIT_ORG ?: "pipeline-training-ws"
env.SHAREDLIB_GIT_REPO = env.SHAREDLIB_GIT_REPO ?: "shared-library"
env.SHAREDLIB_GIT_TAG_DEFAULT = env.SHAREDLIB_GIT_TAG_DEFAULT ?: "main"
env.SHAREDLIB_GIT_CREDENTIALS = env.SHAREDLIB_GIT_CREDENTIALS ?: "gh-pat"
```

| Variable                    | Default                  | Purpose                                                        |
| ---------------------------- | ------------------------- | --------------------------------------------------------------- |
| `SHAREDLIB_GIT_SERVER`       | `https://github.com`      | Git server hosting the Shared Library                          |
| `SHAREDLIB_GIT_ORG`          | `pipeline-training-ws`    | Organization/user owning the Shared Library repo                |
| `SHAREDLIB_GIT_REPO`         | `shared-library`          | Shared Library repository name                                  |
| `SHAREDLIB_GIT_TAG_DEFAULT`  | `main`                    | Branch/tag loaded when `SHAREDLIB_GIT_TAG` isn't set             |
| `SHAREDLIB_GIT_CREDENTIALS`  | `gh-pat`                  | Credentials ID used to fetch the library                        |

If a job or folder sets `SHAREDLIB_GIT_TAG`, that value wins over `SHAREDLIB_GIT_TAG_DEFAULT` — this lets a single branch pin to a specific Shared Library tag (e.g. `dev`) without changing the default used by everyone else.

> ⚠️ For MultiBranch Pipelines, prefer a **GitHub App** over a PAT for `SHAREDLIB_GIT_CREDENTIALS` — a PAT (`gh-pat`) is tied to a single user account.

## Pipeline Delegation

Once the library is loaded, the entire pipeline is delegated to a single global step:

```groovy
pipelineTemplateHelloWorld('ci-config.yaml')
```

- `'ci-config.yaml'` is the marker file expected in each branch/repo; the Shared Library reads it to configure the run.
- The agent and stage definitions live inside the Shared Library (`pipelineTemplateHelloWorld`), not in this Jenkinsfile — consult the Shared Library's own documentation for what it executes.

## Template Parameters

Declared in [`template.yaml`](template.yaml). This template has `templateType: MULTIBRANCH` and configures the MultiBranch project's GitHub branch source directly from its parameters:

| Parameter     | Type        | Default                | Display name        |
| ------------- | ----------- | ----------------------- | -------------------- |
| `repoName`    | `string`      | `sample-app-helloWorld` | Git Repository       |
| `organisation`| `string`      | `pipeline-training-ws`  | GH Organisation       |
| `githubToken` | `CREDENTIALS` | `gh-app`                | GitHubUserToken       |

These feed the `multibranch.branchSource.github` block (`repoOwner`, `repository`, `credentialsId`) and the `markerFile: ci-config.yaml` setting — see [`template.yaml`](template.yaml) and the [CloudBees docs on MultiBranch template parameters](https://docs.cloudbees.com/docs/admin-resources/latest/pipeline-templates-user-guide/managing-multibranch-pipeline-options#_example).

## Running the Pipeline

1. Register this template in a Pipeline Template Catalog (see the [top-level README](../../README.md)).
2. Reference the template from a MultiBranch Pipeline Project or Organization Folder.
3. Ensure each branch/repo provides the `ci-config.yaml` marker file expected by `pipelineTemplateHelloWorld`.
4. Supply the `repoName`, `organisation`, and `githubToken` template parameters when the template is instantiated.

## Troubleshooting

- If the Shared Library fails to load, verify `SHAREDLIB_GIT_SERVER` / `SHAREDLIB_GIT_ORG` / `SHAREDLIB_GIT_REPO`, and that `SHAREDLIB_GIT_CREDENTIALS` has access to the repo.
- If `pipelineTemplateHelloWorld` fails, check that `ci-config.yaml` exists in the branch/repo and matches what the Shared Library expects.
