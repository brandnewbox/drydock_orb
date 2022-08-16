# Brand New Box Drydock Orb

A CircleCI orb for the [Brand New Box](https://brandnewbox.com) workflow.

Based heavily on the [doctl orb](https://circleci.com/orbs/registry/orb/digitalocean/cli).

Check out all the details on [CircleCI](https://circleci.com/orbs/registry/orb/brandnewbox/drydock).

# RspecJunitFormatter

By default, our test suite uses `--format RspecJunitFormatter` to make things look nice for CircleCI. Make sure you have added `gem 'rspec_junit_formatter'` to your Gemfile.

# Rails Credentials

We deploy our applications with secrets encrypted in Rails credentials. In order to unlock those credentials, make sure you set the `RAILS_CREDENTIALS_KEY` in your CircleCI project settings to the appropriate rails key for your deployment.

# Updating?

Don't forget to bump the version in `.circleci/config.yml` and in the example in `src/orb.yml` everytime!
