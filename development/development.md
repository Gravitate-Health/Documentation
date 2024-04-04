# FOSPS Development Manual

This document summarizes the guidelines to be followed to develop FOSPS.

## Contributing: planning, feature requests, bug reporting

FOSPS planning, contribution and collaboration starts [here](https://github.com/Gravitate-Health/mvp-issues/issues). Issues can be open on-demand, and they will be discussed there.

## Version Control

All repositories are under version control in Github, in the [Gravitate-Health organization](https://github.com/orgs/Gravitate-Health), following the [feature branch convention](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow):
- Create Feature Branch: Start by creating a new branch from the main development branch from an issue (often called develop or main).
- Work on Feature: Develop your feature or fix in the feature branch. Keep commits focused and logical.
- Merge to Main: After approval, merge the feature branch into the main development branch (e.g., develop or main).

## Releases

Releases version names follow the [Semantic Versioning (SemVer)](https://semver.org) convention.

## CI/CD pipelines

Github worflows for Gravitate-Health are published and shared by all repositories in the [reusable-workflows repository](https://github.com/Gravitate-Health/reusable-workflows). The main workflow dows the following tasks:
- Semantic versioning and tag release (if push or pull request to main).
- Build of Docker image, tagging and push to registry.
- Security scan with Trivy, scanning vulnerabilities in Docker images and dependencies, detecting secrets leaks and misconfigurations.
- Automatic deployment to the Development environment.

## Builds

All developed components are built as Docker images, as FOSPS is deployed to a Kubernetes cluster.
Build is automated via Github workflows.

## Managing environments

Currently FOSPS has two environments: development and production. The configuration management of the deployments is done via [Kustomize](https://kustomize.io). This enables easy configuration management, as a single configuration has to be maintained, and only the incremental/differential changes have to be configured for each environment, without having to fork the original configuration files for a new environment.

Deployment via kubeflow is done like:

Production: 
```bash
kubectl apply -k kubernetes/base
```

Development: 
```bash
kubectl apply -k kubernetes/dev
```