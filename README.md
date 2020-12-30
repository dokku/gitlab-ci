# gitlab-ci

A collection of gitlab-ci examples

## Usage

All examples require a `SSH_HOST_KEY` environment variable set for the Gitlab CI pipeline. This may be set via a "secret variable"

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

## Examples

All examples below are functionally complete and can be copy-pasted into a `.gitlab-ci.yml` file, with some minor caveats:

- The `GIT_REMOTE_URL` should be changed to match the server and app.
- An [Gitlab Variable](https://docs.gitlab.com/ee/ci/variables/README.html#create-a-custom-variable-in-the-ui) should be set on the Gitlab repository with the name `SSH_PRIVATE_KEY` containing the contents of a private ssh key that has been added to the Dokku installation via the `dokku ssh-keys:add` command.
- As pushing a git repository from a shallow clone does not work, all repository checkous should use a `GIT_DEPTH` of `0`. All examples below have this option set correctly.

For simplicity, each example is standalone, but may be combined as necessary to create the desired effect.

- [Simple Example](/example-pipelines/simple.yml): Deploys a codebase on push or merge to master.
- [Cancel previous runs on new push](/example-pipelines/cancel-previous-runs.yml): This pipeline is particularly useful when triggered by new pushes.
- [Avoid SSH Host Keyscan](/example-pipelines/specify-ssh-host-key.yml): By default, this action will scan the host for it's SSH host key and use that value directly. This may not be desirable for security compliance reasons.

  The `SSH_HOST_KEY` value can be retrieved by calling `ssh-keyscan -t rsa $HOST`, where `$HOST` is the Dokku server's hostname.
- [Specify a custom deploy branch](/example-pipelines/custom-deploy-branch.yml): Certain Dokku installations may use custom deploy branches other than `master`. In the following example, we push to the `develop` branch.
- [Verbose Push Logging](/example-pipelines/verbose-logging.yml): Verbose client-side logging may be enabled with this method. Note that this does not enable trace mode on the deploy, and simply tells the `git` client to enable verbose log output
- [Force Pushing](/example-pipelines/force-push.yml): If the remote app has been previously pushed manually from a location other than CI, it may be necessary to enable force pushing to avoid git errors.
- [Review Apps](/example-pipelines/review-app.yml): Handles creation and deletion of review apps through use of `dokku apps:clone` and `dokku apps:destroy`. Review apps are a great way to allow folks to preview pull request changes before they get merged to production.
