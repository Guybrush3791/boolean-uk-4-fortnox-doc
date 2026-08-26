# SpringBoot Installation

Spring Boot does not require a separate application server or a special IDE. At project level, it is a set of Java dependencies and build plugins managed by Gradle or Maven. Once those dependencies are declared, a Spring Boot application can be compiled, run and debugged like any other Java application.

The **Spring Boot CLI** is different: it is an optional command-line tool that can generate the initial project structure for us. We need it only for the `spring init` command below; we do not need it to run an existing Spring Boot project. ^[https://docs.spring.io/spring-boot/installing.html Spring Boot installation]

## Choose how to install the CLI

There are two available paths:

1. Install the Spring Boot CLI for your operating system by following the [official Spring Boot CLI installation instructions](https://docs.spring.io/spring-boot/installing.html#getting-started.installing.cli).
2. Use Devbox to provide Java 21 and the Spring Boot CLI in an isolated development shell.

If you choose Devbox, make sure it is already installed. Return to [[1 - Jetify DevBox Installation|Jetify DevBox Installation]] if you need the setup instructions.

## Create the Devbox environment

Create an empty folder for this setup and save the following file inside it as `devbox.json`:

```json file:devbox.json
{
  "$schema": "https://raw.githubusercontent.com/jetify-com/devbox/0.16.0/.schema/devbox.schema.json",
  "name": "java-devbox-test1",
  "description": "Java develpment on any os. devbox installs everything, so this works the same on Linux, macOS and Windows (WSL2).",

  "packages": [
    // Java 21 (Eclipse Temurin)
    "temurin-bin@21",
    // Spring Boot CLI
    "spring-boot-cli",
    // Clients
    "curl",
    "postman"
  ],

  "env": {
    // Points JAVA_HOME at the JDK devbox just installed
    "JAVA_HOME": "$DEVBOX_PACKAGES_DIR"
  },
  "shell": {
    "init_hook": [
      "java --version"
    ],
    "scripts": {
      "script-test": "echo 'Hello, World!'"
    }
  }
}
```

Open a terminal in the folder containing `devbox.json`, then enter the development shell:

```sh
devbox shell
```

The first launch may take a little longer while Devbox downloads Java 21 and the Spring Boot CLI. The `init_hook` prints the active Java version when the shell opens. You can also confirm that the Spring Boot CLI is available:

```sh
spring --version
```

## Generate a Spring Boot project

While still inside the Devbox shell, run:

```sh
spring init \
    --type=gradle-project \
    --language=java \
    --boot-version=4.1.1 \
    --group-id=dev.wows.buk \ # [!note] Group ID
    --artifact-id=SpringDemo1 \ # [!note] Artifact
    --name=SpringDemo1 \ # [!note] Project Name
    --package-name=dev.wows.buk.SpringDemo1 \ # [!note] Package Name
    --java-version=21 \
    --dependencies=web,devtools \ # [!note] Dependencies (where most changes will take place)
    --extract \
    SpringDemo1
```

The options describe the project that Spring Initializr will create:

| Option | Purpose |
| --- | --- |
| `--type=gradle-project` | Creates a project built with Gradle. |
| `--language=java` | Selects Java as the source language. |
| `--boot-version=4.1.1` | Selects Spring Boot 4.1.1. |
| `--group-id=dev.wows.buk` | Sets the Group ID used to identify the project. |
| `--artifact-id=SpringDemo1` | Sets the project artifact name. |
| `--name=SpringDemo1` | Sets the project name. |
| `--package-name=dev.wows.buk.SpringDemo1` | Sets the main Java package. |
| `--packaging=war` | Configures the build to produce a WAR package. |
| `--java-version=21` | Configures the project to use Java 21. |
| `--dependencies=web,devtools` | Adds Spring Web and Spring Boot DevTools. |
| `--extract` | Extracts the generated archive immediately. |
| `SpringDemo1` | Names the folder in which the project is created. |

The generated project includes the Gradle Wrapper, so Gradle does not need to be installed separately.

## Run the project

Move into the generated folder and start the application with the Gradle Wrapper:

```sh
cd SpringDemo1
./gradlew bootRun
```

Keep the terminal open while the application is running. Press `Ctrl+C` when you want to stop it.
