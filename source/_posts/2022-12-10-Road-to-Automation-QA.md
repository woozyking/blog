---
title: 'Road to Automation: QA'
date: 2022-12-10 15:54:40
tags:
  - programming
  - software development
  - automation
---

<figure>
    <img src="https://user-images.githubusercontent.com/2837532/207129082-794079b4-c95b-43f8-9fe4-d044396e1434.png" />
    <figcaption align="center">Generated through <a href="https://huggingface.co/CompVis/stable-diffusion-v1-4">Stable Diffusion v1-4</a></figcaption>
</figure>

Software QA (quality assurance), much like documentation, while important, often gets overlooked and buried down in the priority list when compared to research and development efforts.

While there are increasing efforts to automate the various aspects of software QA, many still require substantial effort to get meaningful values (such as writing unit- or end-to-end tests). However, we have sophisticated a few other fronts that can integrate into many projects with minimal setup.
<!-- more -->
## Commit Message Convention Check

Primarily because we already rely on the [same convention](https://engineering.eqworks.io/developments/git/) to facilitate {% post_link Road-to-Automation-Updates "team updates" %} and {% post_link Road-to-Automation-Release-Notes "product release" %} efforts, ensuring that our commit messages follow the said convention becomes even more critical. And for this reason, we created a CLI (command line interface).

```shell
(some repo) % npx @eqworks/commit-watch -b origin/main --verbose
⚠ 6/9 do not match pattern "category[/sub-category] - subject title"

✖ define APIError
✖ fix /readme to reflect changes in selectUser
✖ remove unnecessary await
✖ refactor selectUser to return user || undefined
✖ PR fixes, throws APIError
✖ check if user exists, won't send opt email if user unknown
✓ users - make getUsers async, await db query
✓ login - throw error on /users endpoints when user not found
✓ login - misc PR changes
```

The CLI form factor means it is simple to integrate into any CI/CD (continuous integration/deployment) processes, such as GitHub Actions.

```yaml
name: Main

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  commit-watch:
    runs-on: ubuntu-latest
    if: contains(github.event_name, 'pull_request') # only run on pull requests
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0 # needed for obtaining relevant commit metadata for commit-watch

      - run: npx @eqworks/commit-watch -b ${{ github.event.pull_request.base.sha }} -h ${{ github.event.pull_request.head.sha }} -v
```

And the CLI exits with an error when encountering any unsatisfactory commit messages against the convention. You can find more usage details in the [`commit-watch` repo](https://github.com/EQWorks/commit-watch).

## Environment Variables Usage and Security Sanitization

Another aspect that often haunts developers is environment variable usage, especially when we accidentally need to remember to configure them at build or deployment time. To mitigate that, we created another CLI.

```shell
(@eqworks/updates repo) % npx @eqworks/scan-env
Missing in live context

ORG_TZ:
	cli.js (has default)
	sources/util.js (has default)
GITHUB_TOKEN:
	sources/github/graphql.js
GITHUB_ORG:
	sources/github/graphql.js (has default)
	sources/github/v2.js (has default)
NOTION_TOKEN:
	sources/notion/api.js
BASE_URL:
	sources/notion/index.js (has default)
DATABASE_ID:
	targets/notion/index.js (has default)
SLACK_TOKEN:
	targets/slack.js
SLACK_CHANNEL:
	targets/slack.js
```

The CLI works well with Python and JavaScript (Node.js) projects. In the future, we plan to support more language stacks. And since we use Serverless extensively, we have also built good support to detect when the application's environment variables mismatch with the references dictated in the `serverless.yml` configuration.

```shell
(a-serverless-service) % npx @eqworks/scan-env
Missing in serverless.yml

LOG_LEVEL:
	src/__init__.py (has default)
STAGE:
	src/app.py (has default)
PG_URI:
	src/modules/pg.py (has default)
MONGO_URI:
    src/modules/mongo.py

Unused from serverless.yml

NOT_USED_ANIMO
SNOOP_LEON
NICKY_JELLY
```

Like `commit-watch`, the CLI easily integrates into any CI/CD workflow.

```yaml
name: Main

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  scan-env:
    runs-on: ubuntu-latest
    steps:
      - run: npx @eqworks/scan-env --strict
        env:
          # any build/deploy/run-time environment variables
          PG_URI: ${{ secrets.PG_URI }}
          MONGO_URI: ${{ secrets.MONGO_URI }}
```

You can find more details in [its repo](https://github.com/EQWorks/scan-env).

## Bonus: Slack Notification

While this one is not strictly about QA, we often use it hand-in-hand with all QA-related CLIs in our CI/CD workflows to notify us of various statuses.

```yaml
name: Serverless Deploy - prod stage

on:
  push:
    tags:
      - 'prod-*'

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0

      # ...

      - run: npx serverless deploy --stage=prod
        env:
          # ...

      - name: Notify on deployment status
        if: ${{ always() }}
        run: npx @eqworks/notify deployment $GITHUB_REPOSITORY --commit=$GITHUB_SHA --stage=prod --status=$JOB_STATUS
        env:
          SLACK_HOOK: ${{ secrets.SLACK_HOOK }}
          JOB_STATUS: ${{ job.status }}

      - name: Generate tag associated release changelog
        run: npx @eqworks/release changelog -v --pattern="prod-*" --head=${GITHUB_REF##*/} --github
        env:
          GITHUB_OWNER: ...
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Notify on release changelog update
        if: ${{ success() }}
        run: npx @eqworks/notify send "Release changelog updated for ${GITHUB_REPOSITORY} ${GITHUB_REF##*/}" --link="https://github.com/${GITHUB_REPOSITORY}/releases/tag/${GITHUB_REF##*/}" --status=$JOB_STATUS
        env:
          SLACK_HOOK: ${{ secrets.SLACK_HOOK }}
          JOB_STATUS: ${{ job.status }}
```

There are more details in the [`notify` repo](https://github.com/EQWorks/notify), including its usage as a Node.js library.

## Final Remarks

While far from comprehensive, we've built a little collection to start saying, "There's a CLI for that." The point, though, is to identify common-enough issues to create flexible and composable toolings to form reasonable solutions while getting as much value as possible.

Do you have similar itches? Do you have any other ideas? Let us know by creating issues in the respective repositories:
* https://github.com/EQWorks/commit-watch
* https://github.com/EQWorks/scan-env
* https://github.com/EQWorks/notify
