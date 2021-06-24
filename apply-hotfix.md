1. Make some required changes and push to origin with a new tag


```sh
export COMMIT=$(git rev-list -n 1 {current version number})

git checkout -b hotfix/{hotfix branch} $COMMIT

# make some changes

git add .
git commit -m "{commit message}"

git tag -a {incremented version} -m "source=manual,branch=master,tag={incremented version}" # increment version revision eg v1.2.3 -> v1.2.4

git switch master
git push --force --tags
```

2. Wait for harness to pick up the new tag and run the auto-pipelines, approve the stages until it hits prod

3. Create a PR to merge the changes into master so the changes are permanent
