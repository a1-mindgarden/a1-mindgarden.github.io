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

- **User client**: Where the main functionality of the project is implemented. This will be implemented with KMP for Android and iOS platforms.

- **Admin client**:  Will handle *account suspensions*, *moderate messages*, *contact users*, *appoint other users as moderators*, *view system data*, and others that have not come to mind as of this writing. This will be implemented with Flutter for Desktop. 

> **Why different development technologies?**
>
> As the clients have very different interfaces for each role (except for the chat feature), it would only clutter up the environment on the user client every time I need to implement a feature in the admin client. A workaround could be implemented: modules. However, another reason not to do that is that Firebase (network backend) doesn’t fully support KMP as of this moment, but Flutter does. That way, we can prevent code clutter and have easy access to our backend service.
{: .prompt-info }

## Setting up the user client project

We will only be focusing on the user client right now. Once we make it functional, we will then begin with the admin client development. 

### Adding KMP plugin for Android Studio

To get started working with KMP projects, it's [required](https://www.jetbrains.com/help/kotlin-multiplatform-dev/multiplatform-setup.html#install-the-necessary-tools) to install the KMP plugin in Android Studio.

### Creating a KMP Project

The first step is to visit the [Kotlin Multiplatform Wizard](https://kmp.jetbrains.com) and fill out the required fields in the New Project tab.

I included three components, namely:
- Android;
- iOS with Share UI selected (for future iOS version);

> There will be iOS, but we will not be testing the app against it. We will only be focusing on Android in this journey.
{: .prompt-info }

and downloaded the project. Once finished, I opened the project in Android Studio and let Gradle sync finish.

### Updating necessary dependencies

To work on the latest and most stable version of the libraries, I will perform the upgrade steps suggested by Android Studio.

### Running on Android

This is to test if the project configuration is working.

I used an API level 30 and 35 emulator. Running the project on Android is straightforward, just by clicking the Run button or by using the keyboard shortcut `Shift + F10`. After a while, the app ran successfully on the emulator. This indicates there are no errors in the project.
...

That might be all with the initial setup of the development environment.

Up next would be setting up the theme for the UI. Specifically, we will be using the [Material Theme Builder](https://m3.material.io/theme-builder) for color scheme and typography generation. The project UI will follow the [Material 3 Design Guidelines](https://m3.material.io/) by Google. This will give us more time to implement features rather than design.
















