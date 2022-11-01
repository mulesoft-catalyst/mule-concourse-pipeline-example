# single-job-pipeline

## About

This example pipeline has a single job **job-publish-and-deploy-asset** demonstrating 3 lifecycle steps for a Mule 4 application + Runtime Fabric deployment model:

1. **Build** and **test** the application.
2. **Publish** the application to **Exchange**.
3. **Deploy** the published application to a RTF instance.

## Description

### Resources

|No.|Name|Type|Description
|-|-|-|-
|1|**source-code-resource**|git|Reference to a Mule 4 Application git repository
|2|**pipeline-resource**|git|Reference to this generic Mule 4 Concourse pipeline git repository
|3|**version-resource**|semver|Semver versioning support (git driver)

### Job Plan - Resources

|No.|Step|Description
|-|-|-
|1|**get**: source-code-resource|Get source code to build from git repository
|2|**get**: pipeline-resource|Get generic Mule 4 pipeline resource from git repository
|3|**get**: version-resource|Get version to build using Semver

### Job Plan - Tasks
  
|No.|Task|Description|Maven
|-|-|-|-
|1|**build-and-verify-asset**|Build / verify the Mule 4 application|- `mvn versions:set`<br>- `mvn verify`
|2|**build-and-publish-asset**|Publish the Mule 4 application to Exchange (MMP)|- `mvn versions:set`<br>- `mvn deploy -DskipTests`
|3|**deploy-asset**|Deploy the Mule 4 Application to a Runtime Fabric instance (MMP)|- `mvn versions:set`<br>- `mvn deploy -DmuleDeploy -DskipTests`

### Supporting Jobs

No.|Job|Description
|-|-|-
|1|**increase-major-version**|1. Bump the **major** version<br>2. Trigger the **job-publish-and-deploy-asset** job
|2|**increase-minor-version**|1. Bump the **minor** version <br>2. Trigger the **job-publish-and-deploy-asset** job
|3|**increase-patch-version**|1. Bump the **patch** version<br>2. Trigger the **job-publish-and-deploy-asset** job

### Groups
No.|Group|Jobs
|-|-|-
|1|**publish-and-deploy**|1. job-publish-and-deploy-asset
|2|**version-management**|1. increase-major-version<br>2. increase-minor-version<br>3. increase-patch-version

## Usage

### Credentials

1. Create a **credentials.yml** file based on the **credentials-template.yml** file, e.g.:

The **Source** properties refer to the Mule Application to build, test and deploy, e.g.:

```
source-code-resource-private-key: |
      -----BEGIN OPENSSH PRIVATE KEY-----
      -----END OPENSSH PRIVATE KEY-----
```

The **Pipeline** properties refer to a repository containing the generic pipeline assets as provided in this repository, e.g.:

```
pipeline-resource-private-key: |
      -----BEGIN OPENSSH PRIVATE KEY-----
      -----END OPENSSH PRIVATE KEY-----
```

The **Maven** properties can be used to set the Mule EE Nexus repository and Anypoint Exchange details, e.g.:

```
m2-settings-mule_ee_repo-username: [REPO USERNAME]
m2-settings-mule_ee_repo-password: [REPO PASSWORD]
m2-settings-exchange_repo-org-id: [EXCHANGE ORG ID]
m2-settings-exchange_repo-username: [EXCHANGE USERNAME]
m2-settings-exchange_repo-password: [EXCHANGE PASSWORD]
```

### Properties

1. Create a **properties.yml** file based on the **properties-template.yml** file, and update the following property sections:

The **Source** properties refer to the Mule Application to build, test and deploy, e.g.:

```
source-code-resource-uri: git@github.com:example-org/mule4-api.git
source-code-resource-branch: main
```

The **Pipeline** properties refer to a repository containing the generic pipeline assets as provided in this repository, e.g.:

```
pipeline-resource-uri: git@github.com:example-org/mule-concourse-pipeline.git
pipeline-resource-branch: main
```

The **Platform** properties can be used to set the platform url and [Connected Apps](https://docs.mulesoft.com/access-management/connected-apps-overview) credentials, e.g.:

```
anypoint-platform-url: https://anypoint.mulesoft.com
connected-app-client-id: [CLIENT ID]
connected-app-client-secret: [CLIENT SECRET]
connected-app-grant-type: client_credentials
```

The **Environment** properties can be used to set the Environment and Runtime Fabric instance details, e.g.:

```
environment-name: Development
rtf-instance-name: fabric-name
```

The **Settings** properties can be used to set any relevant application level settings like environment name and autodiscovery properties, e.g.:

```
mule-env-name: dev
application-client-id: [CLIENT ID]
application-client-secret: [CLIENT SECRET]
```

The **Application** properties can be used to set the Application domain name and resource allocation parameters, e.g.:

```
application-domain-name: example.org
application-cpu-reserved: 500m
application-cpu-max: 1000m
application-mem-reserved: 3500Mi
application-mem-max: 3500Mi
```

The **Deployment** properties can be used to set any relevant deployment settings, e.g.:

```
deployment-mule-runtime: 4.3.0
deployment-replicas: 1
deployment-replicas-across-nodes: false
deployment-update-strategy: rolling
deployment-forward-ssl: false
deployment-clustered: false
deployment-last-mile-security: false
```

### Concourse

1. Create/update pipeline, e.g.:

```
$ fly -t anypoint sp -p mule4-workerinfo-single-job -c pipeline.yml -l properties.yml -l credentials.yml
```

The **publish-and-deploy** group:

![Concourse UI - mule4-workerinfo-dev](images/pipeline-1-a.png)

The **version-management** group:

![Concourse UI - mule4-workerinfo-dev](images/pipeline-1-b.png)

2. Unpause pipeline, e.g.:

```
$ fly -t anypoint up -p mule4-workerinfo-single-job
```

An instance of the **job-publish-and-deploy-asset** job (version: **1.0.1-rc.1-af9eff.....7457fcb**):

![Concourse UI - mule4-workerinfo-dev](images/pipeline-1-c.png)

Published Exchange asset (version: **1.0.1-rc.1-af9eff.....7457fcb**):

<img src="images/pipeline-1-d.png" alt="Concourse UI - mule4-workerinfo-dev" width="1280"/>

Deployed Application based on Exchange asset (version: **1.0.1-rc.1-af9eff.....7457fcb**):

![Concourse UI - mule4-workerinfo-dev](images/pipeline-1-e.png)

3. [ **Optional** ] - Trigger job, e.g.:

```
$ fly -t anypoint tj -j mule4-workerinfo-single-job/job-publish-and-deploy-asset
```

4. [ **Optional** ] - Watch job, e.g.:

```
$ fly -t anypoint watch -j mule4-workerinfo-single-job/job-publish-and-deploy-asset
```

5. [ **Optional** ] - Destroy pipeline, e.g.:

```
$ fly -t anypoint dp -p mule4-workerinfo-single-job
```

