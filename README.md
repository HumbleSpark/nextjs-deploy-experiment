# NextJS Deploy Experiment

- github actions
  -- Artifact Build & Upload
  -- Deploy Stage (Previews)
  -- Rollout Stages (Gradual Rollouts / Canary)
- cloud run deploys
- branches & tags trigger cloudrun deploy with hash url
- reverse proxy cloudrun with skew protection & preview support
- ui to promote a deploy to production
- istio setup ... gql

```sh
yarn turbo run dev

# local
docker build -t nextjs-deploy-experiment -f apps/nextjs-deploy-experiment/Dockerfile .
docker run -p "3000:3000" -e "PORT=3000" -i -t nextjs-deploy-experiment

# artifact registry
docker build --platform linux/amd64 -t nextjs-deploy-experiment -f apps/nextjs-deploy-experiment/Dockerfile .
docker tag nextjs-deploy-experiment us-central1-docker.pkg.dev/alert-parsec-404117/nextjs-deploy-experiment/nextjs-deploy-experiment:rev-1
docker push us-central1-docker.pkg.dev/alert-parsec-404117/nextjs-deploy-experiment/nextjs-deploy-experiment:rev-1

```

https://nextjs-deploy-experiment-szlazhzr7q-uc.a.run.app/

## Steps Taken

- Created Github repo
- Created Turborepo + Basic Next application
- Installed Gcloud https://cloud.google.com/sdk/docs/install & signed in.
- Created nextjs-deploy-experiment in GCloud Artifact Registry
- Auth'd gcloud for Docker `gcloud auth configure-docker us-central1-docker.pkg.dev`
- Built docker image and tagged. Pushed to artifact registry.
- Created nextjs-deploy-experiment in cloudrun. Selected image and deployed.
- Auth with Github actions https://cloud.google.com/blog/products/identity-security/enabling-keyless-authentication-from-github-actions ; https://github.com/google-github-actions/auth ; https://gist.github.com/palewire/12c4b2b974ef735d22da7493cf7f4d37
