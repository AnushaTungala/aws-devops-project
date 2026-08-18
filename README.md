# AWS DevOps Static Web Application

## Project Overview

This is a static web application created as part of AWS/DevOps project readiness preparation.

## Technologies

- HTML
- CSS
- Git
- Docker
- Docker Compose
- Jenkins
- AWS

## Project Goal

The application will be:

1. Developed locally
2. Stored in GitHub
3. Containerized using Docker
4. Run using Docker Compose
5. Automated using Jenkins
6. Deployed to AWS

## Current Status

Static application created and tested locally.

## Jenkins CI/CD Pipeline

The repository contains a declarative `Jenkinsfile` checked out by the Jenkins
pipeline job `chatterly-aws-devops-pipeline` (GitHub -> Jenkins integration).

Pipeline stages: Checkout -> Build Docker Image -> Test / Smoke Check ->
Deploy Application -> Verify Deployment -> Push to Registry (optional).

Webhook-based automatic build: a push to `main` delivers an event to the Jenkins
`/github-webhook/` endpoint, which starts the pipeline automatically via the
`githubPush()` trigger in the Jenkinsfile.