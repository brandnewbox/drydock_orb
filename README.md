# Brand New Box Drydock Orb

A CircleCI orb for the [Brand New Box](https://brandnewbox.com) workflow.

Based heavily on the [doctl orb](https://circleci.com/orbs/registry/orb/digitalocean/cli).

Check out all the details on [CircleCI](https://circleci.com/orbs/registry/orb/brandnewbox/drydock).

# Environment Variable

Make sure you set the `RAILS_CREDENTIALS_KEY` in your CircleCI project settings to the appropriate rails key for your deployment.

# Updating?

Don't forget to bump the version in `.circleci/config.yml` and in the example in `src/orb.yml` everytime!

# Accessing DO Registry

We are taking advantage of [this](https://www.digitalocean.com/docs/container-registry/how-to/use-registry-docker-kubernetes/) to authenticate with DO:

>  If you’re in an environment that doesn’t have doctl or want to use an existing API token, you can simulate what doctl registry login does by using an API token string as the username and password when calling docker login. 
 