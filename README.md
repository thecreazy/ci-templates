# Default CI template

This repo contains a series of templates that can be used inside the CI/CD pipeline of JTM.

### How it works

According to the operation / the type of pipeline you have to perform, you can pick here different stages:

- [Test (NB: this is still empty!)](#empty-test-stage)
- [Docker pipeline](#docker-pipeline)
  - Build
  - Push to registry
- Serverless functions deployment
  - [Regional deployment](#google-function-regional-pipeline)
  - [Multi-regional deployment](#google-function-multiregion-pipeline)
- Kubernetes deployment
  - [Deploy on quality (a.k.a. staging)](#kubernetes-quality-pipeline)
  - [Regional deployment](#kubernetes-regional-pipeline)
  - [Multi-regional deployment](#kubernetes-multiregion-pipeline)
  - ["Simple script" execution](#kubernetes-"simple-script"-pipeline)
- [Terraform pipeline](#terraform-pipeline)
- [Google endpoint](#google-endpoint)
- [Google bucket upload](#deploy-to-google-storage)

### How to use it

If you're devops or if you know very well what you're doing, copy the relevant part from this readme to your `.gitlab-ci.yml` file. You can also override some operations by overriding the stages or the operation itself.

# Empty test stage

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/gadiener/ci-templates/v1.0.0/empty-test.yml'

stages:
  - test
```

## Deploy to Google Storage

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/gadiener/ci-templates/v1.0.0/bucket.yml'

stages:
  - deploy

variables:
  GOOGLE_KEY: <google json key>
  GOOGLE_PROJECT: my-project
  BUCKET_NAME: website.com
```

## Google function regional pipeline

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/gadiener/ci-templates/v1.0.0/serverless-regional.yml'

stages:
  - deploy

variables:
  REGION: us-central1
  FUNCTION_NAME: my-function
  RUNTIME: go111
  TRIGGER_HTTP: 1
  GOOGLE_KEY: <google json key>
  DOMAIN: https://us-central1-my-project.cloudfunctions.net/http_function
  SECRET_YAML: <secret base64 encoded>
  SECRET_ENV_LIST: "SUPERSECRET=env"
  TIMEOUT: 30
```

## Google function multiregion pipeline

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/gadiener/ci-templates/v1.0.0/serverless-multiregion.yml'

stages:
  - deploy

variables:
  FUNCTION_NAME: my-function
  RUNTIME: go111
  TRIGGER_HTTP: 1
  TIMEOUT: 30

  # PRODUCTION EUROPE VARIABLES
  GOOGLE_KEY_EUROPE: <google json key>
  REGION_EUROPE: europe-west1
  DOMAIN_EUROPE: https://europe-west1-my-project.cloudfunctions.net/http_function
  SECRET_YAML_EUROPE: <secret base64 encoded>
  SECRET_ENV_LIST_EUROPE: "SUPERSECRET=env"

  # PRODUCTION AMERICA VARIABLES
  GOOGLE_KEY_AMERICA: <google json key>
  REGION_AMERICA: us-central1
  DOMAIN_AMERICA: https://us-central1-my-project.cloudfunctions.net/http_function
  SECRET_YAML_AMERICA: <secret base64 encoded>
  SECRET_ENV_LIST_AMERICA: "SUPERSECRET=env"

  # PRODUCTION ASIA VARIABLES
  GOOGLE_KEY_ASIA: <google json key>
  REGION_ASIA: asia-east2
  DOMAIN_ASIA: https://asia-east2-my-project.cloudfunctions.net/http_function
  SECRET_YAML_ASIA: <secret base64 encoded>
  SECRET_ENV_LIST_ASIA: "SUPERSECRET=env"
```

## Google endpoint

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/gadiener/ci-templates/v1.0.0/endpoint.yml'

stages:
  - deploy

variables:
  ENDPOINT_FILE: endpoint.yaml
  GOOGLE_KEY: <google json key>
```

## Terraform pipeline

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/gadiener/ci-templates/v1.0.0/terraform.yml'

cache:
  paths:
    - /.terraform

stages:
  - validate
  - build
  - deploy
```

## Docker pipeline

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/gadiener/ci-templates/v1.0.0/docker.yml'

stages:
  - build
  - push

variables:
  IMAGES: "app nginx"
```

## Kubernetes quality pipeline

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/gadiener/ci-templates/v1.0.0/kubernetes-quality.yml'

stages:
  - build
  - push
  - manifest
  - deploy
  - rollback

variables:
  IMAGES: "app nginx"
  ROLLOUT_RESOURCES: "deployment/app deployment/worker"

  # QUALITY VARIABLES
  GOOGLE_KEY_QUALITY: <google json key>
  NAMESPACE_QUALITY: "my-namespace"
  CLUSTER_NAME_QUALITY: quality
  CLUSTER_ZONE_QUALITY: europe-west6-a
  SECRET_YAML_QUALITY: <secret base64 encoded>
  DOMAIN_QUALITY: quality.example.com
```

## Kubernetes regional pipeline

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/gadiener/ci-templates/v1.0.0/kubernetes-regional.yml'

stages:
  - build
  - push
  - manifest
  - deploy
  - rollback

variables:
  IMAGES: "app nginx"
  ROLLOUT_RESOURCES: "deployment/app"

  # QUALITY VARIABLES
  GOOGLE_KEY_QUALITY: <google json key>
  NAMESPACE_QUALITY: "my-namespace"
  CLUSTER_NAME_QUALITY: quality
  CLUSTER_ZONE_QUALITY: europe-west6-a
  SECRET_YAML_QUALITY: <secret base64 encoded>
  DOMAIN_QUALITY: quality.example.com

  # PRODUCTION VARIABLES
  GOOGLE_KEY_PRODUCTION: <google json key>
  NAMESPACE_PRODUCTION: "my-namespace"
  CLUSTER_NAME_PRODUCTION: production-europe-west1
  CLUSTER_ZONE_PRODUCTION: europe-west1-b
  SECRET_YAML_PRODUCTION: <secret base64 encoded>
  DOMAIN_PRODUCTION: production.example.com
```

## Kubernetes multiregion pipeline

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/gadiener/ci-templates/v1.0.0/kubernetes-multiregion.yml'

stages:
  - build
  - push
  - manifest
  - deploy
  - rollback

variables:
  IMAGES: "app nginx"
  ROLLOUT_RESOURCES: "deployment/app"

  # QUALITY VARIABLES
  GOOGLE_KEY_QUALITY: <google json key>
  NAMESPACE_QUALITY: "my-namespace"
  CLUSTER_NAME_QUALITY: quality
  CLUSTER_ZONE_QUALITY: europe-west6-a
  SECRET_YAML_QUALITY: <secret base64 encoded>
  DOMAIN_QUALITY: quality.example.com

  # PRODUCTION EUROPE VARIABLES
  GOOGLE_KEY_PRODUCTION_EUROPE: <google json key>
  NAMESPACE_PRODUCTION_EUROPE: "my-namespace"
  CLUSTER_NAME_PRODUCTION_EUROPE: production-europe-west1
  CLUSTER_ZONE_PRODUCTION_EUROPE: europe-west1-b
  SECRET_YAML_PRODUCTION_EUROPE: <secret base64 encoded>
  DOMAIN_PRODUCTION_EUROPE: eu.production.example.com

  # PRODUCTION AMERICA VARIABLES
  GOOGLE_KEY_PRODUCTION_AMERICA: <google json key>
  NAMESPACE_PRODUCTION_AMERICA: "my-namespace"
  CLUSTER_NAME_PRODUCTION_AMERICA: production-us-central1
  CLUSTER_ZONE_PRODUCTION_AMERICA: us-central1-a
  SECRET_YAML_PRODUCTION_AMERICA: <secret base64 encoded>
  DOMAIN_PRODUCTION_AMERICA: am.production.example.com

  # PRODUCTION ASIA VARIABLES
  GOOGLE_KEY_PRODUCTION_ASIA: <google json key>
  NAMESPACE_PRODUCTION_ASIA: "my-namespace"
  CLUSTER_NAME_PRODUCTION_ASIA: production-asia-east1
  CLUSTER_ZONE_PRODUCTION_ASIA: asia-east1-a
  SECRET_YAML_PRODUCTION_ASIA: <secret base64 encoded>
  DOMAIN_PRODUCTION_ASIA: as.production.example.com
```

## Kubernetes "simple script" pipeline

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/gadiener/ci-templates/v1.0.0/kubernetes-task.yml'

stages:
  - deploy

variables:
  NAMESPACE: 'my-namespace'

  # PRODUCTION VARIABLES
  CLUSTER_NAME_PRODUCTION: 'production-europe-west1'
  CLUSTER_ZONE_PRODUCTION: 'europe-west1-b'
  KUPDATE_SCRIPT: |
                  kupdate resource-1 && \
                  kupdate resource-2 && \
                  kupdate statefulset resource-3
```