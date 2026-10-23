# Deploy to ECS

Could be used in any context, it defines a basic pipeline to deploy to ECS.
## `force_rebuild`

By default the action skips the build when an image tagged with `ecr_image_tag` (the commit
SHA) already exists in ECR, and deploys that image. That means re-running a deploy never picks
up a patched base image (e.g. a new `node:24-slim` after a Node security release).

With `force_rebuild: "true"` the action:

- skips the existing-tag lookup and always builds, with `--pull` so the mutable base tag is
  re-resolved instead of reusing the cached layer;
- pushes under `<ecr_image_tag>-rebuild-<YYYYMMDDHHMMSS>` (UTC) instead of the bare tag, because
  the ECR repos are `IMMUTABLE` and re-pushing the same tag would fail with
  `ImageTagAlreadyExistsException`. The timestamp is per action call, so two forced calls in
  the same run (same repo and tag) don't collide;
- renders both task definitions with that new tag.

Each forced rebuild is one more image in the repo, so it counts against the ECR lifecycle policy.
