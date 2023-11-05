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

# for running local docker / building & deploying...
export RUN_PROJECT=nextjs-deploy-experiment
export LOCAL_IMAGE=nextjs-deploy-experiment
export REMOTE_IMAGE=us-central1-docker.pkg.dev/alert-parsec-404117/nextjs-deploy-experiment/nextjs-deploy-experiment
export TAG=$(git rev-parse --short HEAD)
export REGION=us-central1

# local
docker build -t $LOCAL_IMAGE:$TAG -f apps/nextjs-deploy-experiment/Dockerfile .
docker run -p "3000:3000" -e "PORT=3000" -i -t $LOCAL_IMAGE:$TAG

# artifact registry
docker build --platform linux/amd64 -t $LOCAL_IMAGE:$TAG -f apps/nextjs-deploy-experiment/Dockerfile .
docker tag $LOCAL_IMAGE:$TAG $REMOTE_IMAGE:$TAG
docker push $REMOTE_IMAGE:$TAG

# deploy https://cloud.google.com/sdk/gcloud/reference/run/deploy
gcloud run deploy $RUN_PROJECT \
  --region=$REGION \
  --image=$REMOTE_IMAGE:$TAG \
  --revision-suffix=$TAG \
  --tag=$TAG \
  --min-instances=0 \
  --max-instances=5 \
  --port=8080 \
  --no-traffic

# rollout https://cloud.google.com/sdk/gcloud/reference/run/services/update-traffic
gcloud run services update-traffic $RUN_PROJECT --region=$REGION --to-revisions=$RUN_PROJECT-$TAG=10
gcloud run services update-traffic $RUN_PROJECT --region=$REGION --to-revisions=$RUN_PROJECT-$TAG=100
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
