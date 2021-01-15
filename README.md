*Note this is a fork from [automated-quay.io-deployment](https://github.com/lkiesow/automated-quay.io-deployment), but for GitHub actions.*

# Quay.io Deployment Tools

[Quay.io](https://quay.io) is a nice tool for automatically building and
hosting your Docker containers. It supports [Github based build triggers
](https://coreos.com/quay-enterprise/docs/latest/github-build.html) to
automatically build containers every time you push to your repository which is
really nice.

But it does not support any form of automatic rebuilds to ensure that base
images get updated. This essentially means that unless you regularly push to
your Dockerfile repository, your base images will never get updated.

This repository contains a very minimal script to support a custom trigger
using [GitHub actions](https://docs.github.com/en/free-pro-team@latest/actions/quickstart).
Since GitHub actions supports not only build
triggers but also cronjobs, this makes regular builds easy.


## Setting up [Quay.io webhook](https://docs.quay.io/guides/custom-trigger.html)

*Note that this will work for public repositories only.*

Go to the “Builds” section in your repository on quay.io and select “Create
Build Trigger” and “Custom Git Repository Push”. In the following dialog, use
the HTTPS variant for your Github repository URL. That makes it completely
unnecessary to deal with any SSH keys later on.

After it has been successfully created, you will get a Webhook Endpoint URL
which you will need for Travis. The URL should look somewhat like this:

    https://$token:...@quay.io/webhooks/push/trigger/...


## Setting up GitHub actions

Go into your repository settings on GitHub, go to the “Secrets” section
and add an environment secrets, name it.
In “Environment secrets”, add a secret variable called `QUAY_WEBHOOK_URL`
containing the Quay Webhook Endpoint URL you received earlier.

Enable GitHub actions for your repository, by adding a
`.github/workflows/*.yml` file in your repository:

```yaml
run: |
  curl -s https://raw.githubusercontent.com/VincentRouvreau/automated-quay.io-deployment/master/deploy-on-quay | sh
```


This will let you build fail if the deployment to quay did not work. If you
want this to fail silently instead (e.g. to not disrupt development), you can
download the script instead:

```yaml
run: |
  curl -O https://raw.githubusercontent.com/VincentRouvreau/automated-quay.io-deployment/master/deploy-on-quay
  sh deploy-on-quay || true
```

Additionally, you can set the Quay.io default branch by setting the
`QUAY_DEFAULT_BRANCH` environment variable. This is set to `master` by default.
The default branch is used by Quay.io to determine if a container that is built
gets tagged with the `latest` tag.


### Automated Rebuilds

Finally, to schedule automatic rebuilds, go to your repository setting on
Travis and add a cron job to rebuild the containers on a daily, weekly or
monthly basis.


## A Note About Security

Instead of loading the script from a foreign repository and just execute it, it
is obviously possible to directly include this script in the Dockerfile
repository instead.
