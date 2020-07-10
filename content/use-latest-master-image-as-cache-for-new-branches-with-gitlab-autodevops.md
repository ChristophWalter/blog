# Use latest master image as cache for new branches with gitlab autodevops
We noticed that the first build of every feature branch was pretty slow. Gitlab autodevops uses the latest docker image of the current branch for caching. 
To speed up the build we need to make it use the latest image from master if there is none for the current branch.

## Pull the master image before autodevops build script starts
This is the build script for docker images using gitlab autodevops: [build.sh](https://gitlab.com/gitlab-org/cluster-integration/auto-build-image/blob/master/src/build.sh)
In order to use the master image as cache, we need to execute some steps during `before_script` in *gitlab-ci.yaml*:
```yaml
build:
  before_script:
    - ./prefetch-image.sh
```
This script will do the following:
1. Pull Branch Image and exit if it does exist.
2. If it does not exist, pull master image.
3. Tag the locally pulled master image with the branch name.
4. Make sure to exit with code 0 as otherwise the exit command of the last command will be the exit code of the script. This will make it impossible to create a master image if there is no master image.
```bash
# prefetch-image.sh
BRANCH_LATEST_IMAGE=$CI_REGISTRY_IMAGE/$CI_COMMIT_REF_SLUG:latest
MASTER_LATEST_IMAGE=$CI_REGISTRY_IMAGE/master:latest

if ! docker pull $BRANCH_LATEST_IMAGE ; then
  echo "No image found for branch. Trying to pull latest master image."
  if docker pull $MASTER_LATEST_IMAGE ; then
    echo "Master image found. Tagging it as latest branch image."
    docker tag $MASTER_LATEST_IMAGE $BRANCH_LATEST_IMAGE
  fi
fi

exit 0
```

**The gitlab runner will still fail pulling the latest branch image. But it will use the already pulled image when building a new image from cache using the same name.**
