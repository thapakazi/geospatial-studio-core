# The `build` branch

Our integration branch. It is **not** meant to be merged upstream.

`main` and the `feat/*` branches stay exactly as upstream would want them, so a
pull request can be opened from them without carrying any of our build
machinery. Everything deployment-specific lives here instead:

- `.github/workflows/build-images.yml` — publishes images to GHCR
- `verify-signed-commits.yml` is **deleted** here, deliberately. Upstream
  requires signed commits on every branch; we do not need that gate on a branch
  that never leaves the fork.

## Images

| Image | Built from | Used as |
|---|---|---|
| `ghcr.io/<owner>/geospatial-studio-core/geostudio-gateway` | `Dockerfile` | the gateway deployment |
| `ghcr.io/<owner>/geospatial-studio-core/geostudio-pipelines` | `pipelines/Dockerfile` | the pipelines processors **and** `DATASET_PIPELINE_IMAGE` |

Both are `linux/amd64`. Tags: `latest` (tracks this branch), the branch name,
and `sha-<short>` for pinning.

The pipelines image contains every component, including
`claimed_curated_upload_v2.py`, which is what the dataset-onboarding Job runs.
That is why `DATASET_PIPELINE_IMAGE` points at it rather than at a separate
image.

## Keeping up with upstream

```sh
git fetch origin                 # origin = terrastackai/geospatial-studio-core
git checkout main && git merge --ff-only origin/main
git checkout build && git merge main
git push fork build              # triggers a rebuild
```

Feature work is merged in the same way:

```sh
git checkout build
git merge feat/optional-object-storage-credentials
git push fork build
```

## Before the first deploy

GHCR packages are **private by default**. Either make the two packages public,
or give the cluster an imagePullSecret — the chart supports one via
`global.imagePullSecret` and `FT_IMAGE_PULL_SECRETS`.

When pointing the chart at these images, set the per-image `image.name` values.
Do **not** set `CONTAINER_IMAGE_REPOSITORY`: `values-file-generate.sh` runs
`s|geospatial-studio|${CONTAINER_IMAGE_REPOSITORY}|` with no `g` flag, and that
pattern also matches the `geospatial-studio-pipelines:` subchart key in
`values.yaml`, which silently drops the entire pipelines subchart back to its
chart defaults with no error.
