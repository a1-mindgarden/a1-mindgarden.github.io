---
layout: post
title: Setting up development environment
categories:
- Development
tags:
- setup
description: Walking through how we set up Kotlin Multiplatform with Android Studio
  to get started with the development.
date: 2025-03-21 22:03 +0800
---
## Project specifications

The project will require two clients for two different user roles:

- **User client**: Where the main functionality of the project is implemented.

- **Admin client**:  Will handle *account suspensions*, *moderate messages*, *contact users*, *appoint other users as moderators*, *view system data*, and others that have not come to mind as of this writing.

## Setting up

### Adding KMP plugin for Android Studio

To get started working with KMP projects, it's [required](https://www.jetbrains.com/help/kotlin-multiplatform-dev/multiplatform-setup.html#install-the-necessary-tools) to install the KMP plugin in Android Studio.

### Creating a KMP Project

The first step is to visit the [Kotlin Multiplatform Wizard](https://kmp.jetbrains.com) and fill out the required fields in the New Project tab.

I included three components, namely:
- Android (for user client);
- iOS with Share UI selected (for future iOS version);
- Desktop (for Admin client);

and downloaded the project. Once finished, I opened the project in Android Studio and let Gradle sync finish.

### Updating necessary dependencies

To work on the latest and most stable version of the libraries, I will perform the upgrade steps suggested by Android Studio.

### Running on Android and Windows

To test if the project configuration is working, I ran the project on Android and Windows.


For Android, I used an API level 30 emulator. Running the project on Android is straightforward, just clicking the Run button or by just using the keyboard shortcut `Shift + F10`. After a while, the app ran successfully on the emulator. This indicates there are no errors in the project.

For Windows, there is an additional step to run the project. First, I need to make sure that I am on the Project view. Next, I need to locate the `main.kt` of the `desktopMain` module. This is the entry point for the desktop version of the project. I can run the Windows version by clicking on the Play button in the gutter beside the `main` function definition and selecting Run 'main[desktop]'. After building for a while, a window with the same content as the Android UI shows, indicating a successful run.

...

That might be all with the initial setup of the development environment.

Up next would be setting up the theme for the UI. Specifically, we will be using the [Material Theme Builder](https://m3.material.io/theme-builder) for color scheme and typography generation. The project UI will follow the [Material 3 Design Guidelines](https://m3.material.io/) by Google. This will give us more time to implement features rather than design.















