# custom-resource-pipeline

## About

This example pipeline has 3 jobs:

1. **build-and-publish-asset**
3. **deploy-asset**

demonstrating 3 lifecycle steps for a Mule 4 application + Runtime Fabric deployment model:

1. **Build** and **test** the application.
2. **Publish** the application to **Exchange**.
3. **Deploy** the published application to a RTF instance.

## Description

### Resource Types

|No.|Name|Type|Link
|-|-|-|-
|1|**exchange-concourse-resource-type**|Custom - **Anypoint Exchange**|[exchange-concourse-resource](https://github.com/mulesoft-catalyst/exchange-concourse-resource)
|2|**rtf-concourse-resource-type**|Custom - **Anypoint Runtime Fabric**|[rtf-concourse-resource](https://github.com/mulesoft-catalyst/rtf-concourse-resource)

### Resources

|No.|Name|Type|Description
|-|-|-|-
|1|**source-code-resource**|git|Reference to a Mule 4 Application git repository
|2|**pipeline-resource**|git|Reference to this generic Mule 4 Concourse pipeline git repository
|3|**version-resource**|semver|Semver versioning support (git driver)
|4|**exchange-concourse-resource-type**|custom|**Anypoint Exchange** Mule Application asset publication support (Platform APIs)
|5|**rtf-concourse-resource-type**|custom|**Anypoint Runtime Fabric** Mule Application deployment support (Platform APIs)

### Job Plan - Resources - "build-and-publish-asset"

|No.|Step|Description
|-|-|-
|1|**get**: source-code-resource|Get source code to build from git repository
|2|**get**: pipeline-resource|Get generic Mule 4 pipeline resource from git repository
|3|**get**: version-resource|Get version to build using Semver
|4|**put**: exchange-concourse-resource|Publish application asset to Exchange using the Platform APIs

### Job Plan - Resources - "deploy-asset"

|No.|Step|Description
|-|-|-
|1|**get**: exchange-concourse-resource|Check for published application asset in Exchange
|2|**put**: rtf-concourse-resource|Deploy application asset to a Runtime Fabric instance using the Platform APIs

**Note**: The **get** performs a **check**, there is no requirement to get/download the asset, RTF will do this by providing the **GAV** reference).

### Job Plan - Task - "deploy-asset"

NA

### Supporting Jobs

No.|Job|Description
|-|-|-
|1|**increase-major-version**|1. Bump the **major** version<br>2. Trigger the **job-publish-and-deploy-asset** job
|2|**increase-minor-version**|1. Bump the **minor** version <br>2. Trigger the **job-publish-and-deploy-asset** job
|3|**increase-patch-version**|1. Bump the **patch** version<br>2. Trigger the **job-publish-and-deploy-asset** job

### Groups
No.|Group|Jobs
|-|-|-
|1|**publish-and-deploy**|1. build-and-publish-asset<br>3. deploy-asset
|2|**version-management**|1. increase-major-version<br>2. increase-minor-version<br>3. increase-patch-version

## Usage

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

The **Docker** properties refer to the Docker Registry for accessing the Resource Type by Concourse, e.g.:

```
docker-registry-ca-certs:
    - |
      -----BEGIN CERTIFICATE-----

      -----END CERTIFICATE-----
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

The **Exchange** properties refer to a Docker registry containing the image asset for this resourse type, e.g.:

```
exchange-concourse-resource-uri: docker.registry.com:10443/exchange-concourse-resource
exchange-concourse-resource-tag: latest
```

The **Runtime Fabric** properties refer to a Docker registry containing the image asset for this resourse type, e.g.:

```
rtf-concourse-resource-uri: docker.registry.com:10443/rtf-concourse-resource
rtf-concourse-resource-tag: latest
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
application-domain-name: example.com
application-cpu-reserved: 500m
application-cpu-max: 1000m
application-mem-reserved: 3500Mi
application-mem-max: 3500Mi
```

The **Deployment** properties can be used to set any relevant deployment settings, e.g.:

```
deployment-mule-runtime: 4.3.0
deployment-replicas: 1
deployment-replicas-across-nodes: "false"
deployment-update-strategy: rolling
deployment-forward-ssl: "false"
deployment-clustered: "false"
deployment-last-mile-security: "false"
```

The **Exchange** properties can be used to set the platform url, [Connected Apps](https://docs.mulesoft.com/access-management/connected-apps-overview) credentials and artifact details, e.g.:

```
exhange-resource-uri: anypoint.mulesoft.com
exchange-resource-client-id: [CLIENT ID]
exhange-resource-client-secret: [CLIENT SECRET]
exhange-resource-group-id: [GROUP ID]
exhange-resource-artifact-id: mule4-workerinfo
```

### Concourse

1. Create/update pipeline, e.g.:

```
$ fly -t anypoint sp -p mule4-workerinfo-custom-resources -c pipeline.yml -l properties.yml -l credentials.yml
```

The **publish-and-deploy** group:

![Concourse UI - mule4-workerinfo-dev](images/pipeline-3-a.png)

2. Unpause pipeline, e.g.:

```
$ fly -t anypoint up -p mule4-workerinfo-custom-resources
```

3. [ **Optional** ] - Trigger job, e.g.:

```
$ fly -t anypoint tj -j mule4-workerinfo-custom-resources/build-and-publish-asset
```

4. [ **Optional** ] - Watch job, e.g.:

```
$ fly -t anypoint watch -j mule4-workerinfo-custom-resources/build-and-publish-asset
```

5. [ **Optional** ] - Destroy pipeline, e.g.:

```
$ fly -t anypoint dp -p mule4-workerinfo-custom-resources
```