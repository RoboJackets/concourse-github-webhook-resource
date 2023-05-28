# concourse-github-webhook-resource
[![GitHub license](https://img.shields.io/github/license/RoboJackets/concourse-github-check-resource)](https://github.com/RoboJackets/concourse-github-check-resource/blob/main/LICENSE) [![CI](https://concourse.sandbox.aws.robojackets.net/api/v1/teams/information-technology/pipelines/github-webhook/jobs/build-main/badge)](https://concourse.sandbox.aws.robojackets.net/teams/information-technology/pipelines/github-webhook)

Concourse resource to automatically configure webhooks for a GitHub repository

## Source configuration

- `github_token` (required) - GitHub App token to use to authenticate
- `webhook_token` (required) - webhook token configured for your resources
- `debug` (optional) - whether to enable debug logging; must be set to boolean true if present
- `resources` (required) - map of resources to subscribe to GitHub events (keyed by name of the resource)
  - A `webhook_token` must also be specified on each of the resources' definitions in the pipeline-level `resources` block
- `resources.github_uri` (required) - GitHub repository URL to configure
- `resources.events` (required) - list of events to subscribe to (see [the GitHub API documentation](https://docs.github.com/en/developers/webhooks-and-events/webhook-events-and-payloads) for options)

## Behavior
Do not `get` this resource manually, it will not work.

### `check`
Returns an empty list of versions.

### `in`
Does nothing.

### `out`
Configures [resource webhooks](https://concourse-ci.org/resources.html#schema.resource.webhook_token) within GitHub so that Concourse is aware of new versions faster. This should only be called at the end of your main build job.

Specifically, this resource takes the following steps:
- Construct the desired webhook URLs for each repository using the [metadata provided by Concourse](https://concourse-ci.org/implementing-resource-types.html#resource-metadata) and the `source` configuration described above
- Call the constructed webhook URLs to verify the inputs are all correct
- Retrieve from GitHub a list of webhooks configured on each repository and filters down to those pointed at this Concourse server
- Removes any webhooks that most recently returned a 404
- Creates and updates remaining webhooks as needed

`inputs` must be an empty list; this reduces overhead starting the container.
