# Java SpringBoot GitHub Action Example

## Learning objectives

- Generate a Spring Boot 4.1.1 project with Java 21
- Prove one small health-check behaviour before automating it
- Run the project's tests and packaging task in GitHub Actions
- Distinguish a reported candidate version from a published release or deployment

## Prepare the smallest useful Devbox

The project generator needs a Java runtime and the Spring Boot CLI. The generated project includes the Gradle Wrapper, so Devbox does not need Gradle. This example also needs no database, migration tool, API client, or OpenAPI tool.

Start in an empty working folder and save this file as `devbox.json`:

```json file:devbox.json
{
  "$schema": "https://raw.githubusercontent.com/jetify-com/devbox/0.16.0/.schema/devbox.schema.json",
  "packages": [
    "temurin-bin@21",
    "spring-boot-cli"
  ]
}
```

Enter the environment:

```sh file:"Enter the Devbox shell"
devbox shell
```

`temurin-bin@21` supplies the Java 21 runtime and compiler. `spring-boot-cli` supplies the `spring init` command. No services or custom environment variables are required.

## Generate the exact project

From the Devbox shell, run:

```sh file:"Create the project"
spring init \
  --type=gradle-project \
  --language=java \
  --boot-version=4.1.1 \
  --group-id=dev.wows.buk \
  --artifact-id=JavaCiCd \
  --name=JavaCiCd \
  --package-name=dev.wows.buk.JavaCiCd \
  --java-version=21 \
  --dependencies=web,devtools \
  --extract \
  JavaCiCd
```

The command creates `JavaCiCd` and extracts the generated project into it. Move `devbox.json` and its generated `devbox.lock` into `JavaCiCd` so the development environment and `build.gradle` live at the project root. Then enter the project and verify its starting point:

```sh file:"Verify the generated project"
cd JavaCiCd
./gradlew test
```

The `web` dependency supplies Spring MVC and its test support. `devtools` supports local development. The health check does not use PostgreSQL, JPA, Lombok, Flyway, Postman, or Redocly, so none of those belong in this starting environment.

## Add one observable behaviour

Create a feature branch from an updated `main` branch:

```sh file:"Create the feature branch"
git checkout -b feature/health-check
```

Add `src/main/java/dev/wows/buk/JavaCiCd/health/HealthController.java`:

```java file:HealthController.java group:health-check
package dev.wows.buk.JavaCiCd.health;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/health")
public class HealthController {

    @GetMapping
    public String health() {
        return "OK";
    }
}
```

`@RequestMapping` supplies the controller's base path. `@GetMapping` maps an HTTP GET request at that path to `health()`.

Add `src/test/java/dev/wows/buk/JavaCiCd/health/HealthControllerTest.java`:

```java file:HealthControllerTest.java group:health-check
package dev.wows.buk.JavaCiCd.health;

import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.Test;
import org.springframework.boot.webmvc.test.autoconfigure.WebMvcTest;

@WebMvcTest(HealthController.class)
class HealthControllerTest {

    private HealthController healthController;

    public HealthControllerTest() {
        healthController = new HealthController();
    }

    @Test
    void healthEndpointReturnsOK() throws Exception {
        assertEquals("OK", healthController.health());
    }
}
```

This test calls the controller method directly and proves that it returns `"OK"`. It does not send an HTTP request or prove the `/api/health` route. Keep that boundary clear when reading the test result.

Run the test before adding automation:

```sh file:"Run the health-check test"
./gradlew test
```

## Automate the same commands

Create `.github/workflows/pr-validation.yaml`:

```yaml file:pr-validation.yaml
name: Validate Java application

on:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]

jobs:
  test-and-package:
    name: Run tests and build executable JAR
    runs-on: ubuntu-latest

    steps:
      - name: Check out repository source
        uses: actions/checkout@v4

      - name: Install Temurin JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'

      - name: Run automated tests with Gradle
        run: ./gradlew test

      - name: Build the executable Spring Boot JAR
        run: ./gradlew bootJar

      - name: Report candidate version for main
        if: github.event_name == 'push' && github.ref == 'refs/heads/main'
        run: |
          VERSION="1.0.${{ github.run_number }}"
          echo "Candidate version: $VERSION"
          echo "Tests passed and the executable JAR was built."
```

The labels have different jobs:

| YAML value | Where it appears | Meaning |
| --- | --- | --- |
| `Validate Java application` | Actions workflow list | The purpose of the whole workflow |
| `test-and-package` | YAML job identifier | A stable machine-readable job key |
| `Run tests and build executable JAR` | Workflow run | What the job proves |
| Step names | Job log | The exact operation currently running |

The first four steps run for both pull requests and pushes to `main`. The final step runs only for a push to `main`. `${{ github.run_number }}` is GitHub's increasing number for runs of this workflow.

> [!warning] No deployment yet
> The runner builds a JAR and prints a candidate version, but the workflow does not upload the JAR, create a GitHub Release, publish a package, or deploy to an environment. Those would require additional explicit steps.

## Push, review, and merge

Commit the feature and workflow, then push the feature branch:

```sh file:"Push the feature branch"
git add -A
git commit -m "Add health check and Java validation workflow"
git push -u origin feature/health-check
```

Open a pull request targeting `main`. The pull request event should run the test and package steps, while the candidate-version step remains skipped. After review, merge the pull request. The resulting push to `main` runs the complete workflow, including the candidate-version report.

![[GitHub Automation CI-CD push & deploy|1000]]

## Decision boundary

Use this workflow when the immediate requirement is to reject code that does not test or package successfully. Add delivery only when the team has chosen where the built artefact will be stored, and add deployment only when a target environment and release policy are defined.

---

# Links
![[Lessons/2 - Java Back-end/Day 17/__blocks/Links]]
