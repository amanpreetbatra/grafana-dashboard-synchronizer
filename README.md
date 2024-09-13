# Grafana Dashboard Synchronizer

A small CLI tool to do a tag-base automatic synchornization and backup Grafana of dashboards across multiple Grafana instances.

This application can be used to synchronize dashboards, using a Git repository, across multiple Grafana instances.  
A possible use cases:
- Backup certain dashboards regularly using a Git repository
- Push Grafana dashboards from one Grafana instance to a Git repository and import them into another Grafana instance. In addition, users can use tags to determine for themselves when a dashboard should be synchronized.
- Preload new Grafana instances with predefined dashboards.

#### Example Usecase

This can be useful to stage dashboards from "dev" to "prod" environments.

![image](images/dashboard-synchronizer.png)

## Usage

The application can be used as follows:

    $ ./grafana-dashboard-synchronizer [options]

By default, the application will use a configuration file named `configuration.yml` next to the binary. A custom configuration file can be used using the `--config` or `-c` option flag:

    $ ./grafana-dashboard-synchronizer --config /custom/configuration.yml

In addition, a dry-run flag can be used. When the `--dry-run` flag is used, the application does not perform any modifications. This can be useful when testing what changes would be made.

    $ ./grafana-dashboard-synchronizer --dry-run

By default, the application logs in an easy-to-read text format. With the `--log-as-json` flag, the application generates logs in JSON format, which is convenient if the logs are processed by other tools such as Logstash:

    $ ./grafana-dashboard-synchronizer
      INFO[0000] Synchronizing Grafana dashboards...
      ...

compared to:

    $ ./grafana-dashboard-synchronizer --log-as-json
      {"level":"info","msg":"Synchronizing Grafana dashboards...","time":"2022-02-08T16:41:26+01:00"}
      ...

### Configuration

The configuration file can contain multiple jobs, which will be sequentially executed. Furthermore, the push (export) step of a job is executed before its pull (import) step.

See the following configuration for available configuration options:

    - job-name:      "example-job-1"
      # API token to interact with the specified Grafana instance
      grafana-token: "eyJrIjoiSEp4dzhGdVBxMUhBdm..."
      # Base URL of the Grafana instance
      grafana-url:   "http://localhost:3000"
      # SSH-URL of the Git repository to use
      git-repository-url: "<GIT_REPOSITORY_URL>"
      # Auth Option 1: Private key to use for authentication against the Git repository
      private-key-file:   "<PRIVATE_SSH_KEY>"
      # Auth Option 2: Use authentication token (eg. Github or Gitlab token)
      git-username: "<GIT_USERNAME>"
      git-password: "<GIT_PASSWORD>" # this is where your authentication token goes

      # push (export) related configurations
      push-configuration:
         # whether to export dashboards
         enable: true
         # the branch to use for exporting dashboards
         git-branch: "push-branch"
         # only dashboards with match this pattern will be considered in the sync process
         filter: ""
         # the tag to determine which dashboards should be exported
         tag-pattern: "agent"
         # whether the sync-tag should be kept during exporting
         push-tags: true

      # pull (import) related configurations  
      pull-configuration:
         # whether to import dashboards
         enable: true
         # the branch to use for importing dashboards
         git-branch: "pull-branch"
         # only dashboards with match this pattern will be considered in the sync process
         filter: ""

## Development

### Getting started

1. Update and get dependencies:

   ```bash
   go mod tidy
   ```

2. Build binaries for Linux, Windows and Darwin:

   ```bash
   mage -v
   ```

3. List all available Mage targets for additional commands:

   ```bash
   mage -l
   ```

### Releasing the Application

The release process of the application is automated using Github Actions.
On each push to the `main` branch, a new prerelease is created and the corresponding commit is tagged "latest".
Old prereleases will be deleted.

To create a normal release, the commit that is used as the basis for the release must be tagged with the following format: `v*.*.*`.
After that, the release is built and created with the version number extracted from the tag.
Furthermore, a new commit is created, which sets the current version in the `main` branch to the version that has been released.
