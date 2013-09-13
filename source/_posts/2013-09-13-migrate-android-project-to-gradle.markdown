---
layout: post
title: "Migrate Android Project to Gradle Build System"
date: 2013-09-13 13:57
comments: true
categories: [android, gradle]
---

[Gradle] is a new build system that Android is currently promoting. It can be
used to build Android project by adding a [Gradle Android plugin]. It is also
possible that Android would move to Gradle-only build system, ditching the old
ant-way to build things. The [Android Studio] only supports Gradle projects.

### Quickstart

To join the new family, we need to write a new build.gradle file in project
root. An general example of build.gradle would look like the snippet below.
Please note that the system is still in active development, version numbers
could change very soon. I'm using Gradle 1.7, plugin 0.5.6, Android Build Tools
17 and my target SDK version is 15.

``` groovy A simple build.gradle for Android project
buildscript {
  repositories {
    mavenCentral()
  }

  dependencies {
    classpath 'com.android.tools.build:gradle:0.5.6'
  }
}

apply plugin: 'android'

android {
  compileSdkVersion 15
  buildToolsVersion "17"

  sourceSets {
    main {
      manifest.srcFile 'AndroidManifest.xml'
        java.srcDirs = ['src']
        res.srcDirs = ['res']
    }
  }
}
```

This simple gradle file should be able to build most of standard Android
projects. But what if I have special need? I'm going to talk about how to
add support for AndroidAnnotation and token replacement below.

### Android Annotation

[AndroidAnnotation] is a set of very useful annotations that you could use in
your code to make your code shorter and more easier to read. The framework
utilize a annotation processor to scan through your source code file and
generate helper classes. To integrate it, we need to do 2 things:

  - Add processor flags to Java compiler
  - Add androidannotation-api dependency

#### Manipulating Compile Task

To add extra flags to compiler, we need to use the interface Android Gradle
plugin exposed to access relevant tasks. The build plugin organize tasks into
different "variants", some basic variants includes debug and release. To
iterate through all variants, we could use `android.applicationVariants.all`.

``` groovy Add support of Android annotation
// Code based on https://github.com/excilys/androidannotations/issues/676
def getSourceSetName(variant) {
  return new File(variant.dirName).getName();
}

android.applicationVariants.all { variant ->

  def aptOutputDir = project.file("build/source/apt")
  def aptOutput = new File(aptOutputDir, variant.dirName)

  android.sourceSets[getSourceSetName(variant)].java.srcDirs += aptOutput.getPath()

  variant.javaCompile.options.compilerArgs += [
    '-processorpath', configurations.apt.getAsPath(),
    '-s', aptOutput
  ]

  variant.javaCompile.source = variant.javaCompile.source.filter { p ->
    return !p.getPath().startsWith(aptOutputDir.getPath())
  }

  variant.javaCompile.doFirst {
    aptOutput.mkdirs()
  }

}
```

The `configurations.apt` is missing at the moment. We used the variable to
store path to Android Annotation processor and we use Gradle's dependencies
managing feature to resolve the path for us.

``` groovy Resolve Android Annotation processor path
configurations {
  apt
}

dependencies {
  apt files('compile-libs/androidannotations-2.7.1.jar')
}
```

### Add dependencies

For managing dependencies, Gradle uses a `dependencies` block. The following
code snippets shows different ways to include dependencies. `fileTree` can be
used to iterate files in directory, `string` will be resolved using
repositories(default includes Maven Central) and `project` is used to reference
dependent local projects. We will talk about multi-project build support later.

``` groovy Different ways to specify dependencies
dependencies {
  compile fileTree(dir: 'libs', include: '*.jar', exclude: 'android-support-v4.jar')
  compile 'com.android.support:support-v4:18.0.+'
  compile project(':extra:actionbarsherlock')
  compile project(':extra:ViewPagerIndicator')
}
```

Say if you put the Android Annotation API jar into libs directory, the build
system should picked it up by now.

## Token Replacement

Sometimes we would like to have a way to reference build version, git revisions
from Java code. To accomplish this, we need to collect those informations and
put it in a Java source file which will be picked out later during compilation.

Let's say if you have a AppBuild.java looks like below and we put it in
`compile-libs` directory.

``` java Template of AppBuild.java
package idv.Zero.example;

public class AppBuild {
  public static final String GIT_REV = "@git-rev@";
}
```

Note the `@git-rev@` is the token we will replace with current git revision
number each time we build the project. Now we have the template, we need to rig
it into the build system. The way I'm doing this is by adding a generation task
per variant and setup the dependency for compiling task to be depend on this
generation task.

``` groovy Rig token replacement into the build system
import org.apache.tools.ant.filters.ReplaceTokens

android.applicationVariants.all { variant ->
  def taskName = "generateAppBuild${variant.dirName.capitalize()}"

  task (taskName, type: Copy) {
    def todir = "build/source/AppBuild/${variant.dirName}/cc/hypo/pieceroids"
    new File(todir).mkdirs()

    from project.file('compile-libs/AppBuild.java')
    into todir
    outputs.upToDateWhen { false }

    def proc = "git rev-parse --verify HEAD".execute()
    proc.waitFor()

    filter(ReplaceTokens, tokens:['git-rev': proc.in.text.trim()])
  }
  tasks["compile${variant.dirName.capitalize()}"].dependsOn(taskName)
}
```

What this code snippets do is to initiate a copy task by the name
`generateAppBuild${variant}`. Note the outputs.upToDateWhen line would cause
the task to always run, otherwise it'll be skipped if the source file is not
changed, which is not what we want. You'll also need to import ReplaceTokens
filter from our good old friend, `ant`.

## Multi-Projects

It is very common that you might have some sub-projects that needs to be build
together. Previously, I showed that you could have `compile
project(':path:to:project')` in `dependencies` block to have it being included
during compile time. However, you need some extra setup to make it work. You
need an extra file called `settings.gradle` with those lines.

``` groovy settings.gradle for multi-projects
include ":extra:actionbarsherlock"
include ":extra:ViewPagerIndicator"
```

This indicates you have two sub-projects, they're located at
`${project.root}/extra/actionbarsherlock` and
`${project.root}/extra/ViewPagerIndicator`. Gradle will look for build.gradle
inside those directories and set up project dependencies automatically.

## Conclusion

In this article, I showed how to migrate your android project to utilize Gradle
as the build system. Additionally, I also showed a way to integrate
Android Annotations and build information generation. I hope these would be
enough to get your started with Gradle. Here is the final build.gradle you
should have after all these extra codes.

``` groovy Final build.gradle
import org.apache.tools.ant.filters.ReplaceTokens

buildscript {
  repositories {
    mavenCentral()
  }

  dependencies {
    classpath 'com.android.tools.build:gradle:0.5.6'
  }
}

apply plugin: 'android'

configurations {
  apt
}

dependencies {
  compile fileTree(dir: 'libs', include: '*.jar', exclude: 'android-support-v4.jar')
  compile 'com.android.support:support-v4:18.0.+'
  compile project(':extra:actionbarsherlock')
  compile project(':extra:ViewPagerIndicator')

  apt files('compile-libs/androidannotations-2.7.1.jar')
}

def getSourceSetName(variant) {
    return new File(variant.dirName).getName();
}

android.applicationVariants.all { variant ->
  def aptOutputDir = project.file("build/source/apt")
  def aptOutput = new File(aptOutputDir, variant.dirName)

  android.sourceSets[getSourceSetName(variant)].java.srcDirs += aptOutput.getPath()

  variant.javaCompile.options.compilerArgs += [
    '-processorpath', configurations.apt.getAsPath(),
    '-s', aptOutput
  ]

  variant.javaCompile.source = variant.javaCompile.source.filter { p ->
    return !p.getPath().startsWith(aptOutputDir.getPath())
  }

  variant.javaCompile.doFirst {
    aptOutput.mkdirs()
  }

  android.sourceSets[getSourceSetName(variant)].java.srcDirs += "build/source/AppBuild/${variant.dirName}"

  def taskName = "generateAppBuild${variant.dirName.capitalize()}"
  task (taskName, type: Copy) {
    def todir = "build/source/AppBuild/${variant.dirName}/cc/hypo/pieceroids"
    new File(todir).mkdirs()

    from project.file('compile-libs/AppBuild.java')
    into todir
    outputs.upToDateWhen { false }

    def proc = "git rev-parse --verify HEAD".execute()
    proc.waitFor()

    filter(ReplaceTokens, tokens:['git-rev': proc.in.text.trim()])
  }
  tasks["compile${variant.dirName.capitalize()}"].dependsOn(taskName)
}

android {
  compileSdkVersion 15
  buildToolsVersion "17"

  sourceSets {
    main {
      manifest.srcFile 'AndroidManifest.xml'
        java.srcDirs = ['src']
        res.srcDirs = ['res']
    }
  }
}
```

[Gradle]: http://www.gradle.org/
[Gradle Android plugin]: http://tools.android.com/tech-docs/new-build-system/user-guide
[Android Studio]: http://developer.android.com/sdk/installing/studio.html
[AndroidAnnotation]: https://github.com/excilys/androidannotations
