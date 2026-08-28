# Redocly CLI

Day 7 introduced the shared `devbox.json` used for Java and Spring Boot development. Keep that configuration and add the Redocly CLI required to work with OpenAPI specifications:

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
    "postman",
    // OpenAPI documentation
    "redocly-cli"
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

The only new package is `redocly-cli`. It provides the `redocly` command used to validate OpenAPI descriptions and generate browsable API documentation from an OpenAPI YAML file. All other entries are unchanged from Day 7.

Enter the Devbox shell and verify that the command is available:

```sh
devbox shell
redocly --version
```
