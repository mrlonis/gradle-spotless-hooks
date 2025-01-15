# gradle-spotless-hooks

## Table of Contents

- [gradle-spotless-hooks](#gradle-spotless-hooks)
  - [Table of Contents](#table-of-contents)
  - [Usage](#usage)
    - [Automatically Update Submodule With Gradle](#automatically-update-submodule-with-gradle)
      - [Executed Command](#executed-command)
      - [Excluding submodule updates during CI](#excluding-submodule-updates-during-ci)
        - [GitHub Actions](#github-actions)
    - [Install Git Hooks](#install-git-hooks)
  - [Troubleshooting](#troubleshooting)
    - [How to fix "git-sh-setup: file not found" in windows](#how-to-fix-git-sh-setup-file-not-found-in-windows)

## Usage

This repo should be added to another repo as a submodule

```sh
git submodule add -b main https://github.com/mrlonis/gradle-spotless-hooks.git .hooks/
git commit -m "Adding gradle-spotless-hooks"
```

This will add the `gradle-spotless-hooks` repository as a submodule in the `.hooks` folder within your project.

### Automatically Update Submodule With Gradle

Submodules are not cloned by default so we need to add a plugin to our Gradle root `build.gradle` to clone the submodule. The following is the recommended configuration:

```groovy
task updateSubmodule {
    doLast {
        exec {
            commandLine 'git', 'submodule', 'update', '--init', '--remote', '--force'
        }
    }
}

build {
    dependsOn updateSubmodule
}
```

#### Executed Command

The resulting command that is executed is `git submodule update --init --remote --force` which will clone the submodule if it does not exist, update the submodule to the latest commit, throw away local changes in submodules when switching to a different commit; and always run a checkout operation in the submodule, even if the commit listed in the index of the containing repository matches the commit checked out in the submodule.

#### Excluding submodule updates during CI

If you are using a CI/CD pipeline, you may want to exclude the submodule update during the CI/CD pipeline. This can be done by adding the following configuration to the `build.gradle`:

```groovy
task updateSubmodule {
    doLast {
        exec {
            commandLine 'git', 'submodule', 'update', '--init', '--remote', '--force'
        }
    }
    onlyIf {
        System.env['SOME_ENV_VAR'] != null
    }
}

build {
    dependsOn updateSubmodule
}
```

This works by checking for the absence of an environment variable `SOME_ENV_VAR` and if it is not present, the submodule update will be executed. This can be used to exclude the submodule update during the CI/CD pipeline.

##### GitHub Actions

If you are using GitHub Actions, you can exclude the submodule update by adding the following `env` configuration to the `.github/workflows/*.yml` file:

```yaml
---
env:
  SOME_ENV_VAR: this_can_be_anything_since_we_are_checking_for_its_absence_not_its_value
```

### Install Git Hooks

We then need to install the git hooks. This can be done by adding the following configuration to the `build.gradle`:

```groovy
task installGitHooks(type: Copy) {
    from new File(rootProject.rootDir, '.hooks/pre-commit')
    into { new File(rootProject.rootDir, '.git/hooks') }

    from new File(rootProject.rootDir, '.hooks/pre-commit')
    into { new File(rootProject.rootDir, '.git/hooks') }
}

build {
    dependsOn installGitHooks
}
```

## Troubleshooting

### How to fix "git-sh-setup: file not found" in windows

[https://stackoverflow.com/questions/49256190/how-to-fix-git-sh-setup-file-not-found-in-windows](https://stackoverflow.com/questions/49256190/how-to-fix-git-sh-setup-file-not-found-in-windows)

1. In the Windows Search bar, type `Environment Variables` and select `Edit the system environment variables`
2. In the `System Properties` window, click on the `Environment Variables` button
3. In the `Environment Variables` window, under `System variables`, click on `Path` and then click on `Edit`
4. In the `Edit Environment Variable` window, click on `New` and add the following paths:
   - `C:\Program Files\Git\usr\bin`
   - `C:\Program Files\Git\mingw64\libexec\git-core`
5. These will be added to the end of the list. Click on each one, and then click on `Move Up` until they are at the top of the list
