# Getting Started Building Docker Images with Gradle

This guide demonstrates how to create a Docker image with Gradle
using [gradle-docker-plugin](https://bmuschko.github.io/gradle-docker-plugin/current/user-guide/).

## What You'll Build

You'll create a docker image that prints "Hello, Gradle!" when run.

## What You'll Need

- A text editor or IDE - for example [IntelliJ IDEA](https://www.jetbrains.com/idea/download/)
- A Java Development Kit (JDK), version 8 or higher - for example [AdoptOpenJDK](https://adoptopenjdk.net/)
- An installation of [Docker](https://docs.docker.com/get-docker/)
- The latest [Gradle distribution](https://gradle.org/install)

## Steps

Follow these steps to create a new project from scratch,
or check out the [complete sample project](https://github.com/nathan-contino/gradle-docker-plugin-guide) for a working example.

### 1. Create a Project Folder

To begin, create a folder for the new project and change directory into it.

```
$ mkdir example-app
$ cd example-app
```

### 2. Generate a New Gradle Project With `gradle init`

Gradle comes with a built-in task, called `init`, that initializes a new Gradle project in an empty folder.

Run the `init` task using the following command in a terminal:

```
$ gradle init
```

When prompted, select:

1. "application" as the project type
1. "Java" as the implementation language
1. "no" to splitting functionality across multiple subprojects
1. "Kotlin" as the build script DSL
1. "no" to building with new APIs and behavior
1. "Junit 4" as the test framework
1. the default project name (input nothing and press "enter")
1. the default source package (input nothing and press "enter")

After completing the questionnaire, you should see output that resembles the
following:

```
> Task :init
Get more help with your project: https://docs.gradle.org/7.4.1/samples/sample_building_java_applications.html

BUILD SUCCESSFUL in 2m 7s
2 actionable tasks: 2 executed
```

This generates a project with the following structure:

```
├── app
│   ├── build.gradle.kts
│   └── src
│       ├── main
│       │   ├── java
│       │   │   └── example
│       │   │       └── app
│       │   │           └── App.java
│       │   └── resources
│       └── test
│           ├── java
│           │   └── demo
│           │       └── app
│           │           └── AppTest.java
│           └── resources
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
└── settings.gradle.kts
```

### 3. Update Application Code

Open `app/src/main/java/example/app/App.kt` with your favorite text editor.
In `App.getGreeting()`, change the string containing "Hello World!" to
"Hello Gradle!". For use cases that don't just print a string, this
is where you would write the entry point for your application.

### 4. Add Gradle Docker and Java Plugins

Your gradle build file should already contain the `application` plugin.
Add the Gradle Docker Plugin and the Java plugin to the `plugins` block
of your gradle build file with the following lines:

```kotlin
plugins {
    // Apply the application plugin to add support for building a CLI application in Java.
    application

    // Apply the java plugin
    java

    // Apply the gradle-docker-plugin
    id("com.bmuschko.docker-java-application") version "7.3.0"
}
```

> 💡 Tip: You can find your gradle build file at `app/build.gradle.kts`.

### 5. Configure Your Java Language Build Target

By default, Java builds target the latest version of the Java JRE.
However, Java isn't backwards compatible when targeting the latest
version of Java. Instead, configure your build to target a reasonable
target version of Java:

```kotlin
// Configure Java to target a reasonably-low JRE
java {
    toolchain {
        languageVersion.set(JavaLanguageVersion.of(11))
    }
}
```

### 6. Generate a Docker Image

To generate a docker image, configure the `docker` block in
your gradle build file. In this case, we use a Docker base
image that contains OpenJDK 16 and all of its dependencies:

```kotlin
docker {
    javaApplication {
        baseImage.set("adoptopenjdk/openjdk16")
    }
}
```

> 💡 Tip: You can find your gradle build file at `app/build.gradle.kts`.

### 7. Create the Docker File

Now, use Gradle to build a Dockerfile:

```
$ gradle dockerBuildImage
```

You should see output that resembles the following:

```
Using images 'example-app/app:latest'.
Step 1/7 : FROM adoptopenjdk/openjdk16
 ---> 7616401e09db
Step 2/7 : LABEL maintainer=example
 ---> Using cache
 ---> 5abeadb155e3
Step 3/7 : WORKDIR /app
 ---> Using cache
 ---> 500334e3a48e
Step 4/7 : COPY libs libs/
 ---> Using cache
 ---> e3d71f062b07
Step 5/7 : COPY classes classes/
 ---> Using cache
 ---> f48d4e915518
Step 6/7 : ENTRYPOINT ["java", "-cp", "/app/resources:/app/classes:/app/libs/*", "example.app.App"]
 ---> Running in 8118e3b39888
Removing intermediate container 8118e3b39888
 ---> 2c36f9ec10e0
Step 7/7 : EXPOSE 8080
 ---> Running in 11069f36cc15
Removing intermediate container 11069f36cc15
 ---> b3bbcd2e68a2
Successfully built b3bbcd2e68a2
Successfully tagged example-app/app:latest
Created image with ID 'b3bbcd2e68a2'.
```

> 🔥 Warning: Getting a Docker error? Make sure you've
> - started Docker Desktop
> - created (and logged into) a Docker account
>
> If you aren't logged in, you can't download Docker Images
> from Docker Hub.

> 💡 Tip: You can view your generated Dockerfile at `app/build/docker`.

### 8. Build the Docker Image

Now that you've created a Dockerfile, you can use it to create a
Docker Image that contains your application:

```
$ docker build -t example.app app/build/docker
```

You should see output that resembles the following:

```
[+] Building 0.2s (9/9) FINISHED
 => [internal] load build definition from Dockerfile                                                                     0.0s
 => => transferring dockerfile: 250B                                                                                     0.0s
 => [internal] load .dockerignore                                                                                        0.0s
 => => transferring context: 2B                                                                                          0.0s
 => [internal] load metadata for docker.io/adoptopenjdk/openjdk16:latest                                                 0.0s
 => [1/4] FROM docker.io/adoptopenjdk/openjdk16                                                                          0.0s
 => [internal] load build context                                                                                        0.0s
 => => transferring context: 557B                                                                                        0.0s
 => CACHED [2/4] WORKDIR /app                                                                                            0.0s
 => CACHED [3/4] COPY libs libs/                                                                                         0.0s
 => CACHED [4/4] COPY classes classes/                                                                                   0.0s
 => exporting to image                                                                                                   0.0s
 => => exporting layers                                                                                                  0.0s
 => => writing image sha256:20324e4a164f0c3298305e4c0c1b8627f4a0b7761ca3cd0ad2bcbb0c56455923                             0.0s
 => => naming to docker.io/library/example.app.last
 ```

### 9. Run the Docker Image

You have now generated a Docker Image that executes your application
in a container. Run the application through Docker with the following
command:

```
$ docker run example.app
```

You should see output that resembles the following:

```
Hello Gradle!
```

## Summary

That’s it! You’ve now successfully configured and built a Docker Image
with Gradle. You’ve learned how to:

- Add the Gradle Docker Plugin to a Gradle application
- Use the plugin to generate a Dockerfile for your application
- Use the Dockerfile to create a Docker Image
- Run the Docker Image to execute your application in a container

## Next steps

- Read the [Gradle Docker Plugin User Guide](https://bmuschko.github.io/gradle-docker-plugin/current/user-guide/) to learn how to use the plugin with other kinds of projects.
- Check out the [Docker Documentation](https://docs.docker.com/get-started/overview/) for more information about Dockerfiles, Docker Images, and containers.
- Optimize resources used by your build with [Build Scan](https://guides.gradle.org/performance/).
- Use the [Gradle Build Cache](https://guides.gradle.org/using-build-cache/)
to speed up build times.
