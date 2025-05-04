# 🛠️ Spotless Configuration

## 📑 Table of Contents

- [🛠️ Spotless Configuration](#️-spotless-configuration)
  - [📑 Table of Contents](#-table-of-contents)
  - [❓ Why Spotless?](#-why-spotless)
  - [📋 Pre-requisites](#-pre-requisites)
  - [🧰 Gradle Wrapper Setup](#-gradle-wrapper-setup)
  - [🧾 Adding .gitattributes](#-adding-gitattributes)
  - [⚙️ Basic Plugin Setup](#️-basic-plugin-setup)
    - [🦴 Plugin Skeleton](#-plugin-skeleton)
      - [📄 Non-Code Files (Still Important!)](#-non-code-files-still-important)
      - [🎨 Prettier (JSON, HTML, YAML, XML) Configuration](#-prettier-json-html-yaml-xml-configuration)
      - [☕️ Java Configuration](#️-java-configuration)
      - [🧾 Groovy Gradle Configuration (\*.gradle files)](#-groovy-gradle-configuration-gradle-files)
      - [✍️ Markdown Configuration](#️-markdown-configuration)
      - [🛢️ SQL (Surprise its prettier!) Configuration](#️-sql-surprise-its-prettier-configuration)
      - [🐚 Shell Script Configuration](#-shell-script-configuration)
  - [📚 Plugin Documentation](#-plugin-documentation)

## ❓ Why Spotless?

There really isn’t a strong alternative.

Spotless is one of the very few formatting tools that works seamlessly with both Maven and Gradle, and can be integrated into Java projects without requiring new tooling ecosystems or language server configuration. It handles multi-format linting, supports widely accepted formatting styles (like Palantir Java Format), and is extremely customizable—yet doesn't get in your way.

If you’re coming from a frontend or Python world, tools like `prettier`, `black`, or `eslint` offer tight `pre-commit` integration out of the box. In the `Java` ecosystem, that level of integration is oddly lacking. `Spotless` fills that gap with the added benefit of being language-agnostic across file types.

## 📋 Pre-requisites

- Your project must be on `Java 11`
- Your project must have the `Gradle Wrapper` configured

## 🧰 Gradle Wrapper Setup

To add Gradle wrapper to your project, run the following command: `gradle wrapper`

## 🧾 Adding .gitattributes

If your project doesn't have a `.gitattributes` file, create one in the root of your project and add the following lines:

<!-- markdownlint-disable-next-line MD033 -->
<details><summary>View the <code>.gitattributes</code> file</summary>

```gitattributes
/gradlew text eol=lf
*.bat text eol=crlf
# Add other files here before the * text=auto
# *.png binary
# * text=auto should be the last line in the file
* text=auto
```

</details>

> This is **NOT** optional. Failure to do this, and messing up the line endings for `*.bat` or the `gradlew` script files will cause issues with users on Windows and Mac. Mac cannot run `gradlew` if the line endings are `crlf`, and Windows cannot run `*.bat` if the line endings are not `crlf`.

## ⚙️ Basic Plugin Setup

Below is a full-fat `spotless` configuration, configuring many file types, including `Java`, `XML`, `JSON`, `YAML`, `HTML`, `Markdown`, and `SQL`.

This is a recommended `final` configuration for most projects. However, you can, and should, remove the sections that you do not need.

It is recommended to add everything **BUT** the `java` section to start with. Get that working in your local development workflow and your CI/CD. Merge those changes to your main branch. Then, add the `java` section to the `spotless` configuration. This will allow you to get the formatting on the `Java` files without having to setup the overall configuration and process in one go, reducing the burden on code reviewers and limiting the blast radius of merge conflicts on your most important files; `Java` files.

### 🦴 Plugin Skeleton

Below is the overall skeleton of the `pom.xml` file. This is broken out here to show the overall high-level structure of the `spotless` configuration. The actual configurations for specific programming languages and file types are below in separate sections that are intended to be copied and pasted into the `spotless` plugin's `configuration` section, replacing the `...` in the `configuration` section.

<!-- markdownlint-disable-next-line MD033 -->
<details><summary>View <code>pom.xml</code> skeleton</summary>

```groovy
plugins {
  id("com.diffplug.spotless") version "7.0.3"
}

java {
  toolchain {
    // Must be at least Java 11
    languageVersion = JavaLanguageVersion.of(11)
  }
}

build {
  dependsOn spotlessCheck
}

spotless {
  String ref = project.properties["ratchetFrom"]
  if (ref != null) {
    ref = ref.trim()
    if (ref.length() > 0) {
      ratchetFrom ref
    }
  }
  // We will add the formatting configurations below this comment
}
```

</details>

An important callout here is that the `build` task has its `dependsOn` block configured as follows:

```xml
build {
  dependsOn spotlessCheck
}
```

This has the side effect of making the CI/CD run the `spotless` check, **NOT** the `apply` goal. This checks that code was formatted with `spotless` in the CI/CD, but does not apply formatting, and instead, will fail the Gradle build if the code is not formatted correctly. This is a good practice to get into, as it will help you catch formatting issues before they hit your main branch, and identify developers not configuring their local development environment correctly.

#### 📄 Non-Code Files (Still Important!)

This is important because it not only enforces some minor trimming of whitespace and newlines, but also ensures that the `gradlew` and `gradlew.cmd` files are properly formatted for the OS you are on. `spotless` will format all files according to the line endings defined in the `.gitattributes` file. This means, we need to keep `gradlew` and `gradlew.cmd` files in separate blocks, since the first found line ending is used for all files in the `includes` block.

<!-- markdownlint-disable-next-line MD033 -->
<details><summary>View configuration</summary>

```groovy
format 'misc', {
  target '*.gradle', '.gitattributes', '.gitignore', 'prettierc', 'gradlew'
  trimTrailingWhitespace()
  endWithNewline()
}
format 'crlf-misc', {
  target 'gradlew.bat'
  trimTrailingWhitespace()
  endWithNewline()
}
```

</details>

#### 🎨 Prettier (JSON, HTML, YAML, XML) Configuration

<!-- markdownlint-disable-next-line MD033 -->
<details><summary>View configuration</summary>

```groovy
format 'styling', {
  target '.vscode/**/*.json', 'src/**/*.json', 'src/**/*.yaml', 'src/**/*.yml', 'compose.yml', '.mvn/**/*.xml',
      'src/**/*.xml'
  prettier(['prettier': '3.3.2', '@prettier/plugin-xml': '0.10.0'])
  .config(['printWidth': 120, 'plugins': ['@prettier/plugin-xml']]).npmInstallCache()
  trimTrailingWhitespace()
  endWithNewline()
}
```

</details>

#### ☕️ Java Configuration

<!-- markdownlint-disable-next-line MD033 -->
<details><summary>View configuration</summary>

```groovy
java {
  importOrder()
  removeUnusedImports()
  cleanthat().version('2.20').sourceCompatibility('21').addMutator('SafeAndConsensual').addMutator(
      'SafeButNotConsensual')
  palantirJavaFormat('2.47.0').style("PALANTIR").formatJavadoc(true)
  formatAnnotations()
  trimTrailingWhitespace()
  endWithNewline()
}
```

</details>

#### 🧾 Groovy Gradle Configuration (\*.gradle files)

<!-- markdownlint-disable-next-line MD033 -->
<details><summary>View configuration</summary>

```groovy
groovyGradle {
  greclipse()
  trimTrailingWhitespace()
  endWithNewline()
}
```

</details>

#### ✍️ Markdown Configuration

<!-- markdownlint-disable-next-line MD033 -->
<details><summary>View configuration</summary>

```groovy
flexmark {
  target '**/*.md'
  flexmark()
  trimTrailingWhitespace()
  endWithNewline()
}
```

</details>

#### 🛢️ SQL (Surprise its prettier!) Configuration

<!-- markdownlint-disable-next-line MD033 -->
<details><summary>View configuration</summary>

```groovy
format 'styling-sql', {
  target 'src/**/*.sql'
  prettier(['prettier': '^3', 'prettier-plugin-sql': '~0.18'])
  .config(['printWidth': 120, 'plugins': ['prettier-plugin-sql']]).npmInstallCache()
  trimTrailingWhitespace()
  endWithNewline()
}
```

#### 🐚 Shell Script Configuration

<!-- markdownlint-disable-next-line MD033 -->
<details><summary>View configuration</summary>

```groovy
shell {
  target 'scripts/**/*.sh'
  shfmt()
  trimTrailingWhitespace()
  endWithNewline()
}
```

</details>

## 📚 Plugin Documentation

To find out more information please refer to the [Spotless Gradle Plugin Documentation](https://github.com/diffplug/spotless/blob/main/plugin-gradle/README.md). This will give you more information about the configuration options available to you. The configuration options laid out above are a full-fat recommended configuration. All the sections might not apply to you, like the `sql` section. It is also strongly advised, if you are adding `spotless` to an existing project, to remove the `java` portion from the `spotless` configuration for a phase 1 migration. This way, you can start enforcing the `pre-commit` process and get formatting on some non-critical, non-java files. Once you are happy with the configuration, you can then add the `java` portion to the `spotless` configuration. This will allow you to get the formatting on the Java files without having to set up the overall configuration and process in one go.

← Back to [README.md](./README.md)
