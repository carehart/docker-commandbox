Official CommandBox Dockerfiles [![Build Status](https://travis-ci.org/Ortus-Solutions/docker-commandbox.svg)](https://travis-ci.org/Ortus-Solutions/docker-commandbox)
=========================

This is the repository for official Dockerfiles for Commandbox images

## How it works

The Docker files in this repository can be used to create your own custom Docker container for running ColdFusion CFML applications on CommandBox.   Leveraging CommandBox allows you to configure your entire ColdFusion CFML engine environment from a single `server.json` file in the root of your project.

Tags
======

* `:latest` ([Dockerfile](https://github.com/Ortus-Solutions/docker-commandbox/blob/master/builds/base/Dockerfile)) - Latest stable version
* `:4.0.0` - Stable image tagged with the version of CommandBox used to build the image ( `:4.0.0`, `:4.5.0`, `:4.6.0` )
* `:snapshot` - Development/BE version
* `:[tag]-snapshot` - Development/BE version of a tagged variations (e.g. - `:adobe2016-snapshot`)
* `:alpine` ([Dockerfile](https://github.com/Ortus-Solutions/docker-commandbox/blob/master/builds/base/Alpine.Dockerfile)) - [Alpine Linux](https://alpinelinux.org/about/) version of the image - slight decrease in overall size and optimizations for containerized runtimes
* `:[engine][version]` - Containers with warmed-up engines - saves having to download the server WAR during container start: `:lucee45`([Dockerfile](https://github.com/Ortus-Solutions/docker-commandbox/blob/master/builds/Lucee4.Dockerfile)), `:lucee5`([Dockerfile](https://github.com/Ortus-Solutions/docker-commandbox/blob/master/builds/Lucee5.Dockerfile)), `:adobe11`([Dockerfile](https://github.com/Ortus-Solutions/docker-commandbox/blob/master/builds/Adobe11.Dockerfile)) ,`:adobe2016`([Dockerfile](https://github.com/Ortus-Solutions/docker-commandbox/blob/master/builds/Adobe2016.Dockerfile)),`:adobe2018`([Dockerfile](https://github.com/Ortus-Solutions/docker-commandbox/blob/master/builds/Adobe2018.Dockerfile))
* `:[engine][version]-alpine` - Alpine linux versions of the image with warmed-up engines:
`:lucee45-alpine`([Dockerfile](https://github.com/Ortus-Solutions/docker-commandbox/blob/master/builds/Lucee4.Dockerfile)), `:lucee5-alpine`([Dockerfile](https://github.com/Ortus-Solutions/docker-commandbox/blob/master/builds/Lucee5.Dockerfile)), `:adobe11-alpine`([Dockerfile](https://github.com/Ortus-Solutions/docker-commandbox/blob/master/builds/Adobe11.Dockerfile)) ,`:adobe2016-alpine`([Dockerfile](https://github.com/Ortus-Solutions/docker-commandbox/blob/master/builds/Adobe2016.Dockerfile)),`:adobe2018-alpine`([Dockerfile](https://github.com/Ortus-Solutions/docker-commandbox/blob/master/builds/Adobe2016.Dockerfile))

Description 
=================

CommandBox allows you to configure your entire CFML engine environment from a single file in the root of your project.  For more information on how to leverage CommandBox in developing and deploying your applications, see the [official documentation](https://ortus.gitbooks.io/commandbox-documentation/). 

Current CFML engines supported are:

- Lucee:  4+ & 5+
- Adobe ColdFusion 11+

You may also specify a custom WAR for deployment, using the `server.json` configuration.

Usage
================

This section assumes you are using the [Official Docker Image](https://hub.docker.com/r/ortussolutions/commandbox/)

By default, the directory `/app` in the container is mapped as the Commandbox home.  To deploy a new application, first pull the image:

```
docker pull ortussolutions/commandbox
```

Then, from the root of your project, start with

```
docker run -p 8080:8080 -p 8443:8443 -v "/path/to/your/app:/app" ortussolutions/commandbox 
```

By default the process ports of the container are `8080` (insecure) and `8443` (secure - if enabled in your `server.json`) so, once the container comes online, you may access your application via browser using the applicable port (which we explicitly exposed for external access in the `run` command above).  You may also specify different port arguments in your `run` command to assign what is to be used in the container and exposed.  This prevents conflicts with other instances in the Docker machine using those ports:

```
docker run -p 8080:8080 -p 8443:8443 -e "PORT=80" -e "SSL_PORT=443" -v "/path/to/your/app:/app" ortussolutions/commandbox
```

To create your own, customized Docker image, use [our Dockerfile repository](https://github.com/Ortus-Solutions/docker-commandbox) as a reference to begin your customizations.  You can extend any of the base images and add your own additional functionality or modules.  For example, to install the [Ortus Couchbase extension for Lucee](https://www.ortussolutions.com/products/couchbase-lucee):

```
FROM ortussolutions/commandbox:lucee5

ARG COUCHBASE_EMAIL
ARG COUCHBASE_LICENSE_KEY
ARG COUCHBASE_ACTIVATION_CODE

# Copy Couchbase Extension into place for install
ADD http://lucee.ortussolutions.com/ext/couchbase-cache.lex /root/serverHome/WEB-INF/lucee-server/deploy/couchbase-cache.lex

# Seed Couchbase Extension License
ENV COUCHBASE_LICENSE_FILE /root/serverHome/WEB-INF/lucee-server/context/context/ortus/couchbase/license.properties
RUN touch "${CI_PROJECT_DIR}/build/license.properties" \
  && printf "email=${COUCHBASE_EMAIL}\n" >> ${COUCHBASE_LICENSE_FILE} \
  && printf "licenseKey=${COUCHBASE_LICENSE_KEY}\n" >> ${COUCHBASE_LICENSE_FILE} \
  && printf "activationCode=${COUCHBASE_ACTIVATION_CODE}\n" >> ${COUCHBASE_LICENSE_FILE} \
  && printf "serverType=Production\n" >> ${COUCHBASE_LICENSE_FILE}

# WARM UP THE SERVER WITH THE NEW EXTENSION
RUN ${BUILD_DIR}/util/warmup-server.sh
```

Environment Variables
=====================

The CommandBox Docker image supports the use of environmental variables for the configuration of your servers.  Specifically, the image includes the [`cfconfig` CommandBox module](https://www.forgebox.io/view/commandbox-cfconfig), which allows you to provide custom settings for your engine, including the admin password.

##### Port Variables

* `$PORT` - The port which your server should start on.  The default is `8080`.
* `$SSL_PORT` - If applicable, the ssl port used by your server The default is `8443`.


##### Server Configuration Variables

The following environment variables may be provided to modify your runtime server configuration.  Please note that environment variables are case sensitive and, while some lower/upper case aliases are accounted for, you should use consistent casing in order for these variables to take effect.

* `SERVER_HOME_DIRECTORY` - When provided, a custom path to your server home directory will be assigned.  By default, this path is set as `/root/serverHome` ( _Note: You may also provide this variable in your app's customized `server.json` file_ )
* `APP_DIR` - Application directory (web root). By default, this is `/app`.  If you are deploying an application with mappings outside of the root, you would want to provide this environment variable to point to the webroot ( e.g. `/app/wwwroot` )
* `CFENGINE` - Using the `server.json` syntax, allows you to specify the CFML engine for your container ( e.g. `lucee@5` ). Defaults to the CommandBox default ( currently `lucee@4.5`) 
* `cfconfig_[engine setting]` - Any environment variable provided which includes the `cfconfig_` prefix will be determined to be a `cfconfig` setting and the value after the prefix is presumed to be the setting name.  The command `cfconfig set ${settingName}=${value}` will be run to populate your setting in to the `$SERVER_HOME_DIRECTORY`.
* `cfconfigfile` - A `cfconfig`-compatible JSON file may be provided with this environment variable.  The file will be loaded and applied to your server.  If an `adminPassword` key exists, it will be applied as the Server and Web context passwords for Lucee engines.  _Note: The value `CFCONFIG` is aliased to this parameter, but is deprecated._
* `HEADLESS`/`headless` - When set to true, a rewrite configuration will be applied which disallows access to the Lucee Admin or Coldfusion Administrator web interfaces for a secure deployment with no administrator access.
* `BOX_INSTALL`/`box_install` - When set to true, the `box install` command will be run before the server is started to ensure any dependencies configured in your `box.json` file are installed
* `URL_REWRITES`/`url_rewrites` - A boolean value, specifying whether URL rewrites will be enabled/disabled on the server. Rewrite configurations provided within the app's `server.json` file will supersede this argument.
* `USER` - When provided the server process will run under the provided user account name
* `javaVersion` - When provided the server will [start on a custom JRE](https://commandbox.ortusbooks.com/embedded-server/configuring-your-server/custom-java-version) specified by the environment variable.

##### Docker Runtime Variables

* `$HEALTHCHECK_URI` - Specifies the URI endpoint for container [health checks](https://docs.docker.com/engine/reference/builder/#healthcheck).  By default, this defaults to `http://127.0.0.1:${PORT}/` at 20 second intervals, a timeout of 30 seconds,  with 15 retries before the container is marked as failed.  _Note: Since the interval, timeout, and retry settings cannot be set dynamically, if you need to adjust these, you will need to build from a Dockerfile which provides a new [`HEALTHCHECK` command](https://docs.docker.com/engine/reference/builder/#healthcheck)

Docker Secrets
==============

[Docker secrets](https://docs.docker.com/engine/swarm/secrets/) can use two storage mechanisms:

* Secret values stored as files on the host (non-swarm mode).
* `docker secret`-managed key/value pairs (swarm mode).

To use secrets as variables in this image, a placeholder is specified (e.g., `<<SECRET:test_docker_secret>>`) as the variable's value. At run-time, the environment variable's value is replaced with the secret.

Example with a secret using host file storage:

```yml
version: '3.1'

services:

  sut:
    environment:
      - IMAGE_TESTING_IN_PROGRESS=true
      #- ENV_SECRETS_DEBUG # uncomment to debug the placeholder replacements
      # this is a placeholder that will be replaced at runtime with the secret value
      - TEST_DOCKER_SECRET=<<SECRET:test_docker_secret>>
    ...
    
secrets:
  test_docker_secret:
    # this is the file containing the secret value
    file: ./build/tests/secrets/test_docker_secret
```

Deployment
==========

Because, with the exception of the CommandBox default engine of Lucee 5, the CFML server engines are downloaded and installed at container runtime. This can result in significant startup time increases ( even with Lucee 5 already downloaded in the base image, there is a time penalty for a "cold start" ). It is recommended that builds for production use employ a the engine-specific variations for the build, which ensures the server is downloaded, in place, and warmed up on container start.   

For a basic example, the following will suffice:

```
FROM ortussolutions/commandbox:lucee5

# Copy application files to root
COPY ./ ${APP_DIR}/
```

In many cases, you will have tier-specific builds, with custom configuration options.  The following employs a `build` directory, which includes additional configuration files for tier-based deployments:

```
FROM ortussolutions/commandbox:lucee5

ARG CI_ENVIRONMENT_NAME

# Copy application files to root
COPY ./ ${APP_DIR}/

# Copy tier-only files over
COPY ./build/env/${CI_ENVIRONMENT_NAME}/tier/ ${APP_DIR}/

# Install our box.json dependencies
RUN cd ${APP_DIR} && box install

# Warm up and validate our server
RUN ${APP_DIR}/build/env/setup-env.sh

# Remove our build directory from our deployable image
RUN rm -rf ${APP_DIR}/build

# Set our healthcheck to a non-framework route - in this case we only need to know that CFML pages are being served
ENV HEALTHCHECK_URI "http://127.0.0.1:${PORT}/config/Routes.cfm"
```

In the above case, the `setup-env.sh` file might perform an additional server warmup and validation, where in the former case, the server was previously warmed up when the image was built.

Once your customized `Dockerfile` has has been built, you can run the generated image directly, or publish it to a [private registry](https://docs.docker.com/registry/) 

About CommandBox
================

*CommandBox* is a standalone, native, [modular](https://www.forgebox.io/type/commandbox-modules) CFML development and deployment tool for Windows, Mac, and Linux which provides a CLI for server orchestration, developer productivity, tool interaction, package management, application scaffolding, and some sweet ASCII art. 
It is open for extensibility for any ColdFusion (CFML) project and is written in CFML, allowing developers to easily write their own [modules](https://www.forgebox.io/type/commandbox-modules).  It tightly integrates with the CFML open source hub [ForgeBox](https://www.forgebox.io/), so developers can share modules world-wide.

[Learn more about CommandBox](https://www.ortussolutions.com/products/commandbox)


Issues
================

Please submit issues to our repository: [https://github.com/Ortus-Solutions/docker-commandbox/issues](https://github.com/Ortus-Solutions/docker-commandbox/issues)

## LICENSE
Apache License, Version 2.0.

<hr/>

#### HONOR GOES TO GOD ABOVE ALL
Because of His grace, this project exists. If you don't like this, then don't read it, it's not for you.

>"Therefore being justified by faith, we have peace with God through our Lord Jesus Christ:
By whom also we have access by faith into this grace wherein we stand, and rejoice in hope of the glory of God.
And not only so, but we glory in tribulations also: knowing that tribulation worketh patience;
And patience, experience; and experience, hope:
And hope maketh not ashamed; because the love of God is shed abroad in our hearts by the 
Holy Ghost which is given unto us. ." Romans 5:5

#### THE DAILY BREAD
 > "I am the way, and the truth, and the life; no one comes to the Father, but by me (JESUS)" Jn 14:1-12
