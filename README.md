LiquidCore
==========

The LiquidCore library provides independent, instantiable [Node.js] virtual machines each
with its own lifecycle, virtual file system, persistent and non-persistent storage,
and [SQLite] database.

Version
-------
[0.1.0](https://github.com/LiquidPlayer/LiquidCore/releases/tag/v0.1.0) - Get it through [JitPack](https://jitpack.io/#LiquidPlayer/LiquidCore/v0.1.0)

[![](https://jitpack.io/v/LiquidPlayer/LiquidCore.svg)](https://jitpack.io/#LiquidPlayer/LiquidCore)

Version 0.1.0 is currently only suitable for use as an [AndroidJSCore] replacement.

LiquidCore and The LiquidPlayer Project
---------------------------------------

[The LiquidPlayer Project] is an open source mobile micro-app ecosystem and development
environment built around the JavaScript language.  More on that later.

The LiquidCore library is the kernel of LiquidPlayer.  It provides independent,
instantiable [Node.js] virtual machines each with its own lifecycle, virtual file system,
persistent and non-persistent storage, and [SQLite] database.

LiquidCore provides 3 interfaces to the host app:

1. JavaScript API
2. Node.js VM API
3. UI Widgets

## 1. JavaScript API

LiquidCore is built on top of [Node.js], which is in turn built on [V8].  So, the V8 API
is natively available to any app which includes the LiquidCore library.  In addition to
directly interacting with V8 (a powerful, but incredibly complex API), LiquidCore provides
two additional APIs: a Java Native Interface (JNI) API for Android, and a JavaScriptCore
API for iOS and [React Native].

### Java Native Interface (JNI) for Android

The JNI is a near drop-in replacement for [`AndroidJSCore`](https://github.com/ericwlange/AndroidJSCore).
In fact, development on `AndroidJSCore` has ceased and is superseded by LiquidCore.  See
the [Javadocs] for complete documentation of the API.  If you have been using `AndroidJSCore`,
the interface will look familiar.  To migrate from `AndroidJSCore`, you must:

1. Replace the `AndroidJSCore` library with the `LiquidCore` library in `build.gradle`
2. Change the package name from `org.liquidplayer.webkit.javascriptcore` to `org.liquidplayer.javascript` in all of your source files
3. Fix any inconsistencies between the versions.  There aren't many.  It is 99% the same.

Otherwise, to get started, you need to create a JavaScript `JSContext`.  The execution of JS code
occurs within this context, and separate contexts are isolated virtual machines which
do not interact with each other.

```java
JSContext context = new JSContext();
```

This context is itself a JavaScript object.  And as such, you can get and set its properties.
Since this is the global JavaScript object, these properties will be in the top-level
context for all subsequent code in the environment.

```java
context.property("a", 5);
JSValue aValue = context.property("a");
double a = aValue.toNumber();
DecimalFormat df = new DecimalFormat(".#");
System.out.println(df.format(a)); // 5.0
```

You can also run JavaScript code in the context:

```java
context.evaluateScript("a = 10");
JSValue newAValue = context.property("a");
System.out.println(df.format(newAValue.toNumber())); // 10.0
String script =
  "function factorial(x) { var f = 1; for(; x > 1; x--) f *= x; return f; }\n" +
  "var fact_a = factorial(a);\n";
context.evaluateScript(script);
JSValue fact_a = context.property("fact_a");
System.out.println(df.format(fact_a.toNumber())); // 3628800.0
```

You can also write functions in Java, but expose them to JavaScript:

```java
JSFunction factorial = new JSFunction(context,"factorial") {
    public Integer factorial(Integer x) {
        int factorial = 1;
        for (; x > 1; x--) {
        	   factorial *= x;
        }
        return factorial;
    }
};
```

This creates a JavaScript function that will call the Java method `factorial` when
called from JavaScript.  It can then be passed to the JavaScript VM:

```java
context.property("factorial", factorial);
context.evaluateScript("var f = factorial(10);")
JSValue f = context.property("f");
System.out.println(df.format(f.toNumber())); // 3628800.0
```
 
### JavaScriptCore API

There are two major open source JavaScript implementations: [V8], which is popularized by
Google Chrome, and [JavaScriptCore], which is part of [WebKit], backed by Apple's Safari.
LiquidCore uses V8, simply because it is built on Node.js, which is difficult to decouple
from V8.  However, the JavaScriptCore API has a few advantages: (1) it is far simpler to
use than V8, (2) it is a familiar interface to iOS developers as the
[JavaScriptCore framework] has been available since iOS 7, and (3) other very useful
projects, like [React Native] require the library.  So, to take advantage of this,
LiquidCore provides a JavaScriptCore -> V8 API, where projects that require the
JavaScriptCore API can use the V8 backend with little or no modification.

This API is currently experimental and is not yet fully supported.  But feel free to
play with it.


## 2. Node.js VM API

LiquidCore enables app developers to launch virtual Node instances.  These instances
are isolated from each other and the host app.  Each VM consists of its own process
thread, JavaScript context, virtual file system, and SQLite database.  The full Node.js
operating environment is available, including file system access, socket-level networking,
process management, etc.  For security reasons, native extensions and OS-level process
spawning have been disabled, and the file system is restricted.

This API is currently experimental and not yet fully supported.

## 3. UI Widgets

Simple UI widgets to aid with micro app development and debugging will be provided.  Currently,
`NodeConsoleView` is implemented for Android which allows a Node console to be attached
to a `Process`.  This is currently experimental and not yet fully supported.


Use LiquidCore in your Android project
--------------------------------------

#### Step 1. Add the JitPack repository to your build file

Add it in your root build.gradle at the end of repositories:

	allprojects {
		repositories {
			...
			maven { url "https://jitpack.io" }
		}
	}
	
#### Step 2. Add the dependency

	dependencies {
	        compile 'com.github.liquidplayer:LiquidCore:v0.1.0'
	}

You should be all set!

Use LiquidCore in your iOS project
----------------------------------

The iOS version is not yet available.  Stay tuned.


Building the LiquidCore Android library
---------------------------------------

If you are interested in building the library directly and possibly contributing, you must
do the following:

    % git clone https://github.com/liquidplayer/LiquidCore.git
    % cd LiquidCore/LiquidCoreAndroid
    % echo ndk.dir=$ANDROID_NDK > local.properties
    % echo sdk.dir=$ANDROID_SDK >> local.properties
    % ./gradlew assembleRelease

Your library now sits in `LiquidCoreAndroid/build/outputs/aar/LiquidCore-0.1.0-release.aar`.  To use it, simply
add the following to your app's `build.gradle`:

    repositories {
        flatDir {
            dirs '/path/to/lib'
        }
    }

    dependencies {
        compile(name:'LiquidCore-0.1.0-release', ext:'aar')
    }
    
##### Note

The Node.js library (`libnode.so`) is pre-compiled and included in binary form in
`LiquidCoreAndroid/jni/lib/**/libnode.so`, where `**` represents the ABI.  All of the
modifications required to produce the library are included in `deps/node-6.4.0`.  To
build each library (if you so choose), you can do the following:

```
.../LiquidCore/deps/node-6.4.0% ./android-configure /path/to/android/ndk <abi>
```
where \<abi\> is one of `arm`, `arm64`, `x86` or `x86_64`


License
-------

 Copyright (c) 2014-2016 Eric Lange. All rights reserved.

 Redistribution and use in source and binary forms, with or without
 modification, are permitted provided that the following conditions are met:

 - Redistributions of source code must retain the above copyright notice, this
 list of conditions and the following disclaimer.

 - Redistributions in binary form must reproduce the above copyright notice,
 this list of conditions and the following disclaimer in the documentation
 and/or other materials provided with the distribution.

 THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

[NDK]:http://developer.android.com/ndk/index.html
[latest release]:https://github.com/ericwlange/AndroidJSCore/releases
[webkit]:https://webkit.org/
[Javadocs]:https://liquidplayer.github.io/LiquidCoreAndroid/
[JavaScriptCore]:https://developer.apple.com/reference/javascriptcore
[JavaScriptCore framework]:https://www.bignerdranch.com/blog/javascriptcore-example/
[V8]:https://developers.google.com/v8/
[Node.js]:https://nodejs.org/
[SQLite]:https://sqlite.org/
[The LiquidPlayer Project]:https://github.com/LiquidPlayer/
[React Native]:https://facebook.github.io/react-native/
[AndroidJSCore]:https://github.com/ericwlange/AndroidJSCore
