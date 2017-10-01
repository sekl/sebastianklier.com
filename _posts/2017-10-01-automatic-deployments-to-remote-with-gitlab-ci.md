---
layout: post
title:  "Automatic Deployments to a Remote Host Using GitLab-CI"
date:   2017-10-01 19:32:00
categories: [gitlab]
comments: true
---

I recently spent some time cleaning up a deployment process which so far consisted pretty much of manually running a big shell script to prepare the code base and then rsync the files to a staging environment. Any database changes had to be run manually.

Since this project has already been using [GitLab-CI](https://about.gitlab.com/features/gitlab-ci-cd/) to run the test suite, letting it handle deployments too turned out very simple.

Depending on your environments you will have to set up SSH keys for this to work so that your user on the remote server has access to your GitLab repository. Simply add the public key to your repository's deploy keys. Also, since you will be opening a SSH connection from the GitLab-runner, the user on the runner also needs access to your remote environment.

I'm not using Docker for this particular project yet but if you do, you will want to add a [GitLab environment variable](https://docs.gitlab.com/ce/administration/environment_variables.html) with your key and let your container use that.

This is a simplified version of what I ended up with. It will run the test suite and - as long as it passes - run `deploy_staging` if it's the `master` branch, by opening a SSH connection from your runner to the remote machine, pulling your code and running any necessary jobs to handle dependencies or database migrations.

```yaml
stages:
  - test
  - staging

test:
  stage: test
  script:
    - bin/phpunit --verbose --debug

deploy_staging:
  stage: staging
  script:
    - ssh user@example.com "cd /srv/www/staging.example.com &&
        git checkout master &&
        git pull origin master &&
        composer install -n &&
        bin/console doctrine:migrations:migrate"
  environment: staging
  only:
    - master
```

Using this short GitLab config file we're able to do what we previously used a 100-line Shell script for, and it even offers you a log and an easy way to roll-back right from the GitLab UI. I hope this explains the general concept well enough so you can adjust it to your needs.
