---
title: Build Jasig CAS from Zero (1)  - Build CAS Server
date: 2016-06-09 10:54:17
tags:
- jasig
- cas
- gradle
---



# Download source tarball from github

You can get CAS from git easily:)

```
git clone https://github.com/apereo/cas.git
git fetch
git checkout 4.2.x
```

The master branch is not a can-be-compiled version. I got failure information when I run compile commands in `master` branch:

```
$./dev-build-no-tests.sh

FAILURE: Build failed with an exception.

* Where:
Build file '/private/tmp/cas/cas-management-webapp/build.gradle' line: 28

* What went wrong:
A problem occurred evaluating project ':cas-management-webapp'.
> No such property: java for class: org.gradle.api.java.archives.internal.DefaultManifest

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output.

BUILD FAILED

Total time: 11.67 secs
```

The stable version version in [cas release page](https://github.com/apereo/cas/releases) is 4.2.2. So just checkout out the stable version.

# compile
## get gradle
CAS can be compiled by `gradle`, when you first run `gradlew`(gradle wrapper), the project will download gradle-XXX-bin.zip. If you already have it or you have some problems downloading it(such as GFW), you can simply edit `{CAS_PWD}/gradle/wrapper/gradle-wrapper.properties" like below:

```
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-2.13-all.zip

```
Replace `distributionUrl` with your own url.

If you have local gradle zip file, use the zip file name directly and put zip file into `{CAS_PWD}/gradle/wrapper`. For example if my gradle zip file is `gradle-2.10-bin.zip`, you need parameter `distributionUrl` like this:

```
...
distributionUtl=gradle-2.10-bin.zip
```
## compile cas project
There is two bootstrap shell in cas directory, `dev-build.sh` and `dev-build-no-tests.sh`. The difference between them is obviously shown in their names - `dev-build-no-tests.sh` will set this flag `-DskipAspectJ=true`

When you run `dev-build.sh` or `dev-build-no-tests.sh` for a couple of minutes, you will probably get this conflicts failure.

```
:cas-server-support-pac4j:compileAspect
Download ...
> :cas-server-support-pac4j:compileAspecFAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':cas-server-support-pac4j:compileAspect'.
> Could not resolve all dependencies for configuration ':cas-server-support-pac4j:compile'.
   > A conflict was found between the following modules:
      - commons-io:commons-io:2.4
      - commons-io:commons-io:2.5

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output.

BUILD FAILED

Total time: 10 mins 48.643 secs
```

It is casuse by the conflict between `commons-io:2.4` and `commons-io:2.5` in `cas-server-support-pac4j`, we need to add `-DskipVersionConflict=true` in bootstrap scripts.

If you build it in IntelliJ IDEA, set this parameter in `Preferences-Build,Execution,Deployment-Build Tools-Gradle-Gradle VM options`.

Plus, if you want to use http proxy in gradle, you need to set these parameters:

```
-DproxySet=true -DproxyHost=127.0.0.1 -DproxyPort=8123
```

## For IDE

We need to execute extra command for IDE

### IDEA

```
./gradlew idea
```

### Eclipse

```
./gradlew eclipse
```

# To Be Contined...w