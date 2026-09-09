# GitHub actions CI/CD

## Repository
https://github.com/Guybrush3791/boolean-uk-1-fortnox-github-actions-ci-cd.git

## Learning Objectives

- CI/CD through Github Actions

## Instructions

1. Create a new repository in your account with given name `boolean-uk-1-fortnox-github-actions-ci-cd`
2. Create a new project and open it in *VSCode*

## Activity
### Core
Create a Java SpringBoot application with automated CI/CD pipeline using *GitHub Actions*, following the same pattern demonstrated in the lesson materials.

#### Feature
The target feature in the new project is `status-endpoint`:

**GET** `/api/status` --> "Service is Active"

#### Test
Make test in advance, the provide actual implementation code

#### Build & Deploy through CI/CD
Create a dedicate `yaml` workflow file and include *test execution* during *push* and *pr* on *main* branch

### Extension
Create a very simple `Entity` into the project and develop some CRUD functionalities 

#### Feature
The goal is been able to test database feature into *GitHub Actions* CI/CD context through any version of database is available in the *GitHub Actions* context.
