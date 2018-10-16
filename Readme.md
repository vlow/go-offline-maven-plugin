# Go Offline Maven Plugin
Maven Plugin used to download all Dependencies and Plugins required in a Maven build,
so the build can be run without an internet connection afterwards.

This is especially relevant with modern CI-Systems like Gitlab and Circle-CI that
need a consistent local Maven repository in their cache to build efficiently.

Maven already has an official to do this: The maven-dependency-plugin go-offline goal
Unfortunately, the go-offline plugin suffers from several drawbacks:

- Multi-Module builds are not supported since the plugin tries to download Reactor-Dependencies from the Remote Repository
- Most parameters simply do not work
- No option to download dynamic dependencies

The Go Offline Maven Plugin fixes these drawbacks.  

## Requirements
- Java 1.7 or higher
- Maven 3.1.x or higher

## Goals
The Go Offline Maven Plugin only has one goal: "resolve-dependencies". This goal downloads
all external Dependencies and Plugins needed for your build to your local Repository.
Dependencies that are built inside the reactor build of your project are excluded. For Downloading,
the repositories specified in your pom.xml are used.

## Usage
Simply add the plugin to the pom.xml of your project. Use the Root Reactor pom in case of a multi module project.
Make sure to configure any dynamic dependency your project has (see below).

    <plugin>
        <groupId>de.qaware.maven</groupId>
        <artifactId>go-offline-maven-plugin</artifactId>
        <version>1.0</version>
        <configuration>
            <dynamicDependencies>
                <DynamicDependency>
                    <groupId>org.apache.maven.surefire</groupId>
                    <artifactId>surefire-junit4</artifactId>
                    <version>2.20.1</version>
                    <repositoryType>PLUGIN</repositoryType>
                </DynamicDependency>
            </dynamicDependencies>
        </configuration>
    </plugin>
    
To download all depenencies to your local repository, use
    
    mvn de.qaware.maven:go-offline-maven-plugin:resolve-dependencies

Make sure to activate any Profiles etc. so that all relevant modules of your project are included
in the Maven run.

### Dynamic Dependencies
Unfortunately some Maven Plugins dynamically load additional dependencies when they are run. Since those
dependencies are not necessarily specified anywhere in the plugins pom.xml, the Go Offline Maven Plugin
cannot know that it has to download those dependencies. Most prominently, the surefire-maven-plugin dynamically
loads test-providers based on test it finds in your project.

You must tell the Go Offline Maven Plugin of those dynamic depenencies to ensure all dependencies are downloaded.
For each Dependency, add a DynamicDependency block to the plugins configuration as seen in the Usage section.
Each dynamic dependency block consists of four parameters:

- *groupId* The GroupId of the dynamic dependency to download
- *artifactId* The ArtifactId of the dynamic dependency to download
- *version* The version of the dynamic dependency to download
- *repositoryType* Either 'MAIN' or 'PLUGIN' to control from which repository the dependency is downloaded

Note that Plugins are not consistent about where they pull their dynamic dependencies from. Some use the Plugin-Repository
, some the Main-Repository. If one doesn't work, try the other.

### Downloading Sources
The plugin can also download the source files of the projects transitive dependencies. This behaviour can either be activated via the pom.xml
or a command line parameter.

    <plugin>
        <groupId>de.qaware.maven</groupId>
        <artifactId>go-offline-maven-plugin</artifactId>
        <version>1.0</version>
        <configuration>
           <downloadSources>true</downloadSources>
        </configuration>
    </plugin>          
    
or

    mvn de.qaware.maven:go-offline-maven-plugin:resolve-dependencies -DdownloadSources
    
##License

Apache 2.0 (https://www.apache.org/licenses/LICENSE-2.0.txt) 
    