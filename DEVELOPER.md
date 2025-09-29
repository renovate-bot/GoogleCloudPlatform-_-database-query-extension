# DEVELOPER.md

## Before you begin

1. Make sure you've [setup
   Toolbox](README.md#launch-the-toolbox-server-choose-one).

1. Install Python 3.11+

1. Install dependencies. We recommend using a virtualenv:

    ```bash
    pip install -r requirements.txt
    ```

1. Install test dependencies:

    ```bash
    pip install -r requirements-test.txt
    ```

## Run the App

### Setup Database

To setup the datasource to run with Toolbox, follow [these
steps](README.md#one-time-database--tool-configuration).

### Setup Toolbox

To setup Toolbox (locally or on Cloud Run), follow [these
steps](README.md#launch-the-toolbox-server-choose-one).

### Run Agent App

1. Configure the Toolbox URL

    You need to set the `TOOLBOX_URL` environment variable to point to your
    running Toolbox server. Choose the option below that matches your setup.

    #### **Option A:** If you are running Toolbox locally
    Set the `TOOLBOX_URL` environment variable to your local server, which is
    typically running on port `5000`.

    ```bash
    export TOOLBOX_URL="http://localhost:5000"
    ```

    #### **Option B:** If you are running Toolbox on Cloud Run
    1. First, authenticate your gcloud CLI with your user credentials:

        ```bash
        gcloud auth login
        ```

    1. Next, set the `TOOLBOX_URL` environment variable by fetching your live
      service's URL:

        ```bash
        export TOOLBOX_URL=$(gcloud run services describe toolbox --format 'value(status.url)')
        ```

    1. Allow your account to invoke the Cloud Run service by granting the [role
       Cloud Run invoker][invoker]

    </details>

1. [Optional] Turn on debugging by setting the `DEBUG` environment variable:

    ```bash
    export DEBUG=True
    ```

1. To run the app using uvicorn, execute the following:

    ```bash
    python run_app.py
    ```

1. View the app in your browser at http://localhost:8081.

> [!TIP]
> For hot-reloading during development, use the `--reload` flag:
> ```bash
> python run_app.py --reload
> ```

## Testing

### Run tests locally

The unit tests for this application mock the API calls to the MCP Toolbox, so
you do not need a live database or a running Toolbox instance to run them.

```bash
pytest
```

### CI Platform Setup

Cloud Build is used to run tests against Google Cloud resources in test project:
`extension-demo-testing`.

Each test has a corresponding Cloud Build trigger, see [all triggers][triggers].

#### Trigger Setup
Create a Cloud Build trigger via the UI or `gcloud` with the following specs:

* Event: Pull request
* Region:
    * `us-central1` - for AlloyDB to connect to private pool in VPC
    * `global` - for default worker pools
* Source:
  * Generation: 1st gen
  * Repo: GoogleCloudPlatform/cymbal-air-toolbox-demo (GitHub App)
  * Base branch: `^main$`
* Comment control: Required except for owners and collaborators
* Filters: add directory filter
* Config: Cloud Build configuration file
  * Location: Repository (add path to file)
* Substitution variables:
  * Add `_DATABASE_HOST` for non-cloud postgres
* Service account: set for demo service to enable ID token creation to use to
  authenticated services

#### Project Setup

1. Follow instructions to setup the test project:
    * [Set up and configure database](README.md#one-time-database--tool-configuration)
    * [Instructions for Toolbox setup](README.md#launch-the-toolbox-server-choose-one)
1. Setup Cloud Build triggers ([above](#trigger-setup))

##### Setup for Toolbox

1. Create a Cloud Build private pool
1. Enable Secret Manager API
1. Create secret, `db_user` and `db_pass`, with your database user and database password defined [here](https://googleapis.github.io/genai-toolbox/resources/sources/).

1. Allow Cloud Build to access secret
1. Add role Vertex AI User (`roles/aiplatform.user`) to Cloud Build Service
   account. Needed to run database init script.

##### Setup for Agent App

Add roles `Cloud Run Admin`, `Service Account User`, `Log Writer`, and `Artifact
   Registry Admin` to the demo service's Cloud Build trigger service account.

#### Run Integration Tests

```bash
gcloud builds submit --config integration.cloudbuild.yaml
```

> [!NOTE]
> Make sure to setup secrets described in [Setup for Toolbox](#setup-for-toolbox)

#### Trigger

To run Cloud Build tests on GitHub from external contributors, ie RenovateBot,
comment: `/gcbrun`.

#### Code Coverage
Please make sure your code is fully tested.

## Versioning

This app will be released based on version number `MAJOR.MINOR.PATCH`:

- `MAJOR`: Breaking change is made, requiring user to redeploy all or some of the app.
- `MINOR`: Backward compatible feature change or addition that doesn't require redeploying.
- `PATCH`: Backward compatible bug fixes and minor updates

[alloydb-proxy]: https://cloud.google.com/alloydb/docs/auth-proxy/connect
[cloudsql-proxy]: https://cloud.google.com/sql/docs/mysql/sql-proxy
[tunnel]: https://github.com/GoogleCloudPlatform/cymbal-air-toolbox-demo/blob/main/docs/datastore/alloydb.md#set-up-connection-to-alloydb
[config]: https://github.com/GoogleCloudPlatform/cymbal-air-toolbox-demo/blob/main/docs/datastore/alloydb.md#initialize-data-in-alloydb
[triggers]: https://console.cloud.google.com/cloud-build/triggers?e=13802955&project=extension-demo-testing
[invoker]: https://cloud.google.com/run/docs/securing/managing-access#add-principals
[vertex-ai-experiments]: https://pantheon.corp.google.com/vertex-ai/experiments/experiments
