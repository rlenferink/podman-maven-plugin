![Java CI with Maven](https://github.com/lexemmens/podman-maven-plugin/workflows/Java%20CI%20with%20Maven/badge.svg) [![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=lexemmens_podman-maven-plugin&metric=alert_status)](https://sonarcloud.io/dashboard?id=lexemmens_podman-maven-plugin)

# Podman Maven Plugin
A maven plugin to build, tag and push OCI compliant images configured using a Dockerfile and built with Podman.

## About
This plugin was created based on the need to build container images with Podman, whilst not having an appropriate plugin to do so. Initially
using the [docker-maven-plugin](https://github.com/fabric8io/docker-maven-plugin) from fabric8io it quickly became clear that simply swapping
docking for podman (`alias docker=podman`) did not work as the plugin requires a Docker outlet to communicate with. With other plugins currently 
not yet available, the idea was born.

This plugin uses a `ProcessExecutor` from ZeroTurnaround to do its magic under water. Although this plugins' structure is similar to that from the 
`docker-maven-plugin`, its implementation is not. In an effort to keep things simple, I have chosen to use a `ProcessExecutor` to run `podman` rather
than implementing the logic myself. 

Also, only a subset of the `podman`s capabilities have been implemented (build, push and save).

## Requirements
- Maven 3.6.2 or later
- Java 9 or later
- Podman to be installed on the OS where this plugin will run
- Registry configuration to be present in Maven settings.

## Goals
| Goal                                             | Description                | Default Lifecycle Phase |
| -------------------------------------------------| -------------------------- | ----------------------- |
| `podman:build`                                   | Build images               | install                 | 
| `podman:push`                                    | Push images to a registry  | deploy                  |
| `podman:save`                                    | Save image to a file       |                         |

## Usage
The plugin is currently **NOT** available via Maven Central and can be used as follows:

1. Ensure that there is a `Dockerfile` places in the module's root directory
2. Configure the plugin in your pom file, like so: 
```XML
<plugin>
    <groupId>nl.lexemmens</groupId>
    <artifactId>podman-maven-plugin</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</plugin>
```

You can also call this plugin from the command line by invoking either of its Mojo's via `mvn podman:build` or `mvn podman:push` after running a `mvn install`.

### Configuration
With respect to configuration, it is important to outline the difference to the plugins configuration and configuration that is required by the plugin to work.

#### Mandatory configuration
This plugin requires for _all_ configured registries to have credentials present in the Maven settings file (usually located in `~/.m2/settings.xml`). Registry
configuration should look like this:
```XML
<server>
  <id>examle-registry.com</id>
  <username>username</username>
  <password>password</password>
</server>
```

The password may be encrypted. The id of the server **must** match the registry configured in the plugin (see below). The plugin will fail if
credentials are missing for any of the provided registries.

This plugin will also fail if there are no registries configured, but authentication is not skipped. Please refer to the table below for all configuration
options.

#### Configuration parameters 
The following parameters are supported by this plugin:

| Parameter                 | Command line alias             | Type    | Required | Required by                                  | Default value      | Description                                    |
| ------------------------- | ------------------------------ | ------- | -------- | -------------------------------------------- | ------------------ | ---------------------------------------------- |
| dockerFileDir             | podman.dockerfile.dir          | String  | N        | `podman:build`                               | ${project.basedir} | Source directory of the (raw) Dockerfile       |
| registries                | podman.registries              | Array   | Y        | `podman:build`, `podman:save`, `podman:push` | -                  | All registries this plugin might reach out to during execution (for building (i.e. pulling), pushing and saving) |
| targetRegistry            | podman.image.target.registry   | String  | Y        | `podman:build`, `podman:push`                | -                  | The target registry where the container image will be pushed to |
| tags                      | podman.image.tags              | Array   | Y        | `podman:build`, `podman:save`, `podman:push` | -                  | The tags of the container image, used for tagging (build), saving and pushing the image |
| useMavenProjectVersion    | podman.image.version.maven     | Boolean | N        | `podman:build`, `podman:save`, `podman:push` | true               | Specified whether the version of the Maven project should be used for tagging the container image (default: yes). When set to false, it requires `tagVersion` or `podman.image.version` to be specified. |
| tagVersion                | podman.image.version           | String  | N        | `podman:build`, `podman:save`, `podman:push` | -                  | When set, this is the version the container image |
| createLatestTag           | podman.image.tag.latest        | Boolean | N        | `podman:build`, `podman:save`, `podman:push` | false              | Specified whether an image should be tagged 'latest'. The built image will receive a normal version nonetheless |
| tlsVerify                 | podman.tls.verify              | Enum    | N        | `podman:build`, `podman:save`, `podman:push` | NOT_SPECIFIED      | Allows setting of the --tls-verify command when building, pushing or saving container images. When not specified this will fallback to default `Podman` behavior |
| skip                      | podman.skip                    | Boolean | N        | `podman:build`, `podman:save`, `podman:push` | false              | Skip all actions. |
| skipBuild                 | podman.skip.build              | Boolean | N        | `podman:build`                               | false              | Skip building container image |
| skipTag                   | podman.skip.tag                | Boolean | N        | `podman:build`                               | false              | Skip tagging container image after build |
| skipPush                  | podman.skip.push               | Boolean | N        | `podman:push`                                | false              | Will skip pushing the container image to the `targetRegistry` |
| deleteLocalImageAfterPush | podman.image.delete.after.push | Boolean | N        | `podman:push`                                | false              | Will delete the final image from the local registry. **NOTE:** All other pulled images (such as base images) will continue to exist. |
| skipSave                  | podman.skip.save               | Boolean | N        | `podman:save`                                | false              | Will skip saving the container image |
| skipAuth                  | podman.skip.auth               | Boolean | N        | `podman:build`, `podman:save`, `podman:push` | false              | Skip registry authentication check at the beginning. **NOTE:** This may cause access denied errors when building, pushing or saving container images. | 

### Using parameters in your Dockerfile
It is possible to specify properties in your pom file and use those properties in the Dockerfile, just like you would in a pom file:
```Dockerfile
FROM ${container.base.image}

WORKDIR /application
COPY ${project.artifactId}.jar ./
```

## Contributing
Feel free to open a Pull Request if you want to contribute!