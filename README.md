# kotlin-frontend-plugin  [![](https://jitpack.io/v/Kotlin/kotlin-frontend-plugin.svg)](https://jitpack.io/#Kotlin/kotlin-frontend-plugin)
Gradle plugin for Kotlin frontend development

The plugin provides an easy way to gather Maven and npm dependencies, pack bundles (via [webpack](https://webpack.github.io/)) and test a frontend application using [Karma](http://karma-runner.github.io/1.0/index.html). By default the plugin generates all required configs for webpack, karma and manages the corresponding daemons.

By using Gradle continuous build, you also can get hot module replacement feature (apply code changes in browser on the fly). See corresponding [section below](#hot-module-replacement).

# Howto

### Configure Gradle project

Fist of all you have to apply plugin `org.jetbrains.kotlin.frontend` and setup Kotlin:

```gradle
buildscript {
    ext.kotlin_version = '1.1.0'

    repositories {
        jcenter()
        maven {
            url "https://jitpack.io"
        }
    }

    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
        classpath "com.github.Kotlin:kotlin-frontend-plugin:568610baa1"
    }
}


// apply plugin
apply plugin: 'org.jetbrains.kotlin.frontend'

// apply kotlin2js
apply plugin: 'kotlin2js'

// configure kotlin compiler
compileKotlin2Js {
    kotlinOptions.metaInfo = true
    kotlinOptions.outputFile = "$project.buildDir.path/js/${project.name}.js"
    kotlinOptions.sourceMap = true
    kotlinOptions.moduleKind = 'commonjs'
    kotlinOptions.main = "call"
}
```

### Setup npm dependencies

All frontend plugin settings are applied in `kotlinFrontend` section:

```
kotlinFrontend {
    npm {
        dependency "style-loader" // production dependency
        devDependency "karma"     // development dependency
    }
}
```

### webpack bundler

To create a webpack bundle (for both packaging and running the dev server):

```
kotlinFrontend {
    webpackBundle {
        bundleName = "main"
    }
}
```

### Complete example

See [examples/frontend-only/build.gradle](examples/frontend-only/build.gradle) for a full example.

# Building and running

To run dev server (that also will build kotlin sources):

`gradlew run`

To pack the bundle:

`gradle bundle`

To stop running webpack and Karma daemons:

`gradle stop`

# webpack

webpack configuration: 

```
kotlinFrontend {
    webpackBundle {
        bundleName = "main"
        sourceMapEnabled = true | false   // enable/disable source maps 
        contentPath = file(...) // a file that represents a directory to be served by dev server)
        publicPath = "/"  // web prefix
        port = 8088   // dev server port
        proxyUrl = "" | "http://...."  // URL to be proxied, useful to proxy backend webserver
        stats = "errors-only"  // log level
    }
}
```

dev server log is located at `build/logs/webpack-dev-server.log`

config file is generated at `build/webpack.config.js`

## webpack configuration customization

To customize webpack coniguration, you can apply additional scripts by placing them in the directory `webpack.config.d`. The scripts will be appended to the end of config script. Use number prefix to change order (it is very similar to UNIX rc.d config directories)

Sample structure:

- [DIR] webpack.config.d
  - css.js
  - minify.js
  - 10-apply-ealier.js
  - 20-apply-later.js

# Karma

Karma configuration:

```
kotlinFrontend {
    karma {
        port = 9876
        runnerPort = 9100
        reporters = listOf("progress") 
        frameworks = listOf("qunit") // for now only qunit works as intended
        preprocessors = listOf("...")
    }
}
```

karma log is located at `build/logs/karma.log`

config file is generated at `build/karma.conf.js`

# Hot module replacement

Webpack provides ability to apply code changes on the fly with no page reload (if possible). For reference see [Webpack Hot Module Replacement documentation](https://webpack.js.org/concepts/hot-module-replacement/)

Webpack does a lot of work for you however to get it working well most likely you have to implement state save and restore functionality via webpack's API. See [HMR.kt](examples/frontend-only/src/main/kotlin/test/hello/HMR.kt) for corresponding Kotlin external declarations for webpack API and [main.kt](examples/frontend-only/src/main/kotlin/test/hello/main.kt) for sample save/load.

Briefly at module load accept HMR feature and listen for disposal

```kotlin
module.hot?.let { hot ->
    hot.accept() // accept hot reload
    
    hot.dispose { data -> // listen for disposal events
        data.my-fields = [your application state] // put your state in the 'data' object
    }
}
```

To get previously saved state at module load use `module.hot?.data`

```kotlin
    module.hot?.data?.let { data -> // if we have previous state then we are in the middle of HMR
        myRestoreFunction(data) // so get state from the 'data' object
    }
```

Finally use Gradle continuous build with run task to get live replacement every time you change your code.

```
gradlew -t run
```
