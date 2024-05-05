# ARM Memory Tagging Extension - A Deep Dive

## Introduction
Memory safety errors, such as temporal and spatial errors, are a large percentage of security vulnerabilities. Microsoft has found that, in severe-level security bugs, 70 percent were a result of underlying memory safety problems.  With memory safety prevention becoming a prevalent field within academia, industry, and governments, the line between the security guarantees being offered and their associated overheads has become blurred and/or obfuscated. The Memory Tagging Extension (MTE) is one memory safety protection mechanism introduced by ARM in ARMv8.5-A which “aims to increase the memory safety of code written in unsafe languages without requiring source changes, and in some cases, without requiring recompilation”. While MTE has notable security and memory promises alongside industry praise, there is little third-party exploratory research into these claims. Thus, our research aimed to explore four sides of MTE:
1. Claims of minimal or negligible overhead: Google claims a RAM overhead of 3-5% and CPU overhead of “low-single-digit%”
2. Bug Detection Probabilities: 93-100%, or 15/16 to 1/1 odds of catching bugs
3. Tag Storage and Generation: due to the closed-source nature of MTE, we aim to reverse-engineer where and how tags are stored. 
4. Conglomerate previous MTE research: to demystify MTE, we would like to group previous findings to allow developers to decide whether MTE meets their use case.

# Installation

## Prerequisites
### Hardware
- Pixel 8 with Root Access
- Android Studio and Android NX
- C/C++ Test Project
### Emulation
- TODO
  
### Steps
1. enable MTE on the phone by performing these steps:
  i. Enable developer settings by Settings > About phone > pressing "Build number" 7 times
 ii. Reboot the phone
iii. Head to Settings > Developer Settings > Memory Tagging Extension and press "Enable MTE until you turn it off"
iv.  Reboot the phone
2. Create a sample C/C++ Project in Android Studio as seen here: https://developer.android.com/studio/projects/add-native-code. The hello world example is a great starting point you can hijack for this purpose!
3. In the C/C++ project, edit AndroidManifest.xml as follows:

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <application
        android:memtagMode="sync"
        tools:replace="android:memtagMode"
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.MteStudy_Benchmarking"
        tools:targetApi="31">
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

4. Edit CMakeLists.txt as follows: 

```
# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html.
# For more examples on how to use CMake, see https://github.com/android/ndk-samples.

# Sets the minimum CMake version required for this project.
cmake_minimum_required(VERSION 3.22.1)

# Declares the project name. The project name can be accessed via ${ PROJECT_NAME},
# Since this is the top level CMakeLists.txt, the project name is also accessible
# with ${CMAKE_PROJECT_NAME} (both CMake variables are in-sync within the top level
# build script scope).
project("mtestudy_benchmarking")

# Creates and names a library, sets it as either STATIC
# or SHARED, and provides the relative paths to its source code.
# You can define multiple libraries, and CMake builds them for you.
# Gradle automatically packages shared libraries with your APK.
#
# In this top level CMakeLists.txt, ${CMAKE_PROJECT_NAME} is used to define
# the target library name; in the sub-module's CMakeLists.txt, ${PROJECT_NAME}
# is preferred for the same purpose.
#
# In order to load a library into your app from Java/Kotlin, you must call
# System.loadLibrary() and pass the name of the library defined here;
# for GameActivity/NativeActivity derived applications, the same library name must be
# used in the AndroidManifest.xml file.
add_library(${CMAKE_PROJECT_NAME} SHARED
        # List C/C++ source files with relative paths to this CMakeLists.txt.
        native-lib.cpp)

# Specifies libraries CMake should link to your target library. You
# can link libraries from various origins, such as libraries defined in this
# build script, prebuilt third-party libraries, or Android system libraries.
target_link_libraries(${CMAKE_PROJECT_NAME}
        # List libraries link to the target library
        android
        log)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -march=armv8.5-a+memtag")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -march=armv8.5-a+memtag")
```

5. Copy and paste any of the example .cpp codes into the native-lib.cpp file and you will be able to use MTE!
