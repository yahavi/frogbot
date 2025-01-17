[Go back to the main documentation page](https://github.com/jfrog/frogbot)

# Creating the frogbot-config.yml file

## What is the frogbot-config.yml file?
The [frogbot-config.yml](templates/.frogbot/frogbot-config.yml) file includes configuration related to your projects, to help Frogbot scan your Git repositories.

## Is the frogbot-config.yml file mandatory?
Not all projects require the **frogbot-config.yml** file, but any project can use it.
For projects with a single descriptor file (package.json, pom.xml, etc.), which is located 
at the root directory of the project, the **frogbot-config.yml** file isn't mandatory.
In other words, if the following conditions apply to your project, you don't have to create the file. 

1. The project has only one descriptor file (pom.xml, package.json, go.mod, etc.) 
2. The descriptor file is at the root directory of the project 

If your project doesn't use a **frogbot-config.yml** file, all of the configuration Frogbot requires  
should be provided as variables as part of the Frogbot workflows.

## How does the frogbot-config.yml file helps Frogbot scan the repository?
When your Git repository includes multiple subprojects, and each subproject has its own descriptor file (package.json in the case of npm), the **frogbot-config.yml** file should 
include the relative paths to the subprojects. Frogbot uses this configuration to scan each subproject separately. 
In the following example, there are two subprojects under `path/to/project-1` and `path/to/project-2`.
The dependencies of each subproject are downloaded using the `npm i` command.`  
```yaml
- params:
    git:
      repoName: my-git-repo-name
      branches:
        - master
    scan:
      projects:
        - installCommand: npm i
          workingDirs:
            - path/to/npm/project-1
            - path/to/npm/project-2
```

Here's another example for a repository that uses the `npm i` command to download the dependencies of one subproject, and `nuget restore` to download the dependencies of the second.
```yaml
- params:
    git:
      repoName: my-git-repo-name
      branches:
        - master
    scan:
      projects:
        - installCommand: npm i
          workingDirs:
            - path/to/node/project
        - installCommand: nuget restore
          workingDirs:
            - path/to/.net/project
```

The next example is for a repository that uses both `npm` and `mvn` to download the dependencies.
Notice that for Maven projects, there's no need to set the `installCommand` property.
```yaml
- params:
    git:
      repoName: my-git-repo-name
      branches:
        - master
    scan:
      projects:
        - installCommand: npm i
          workingDirs:
            - path/to/node/project
        - workingDirs:
            - path/to/maven/project
```

See the full **frogbot-config.yml** structure [here](templates/.frogbot/frogbot-config.yml).

## Can one frogbot-config.yml file be used for multiple Git repositories?
You have the option of using a single **frogbot-config.yml** file for scanning multiple Git repositories in the same organization, if one of the following platforms are used.
- GitHub with Jenkins or JFrog Pipelines
- Bitbucket Server
- Azure Repos

The file can be placed in any repository, if it's in the same organization as all the repositories referenced in the file. 
Here's an example for a **frogbot-config.yml** referencing multiple repositories.
```yaml
- params:
    git:
      repoName: repo-1
      branches:
        - master
    scan:
      projects:
        - installCommand: npm i
- params:
    git:
      repoName: repo-2
      branches:
        - master
        - dev
- params:
    git:
      repoName: repo-3
      branches:
        - master
    scan:
      projects:
        - pipRequirementsFile: requirements.txt
```

If however you're using one of the following platforms, each repository that needs to be scanned by Frogbot should include its own **frogbot-config.yml** file.
- GitHub with GitHub actions
- GitLab

## Where should the frogbot-config.yml file be placed in the repository?
Frogbot expects the frogbot-config.yml file to be in the following path from the root of the Git repository: `.frogbot/frogbot-config.yml` .

## The frogbot-config.yml file structure

The [frogbot-config.yml](templates/.frogbot/frogbot-config.yml) file has the following structure.

### Params

This section represents a single Git repository. It includes the **git**, **jfrogPlatform** and **scan** sections.

#### git

This section includes the git repository related parameters.

- **repoName** - [Mandatory] The name of the Git repository to scan.
- **branches** - [Mandatory] The branches to scan

#### scan

This section includes the scanning options for Frogbot.

- **includeAllVulnerabilities** - [Default: false] Frogbot displays all the existing vulnerabilities, including the ones that were added by the pull request and the ones that are inside the target branch already.

- **failOnSecurityIssues** - [Default: true] Frogbot fails the task if any security issue is found.
- **projects** - List of sub-projects / project dirs.
  - **workingDirs** - [Default: root directory] A list of relative path's inside the Git repository. Each path should point to the root of a sub-project to be scannedby Frogbot.
  - **installCommand** - [Mandatory for projects which use npm, yarn 2, NuGet and .NET to download their dependencies] The command to download the projectdependencies. For example: 'npm install', 'nuget restore'.
  - **pipRequirementsFile** [Mandatory for projects which use the pip package manager to download their dependencies, if pip requires the requirements file]
  - **useWrapper** - [Default: true] Determines whether to use the Gradle Wrapper for projects which are using Gradle.
  - **repository** - [Optional] Name of a Virtual Repository in Artifactory to resolve (download) the project dependencies from.

#### jfrogPlatform

The section includes the JFrog Platform settings

- **jfrogProjectKey** - [Optional] The JFrog project key. Learn more about it [here](https://www.jfrog.com/confluence/display/JFROG/Projects).
- **watches** - [Optional] The list of Xray watches. Learn more about it [here](https://www.jfrog.com/confluence/display/JFROG/Configuring+Xray+Watches).
