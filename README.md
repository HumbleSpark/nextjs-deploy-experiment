# NextJS Deploy Experiment

https://nextjs-deploy-experiment-szlazhzr7q-uc.a.run.app/

- github actions
  -- Artifact Build & Upload
  -- Deploy Stage (Previews)
  -- Rollout Stages (Gradual Rollouts / Canary)
- cloud run deploys
- branches & tags trigger cloudrun deploy with hash url
- reverse proxy / middleware for skew protection
- ui to promote a deploy to production
- istio setup ... gql

```sh
yarn turbo run dev

# local
docker build -t nextjs-deploy-experiment -f apps/nextjs-deploy-experiment/Dockerfile .
docker run -p "3000:3000" -e "PORT=3000" -i -t nextjs-deploy-experiment

# artifact registry
export SHA_SHORT=$(git rev-parse --short HEAD)
docker build --platform linux/amd64 -t nextjs-deploy-experiment -f apps/nextjs-deploy-experiment/Dockerfile .
docker tag nextjs-deploy-experiment us-central1-docker.pkg.dev/alert-parsec-404117/nextjs-deploy-experiment/nextjs-deploy-experiment:$SHA_SHORT
docker push us-central1-docker.pkg.dev/alert-parsec-404117/nextjs-deploy-experiment/nextjs-deploy-experiment:$SHA_SHORT

# deploy https://cloud.google.com/sdk/gcloud/reference/run/deploy
gcloud run deploy nextjs-deploy-experiment \
  --image=us-central1-docker.pkg.dev/alert-parsec-404117/nextjs-deploy-experiment/nextjs-deploy-experiment:$SHA_SHORT \
  --region=us-central1 \
  --revision-suffix=$SHA_SHORT \
  --tag=$SHA_SHORT \
  --min-instances=0 \
  --max-instances=5 \
  --port=8080 \
  --no-traffic

# rollout https://cloud.google.com/sdk/gcloud/reference/run/services/update-traffic
gcloud run services update-traffic nextjs-deploy-experiment \
  --region=us-central1 \
  --update-tags=latest=nextjs-deploy-experiment-$SHA_SHORT \
  --to-revisions=nextjs-deploy-experiment-$SHA_SHORT=10

gcloud run services update-traffic nextjs-deploy-experiment \
  --region=us-central1 \
  --to-revisions=nextjs-deploy-experiment-$SHA_SHORT=100
```

## Steps Taken

- Created Github repo
- Created Turborepo + Basic Next application
- Installed Gcloud https://cloud.google.com/sdk/docs/install & signed in.
- Created nextjs-deploy-experiment in GCloud Artifact Registry
- Auth'd gcloud for Docker `gcloud auth configure-docker us-central1-docker.pkg.dev`
- Built docker image and tagged. Pushed to artifact registry.
- Created nextjs-deploy-experiment in cloudrun. Selected image and deployed.
- Auth with Github actions https://cloud.google.com/blog/products/identity-security/enabling-keyless-authentication-from-github-actions ; https://github.com/google-github-actions/auth ; https://gist.github.com/palewire/12c4b2b974ef735d22da7493cf7f4d37

## Ideas

- Configure last n production-deployed images to have n min instances
- Deploy to multiple Gcloud regions
