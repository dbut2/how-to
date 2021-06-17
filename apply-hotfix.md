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
