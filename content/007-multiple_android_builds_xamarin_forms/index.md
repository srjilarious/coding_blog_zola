+++
tags = ["Android"]
date = 2017-07-29
title = "Multiple Android builds on Xamarin Forms"
+++

While working on a Xamarin.Forms based project I ran into the issue of wanting to have multiple builds of my android app differentiated by the build configuration.  For instance, I want my normal build to point to the production server, but when I'm developing locally I want a different app that points to my local dev backend.  I still want to be able to use the _current production app_ and have a _development version_ installed at the **same time**.

Part of the problem I had with a single apk, is that I would forget which version was installed and not remember which server I was pointing to when using the app.

I started working on a solution for iOS as well, but it would involve a script to swap out the Info.plist while as a pre-build step.  Since I do most of my development on my Windows machine with Android, I've left the iOS fix for later.

## Setting up Visual Studio for Multiple Android Builds

The first step is to have different build configurations in Visual Studio so that we can differentiate between the different apks that we want to build.  First, I went to `Build -> Configuration Manahger` and then I selected that I wanted to create a new configuration, copying the `Debug` configuration.

We can use the configuration not only to have configuration specific preprocessor flags that we use in our code, but we can conditionally include files by configuration with MSBuild, even though some options are not always available in Visual Studio's UI.

There are two things that we want for different build configurations: A different package name so that Android can differentiate between our builds and a different label so that we can.

The first step is to create Android manifests for each configuration:

- Create copies of your AndroidManifest, naming them so that you can identify which one belongs to which configuration.  
For Instance, I create a LocalDev configuration, so I have an AndroidManifest.LocalDev.manifest.
- Add a new package name in your copied manifest's `manifest` element
- Add a different label to the `application` element so we can differentiate when looking at the launcher which apk is which.

Here's my full LocalDev AndroidManifest:

    <?xml version="1.0" encoding="utf-8"?>
    <manifest 
        xmlns:android="http://schemas.android.com/apk/res/android" 
        android:installLocation="auto" 
        package="MyApp.Droid.LocalDev">

        <uses-sdk android:minSdkVersion="15" android:targetSdkVersion="23" />
        <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
        <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
        <application android:label="MyApp LocalDev"
                     android:theme="@android:style/Theme.Material.Light" 
                     android:icon="@drawable/icon"></application>
    </manifest>

The second step is to set MSBuild to use our new Android Manifest in the new build configuration.  You can do this by editing your App's Android .csproj in a text editor.  Notice that the `PropertyGroup` for each configuration can add its own element for the `AndroidManifest` to use.

    <AndroidManifest>Properties\AndroidManifestLocalProd.xml</AndroidManifest>


My .csproj configuration `PropertyGroup` block for my LocalDev build configuration looks like:

    <PropertyGroup Condition="'$(Configuration)|$(Platform)' == 'DebugLocalDevServer|AnyCPU'">
        <DebugSymbols>true</DebugSymbols>
        <OutputPath>bin\DebugLocalProdServer\</OutputPath>
        <DefineConstants>TRACE;DEBUG;USE_LOCAL_PROD_SERVER</DefineConstants>
        <DebugType>full</DebugType>
        <PlatformTarget>AnyCPU</PlatformTarget>
        <GenerateSerializationAssemblies>Off</GenerateSerializationAssemblies>
        <ErrorReport>prompt</ErrorReport>
        <CodeAnalysisRuleSet>MinimumRecommendedRules.ruleset</CodeAnalysisRuleSet>
        <AndroidManifest>Properties\AndroidManifestLocalDev.xml</AndroidManifest>
    </PropertyGroup>

**Note:** the actual manifest that is included in your app's apk is created using this manifest file as a base and then merging in class attribute related information to create a final version.  The attributes take precedence, so if you have a label set on your activity or application class through an attribute, that will negate what you've placed in your manifest file.

Check the following Xamarin article for more details: [Working with Android Manifest](https://developer.xamarin.com/guides/android/advanced_topics/working_with_androidmanifest.xml/)


## Final thoughts
A downside with this approach is that you now have to work to keep your different manifests in sync for the pieces that do show up in there.  

As I mentioned in the beginning I'm looking to make a script for iOS builds to do the same thing.  This would be applicable to Android as well and allow you to have just a template manifest that is used to generate the correct configuration manifest as a prebuild step.  

The open question there is if the generated manifest gets picked up properly on configuration swaps or if a cached version is used, etc.  Seeing as how the process above has been working really well for me so far, the iOS version has a been a lower priority on my task list.

I hope this can help some other developers out there as well to get around the prod/dev apk issue I was hitting.

<div id="commento"></div>
<script src="https://cdn.commento.io/js/commento.js"></script>
