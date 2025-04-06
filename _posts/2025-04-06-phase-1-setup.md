---
layout: post
title: 'Phase 1: Project Setup and Foundation'
categories:
- Development
date: 2025-04-06 11:50 +0800
tags:
- setup
- firebase
- kmp
- android studio
- vs code
- nodejs
description: This phase establishes the core Firebase project and prepares the local development environments necessary for building MindGarden.
---
## Overview

This phase establishes the core Firebase project and prepares the local development environments necessary for building MindGarden.

## Target Platforms & Technologies

*   **User/Moderator Client:** Kotlin Multiplatform (KMP) targeting Android (initially) and iOS (later).
*   **Admin Client:** Flutter targeting Windows.
*   **Backend:** Firebase (Firestore, Cloud Functions, Authentication).

## Steps Completed

### 1. Firebase Project Creation

*   Created a new project in the [Firebase Console](https://console.firebase.google.com/).
*   Noted the unique project ID.
*   Handled Google Analytics option.

### 2. Local Development Environment Setup

*   **Node.js/npm:** Installed for Firebase CLI and Cloud Functions.
*   **Firebase CLI:** Installed globally (`npm install -g firebase-tools`) and logged in (`firebase login`).
*   **Flutter SDK:** Installed for the Admin Desktop client, enabled desktop support, and verified (`flutter doctor`).
*   **JDK:** Ensured a compatible JDK is installed.
*   **Android Studio:** Installed for KMP/Android development, including Android SDK.
*   **KMP Tooling:** Ensured Android Studio is ready for KMP development.
*   **IDE(s):** Android Studio and VS Code.

### 3. Project Workspace Structure

*   Created a main workspace directory.
*   Established subdirectories for `backend`, `user_app` (KMP), and `admin_app` (Flutter).

### 4. Firebase Initialization (Backend)

*   Navigated to the `backend` directory.
*   Ran `firebase init`.
*   Selected features: `Firestore`, `Functions`.
*   Linked to the existing Firebase project.
*   Generated initial backend configuration files (`firebase.json`, `firestore.rules`, etc.) and the `functions` directory structure.

### 5. Initial Client Project Setup

*   Created a placeholder KMP project structure in `user_app/`.
*   Created a placeholder Flutter project structure in `admin_app/`.

## Outcome

*   A live Firebase project exists.
*   Local environments are set up for Node.js, KMP/Android, and Flutter Desktop development.
*   A structured local workspace is organized.
*   Firebase backend configuration is initialized locally in the `backend` directory.
*   Placeholder client project structures are created.

## Next Steps

*   Confirm the **Authentication Strategy** (Firebase Standard Auth + ID Field + MIS Validation).
*   Begin implementing the chosen authentication flow (Cloud Functions, Firestore setup).
*   Configure Firebase within the client projects (Android, Flutter Desktop).
