# Artist [![Build Status](https://travis-ci.org/uber/artist.svg?branch=master)](https://travis-ci.org/uber/artist)

As Android apps grow, providing common features and consistent functionality across Views becomes challenging. Typically, this results in copy-pasting features across views, monolithic classes, or complicated inheritance trees. Artist is a highly-extensible platform for creating and maintaining an app’s base set of Android views.

## Overview

Artist is a Gradle plugin written in Kotlin that generates a base set of Android `View`s. Artist-generated views are created using a stencil and trait system. Each view type is declared with a single stencil, which is comprised of  a set of  traits. All of this comes together to create an easily maintainable system of stencils and traits.

*Stencils*: A 1:1 ratio of stencils to corresponding generated views. Each Stencil can declare a set of traits they exhibit, while also implementing custom code that’s specific to their view (e.g. RecyclerView’s APIs).

*Traits*: Each trait can be declared by multiple stencils, and generate otherwise duplicate code across all views that exhibit them. Common examples include clicks, attach events, visibility changes, etc. They are a hook into the stencil’s code get process that are called during each stencil’s generation.

## Motivation

#### Common Façade

Everything is behind the façade of commonly named classes, basically "[YOUR_PREFIX]ViewName". This allows you to push as much functionality as you want behind them whilst not changing the front facing entry point. Things we can push behind them include new functionality, other base classes, framework bug fixes, etc.

#### Intelligence

Artist-generated views have deep internal knowledge of their internal state and interactions. This gives you flexibility to do a number of interesting, contextual actions under the hood.

*Automatic Instrumentation*: Artist-generated views know when they're being attached, changed to visible, clicked, etc. This allows you to do automatic instrumentation of impressions and taps in views when they occur, provided the developer has provided an ID. You can also detect and signal a developer if an ID is missing where there should be one.

*Accessibility*: This intelligence gives you enough insight into the state of the view hierarchy to make accessibility a first class citizen in the daily development cycle. Artist-generated views can intelligently infer if there are content description errors associated with them, and signal them to developers in the apps.

#### Reactive Semantics

Artist-generated views can have [RxBinding](https://github.com/JakeWharton/RxBinding) APIs as first class citizens in their public APIs. In a increasingly reactive world, this gracefully bridges common UI listener interactions to RxJava streams.

#### Sane, simple maintainability

The stencil and trait system ensures that base views are defined in one place and that extra functionality is divided up into single-focus traits.

## Usage

#### Create the Provider module
- Create a new plain Java/Kotlin module (non-Android)
- Add Artist dependencies (API, Traits, Traits-Rx)
- Create a class that implements `ViewStencilProvider`
- Put the fully qualified class name of the stencil provider in a file `src/main/resources/META-INF/services/com.uber.artist.api.ViewStencilProvider`
- If you have custom traits, then create a class that implements `TraitProvider`
- Put the fully qualified class name of the trait provider in a file `src/main/resources/META-INF/services/com.uber.artist.api.TraitProvider`

#### Setup buildSrc
- Create a dir at root of project named `buildSrc`
- Navigate to `buildSrc` and add a relative symlink to the provider module `cd $PROJECT_ROOT/buildSrc; ln -s ../path/to/provider/module/root custom-artist-providers`
- Create a `settings.gradle` in `buildSrc` and add `include :custom-artist-providers`
- Update the `build.gradle` for the `buildSrc` project to ensure that the `custom-artist-providers` module is added the buildScript classpath so it is available to the Artist plugin
```
subprojects { subproject ->
    if (subproject.buildFile.exists()) {
        repositories {
            jcenter()
            mavenCentral()
        }

        rootProject.dependencies {
            runtime project(path)
        }
    }
    subproject.afterEvaluate {
        // Disable useless tasks in buildSrc
        if (subproject.plugins.hasPlugin("kotlin")) {
            subproject.tasks.withType(org.jetbrains.kotlin.gradle.tasks.KotlinCompile).all {
                kotlinOptions.suppressWarnings = true
            }
        }

        subproject.tasks.findAll {
            it.name.toLowerCase().contains("test") ||
                it.name.toLowerCase().contains("lint") ||
                it.name.toLowerCase().contains("checkstyle") }.each {
            it.enabled = false
        }
    }
}
```

## Download

Artist Plugin [![Maven Central](https://img.shields.io/maven-central/v/com.uber.artist/artist.svg)](https://mvnrepository.com/artifact/com.uber.artist/artist)
```gradle
classpath 'com.uber.artist:artist:0.0.1'
```

Artist API [![Maven Central](https://img.shields.io/maven-central/v/com.uber.artist/artist-api.svg)](https://mvnrepository.com/artifact/com.uber.artist/artist-api)
```gradle
classpath 'com.uber.artist:artist-api:0.0.1'
```

Artist Traits [![Maven Central](https://img.shields.io/maven-central/v/com.uber.artist/artist-traits.svg)](https://mvnrepository.com/artifact/com.uber.artist/artist-traits)
```gradle
classpath 'com.uber.artist:artist-traits:0.0.1'
```

Artist Rx Traits [![Maven Central](https://img.shields.io/maven-central/v/com.uber.artist/artist-traits-rx.svg)](https://mvnrepository.com/artifact/com.uber.artist/artist-traits-rx)
```gradle
classpath 'com.uber.artist:artist-traits-rx:0.0.1'
```

## License

```
Copyright (C) 2017 Uber Technologies

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```