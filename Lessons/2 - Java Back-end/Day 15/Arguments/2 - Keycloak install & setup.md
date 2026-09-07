# Keycloak install & setup

## Learning Objectives

- Run Keycloak inside the existing Devbox environment.
- Create the lesson realm, client and user through the Keycloak dashboard.
- Keep Keycloak configuration available across service restarts.

## Prepare the Devbox environment

This lesson uses a project-local Devbox environment to provide Java 21, the Spring Boot CLI and Keycloak. Keycloak is a service of this environment, not a separate application that learners must install globally.

Use this complete `devbox.json` in the folder that will contain `JavaOidc`:

```jsonc file:devbox.json
{
  "$schema": "https://raw.githubusercontent.com/jetify-com/devbox/0.16.0/.schema/devbox.schema.json",
  "name": "java-devbox-test1",
  "description": "Java development on any OS. Devbox installs everything, so this works the same on Linux, macOS and Windows (WSL2).",

  "packages": [
    "temurin-bin@21",
    "curl",
    "postman",
    "redocly",
    "postgresql@17",
    "dbeaver-bin",
    "flyway",
    "unzip",
    "spring-boot-cli@latest"
  ],

  "env": {
    "JAVA_HOME": "$DEVBOX_PACKAGES_DIR",
    "PGUSER": "postgres",
    "PGPORT": "5432",
    "PGDATABASE": "booleandb",
    "KEYCLOAK_VERSION": "26.3.0",
    "KC_HTTP_PORT": "8080",
    "KC_BOOTSTRAP_ADMIN_USERNAME": "admin",
    "KC_BOOTSTRAP_ADMIN_PASSWORD": "admin"
  },

  "shell": {
    "init_hook": [
      "test -f \"$PGDATA/PG_VERSION\" || initdb --username=$PGUSER --auth=trust --encoding=UTF8 --locale=C",
      "test -x \"$DEVBOX_PROJECT_ROOT/.keycloak/keycloak-$KEYCLOAK_VERSION/bin/kc.sh\" || { echo \"Downloading Keycloak $KEYCLOAK_VERSION (one-off, ~130 MB)...\"; mkdir -p \"$DEVBOX_PROJECT_ROOT/.keycloak\"; curl -fL --progress-bar -o \"$DEVBOX_PROJECT_ROOT/.keycloak/keycloak.zip\" \"https://github.com/keycloak/keycloak/releases/download/$KEYCLOAK_VERSION/keycloak-$KEYCLOAK_VERSION.zip\" && unzip -q \"$DEVBOX_PROJECT_ROOT/.keycloak/keycloak.zip\" -d \"$DEVBOX_PROJECT_ROOT/.keycloak\" && rm -f \"$DEVBOX_PROJECT_ROOT/.keycloak/keycloak.zip\" || echo 'Keycloak download failed. Check KEYCLOAK_VERSION in devbox.json.'; }",
      "java --version"
    ],
    "scripts": {
      "script-test": "echo 'Hello, World!'",
      "db-shell": "psql",
      "db-reset": "test -n \"$PGDATA\" && rm -rf \"$PGDATA\" && echo 'Database deleted. Run: devbox services up'",
      "kc-reset": "rm -rf \"$DEVBOX_PROJECT_ROOT/.keycloak/keycloak-$KEYCLOAK_VERSION/data\" && echo 'Keycloak realms/users deleted. Run: devbox services restart keycloak'",
      "kc-purge": "rm -rf \"$DEVBOX_PROJECT_ROOT/.keycloak\" && echo 'Keycloak removed. Run: devbox shell'"
    }
  }
}
```

The `init_hook` downloads the Keycloak distribution into `.keycloak` the first time it is needed. The downloaded distribution and the realm data stay local to this lesson folder.

Create `process-compose.yaml` beside `devbox.json`:

```yaml file:process-compose.yaml
version: "0.5"

processes:
  createdb:
    command: "createdb $PGDATABASE || true"
    depends_on:
      postgresql:
        condition: process_healthy

  keycloak:
    command: 'exec "${DEVBOX_PROJECT_ROOT:-.}/.keycloak/keycloak-$KEYCLOAK_VERSION/bin/kc.sh" start-dev --http-port=$KC_HTTP_PORT'
    availability:
      restart: on_failure
      max_restarts: 3
    readiness_probe:
      http_get:
        scheme: http
        host: 127.0.0.1
        port: 8080
        path: /realms/master
      initial_delay_seconds: 20
      period_seconds: 5
      failure_threshold: 40
```

In one terminal, enter the environment so Devbox can install the tools and download Keycloak:

```sh file:"Prepare the Devbox environment"
devbox shell
```

Keep that shell available for the project work. In a second terminal, start Keycloak from the same folder and keep the service process running:

```sh file:"Start Keycloak"
devbox services up keycloak
```

The development service listens on `http://localhost:8080`. Its data directory is under `.keycloak`, so stopping and restarting the service does not remove the realm, client or users.

## Understand the dashboard setup

![[Keycloak and SSO introduction|1600]]

In the diagram, Postman is the `ClientApp`. It sends the browser to Keycloak, Keycloak checks the user's credentials, and Postman receives the token. The `JavaOidc` API only validates that token when Postman requests a protected resource.

## Configure Keycloak in the dashboard

Open [http://localhost:8080](http://localhost:8080). Sign in to the Keycloak administration dashboard with username `admin` and password `admin`.

![[keycloak-login-page.png]]

### Realm concept
In Keycloak, a **realm** is an isolated security domain containing its own users, roles, authentication flows and client applications. Everything created below belongs to the lesson realm rather than the built-in `master` realm.

### Create a dedicated realm for the Spring Boot application
From the left menu, select **Manage realms**, then **Create realm**, and define the realm name as `springboot-realm-1`.

![[Keycloak create realm|1200]]

### Create a client
First, confirm that the correct realm is selected.

![[Keycloack check proper realm|1200]]

Open **Clients**, select **Create client**, and enter the following settings.

#### General settings

| Key | Value |
| --- | --- |
| Client type | OpenID Connect |
| Client ID | `springboot-realm-1` |

Continue to **Capability config** and use:

| Key | Value |
| --- | --- |
| Client authentication | Off |
| Authorization | Off |
| Standard flow | On |
| PKCE method | S256 |

This creates a public client that can use the authorisation code flow with Proof Key for Code Exchange (PKCE) and without a client secret.

#### Login settings

| Key | Value | Note |
| --- | --- | --- |
| Valid redirect URIs | https://oauth.pstmn.io/v1/callback | This is the Postman callback. |

Leave **Root URL**, **Home URL**, **Web origins** and **Admin URL** empty. `JavaOidc` is a resource server, not an OAuth2 login client, so it does not receive a login callback.

### Create a user
Confirm once more that the correct realm is selected.

![[Keycloack check proper realm|1200]]

To access services, create a user who can perform the login. Open **Users**, select **Create new user**, provide suitable values, and create the user.

![[keycloack-create-user.png|1200]]

After creating the user, configure a password through **Users**, select the username, open **Credentials**, then choose **Set password**.

![[Keycloak user password create|1200]]

Set **Temporary** to **Off** if the password should work without a forced password-change screen during the Postman login.

Use the dashboard for realm, client and user setup. Use Devbox only to install, start, stop or reset the local Keycloak service.

---

# Links
![[Lessons/2 - Java Back-end/Day 15/__blocks/Links]]
