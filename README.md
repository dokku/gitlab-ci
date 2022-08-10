# gitlab-ci

A collection of gitlab-ci examples

## Requirements

Please note that these workflows are compatible with `dokku >= 0.11.6`.

## Usage

All examples require a `SSH_PRIVATE_KEY` environment variable set for the Gitlab CI pipeline. This may be set via a "secret variable". See [this doc](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) for instructions on creating a new ssh key. Be careful not to overwrite existing keys on the generating machine by using a new name.

### Adding a secret variable

Browse to the repository in question and visit the following path: `the Gitlab project > Settings > CI/CD.`

Click on `Secret variables > Expand` and fill in the blanks.

- Key: `SSH_PRIVATE_KEY`
- Value: paste in an SSH private key registered in Dokku:

    -----BEGIN RSA PRIVATE KEY-----
    ...
    -----END RSA PRIVATE KEY-----

- Environment scope: `production` (This make sure that `SSH_PRIVATE_KEY` is not available on merge requests or tests)
- Protected: Do not check this checkbox unless you know what you are doing

## Environment Variables

- `BRANCH`: (_optional_) The branch to deploy when pushing to Dokku. Useful when a [custom deploy branch](http://dokku.viewdocs.io/dokku/deployment/methods/git/#changing-the-deploy-branch) is set on Dokku.
  - default: `master`
  - example value: `main`
- `CI_BRANCH_NAME`: (_optional_) The branch name that triggered the deploy. Automatically detected from `CI_COMMIT_REF_NAME`.
  - example value: `develop`
- `CI_COMMIT`: (_optional_) The commit sha that will be pushed. Automatically detected from `CI_COMMIT_SHA`.
  - example value: `0aa00d8dd7c971c121e3d1e471d0a35e1daf8abe`
- `COMMAND`: (_optional_) The command to run for the action.
  - default: `deploy`
  - valid values:
    - `deploy`
    - `review-apps:create`: Used to create a review app - via `dokku apps:clone` - based on the `appname` configured in the `git_remote_url`. If the review app already exists, this action will not recreate the app. In both cases, the current commit will be pushed to the review app.
    - `review-apps:destroy`: Destroys an existing review app.
- `GIT_PUSH_FLAGS`: (_optional_) A string containing a set of flags to set on push. This may be used to enable force pushes, or trigger verbose log output from git.
  - example value: `--force -vvv`
- `GIT_REMOTE_URL`: (**required**) The dokku app's git repository url in SSH format.
  - example value: `ssh://dokku@dokku.myhost.ca:22/appname`
- `REVIEW_APP_NAME`: (_optional_) The name of the review app to create or destroy. Computed as `review-$APPNAME-$BRANCH_NAME` if not specified, where:

  ```text
  $APPNAME: The parsed app name from the `git_remote_url`
  $BRANCH_NAME: The inflected git branch name
  ```

  - example value: `review-appname`
- `SSH_HOST_KEY`: (_optional_) The results of running `ssh-keyscan -t rsa $HOST`. The github-action will otherwise generate this on the fly via `ssh-keyscan`.
  - example value:

    ```text
    # dokku.com:22 SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.1
    dokku.com ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCvS+lK38EEMdHGb...
    ```

- `SSH_PRIVATE_KEY`: (**required**) A private ssh key that has push access to the Dokku instance.
  - tip: It is recommended to use [Encrypted Secrets](https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets) to store sensitive information such as SSH Keys.
  - example value:

    ```text
    -----BEGIN OPENSSH PRIVATE KEY-----
    MIIEogIBAAKCAQEAjLdCs9kQkimyfOSa8IfXf4gmexWWv6o/IcjmfC6YD9LEC4He
    qPPZtAKoonmd86k8jbrSbNZ/4OBelbYO0pmED90xyFRLlzLr/99ZcBtilQ33MNAh
    ...
    SvhOFcCPizxFeuuJGYQhNlxVBWPj1Jl6ni6rBoHmbBhZCPCnhmenlBPVJcnUczyy
    zrrvVLniH+UTjreQkhbFVqLPnL44+LIo30/oQJPISLxMYmZnuwudPN6O6ubyb8MK
    -----END OPENSSH PRIVATE KEY-----
    ```

## Examples

All examples below are functionally complete and can be copy-pasted into a `.gitlab-ci.yml` file, with some minor caveats:

- The `GIT_REMOTE_URL` should be changed to match the server and app.
- An [Gitlab Variable](https://docs.gitlab.com/ee/ci/variables/README.html#create-a-custom-variable-in-the-ui) should be set on the Gitlab repository with the name `SSH_PRIVATE_KEY` containing the contents of a private ssh key that has been added to the Dokku installation via the `dokku ssh-keys:add` command.
- As pushing a git repository from a shallow clone does not work, all repository checkouts should use a `GIT_DEPTH` of `0`. All examples below have this option set correctly.

For simplicity, each example is standalone, but may be combined as necessary to create the desired effect.

- [Simple Example](/example-pipelines/simple.yml): Deploys a codebase on push or merge to master.
- [Build in CI and Deploy an image](/example-pipelines/build-and-deploy.yml): Builds a docker image in CI, pushes the image to the remote Docker Hub repository, and then notifies Dokku to deploy the built image.
- [Cancel previous runs on new push](/example-pipelines/cancel-previous-runs.yml): This pipeline is particularly useful when triggered by new pushes.
- [Avoid SSH Host Keyscan](/example-pipelines/specify-ssh-host-key.yml): By default, this action will scan the host for it's SSH host key and use that value directly. This may not be desirable for security compliance reasons.

  The `SSH_HOST_KEY` value can be retrieved by calling `ssh-keyscan -t rsa $HOST`, where `$HOST` is the Dokku server's hostname.
- [Specify a custom deploy branch](/example-pipelines/custom-deploy-branch.yml): Certain Dokku installations may use custom deploy branches other than `master`. In the following example, we push to the `develop` branch.
- [Verbose Push Logging](/example-pipelines/verbose-logging.yml): Verbose client-side logging may be enabled with this method. Note that this does not enable trace mode on the deploy, and simply tells the `git` client to enable verbose log output
- [Force Pushing](/example-pipelines/force-push.yml): If the remote app has been previously pushed manually from a location other than CI, it may be necessary to enable force pushing to avoid git errors.
- [Review Apps](/example-pipelines/review-app.yml): Handles creation and deletion of review apps through use of `dokku apps:clone` and `dokku apps:destroy`. Review apps are a great way to allow folks to preview pull request changes before they get merged to production.
