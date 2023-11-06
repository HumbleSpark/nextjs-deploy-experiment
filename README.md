# NextJS Deploy Experiment

This deploys a Turborepo, NextJS app using Github Actions and GCP Cloud Run. Gets a Preview URL on main deploys, then lets us rollout traffic. Mostly for experimenting with GCP, service accounts, etc.

90% of requests will show `Hello world` with 10% `Hello world!?` hitting canary.

- [Prod, us-central1](https://nextjs-deploy-experiment-szlazhzr7q-uc.a.run.app/)
- [Prod, us-east1](https://nextjs-deploy-experiment-szlazhzr7q-ue.a.run.app/)
- [rev-989b088 aka canary](https://rev-989b088---nextjs-deploy-experiment-szlazhzr7q-uc.a.run.app/)

## Github Actions

- **Main Build and Preview (main merge):** builds a Docker image, uploads to GCS Artifact Registry, and deploys to Cloud Run for preview deploys.
- **Update Production Traffic (manual):** input Git SHA to ramp to 10% or 100% across datacenters.
- **Describe Production Traffic (manual):** outputs current traffic allocation across datacenters.

## Todo

- [ ] Upload assets to GCS from Docker image.
- [ ] PRs action to deploy to separate project for branch previews.
- [ ] Middleware / reverse proxy for skew protection (pointing at specific hash deploy via headers).
- [ ] Hook this up to Remote Cache. See how Docker build caching is.
- [ ] Better UI for managing deployments. Simple, Vercel like (google cloud authentication; summary view with whats live in canary/all-traffic, with datacenter tags; list of commits with message + branch info; promote to canary; promote to all-traffic; rollback from canary).
- [ ] Istio investigation / setup GQL API to connect to.
- [ ] Potentially write up a blog post on simple Vercel-like deploy setup?!

## Steps Taken / How to Recreate

- Created Github repo.
- Created Turborepo + Basic Next application.
- [Installed GCloud](https://cloud.google.com/sdk/docs/install) & signed in.
- Created `nextjs-deploy-experiment` in GCloud Artifact Registry via UI.
- Auth'd GCloud for Docker `gcloud auth configure-docker us-central1-docker.pkg.dev`
- Built Docker Image and tagged. Pushed to artifact registry. See helpful commands.
- Created `nextjs-deploy-experiment` in Cloud Run via UI. Selected image and deployed via UI.
- Configured GCP Auth with Github actions (followed [one](https://cloud.google.com/blog/products/identity-security/enabling-keyless-authentication-from-github-actions) ; [two](https://github.com/google-github-actions/auth); [three](https://gist.github.com/palewire/12c4b2b974ef735d22da7493cf7f4d37)). Ensured service account had all correct roles (`roles/artifactregistry.writer`, `roles/iam.serviceAccountUser`, `roles/run.admin`, `roles/iam.workloadIdentityUser`).
- Recreated manual Docker builds & Cloud Run deploys using Github Actions. Starting with Build & Publish to Artifact Registry, then Cloud Run deploy.

## Helpful Commands

```sh
yarn turbo run dev

# for running local docker / building & deploying...
export RUN_REGION=us-central1
export RUN_PROJECT=nextjs-deploy-experiment
export LOCAL_IMAGE=nextjs-deploy-experiment
export REMOTE_IMAGE=us-central1-docker.pkg.dev/alert-parsec-404117/nextjs-deploy-experiment/nextjs-deploy-experiment
export TAG=$(git rev-parse --short HEAD)

# local
docker build -t $LOCAL_IMAGE:$TAG -f apps/nextjs-deploy-experiment/Dockerfile .
docker run -p "3000:3000" -e "PORT=3000" -i -t $LOCAL_IMAGE:$TAG

# artifact registry
docker build --platform linux/amd64 -t $LOCAL_IMAGE:$TAG -f apps/nextjs-deploy-experiment/Dockerfile .
docker tag $LOCAL_IMAGE:$TAG $REMOTE_IMAGE:$TAG
docker push $REMOTE_IMAGE:$TAG

deploy https://cloud.google.com/sdk/gcloud/reference/run/deploy
gcloud run deploy $RUN_PROJECT \
  --region=$RUN_REGION \
  --image=$REMOTE_IMAGE:$TAG \
  --revision-suffix=$TAG \
  --tag=$TAG \
  --min-instances=0 \
  --max-instances=5 \
  --port=8080 \
  --no-traffic

# rollout https://cloud.google.com/sdk/gcloud/reference/run/services/update-traffic
gcloud run services update-traffic $RUN_PROJECT --region=$RUN_REGION --to-revisions=$RUN_PROJECT-$TAG=10
gcloud run services update-traffic $RUN_PROJECT --region=$RUN_REGION --to-revisions=$RUN_PROJECT-$TAG=100
gcloud run services describe $RUN_PROJECT --region=$RUN_REGION --format=json
```
