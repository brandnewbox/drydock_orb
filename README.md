# Brand New Box Drydock Orb

A CircleCI orb for the [Brand New Box](https://brandnewbox.com) workflow.

Based heavily on the [doctl orb](https://circleci.com/orbs/registry/orb/digitalocean/cli).

Check out all the details on [CircleCI](https://circleci.com/orbs/registry/orb/brandnewbox/drydock).

Everything below is required for setup. However you will have to manually add the 2 steps below.

# RspecJunitFormatter

By default, our test suite uses `--format RspecJunitFormatter` to make things look nice for CircleCI. Make sure you have added `gem 'rspec_junit_formatter'` to your Gemfile.

# Rails Credentials

We deploy our applications with secrets encrypted in Rails credentials. In order to unlock those credentials, make sure you set the `RAILS_CREDENTIALS_KEY` in your CircleCI project settings to the appropriate rails key for your deployment.

Everything that follows should already be added through the `DO_BNB_REGISTRY` context. Check out these notes to see how we got here.

# Accessing Your Private Registry

We are taking advantage of [CircleCI's ability to use auth credentials in the image configuration](https://circleci.com/docs/2.0/private-images/) to authenticate with a private registry:

>  If you’re in an environment that doesn’t have doctl or want to use an existing API token, you can simulate what doctl registry login does by using an API token string as the username and password when calling docker login.

To do this, we use two environment variables: `PRIVATE_REGISTRY_USERNAME` and `PRIVATE_REGISTRY_PASSWORD`. We recommend keeping these secure in a CircleCI context and passing that context to the `build-and-push-builder` and `build-and-push-final` jobs.

# Deploying to Digital Ocean Kubernetes Cluster

This orb supports deploying built images into Digital Ocean kubernetes clusters. In order to take advantage of this, your private registry must also be on Digital Ocean. You will also need an environment variable of `DIGITAL_OCEAN_ACCESS_TOKEN`. We recommend keeping this secure in a CircleCI context and passing that context to the `create-deployment` job.

# Test Jobs

We used to include a bunch of jobs in this orb to perform tests. But it turned into `test-with-x` and `test-with-y` and `test-with-z-but-not-a` over time. So we decided to move all tests to a local level configuration and keep this orb focused on S2I builds and deploying to K8S.

To help ease the transition, those jobs have been archived in [archived_test_jobs.yml](https://github.com/brandnewbox/drydock_orb/blob/master/archived_test_jobs.yml) for reference.

# Updating?

Don't forget to bump the version in `.circleci/config.yml` and in the example in `src/orb.yml` everytime!
