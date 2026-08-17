# Jetify DevBox
Devbox is a command-line tool that lets you easily create isolated shells for development. You start by defining the list of packages required for your project, and Devbox creates an isolated, reproducible environment with those packages installed.

In practice, Devbox works similar to a package manager like yarn – except the packages it manages are at the operating-system level (the sort of thing you would normally install with brew or apt-get). ^[https://www.jetify.com/docs/devbox]

## Installation Script

Run the following install script to install the latest version of Devbox
```sh
curl -fsSL https://get.jetify.com/devbox | bash
```

> [!attention] You can use Devbox on Windows using Windows Subsystem for Linux 2 ^[https://www.jetify.com/docs/devbox/installing-devbox#installing-wsl2]
> To install WSL2 with the default Ubuntu distribution, open Powershell or Windows Command Prompt as an administrator, and run
> ```sh
> wsl --install
> ```
>
> If WSL2 is already installed, you can install Ubuntu by running
> ```sh
> wsl --install -d Ubuntu
> ```
>
> If you are running an **older version of Windows**, you may need to follow the [manual installation](https://learn.microsoft.com/en-us/windows/wsl/install-manual) steps to enable virtualization and WSL2 on your system. See the [official docs](https://learn.microsoft.com/en-us/windows/wsl/install) for more details.

## Usage
Each Devbox project starts with a `devbox.json` file. It describes the tools and versions the project needs, along with any environment variables or setup commands that should run when you enter the development shell. Devbox reads this file, installs the declared dependencies and gives everyone the same development environment, regardless of the operating system they use.

Some projects also include a `process-compose.yml` file. This describes supporting services that need to run alongside the application, such as PostgreSQL or Kafka, so they can be started together in a predictable way.

You will not be expected to create or change these files. The important idea is that they are the instructions Devbox uses to prepare the environment supplied with a project.

## Testing
To test the setup, create an empty folder and save the following file inside it as `devbox.json`. Then use your terminal to enter that folder and start the environment:

```sh
mkdir java-devbox-test1
cd java-devbox-test1
# Save devbox.json here before running the next command.
devbox shell
```

The first launch may take a little longer while Devbox downloads the packages. Once it is ready, you will be inside a shell containing the Java 21 development tools provided by `temurin-bin@21`. The `init_hook` runs `java --version` automatically, so the Java version should appear in the terminal as soon as you enter the shell.

```sh title:"Expected Output"
devbox shell
Info: Ensuring packages are installed.
✓ Computed the Devbox environment.
Starting a devbox shell...
openjdk 21.0.8 2025-07-15 LTS
OpenJDK Runtime Environment Temurin-21.0.8+9 (build 21.0.8+9-LTS)
OpenJDK 64-Bit Server VM Temurin-21.0.8+9 (build 21.0.8+9-LTS, mixed mode, sharing)
```

### `devbox.json`
As you read the configuration, focus on these sections:

- `packages` installs the Java 21 toolchain used in this course
- `env` defines environment variables; here, `JAVA_HOME` tells Java tools where to find the *JDK* installed by *Devbox*.
- `shell.init_hook` lists commands that run whenever you enter the *Devbox shell*; in this example it prints the active *Java version*
- `shell.scripts` defines named commands that can be run inside the environment; the example `script-test` just prints `Hello, World!`
```json file:devbox.json
{
  "$schema": "https://raw.githubusercontent.com/jetify-com/devbox/0.16.0/.schema/devbox.schema.json",
  "name": "java-devbox-test1",
  "description": "Java develpment on any os. devbox installs everything, so this works the same on Linux, macOS and Windows (WSL2).",

  "packages": [
    // Java 21 (Eclipse Temurin)
    "temurin-bin@21"
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

At the end of this setup, you should be able to open a project folder, run `devbox shell` and enter a ready-to-use development environment with the correct version of Java available. This gives the class a consistent starting point and reduces differences between *Linux*, *macOS* and *Windows* machines.

> [!attention] Attention
> During the academy, the required Devbox configuration will be provided in the project repository. You will normally clone the repository and use the supplied files rather than writing them yourself. You do not need to memorise the configuration syntax, but you should understand its purpose and recognise how it defines the tools, environment variables and startup behaviour for a project.