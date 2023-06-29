# aws-oidc-helper

<!---
[![CircleCI Build Status](https://circleci.com/gh/<organization>/<project-name>.svg?style=shield "CircleCI Build Status")](https://circleci.com/gh/<organization>/<project-name>) [![CircleCI Orb Version](https://badges.circleci.com/orbs/<namespace>/<orb-name>.svg)](https://circleci.com/developer/orbs/orb/<namespace>/<orb-name>) [![GitHub License](https://img.shields.io/badge/license-MIT-lightgrey.svg)](https://raw.githubusercontent.com/<organization>/<project-name>/master/LICENSE) [![CircleCI Community](https://img.shields.io/badge/community-CircleCI%20Discuss-343434.svg)](https://discuss.circleci.com/c/ecosystem/orbs)

--->

This Orb help circleci workflow to assume IAM role with OpenID Connect(OIDC).

## Setup
According to [the official guide](https://circleci.com/docs/ja/openid-connect-tokens/), create necessary resources for AWS OIDC federation(OIDC provider, IAM role, context, etc...)

If you use get-and-pass-ecr-password job, you should enable [dynamic config](https://circleci.com/docs/ja/dynamic-config/#getting-started-with-dynamic-config-in-circleci) at first.

## Usage example
```yaml
version: 2.1
setup: << pipeline.parameters.is-setup >>
parameters:
is-setup:
    type: boolean
    default: true
ecr-password:
    type: string
    default: ""
orbs:
    aws-oidc-helper: mniw/aws-oidc-helper@0.0.1
jobs:
build:
    docker:
        - image: (aws-account-id).dkr.ecr.ap-northeast-1.amazonaws.com/ci_image:latest
        auth:
            username: AWS
            password: << pipeline.parameters.ecr-password >>
    environment:
        AWS_DEFAULT_REGION: ap-northeast-1
        AWS_ROLE_ARN: arn:aws:iam::(aws-account-id):role/role-for-circleci-oidc
    steps:
        - checkout
        - aws-oidc-helper/assume-role:
            aws_role_arn: ${AWS_ROLE_ARN}
        - run: your necessary build process
workflows:
version: 2
setup:
    when: << pipeline.parameters.is-setup >>
    jobs:
        - aws-oidc-helper/get-and-pass-ecr-password:
            aws_role_arn: ${AWS_ROLE_ARN}
            context:
                - aws-oidc-context
build:
    when:
        not: << pipeline.parameters.is-setup >>
    jobs:
        - build:
            context:
                - aws-oidc-context
```

---

## Resources

[CircleCI Orb Registry Page](https://circleci.com/developer/orbs/orb/<namespace>/<orb-name>) - The official registry page of this orb for all versions, executors, commands, and jobs described.

[CircleCI Orb Docs](https://circleci.com/docs/orb-intro/#section=configuration) - Docs for using, creating, and publishing CircleCI Orbs.

### How to Contribute

We welcome [issues](https://github.com/<organization>/<project-name>/issues) to and [pull requests](https://github.com/<organization>/<project-name>/pulls) against this repository!

### How to Publish An Update
1. Merge pull requests with desired changes to the main branch.
    - For the best experience, squash-and-merge and use [Conventional Commit Messages](https://conventionalcommits.org/).
2. Find the current version of the orb.
    - You can run `circleci orb info <namespace>/<orb-name> | grep "Latest"` to see the current version.
3. Create a [new Release](https://github.com/<organization>/<project-name>/releases/new) on GitHub.
    - Click "Choose a tag" and _create_ a new [semantically versioned](http://semver.org/) tag. (ex: v1.0.0)
      - We will have an opportunity to change this before we publish if needed after the next step.
4.  Click _"+ Auto-generate release notes"_.
    - This will create a summary of all of the merged pull requests since the previous release.
    - If you have used _[Conventional Commit Messages](https://conventionalcommits.org/)_ it will be easy to determine what types of changes were made, allowing you to ensure the correct version tag is being published.
5. Now ensure the version tag selected is semantically accurate based on the changes included.
6. Click _"Publish Release"_.
    - This will push a new tag and trigger your publishing pipeline on CircleCI.