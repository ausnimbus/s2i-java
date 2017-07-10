# AusNimbus Builder for Java [![Build Status](https://travis-ci.org/ausnimbus/s2i-java.svg?branch=master)](https://travis-ci.org/ausnimbus/s2i-java) [![Docker Repository on Quay](https://quay.io/repository/ausnimbus/s2i-java/status "Docker Repository on Quay")](https://quay.io/repository/ausnimbus/s2i-java)

[![Java](https://user-images.githubusercontent.com/2239920/27720635-f9da4d7c-5d9d-11e7-9b1d-82f21acd2944.jpg)](https://www.ausnimbus.com.au/)

The [AusNimbus](https://www.ausnimbus.com.au/) builder for Java provides a fast, secure and reliable [Java hosting](https://www.ausnimbus.com.au/languages/java-hosting/) environment.

This document describes the behaviour and environment configuration when running your Java apps on AusNimbus.

- [Runtime Environments](#runtime-environments)
- [Web Process](#web-process)
- [Dependency Management](#dependency-management)
- [Environment Configuration](#environment-configuration)
- [Extending](#extending)
  - [Build Stage (assemble)](#build-stage-assemble)
  - [Runtime Stage (run)](#runtime-stage-run)
  - [Persistent Environment Variables](#persistent-environment-variables)
- [Debug Mode](#debug-mode)

## Runtime Environments

AusNimbus supports the latest stable Java release.

Maven is used to build your application and OpenJDK is used to run it.

The most recent version of Maven will be used unless a [Maven wrapper](https://github.com/takari/maven-wrapper) is found. To use the Maven wrapper you need to include the `mvnw` file and `.mvn` directory in your repository.

The currently supported versions are `jdk8`

## Web Process

Your application's web processes must bind to port `8080`.

AusNimbus handles SSL termination at the load balancer.

## Dependency Management

You are able to pass your own parameters to `maven` during the build stage by using the following environment variable:

NAME               | Description
-------------------|-------------
MAVEN_CUSTOM_OPTS  | Parameters passed to the Maven build process. Default: `-DskipTests`
MAVEN_CUSTOM_GOALS | Parameters passed to the Maven build process. Default: `clean dependency:list install`

## Environment Configuration

The following environment variables are available for you to configure your Java environment:

NAME                | Description
--------------------|-------------
MAVEN_JAVA_OPTS     | Options passed to Java. (Default: `-Xmx$((MEMORY_LIMIT * 90 / 100))m`)

By default the builder will look for `settings.xml` in the root repository. You can use either of the following environment variables to specify alternate file locations

NAME                | Description
--------------------|------------------------
MAVEN_SETTINGS_PATH | Define the specific file path for the `settings.xml`
MAVEN_SETTINGS_URL  | Define a URL where the `settings.xml` can be downloaded from

## Extending

AusNimbus builders are split into two stages:

- Build
- Runtime

Both stages are completely extensible, allowing you to customize or completely overwrite each stage.

### Build Stage (assemble)

If you want to customize the build stage, you need to add the executable `.s2i/bin/assemble` file in your repository.

This file should contain the logic required to build and install any dependencies your application requires.

If you only want to extend the build stage, you may use this example:

```sh
#!/bin/bash

echo "Logic to include before"

# Run the default builder logic
. /usr/libexec/s2i/assemble

echo "Logic to include after"
```

### Runtime Stage (run)

If you only want to change the executed command for the run stage you may the following environment variable.

NAME        | Description
------------|-------------
APP_RUN     | Define a custom command to start your application.

**NOTE:** `APP_RUN` will overwrite any builder's runtime configuration (including the [Debug Mode](#debug-mode) section)

Alternatively you may customize or overwrite the entire runtime stage by including the executable file `.s2i/bin/run`

This file should contain the logic required to execute your application.

If you only want to extend the run stage, you may use this example:

```sh
#!/bin/bash

echo "Logic to include before"

# Run the default builder logic
. /usr/libexec/s2i/run
```

As the run script executes every time your application is deployed, scaled or restarted it's recommended to keep avoid including complex logic which may delay the start-up process of your application.

### Persistent Environment Variables

The recommend approach is to set your environment variables in the AusNimbus dashboard.

However it is possible to store environment variables in code using the `.s2i/environment` file.

The file expects a key=value format eg.

```
KEY=VALUE
FOO=BAR
```

## Debug Mode

The AusNimbus builder provides a convenient environment variable to help you debug your application.

NAME        | Description
------------|-------------
DEBUG       | Set to "TRUE" to enable Debug Mode


Web processes must bind to port `8080` and only the HTTP protocol is permitted for incoming connections.
