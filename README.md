# Bulk Merger

Bulk merge GitHub Pull Requests across an entire Organization.

## Installation

- Use [rvm](http://rvm.io/) to install Ruby 2.6.6: `rvm install 2.6.6` (this may take a while).
- Install the Gems by running `bundle install` in the repo root.
- Optionally add the `./bin` directory to your PATH.

## Setup

Create a [Personal access token](https://github.com/settings/tokens), with at least the `repo/public_repo`
scope. Add your token with repo access to `bin/.env`, and configure other parameters:

```
# REQUIRED: API token
export GITHUB_TOKEN=<your-token>

# REQUIRED: Username/Organization (e.g. "alphagov")
export GITHUB_USER=<username>

# OPTIONAL: String to search for in open PR titles (default is "[DEPENDABOT]")
# export QUERY_STRING=

# OPTIONAL: A topic/tag set on one or more repositories in the org. Only these
#  repositories will be searched for PRs. If not defined, all repositories in
#  the org will be searched. (e.g. "govuk")
# export TOPIC=
```

The scripts will use this token to talk to GitHub.

## Bulk approve PRs

```bash
# ./review [<QUERY>] [<TOPIC>]
$ ./review gds-api-adapters
```

- QUERY: Optional piece of text to search for in open PR titles. Defaults to `"[DEPENDABOT]"`, which is the [commit prefix](https://docs.github.com/en/github/administering-a-repository/configuration-options-for-dependency-updates#commit-message) we add to all our Dependabot PRs. You can use this to search for `"babel/core"`, for example, which would find all open PRs across, all repos in the `GITHUB_USER` org, that related to upgrading that dependency.
    -   Tip: To use the default QUERY but specify a TOPIC, call `./review "" topic` (pass an empty string as the QUERY).
- TOPIC: Optional topic/tag to override what's in the `./.env` file.

The above example looks for unreviewed PRs in the configured organization with "gds-api-adapters" (an NPM package name) in the title. If it finds any, it will list them out and ask you to confirm if you want to approve them.

Use this command for third-party libraries that need [a second GOV.UK reviewer](https://docs.publishing.service.gov.uk/manual/manage-ruby-dependencies.html#who-can-merge-dependabot-prs).

```bash
$ ./review gds-api-adapters
Searching for PRs containing 'gds-api-adapters'
Found 3 unreviewed PRs:

- 'Bump gds-api-adapters from 53.2.0 to 54.0.0' (https://github.com/alphagov/short-url-manager/pull/262)
- 'Bump gds-api-adapters from 53.2.0 to 54.0.0' (https://github.com/alphagov/local-links-manager/pull/328)
- 'Bump gds-api-adapters from 53.2.0 to 54.0.0' (https://github.com/alphagov/email-alert-service/pull/205)

Have you reviewed the changes, and you want to approve all these PRs? [y/N]
y
OK! Approving away...
Reviewing PR 'Bump gds-api-adapters from 53.2.0 to 54.0.0' (https://github.com/alphagov/short-url-manager/pull/262) ✅
Reviewing PR 'Bump gds-api-adapters from 53.2.0 to 54.0.0' (https://github.com/alphagov/local-links-manager/pull/328) ✅
Reviewing PR 'Bump gds-api-adapters from 53.2.0 to 54.0.0' (https://github.com/alphagov/email-alert-service/pull/205) ✅
```

## Approve and merge all Pull Requests

```bash
# ./merge <QUERY> [<TOPIC>]
$ ./merge gds-api-adapters
```

This will run the `./review` script above, but also merge the approved PRs.

> 👉 If a repository has [branch protection turned on](https://docs.github.com/en/github/administering-a-repository/about-protected-branches), you won't be able to merge PRs that haven't passed on CI.

```bash
Found 4 reviewed but unmerged PRs:

- 'Bump gds-api-adapters from 53.2.0 to 54.0.0' (https://github.com/alphagov/short-url-manager/pull/262)
- 'Bump gds-api-adapters from 53.1.0 to 54.0.0' (https://github.com/alphagov/frontend/pull/1658)
- 'Bump gds-api-adapters from 53.2.0 to 54.0.0' (https://github.com/alphagov/local-links-manager/pull/328)
- 'Bump gds-api-adapters from 53.2.0 to 54.0.0' (https://github.com/alphagov/email-alert-service/pull/205)

Have you reviewed the changes, and you want to MERGE all these PRs? [y/N]
y
OK! Approving away...
Merging PR 'Bump gds-api-adapters from 53.2.0 to 54.0.0' (https://github.com/alphagov/short-url-manager/pull/262) ❌ Failed to merge: "PUT https://api.github.com/repos/alphagov/short-url-manager/pulls/262/merge: 405 - Required status check \"continuous-integration/jenkins/branch\" is errored. // See: https://help.github.com/articles/about-protected-branches"
Merging PR 'Bump gds-api-adapters from 53.1.0 to 54.0.0' (https://github.com/alphagov/frontend/pull/1658) ❌ Failed to merge: "PUT https://api.github.com/repos/alphagov/frontend/pulls/1658/merge: 405 - 2 of 2 required status checks have not succeeded: 1 expected and 1 pending. // See: https://help.github.com/articles/about-protected-branches"
Merging PR 'Bump gds-api-adapters from 53.2.0 to 54.0.0' (https://github.com/alphagov/local-links-manager/pull/328) ✅
Merging PR 'Bump gds-api-adapters from 53.2.0 to 54.0.0' (https://github.com/alphagov/email-alert-service/pull/205) ✅
```

## List only

```bash
# ./list [<QUERY>] [<TOPIC>]
```

Only list unreviewed PRs with the given query and optional topic. Basically, `./review` without the approval part.

## Merge only approved PRs

```bash
# ./merge-only [<QUERY>] [<TOPIC>]
$ ./merge-only gds-api-adapters
```

Only merge approved PRs with the given query and optional topic. Basically, `./merge` without the review part, only for already-approved PRs (that have passed CI, if branch protection is enabled).

## Licence

[MIT License](LICENSE)
