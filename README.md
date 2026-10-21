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

# Running Migrations Before Deployments

Use `drydock/run-migrations` to run migrations once before rolling out your deployments. It creates a one-off Kubernetes Job named `rails-migrate-<sha7>` using the new image and the template deployment's pod spec (by default, `rails-puma-deployment`). It copies the first container's `env`, `envFrom`, and `resources`, along with the pod's `serviceAccountName` and `imagePullSecrets`. If migrations fail, the CircleCI job fails and the deployments don't run.

Add the following jobs to your production workflow, using the same pipeline parameters as your existing deployments and `brandnewbox/drydock@2.3.0` or later. Keep your existing `build-and-push-production` and `test` jobs, and apply the same production branch filters to all three jobs below.

```yaml
workflows:
  devops_flow:
    jobs:
      - drydock/run-migrations:
          name: migrate
          rancher-context: << pipeline.parameters.rancher-context >>
          rancher-namespace: << pipeline.parameters.rancher-namespace >>
          image: << pipeline.parameters.registry >>/<< pipeline.parameters.final-image >>:${CIRCLE_SHA1}
          credentials-key-env-var: RAILS_PRODUCTION_KEY
          context: DO_BNB_REGISTRY
          serial-group: << pipeline.project.slug >>/production-migrate
          requires: [build-and-push-production, test]
      - drydock/create-deployment:
          name: puma-deployment
          rancher-deployment: rails-puma-deployment
          rancher-container: rails-puma-container
          rancher-context: << pipeline.parameters.rancher-context >>
          rancher-namespace: << pipeline.parameters.rancher-namespace >>
          image: << pipeline.parameters.registry >>/<< pipeline.parameters.final-image >>:${CIRCLE_SHA1}
          should-update-credentials: false
          context: DO_BNB_REGISTRY
          requires: [migrate]
      - drydock/create-deployment:
          name: sidekiq-deployment
          rancher-deployment: rails-sidekiq-deployment
          rancher-container: rails-sidekiq-container
          rancher-context: << pipeline.parameters.rancher-context >>
          rancher-namespace: << pipeline.parameters.rancher-namespace >>
          image: << pipeline.parameters.registry >>/<< pipeline.parameters.final-image >>:${CIRCLE_SHA1}
          should-update-credentials: false
          context: DO_BNB_REGISTRY
          requires: [migrate]
```

Set `serial-group` where you invoke the job in your workflow. Overlapping pipelines take turns in the same group, so two migration runs never overlap. The orb can't build this in. Both jobs use the shared `setup-rancher-namespace` command to prepare registry access and credentials; the deployments set `should-update-credentials: false` because the migration job already refreshed the credentials.

The default `migrate-command` is `bundle exec rails db:migrate`, run through `/bin/sh -c "exec ..."`. For apps with tenant databases, override it on the `drydock/run-migrations` entry:

```yaml
migrate-command: bundle exec rails deploy:migrate
```

The migration deadline defaults to `timeout-seconds: 1800`. Keep `no-output-timeout` (default `"35m"`) above that deadline, because logs only print when the Kubernetes Job finishes. You can also set `template-deployment` to use a different deployment, and `should-update-credentials`, `credentials-key-env-var`, and `service-account` work the same way as on `create-deployment`.

Remove the `after_initialize` migrate block from your app's `config/environments/production.rb`. Write migrations that work with the code that's already running: old pods keep serving requests until the rollout finishes. See [#11](https://github.com/brandnewbox/drydock_orb/issues/11) for the earlier boot-time migration setup.

# Test Jobs

We used to include a bunch of jobs in this orb to perform tests. But it turned into `test-with-x` and `test-with-y` and `test-with-z-but-not-a` over time. So we decided to move all tests to a local level configuration and keep this orb focused on S2I builds and deploying to K8S.

To help ease the transition, those jobs have been archived in [archived_test_jobs.yml](https://github.com/brandnewbox/drydock_orb/blob/master/archived_test_jobs.yml) for reference.

# Updating?

Don't forget to bump the version in `.circleci/config.yml` and in the example in `src/orb.yml` everytime!
