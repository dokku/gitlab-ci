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
