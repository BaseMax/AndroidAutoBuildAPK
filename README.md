# Android Auto-Build APK

This is an example of how to build an APK using GitHub Actions.
Here, we created a simple Android application that displays a Hello World!.
We then created a workflow that builds the APK and uploads it as a new release.

You maybe not want to auto-build APK for every push, so you can change the workflow to run on every push to a specific branch. For example, you can change the workflow to run on every push to the `main` branch suppose that you are working on `dev` branch in development mode.

> Another idea is to write a comment in [this issue](https://github.com/BaseMax/AndroidAutoBuildAPK/issues/1) to trigger the workflow. By this way, you can ask workflow to build APK for you any time you want.

### **Understanding GitHub Actions**
#### **Overview**

GitHub Actions is a continuous integration and continuous delivery (CI/CD) platform that allows you to automate your build, test, and deployment pipeline. You can create workflows that build and test every pull request to your repository or deploy merged pull requests to production.

GitHub Actions goes beyond just DevOps and lets you run workflows when other events happen in your repository.

For example, you can run a workflow to automatically add the appropriate labels whenever someone creates a new issue in your repository.

GitHub provides Linux, Windows, and macOS virtual machines to run your workflows, or you can host your own self-hosted runners in your own data center or cloud infrastructure.

#### Before we conitinue lets talk about CI/CD.
### **What is CI/CD?**
The main concepts attributed to CI/CD are continuous integration, continuous delivery, and continuous deployment.
#### **What is Continuous Integration?**
Continuous integration ( CI) helps to ensure that software components are working together. Integration should be completed frequently; on an hourly or daily basis, if possible.
#### **What is Continuous Delivery?**
Continuous delivery (CD) picks up where continuous integration is over. While CI is the process to build and test automatically, CD deploys all code changes to the testing or staging environment in the build.

### **The components of GitHub Actions**
#### **- Workflow**
A workflow is a configurable automated process that will run one or more jobs. Workflows are defined by a YAML file checked in to your repository and will run when triggered by an event in your repository, or they can be triggered manually, or at a defined schedule.
Workflows are defined in the .github/workflows directory in a repository, and a repository can have multiple workflows, each of which can perform a different set of tasks. For example, you can have one workflow to build and test pull requests, another workflow to deploy your application every time a release is created, and still another workflow that adds a label every time someone opens a new issue.
#### **- Events**
An event is a specific activity in a repository that triggers a workflow run. For example, activity can originate from GitHub when someone creates a pull request, opens an issue, or pushes a commit to a repository. You can also trigger a workflow run on a schedule, by posting to a REST API, or manually.
#### **- Jobs**
A job is a set of steps in a workflow that execute on the same runner. Each step is either a shell script that will be executed, or an action that will be run. Steps are executed in order and are dependent on each other. Since each step is executed on the same runner, you can share data from one step to another. For example, you can have a step that builds your application followed by a step that tests the application that was built.

You can configure a job's dependencies with other jobs; by default, jobs have no dependencies and run in parallel with each other. When a job takes a dependency on another job, it will wait for the dependent job to complete before it can run. For example, you may have multiple build jobs for different architectures that have no dependencies, and a packaging job that is dependent on those jobs. The build jobs will run in parallel, and when they have all completed successfully, the packaging job will run.
#### **- Actions**
An action is a custom application for the GitHub Actions platform that performs a complex but frequently repeated task. Use an action to help reduce the amount of repetitive code that you write in your workflow files. An action can pull your git repository from GitHub, set up the correct toolchain for your build environment, or set up the authentication to your cloud provider.
You can write your own actions, or you can find actions to use in your workflows in the GitHub Marketplace.
#### **- Runners**
A runner is a server that runs your workflows when they're triggered. Each runner can run a single job at a time. GitHub provides Ubuntu Linux, Microsoft Windows, and macOS runners to run your workflows; each workflow run executes in a fresh, newly-provisioned virtual machine. GitHub also offers larger runners, which are available in larger configurations. 

| Hello World! | 
| :---: | 
| ![](screenshots/1.png) | 

## How to use

**Build after every push:**

```yml
name: Build project after push

on:
  push:
    branches:
      - main
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: set up JDK 11
        uses: actions/setup-java@v3
        with:
          java-version: '11'
          distribution: 'temurin'
          cache: gradle

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Build with Gradle
        run: ./gradlew build

      - name: Build debug APK
        run: bash ./gradlew assembleDebug --stacktrace

      - name: Check files
        run: ls -al app/build/outputs/apk/debug

      - name: Create Release
        id: create_release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.TOKEN }}
        with:
          tag_name: ${{ github.run_number }}
          release_name: ${{ github.event.repository.name }} v${{ github.run_number }}

      - name: Upload Release APK
        id: upload_release_asset
        uses: actions/upload-release-asset@v1.0.1
        env:
          GITHUB_TOKEN: ${{ secrets.TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: app/build/outputs/apk/debug/app-debug.apk
          asset_name: ${{ github.event.repository.name }}.apk
          asset_content_type: application/vnd.android.package-archive
```

**Build after every comment in issue #1:**

```yml
name: Build project after push

on:
  issue_comment:
    types: [ created, edited ]

jobs:
  build:
    if: ${{ !github.event.issue.pull_request && github.event.issue.number == 1 }}
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: set up JDK 11
        uses: actions/setup-java@v3
        with:
          java-version: '11'
          distribution: 'temurin'
          cache: gradle

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Build with Gradle
        run: ./gradlew build

      - name: Build debug APK
        run: bash ./gradlew assembleDebug --stacktrace

      - name: Check files
        run: ls -al app/build/outputs/apk/debug

      - name: Create Release
        id: create_release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.TOKEN }}
        with:
          tag_name: ${{ github.run_number }}
          release_name: ${{ github.event.repository.name }} v${{ github.run_number }}

      - name: Upload Release APK
        id: upload_release_asset
        uses: actions/upload-release-asset@v1.0.1
        env:
          GITHUB_TOKEN: ${{ secrets.TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: app/build/outputs/apk/debug/app-debug.apk
          asset_name: ${{ github.event.repository.name }}.apk
          asset_content_type: application/vnd.android.package-archive
```

Check out the [workflow](.github/workflows/build.yml) for more details.
