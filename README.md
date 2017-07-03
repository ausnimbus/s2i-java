# AusNimbus Builder for Java [![Build Status](https://travis-ci.org/ausnimbus/s2i-java.svg?branch=master)](https://travis-ci.org/ausnimbus/s2i-java) [![Docker Repository on Quay](https://quay.io/repository/ausnimbus/s2i-java/status "Docker Repository on Quay")](https://quay.io/repository/ausnimbus/s2i-java)

[![Java](https://user-images.githubusercontent.com/2239920/27720635-f9da4d7c-5d9d-11e7-9b1d-82f21acd2944.jpg)](https://www.ausnimbus.com.au/)

The [AusNimbus](https://www.ausnimbus.com.au/) builder for Java provides a fast, secure and reliable [Java hosting](https://www.ausnimbus.com.au/languages/java-hosting/) environment.

It uses Maven to build your application and runs it with OpenJDK.

The default version of Maven will be used unless the [Maven wrapper](https://github.com/takari/maven-wrapper) is found. To use the Maven wrapper you need to include the `mvnw` file and `.mvn` directory in your repository.

Web processes must bind to port `8080` and only the HTTP protocol is permitted for incoming connections.

# Environment Variables

* **MAVEN_CUSTOM_GOALS**

  * Parameter passed to the Maven build process
  * Default: `clean dependency:list install`

* **MAVEN_CUSTOM_OPTS**

  * Parameter passed to the Maven build process
  * Default: `-DskipTests`

* **MAVEN_JAVA_OPTS**

  * Default: `-Xmx$((MEMORY_LIMIT * 90 / 100))m`

* **MAVEN_SETTINGS_PATH, MAVEN_SETTINGS_URL**

  * By default the builder will look for `settings.xml` in the root repository.
  * You can define a specific file instead by using `MAVEN_SETTINGS_PATH`
  * You can alternatively define a URL by using `MAVEN_SETTINGS_URL`

# Versions

- jdk8
