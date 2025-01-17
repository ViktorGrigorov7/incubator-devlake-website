---
title: "Webhook"
description: >
  Webhook Plugin
---

## Summary

Incoming Webhooks are your solution to bring data to Apache DevLake when there isn't a specific plugin ready for your DevOps tool. An Incoming Webhook allows users to actively push data to DevLake.

When you create an Incoming Webhook within DevLake, DevLake generates a unique URL. You can then post JSON payloads to this URL to push data directly to your DevLake instance.

In v0.14+, users can push "incidents" and "deployments" required by DORA metrics to DevLake via Incoming Webhooks.

## Entities

Check out the [Incoming Webhooks entities](/Overview/SupportedDataSources.md#data-collection-scope-by-each-plugin) collected by this plugin.

## Metrics

Metrics that can be calculated based on the data collected from Incoming Webhooks:

- [Requirement Delivery Rate](/Metrics/RequirementDeliveryRate.md)
- [Requirement Granularity](/Metrics/RequirementGranularity.md)
- [Bug Age](/Metrics/BugAge.md)
- [Bug Count per 1k Lines of Code](/Metrics/BugCountPer1kLinesOfCode.md)
- [Incident Age](/Metrics/IncidentAge.md)
- [Incident Count per 1k Lines of Code](/Metrics/IncidentCountPer1kLinesOfCode.md)
- [DORA - Deployment Frequency](/Metrics/DeploymentFrequency.md)
- [DORA - Lead Time for Changes](/Metrics/LeadTimeForChanges.md)
- [DORA - Median Time to Restore Service](/Metrics/MTTR.md)
- [DORA - Change Failure Rate](/Metrics/CFR.md)

## Configuration

- Configuring Incoming Webhooks via [Config UI](/Configuration/webhook.md)

## API Sample Request

### Deployment

If you want to collect deployment data from your system, you can use the incoming webhooks for deployment.

#### Payload Schema

You can copy the generated deployment curl commands to your CI/CD script to post deployments to Apache DevLake. Below is the detailed payload schema:

|     Key     | Required | Notes                                                                                                                                                                           |
| :---------: | :------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| pipeline_id |  ✖️ No   | related Domain Layer `cicd_pipelines.id`                                                                                                                                         |
| environment |  ✖️ No   | the environment this deployment happens. For example, `PRODUCTION` `STAGING` `TESTING` `DEVELOPMENT`. <br/>The default value is `PRODUCTION`                  |
|  repo_url   |  ✔️ Yes  | the repo URL of the deployment commit<br />If there is a row in the domain layer table `repos` where `repos.url` equals `repo_url`, the `repoId` will be filled with `repos.id`. |
|   repo_id   |  ✖️ No   | related Domain Layer `repos.id` <br/> No default value.                                                                                                                          |
|    name     |  ✖️ No   | deployment name. The default value is "deployment for `request.commit_sha`"                                                                                   |
|  ref_name   |  ✖️ No   | related branch/tag<br/> No default value.                                                                                                                     |
| commit_sha  |  ✔️ Yes  | the sha of the deployment commit                                                                                                                              |
| commit_msg  |  ✖️ No   | the sha of the deployment commit message                                                                                                                      |
| create_time |  ✖️ No   | Time. Eg. 2020-01-01T12:00:00+00:00<br/> No default value.                                                                                                                       |
| start_time  |  ✔️ Yes  | Time. Eg. 2020-01-01T12:00:00+00:00<br/> No default value.                                                                                                    |
|  end_time   |  ✖️ No   | Time. Eg. 2020-01-01T12:00:00+00:00<br/> The default value is the time when DevLake receives the POST request.                                                                   |
|   result    |  ✖️ No   | deployment result, one of the values : `SUCCESS`, `FAILURE`, `ABORT`, `MANUAL`, <br/> The default value is `SUCCESS`.                                                            |
| deploymentCommits[]    |  ✖️ yes  | Allow deployment webhook to push deployments to multiple repos in one request, includes repo_url,commit_sha,commit_msg,name,ref_name    |



#### Register a Deployment - Sample API Calls

To deploy on a single repository, use the following command:
```
curl <devlake-host>/api/rest/plugins/webhook/1/deployments -X 'POST' -d '{
    "pipeline_id": "optional-pipeline-id",
    "environment":"PRODUCTION",
    "repo_url":"https://github.com/apache/incubator-devlake/",
    "repo_id": "optional-repo-id",
    "name": "optional-deployment-name. If you do not post a name, DevLake will generate one for you.",
    "ref_name": "optional-release-v0.17",
    "commit_sha":"015e3d3b480e417aede5a1293bd61de9b0fd051d",
    "commit_msg":"optional-commit-message",
    "create_time":"2020-01-01T11:00:00+00:00",
    "start_time":"2020-01-01T12:00:00+00:00",
    "end_time":"2020-01-02T13:00:00+00:00",
    "result": "FAILURE"
  }'
```

To deploy across multiple repositories (refer to the [discussion](https://github.com/apache/incubator-devlake/discussions/6162)), use the following command:
```
curl <devlake-host>/api/rest/plugins/webhook/1/deployments -X 'POST' -d '{
    "pipeline_id": "optional-pipeline-id",
    "environment":"PRODUCTION",
    "repo_id": "optional-repo-id",
    "name": "optional-deployment-name. If you do not post a name, DevLake will generate one for you.",
    "create_time":"2020-01-01T11:00:00+00:00",
    "start_time":"2020-01-01T12:00:00+00:00",
    "end_time":"2020-01-02T13:00:00+00:00",
    "result": "FAILURE",
    "deploymentCommits":[
       {
           "repo_url":"repo-1",
           "name":"optional, if null, it will be deployment for {commit_sha}",
           "ref_name": "optional-release-v0.17",
           "commit_sha":"c1",
           "commit_msg":"optional-msg-1"
       },
       {
           "repo_url":"repo-2",
           "name":"optional, if null, it will be deployment for {commit_sha}",
           "ref_name": "optional-release-v0.17",
           "commit_sha":"c2",
           "commit_msg":"optional-msg-2"
       }
    ]
  }'
```

If you have set a [username/password](GettingStarted/Authentication.md) for Config UI, you'll need to add them to the curl command to register a `deployment`:

```
curl <devlake-host>/api/rest/plugins/webhook/1/deployments -X 'POST' -u 'username:password' -d '{
    "deploymentCommits":[
        {
        "commit_sha":"the sha of deployment commit1",
        "repo_url":"the repo URL of the deployment commit"
        }
    ],
    "start_time":"2020-01-01T12:00:00+00:00",
    "end_time":"2020-01-02T12:00:00+00:00"
  }'
```

#### A real-world example - Push CircleCI deployments to DevLake

The following demo shows how to post "deployments" to DevLake from CircleCI. In this example, the CircleCI job 'deploy' is used to manage deployments.

```
version: 2.1

jobs:
  build:
    docker:
      - image: cimg/base:stable
    steps:
      - checkout
      - run:
          name: "build"
          command: |
            echo Hello, World!

  deploy:
    docker:
      - image: cimg/base:stable
    steps:
      - checkout
      - run:
          name: "deploy"
          command: |
            # The time a deploy started
            start_time=`date '+%Y-%m-%dT%H:%M:%S%z'`

            # Some deployment tasks here ...
            echo Hello, World!

            # Send the request to DevLake after deploy
            # The values start with a '$CIRCLE_' are CircleCI's built-in variables
            curl <devlake-host>/api/rest/plugins/webhook/1/deployments -X 'POST' -d "{
              \"commit_sha\":\"$CIRCLE_SHA1\",
              \"repo_url\":\"$CIRCLE_REPOSITORY_URL\",
              \"start_time\":\"$start_time\"
            }"

workflows:
  build_and_deploy_workflow:
    jobs:
      - build
      - deploy
```

### Incident / Issue

If you want to collect issue or incident data from your system, you can use the two webhooks for issues.

#### Register Issues - Update or Create Issues

`POST <devlake-host>/api/rest/plugins/webhook/1/issues`

needs to be called when an issue or incident is created. The body should be a JSON and include columns as follows:

|          Keyname          | Required | Notes                                                         |
| :-----------------------: | :------: | ------------------------------------------------------------- |
|            url            |  ✖️ No   | issue's URL                                                   |
|         issue_key         |  ✔️ Yes  | issue's key, needs to be unique in a connection               |
|           title           |  ✔️ Yes  |                                                               |
|        description        |  ✖️ No   |                                                               |
|         epic_key          |  ✖️ No   | in which epic.                                                |
|           type            |  ✖️ No   | type, such as bug/incident/epic/...                           |
|          status           |  ✔️ Yes  | issue's status. Must be one of `TODO` `DONE` `IN_PROGRESS`    |
|      original_status      |  ✔️ Yes  | status in your system, such as created/open/closed/...        |
|        story_point        |  ✖️ No   |                                                               |
|      resolution_date      |  ✖️ No   | date, Format should be 2020-01-01T12:00:00+00:00              |
|       created_date        |  ✔️ Yes  | date, Format should be 2020-01-01T12:00:00+00:00              |
|       updated_date        |  ✖️ No   | date, Format should be 2020-01-01T12:00:00+00:00              |
|     lead_time_minutes     |  ✖️ No   | how long from this issue accepted to develop                  |
|     parent_issue_key      |  ✖️ No   |                                                               |
|         priority          |  ✖️ No   |                                                               |
| original_estimate_minutes |  ✖️ No   |                                                               |
|    time_spent_minutes     |  ✖️ No   |                                                               |
|  time_remaining_minutes   |  ✖️ No   |                                                               |
|        creator_id         |  ✖️ No   | the user id of the creator                                    |
|       creator_name        |  ✖️ No   | the user name of the creator, it will just be used to display |
|        assignee_id        |  ✖️ No   |                                                               |
|       assignee_name       |  ✖️ No   |                                                               |
|         severity          |  ✖️ No   |                                                               |
|         component         |  ✖️ No   | which component is this issue in.                             |

More information about these columns at [DomainLayerIssueTracking](https://devlake.apache.org/docs/DataModels/DevLakeDomainLayerSchema#domain-1---issue-tracking).

#### Register Issues - Close Issues (Optional)

`POST <devlake-host>/api/rest/plugins/webhook/1/issue/:issueId/close`

needs to be called when an issue or incident is closed. Replace `:issueId` with specific strings and keep the body empty.

#### Register Issues - Sample API Calls

Sample CURL for creating an incident:

```
curl <devlake-host>/api/rest/plugins/webhook/1/issues -X 'POST' -d '{
  "issue_key":"DLK-1234",
  "title":"a feature from DLK",
  "description":"",
  "url":"",
  "type":"INCIDENT",
  "status":"TODO",
  "created_date":"2020-01-01T12:00:00+00:00",
  "updated_date":"2020-01-01T12:00:00+00:00",
  "priority":"",
  "severity":"",
  "creator_id":"user1131",
  "creator_name":"Nick name 1",
  "assignee_id":"user1132",
  "assignee_name":"Nick name 2"
}'
```

Sample CURL for Issue Closing:

```
curl <devlake-host>/api/rest/plugins/webhook/1/issue/DLK-1234/close -X 'POST'
```

## References

- [references](/DeveloperManuals/DeveloperSetup.md#references)
