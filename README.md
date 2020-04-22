# Kermit <sub>the log</sub>

Kermit is a Kotlin Multiplatform logging utility with composable log outputs. The library provides prebuilt loggers for outputting to platform logging tools such as Logcat and NSLog.

## Usage

Logging calls are made to a `Kermit` instance which delegates logging to implementations of the `Logger` interface which are provided during instantiation. If no `Logger`s are provided, `CommonLogger`(which outputs to println) will be used by default.  

```kotlin
val kermit = Kermit(LogcatLogger(),CommonLogger(),CustomCrashLogger())
kermit.i("CustomTag", optionalThrowable) { "Message" }
```

`Kermit` provides convenience functions for each severity level:
* `v()` - Verbose
* `d()` - Debug
* `i()` - Info
* `w()` - Warn
* `e()` - Error
* `wtf()` - Assert

Each call takes optional parameters for tag and throwable and a function parameter which returns the string to be logged. The message function will only be invoked if there are loggers configured to log that particular message.

### Tags

Tags are passed to `Logger` implementations which can decide how to use them (or ignore them). `LogcatLogger` passes it along to Logcat's tag field. 

Tags can be set for logs in a few different ways:
* If no tag is specified, a default tag of "Kermit" will be used
* The default tag can be overridden in the `Kermit` constructor
* A copy of `Kermit` with a new default tag can be obtained with `kermit.withTag("newTag")`. This is handy for using a tag within a particular scope
* Each log can specify it's own tag with the optional tag parameter

### Loggers

Implementations of the `Logger` interface are passed into Kermit and determine the destination(s) of each log statement. 

#### Prebuilt Loggers

The Kermit library provides prebuilt loggers for common mobile cases:

* `CommonLogger` - Uses println to send logs in Kotlin
* `LogCatLogger` - Uses LogCat to send logs in Android
* `NSLogLogger`  - Uses NSLog to send logs in iOS

These can be created and passed into the Kermit object during initialization
```kotlin
val kermit = Kermit(LogcatLogger(),CommonLogger())
```

#### Custom Logger

If you want to customize what happens when a log is received, you can create a customized logger. For a simple `Logger` you only need to implement the `log` method, which handles all logs of every Severity.

```kotlin
class NSLogLogger : Logger() {
    override fun log(severity: Severity, message: String, tag: String, throwable: Throwable?) {
        NSLog("%s: (%s) %s", severity.name, tag, message)
    }
}
```

You can optionally override the severity convenience functions if desired.

```kotlin
override fun v(message: String, tag: String, throwable: Throwable?) {
    Log.v(tag, message, throwable)
}
```

#### isLoggable

Custom loggers may also override `fun isLoggable(severity: Severity): Boolean`. Kermit will check this value before logging to this Logger

## Sample

Kermit comes with a Sample Multiplatform project which gives a brief example of how it can be used in both Android and iOS. The example demonstrates logs being called from within common code as well as from each platform.

### OSLogLogger

The iOS sample also includes a custom logger for outputting using `os_log`. `os_log` is not currently available in Kotlin/Native and consequently could not be included as a prebuilt logger in the library, but does serve as a good demonstration of custom logger implementation

