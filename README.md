# Brand New Box Drydock Orb

A CircleCI orb for the [Brand New Box](https://brandnewbox.com) workflow.

Based heavily on the [doctl orb](https://circleci.com/orbs/registry/orb/digitalocean/cli).

Check out all the details on [CircleCI](https://circleci.com/orbs/registry/orb/brandnewbox/drydock).

# Environment Variable

Make sure you set the `RAILS_CREDENTIALS_KEY` in your CircleCI project settings to the appropriate rails key for your deployment.

# Updating?

Don't forget to bump the version in `.circleci/config.yml` and in the example in `src/orb.yml` everytime!

# Accessing DO Registry

We are taking advantage of [DigitalOcean's ability to authenticate for a registry](https://www.digitalocean.com/docs/container-registry/how-to/use-registry-docker-kubernetes/) and [CircleCI's ability to use auth credentials in the image configuration](https://circleci.com/docs/2.0/private-images/) to authenticate with DO:

>  If you’re in an environment that doesn’t have doctl or want to use an existing API token, you can simulate what doctl registry login does by using an API token string as the username and password when calling docker login. 
 
But this is currently failing with:

```bas
Starting container registry.digitalocean.com/brandnewbox/tasn-bdgt-builder:780b33087f15c5d33ff9748ac5ab88f967bd375d
  image cache not found on this host, downloading registry.digitalocean.com/brandnewbox/tasn-bdgt-builder:780b33087f15c5d33ff9748ac5ab88f967bd375d

Error response from daemon: Get https://registry.digitalocean.com/v2/brandnewbox/tasn-bdgt-builder/manifests/780b33087f15c5d33ff9748ac5ab88f967bd375d: unauthorized: authentication required
```

Internet says:

> The username should be your Api Token, while the password would be that’s API personal access token. The API personal access token is shown only once on creation.