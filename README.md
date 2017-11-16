## Introduction
Recently I migrated my project from Maven to Gradle. In the process I faced lots of challenges, one of them is 
compiling JasperReport. 

This post explains how to compile jasperreport's ```jrxml``` to ```jasper``` and package ```jasper``` 
to the jar.

## Prerequisites
1. JDK 1.7 or later
2. Gradle 2

## Project Structure
It's a typical gradle structure except ```src/main/jasperreports``` which will have the ```*.jrxml``` files.

    Project Root
        |--build.gradle
        |--settings.gradle
        |--src/
            |--main/
                |--jasperreports/
                |--java/
          
## Gradle Script

```groovy
apply plugin: 'java'

repositories {
    mavenCentral()
    jcenter()
    maven{url "http://jasperreports.sourceforge.net/maven2/"}
    maven{url "http://jaspersoft.artifactoryonline.com/jaspersoft/third-party-ce-artifacts/"}
}

configurations {
    jasperreports {
        transitive = true
    }
}

gradle.projectsEvaluated {
    processResources.dependsOn(compileJasperReports)
}
dependencies {
    jasperreports 'net.sf.jasperreports:jasperreports:5.6.0',
            'org.codehaus.groovy:groovy-all:2.3.6'
}



task compileJasperReports {
    def jasperSourceDir = file('src/main/jasperreports')
    ant {
        taskdef(name: 'jrc', classname: 'net.sf.jasperreports.ant.JRAntCompileTask', classpath: configurations.jasperreports.asPath)
        sourceSets.main.output.classesDir.mkdirs()
        jrc(srcdir: jasperSourceDir, destdir: sourceSets.main.output.classesDir) {
            classpath(path: sourceSets.main.output.classesDir)
            include(name: '**/*.jrxml')
        }
    }
}
```

This build script has two maven urls which has the dependency for jasperreport which are not available in maven central repo.

## Conclusion
When the project is build the ```*.jrxml``` files will be compiled to ```*.jasper``` filed and copied to the classes folder.
So when the project is packaged these ```*.jrxml``` files will be available in classpath
