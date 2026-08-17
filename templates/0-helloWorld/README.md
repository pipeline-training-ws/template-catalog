## Overview

This directory contains a Jenkins pipeline template that loads the [`shared-library`](../../../shared-library) Shared Library and runs on a Kubernetes agent.
The pipeline executes the shared library's `helloWorld` step across two explicit stages (`Stage1`, `Stage2`).

## Prerequisites

Before running this pipeline, ensure you have the following:

- Jenkins with Kubernetes plugin installed
- A Kubernetes cluster configured as an agent for Jenkins
- The following Jenkins plugins installed:
    - **Pipeline: Shared Groovy Libraries**
    - **Kubernetes Plugin**
    - **Git Plugin**

## Setting Up the Shared Library

This pipeline resolves the Shared Library's Git coordinates from environment variables (each with a sensible default), so they can be overridden at the folder or controller level without touching the Jenkinsfile:

```groovy
env.SHAREDLIB_GIT_SERVER = env.SHAREDLIB_GIT_SERVER ?: "https://github.com"
env.SHAREDLIB_GIT_ORG = env.SHAREDLIB_GIT_ORG ?: "pipeline-training-ws"
env.SHAREDLIB_GIT_REPO = env.SHAREDLIB_GIT_REPO ?: "shared-library"
env.SHAREDLIB_GIT_TAG_DEFAULT = env.SHAREDLIB_GIT_TAG_DEFAULT ?: "main"
env.SHAREDLIB_GIT_CREDENTIALS = env.SHAREDLIB_GIT_CREDENTIALS ?: "gh-pat"

library identifier: "${env.SHAREDLIB_GIT_REPO}@${env.SHAREDLIB_GIT_TAG_}", retriever: modernSCM(
        [$class: 'GitSCMSource',
         remote: "${env.SHAREDLIB_GIT_SERVER}/${env.SHAREDLIB_GIT_ORG}/${env.SHAREDLIB_GIT_REPO}.git",
         credentialsId: "${env.SHAREDLIB_GIT_CREDENTIALS}"
        ]
)
```

If a job or folder sets `SHAREDLIB_GIT_TAG`, that value wins over `SHAREDLIB_GIT_TAG_DEFAULT`, letting a single branch pin to a specific Shared Library tag without changing the default used by everyone else.

## Agent Configuration

This pipeline runs on a Kubernetes agent using the shared library's general-purpose pod template, loaded via `libraryResource` and set as the default container:

```groovy
def pod = libraryResource 'podtemplates/agent.yaml'

pipeline {
    agent {
        kubernetes {
            defaultContainer 'maven'
            yaml pod
        }
    }
```

- The `defaultContainer` is set to `maven` (the only container defined in `podtemplates/agent.yaml`).
- The `yaml` variable loads the pod template YAML from the shared library's `resources/podtemplates/agent.yaml`.

## Pipeline Stages

The pipeline consists of the following stages:

### **1. Stage1**

- Runs in the default container (`maven`).
- Prints the hostname of the agent.
- Calls the shared library's `helloWorld` step with `firstname` and `lastname`.
- `firstname` and `lastname` are declared in `template.yaml`.

### **2. Stage2**

- Repeats the steps from `Stage1`, but explicitly declares `container("maven")`.

Example stage execution:

```groovy
stage('Stage1') {
    steps {
        sh 'hostname'
        helloWorld "${firstname}"
        helloWorld "${lastname}"
    }
}
```

## Running the Pipeline

To execute this pipeline:

1. Ensure Jenkins is configured with access to the Kubernetes cluster.
2. Optionally override `SHAREDLIB_GIT_SERVER`, `SHAREDLIB_GIT_ORG`, `SHAREDLIB_GIT_REPO`, `SHAREDLIB_GIT_TAG`, or `SHAREDLIB_GIT_CREDENTIALS` at the folder/controller level.
3. Run the pipeline from Jenkins UI or via a Jenkinsfile, supplying the `firstname`/`lastname` template parameters.

## Troubleshooting

- If the shared library fails to load, verify `SHAREDLIB_GIT_SERVER` / `SHAREDLIB_GIT_ORG` / `SHAREDLIB_GIT_REPO`, and that `SHAREDLIB_GIT_CREDENTIALS` has access to the repo.
- If the Kubernetes agent does not start, check `resources/podtemplates/agent.yaml` in the shared library and ensure the Kubernetes plugin is installed and configured correctly.

## Contributing

Feel free to submit issues or pull requests to improve this pipeline setup.

## License

This project is licensed under the MIT License.


