# Train-Regression-Model-CiCd
Train a Regression Model and Tune Hyperparameters
1. Create github repository “Train-Regression-Model-CiCd”
2. Create new project on Google Cloud “mlops-01”
3. Create repository “Train-Regression-Model-CiCd” in europe-central2 region and link it to github repository “Train-Regression-Model-CiCd”
4. Create Google Cloud Service account an give it necessary rights (Artifact registry Writer/Reader)

gcloud projects add-iam-policy-binding mlops-01-443112  --member="serviceAccount:mlops-pusher@mlops-01-443112.iam.gserviceaccount.com"  --role="artifactregistry.writer"

5. Create Ci-Cd trigger and configure Cloud Build configuration file to start pipeline after new files have been pushed to git repository

Cloud Build configuration file:
substitutions:
  _LOCATION: europe-central2
  _PROJECT_ID: mlops-01-443112
  _IMAGE_NAME: hello-world
  _REGISTRY: mlops-test
  _VERSION: "1.0.1"
  _WORKDIR: .
steps:
  - id: build container image
    name: gcr.io/cloud-builders/docker
    env:
      - DOCKER _BUILDKIT=1
    dir: $_WORKDIR
    args:
      - build
      - -t
      - $_LOCATION-docker.pkg.dev/$_PROJECT_ID/$_REGISTRY/$_IMAGE_NAME:$_VERSION
      - -t
      - $_LOCATION-docker.pkg.dev/$_PROJECT_ID/$_REGISTRY/$_IMAGE_NAME:latest
      - -f
      - Dockerfile
      - .

images:
  - $_LOCATION-docker.pkg.dev/$_PROJECT_ID/$_REGISTRY/$_IMAGE_NAME
options:
  requestedVerifyOption: VERIFIED
  logging: CLOUD_LOGGING_ONLY
  sourceProvenanceHash: ['SHA256']




6. Add image to repository mlops-test
docker tag hello-world europe-central2-docker.pkg.dev/mlops-01-443112/mlops-test/hello-world:hello-test
docker push europe-central2-docker.pkg.dev/mlops-01-443112/mlops-test/hello-world:hello-test


gcloud builds submit --config cloudbuild.yaml .
