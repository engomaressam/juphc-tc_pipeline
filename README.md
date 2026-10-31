# Tekton CI pipeline for a Node.js tax calculator

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)

A Tekton pipeline that takes a Node.js app from a Git repository to a container image in IBM Cloud Container Registry. Each push runs install, unit tests and image build as separate tasks, sharing one workspace on a persistent volume. I wrote it for the final project of IBM's *Get Started with Cloud Native, DevOps, Agile, and NoSQL* course (Full-Stack JavaScript Developer Professional Certificate).

## Pipeline

```
init (cleanup) ──► clone (git-clone) ──► npminstall (npm) ──► tests (jasmine) ──► build (buildah ──► us.icr.io)
```

| Task | Defined in | What it does |
|---|---|---|
| `init` | `tasks.yaml` (`cleanup`) | Empties the shared workspace, including hidden files, without deleting the mount point |
| `clone` | `git-clone` ClusterTask | Checks out `repo-url` at `branch` |
| `npminstall` | `tasks.yaml` (`npm`) | Cleans the npm cache and runs `npm install` on `node:20-buster-slim` |
| `tests` | `tasks.yaml` (`jasmine`) | Runs the Jasmine unit tests with `npx jasmine` |
| `build` | `buildah` ClusterTask | Builds the image and pushes it to `us.icr.io/<namespace>/<app-name>` |

## Files

- `pipeline.yaml`: the `juphc-tc-pipeline` Pipeline: its parameters (`repo-url`, `app-name`, `branch`), two workspaces (source and registry credentials) and the task order.
- `tasks.yaml`: the custom `cleanup`, `npm` and `jasmine` Tasks.
- `pvc.yaml`: the 1 GiB PersistentVolumeClaim the tasks share.
- `run.yaml`: a PipelineRun for the course's tax-calculator app, with the registry credentials mounted from the `docker-reg-creds` secret.

## Running it

On a cluster with Tekton Pipelines and the `git-clone` and `buildah` ClusterTasks installed:

```bash
kubectl apply -f pvc.yaml -f tasks.yaml -f pipeline.yaml
kubectl create -f run.yaml          # generateName gives each run a new name
tkn pipelinerun logs --last -f
```

The registry secret (`docker-reg-creds`) and the `pipeline` service account have to exist in the namespace first.

## Credits

Forked from [ibm-developer-skills-network/juphc-tc_pipeline](https://github.com/ibm-developer-skills-network/juphc-tc_pipeline), the course's starter repository (Apache 2.0). The starter had the `init` and `clone` steps and the `cleanup` task. The `npm` and `jasmine` tasks, the install, test and build stages, and the PipelineRun are my additions.
