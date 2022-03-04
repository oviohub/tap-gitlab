# `tap-gitlab`

GitLab tap class.

Built with the [Meltano SDK](https://sdk.meltano.com) for Singer Taps and Targets.

## Capabilities

* `catalog`
* `state`
* `discover`
* `about`
* `stream-maps`
* `schema-flattening`

## Settings

| Setting                    | Required | Default | Description |
|:---------------------------|:--------:|:-------:|:------------|
| api_url                    | False    | None    | Overrides the base URL. |
| private_token              | True     | None    | TODO        |
| groups                     | False    | None    | TODO        |
| projects                   | False    | None    | TODO        |
| start_date                 | False    | None    | TODO        |
| ultimate_license           | False    | None    | TODO        |
| fetch_merge_request_commits| False    | None    | TODO        |
| fetch_pipelines_extended   | False    | None    | TODO        |
| fetch_group_variables      | False    | None    | TODO        |
| fetch_project_variables    | False    | None    | TODO        |
| stream_maps                | False    | None    | Config object for stream maps capability. |
| stream_map_config          | False    | None    | User-defined config values to be used within map expressions. |
| flattening_enabled         | False    | None    | 'True' to enable schema flattening and automatically expand nested properties. |
| flattening_max_depth       | False    | None    | The max depth to flatten schemas. |

A full list of supported settings and capabilities is available by running: `tap-gitlab --about`

# tap-gitlab

`tap-gitlab` is a Singer tap for GitLab.

Built with the [Meltano Tap SDK](https://sdk.meltano.com) for Singer Taps.

## Installation

- [ ] `Developer TODO:` Update the below as needed to correctly describe the install procedure. For instance, if you do not have a PyPi repo, or if you want users to directly install from your git repo, you can modify this step as appropriate.

```bash
pipx install tap-gitlab
```

## Configuration

The following settings are expected:

```js
{
  "api_url": "https://gitlab.com",   //overrides the base URL
  "private_token": "your-access-token",
  "groups": "myorg mygroup",
  "projects": "myorg/repo-a myorg/repo-b",
  "start_date": "2018-01-01T00:00:00Z",
  "ultimate_license": true,
  "fetch_merge_request_commits": false,
  "fetch_pipelines_extended": false,
  "fetch_group_variables": false,
  "fetch_project_variables": false
}
```

### Accepted Config Options

TODO

### About the streams

This tap:
- Pulls raw data from GitLab's [REST API](https://docs.gitlab.com/ee/api/README.html)
- Extracts the following resources from GitLab:
  - [Branches](https://docs.gitlab.com/ee/api/branches.html)
  - [Commits](https://docs.gitlab.com/ee/api/commits.html)
  - [Issues](https://docs.gitlab.com/ee/api/issues.html)
  - [Pipelines](https://docs.gitlab.com/ee/api/pipelines.html)
  - [Jobs](https://docs.gitlab.com/ee/api/jobs.html)
  - [Projects](https://docs.gitlab.com/ee/api/projects.html)
  - [Project milestones](https://docs.gitlab.com/ee/api/milestones.html)
  - [Project Merge Requests](https://docs.gitlab.com/ee/api/merge_requests.html)
  - [Users](https://docs.gitlab.com/ee/api/users.html)
  - [Groups](https://docs.gitlab.com/ee/api/group_milestones.html)
  - [Group Milestones](https://docs.gitlab.com/ee/api/users.html)
  - [Group and Project members](https://docs.gitlab.com/ee/api/members.html)
  - [Tags](https://docs.gitlab.com/ee/api/tags.html)
  - [Releases](https://docs.gitlab.com/ee/api/releases/index.html)
  - [Group Labels](https://docs.gitlab.com/ee/api/group_labels.html)
  - [Project Labels](https://docs.gitlab.com/ee/api/labels.html)
  - [Epics](https://docs.gitlab.com/ee/api/epics.html) (only available for GitLab Ultimate and GitLab.com Gold accounts)
  - [Epic Issues](https://docs.gitlab.com/ee/api/epic_issues.html) (only available for GitLab Ultimate and GitLab.com Gold accounts)
  - [Vulnerabilities](https://docs.gitlab.com/ee/api/project_vulnerabilities.html)
  - [Group Variables](https://docs.gitlab.com/ee/api/group_level_variables.html)
  - [Project Variables](https://docs.gitlab.com/ee/api/project_level_variables.html)
- Outputs the schema for each resource
- Incrementally pulls data based on the input state

## Quick start

1. Install

Currently this project is not hosted on Python Package Index. To install, run:
```
pip install git+https://gitlab.com/meltano/tap-gitlab.git
```

2. Get your GitLab access token

    - Login to your GitLab account
    - Navigate to your profile page
    - Create an access token

3. Create the config file

    Create a JSON file called `config.json` containing:
    - Access token you just created
    - API URL for your GitLab account. If you are using the public gitlab.com this will be `https://gitlab.com/api/v4`
    - Groups to track (space separated)    
    - Projects to track (space separated)

    Notes on group and project options:
    - either groups or projects need to be provided
    - filling in 'groups' but leaving 'projects' empty will sync all group projects.
    - filling in 'projects' but leaving 'groups' empty will sync selected projects.
    - filling in 'groups' and 'projects' will sync selected projects of those groups.

    ```json
    {
      "api_url": "https://gitlab.com",
      "private_token": "your-access-token",
      "groups": "myorg mygroup",
      "projects": "myorg/repo-a myorg/repo-b",
      "start_date": "2018-01-01T00:00:00Z",
      "ultimate_license": true,
      "fetch_merge_request_commits": false,
      "fetch_pipelines_extended": false,
      "fetch_group_variables": false,
      "fetch_project_variables": false
    }
    ```

    The `api_url` requires only the base URL of the GitLab instance, e.g. `https://gitlab.com`. `tap-gitlab` automatically uses the latest (v4) version of GitLab's API. If you really want to set a different API version, you can set the full API URL, e.g. `https://gitlab.com/api/v3`, but be warned that this tap is built for API v4.

    If `ultimate_license` is true (defaults to false), then the GitLab account used has access to the GitLab Ultimate or GitLab.com Gold features. It will enable fetching Epics, Epic Issues and other entities available for GitLab Ultimate and GitLab.com Gold accounts.

    If `fetch_merge_request_commits` is true (defaults to false), then for each Merge Request, also fetch the MR's commits and create the join table `merge_request_commits` with the Merge Request and related Commit IDs. In the current version of GitLab's API, this operation requires one API call per Merge Request, so setting this to True can slow down considerably the end-to-end extraction time. For example, in a project like `gitlab-org/gitlab-foss`, this would result to 15x more API calls than required for fetching all the other Entities supported by `tap-gitlab`.

    If `fetch_pipelines_extended` is true (defaults to false), then for every Pipeline fetched with `sync_pipelines` (which returns N pages containing all pipelines per project), also fetch extended details of each of these pipelines with `sync_pipelines_extended`. Similar concerns as those related to `fetch_merge_request_commits` apply here - every pipeline fetched with `sync_pipelines_extended` requires a separate API call.

    If `fetch_group_variables` is true (defaults to false), then Group-level CI/CD variables will be retrieved for each available / specified group. This feature is treated as an opt-in to prevent users from accidentally extracting any potential secrets stored as Group-level CI/CD variables.

    If `fetch_project_variables` is true (defaults to false), then Project-level CI/CD variables will be retrieved for each available / specified project. This feature is treated as an opt-in to prevent users from accidentally extracting any potential secrets stored as Project-level CI/CD variables.

4. [Optional] Create the initial state file

    You can provide JSON file that contains a date for the API endpoints
    to force the application to only fetch data newer than those dates.
    If you omit the file it will fetch all GitLab data

    ```json
    {
      "project_278964": "2017-01-17T00:00:00Z",
      "project_278964_issues": "2017-01-17T00:00:00Z",
      "project_278964_merge_requests": "2017-01-17T00:00:00Z",
      "project_278964_commits": "2017-01-17T00:00:00Z"
    }
    ```

    Note:
    - You have to provide the id of each project you are syncing. For example, in the case of `gitlab-org/gitlab` it is 278964.
    - You can find the Project ID for a project in the homepage for the project, under its name.

5. Run the application

    `tap-gitlab` can be run with:

    ```bash
    tap-gitlab --config config.json [--state state.json]
    ```

### Source Authentication and Authorization

- [ ] `Developer TODO:` If your tap requires special access on the source system, or any special authentication requirements, provide those here.

## Usage

You can easily run `tap-gitlab` by itself or in a pipeline using [Meltano](https://meltano.com/).

### Executing the Tap Directly

```bash
tap-gitlab --version
tap-gitlab --help
tap-gitlab --config CONFIG --discover > ./catalog.json
```

## Developer Resources

- [ ] `Developer TODO:` As a first step, scan the entire project for the text "`TODO:`" and complete any recommended steps, deleting the "TODO" references once completed.

### Initialize your Development Environment

```bash
pipx install poetry
poetry install
```

### Create and Run Tests

Create tests within the `tap_gitlab/tests` subfolder and
  then run:

```bash
poetry run pytest
```

You can also test the `tap-gitlab` CLI interface directly using `poetry run`:

```bash
poetry run tap-gitlab --help
```

### Testing with [Meltano](https://www.meltano.com)

_**Note:** This tap will work in any Singer environment and does not require Meltano.
Examples here are for convenience and to streamline end-to-end orchestration scenarios._

Your project comes with a custom `meltano.yml` project file already created. Open the `meltano.yml` and follow any _"TODO"_ items listed in
the file.

Next, install Meltano (if you haven't already) and any needed plugins:

```bash
# Install meltano
pipx install meltano
# Initialize meltano within this directory
cd tap-gitlab
meltano install
```

Now you can test and orchestrate using Meltano:

```bash
# Test invocation:
meltano invoke tap-gitlab --version
# OR run a test `elt` pipeline:
meltano elt tap-gitlab target-jsonl
```

### SDK Dev Guide

See the [dev guide](https://sdk.meltano.com/en/latest/dev_guide.html) for more instructions on how to use the SDK to 
develop your own taps and targets.
