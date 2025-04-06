---
layout: post
title: 'Phase 2: Authentication & Initial Data Setup'
description: This phase implements the complete authentication flow using Standard
  Email/Password (Personal Emails) combined with mandatory manual admin approval after
  in-person verification. It details the setup of required Firestore collections (`/schoolDirectory`,
  `/plantConfigs`, `/users`), the necessary Cloud Functions (TypeScript), full Firestore
  Security Rules, and the plan for password reset. Unapproved users are blocked from
  core app functionality.
categories:
- Backend development
tags:
- firebase
- nodejs
- typescript
date: 2025-04-06 17:55 +0800
---

## Core Strategy

*   **User Authentication:** Standard Email/Password (Personal Email).
*   **Login Identifier:** Personal Email + Password.
*   **Verification Process:** 1) ID/Name check against `/schoolDirectory` during sign-up. 2) Account created (`isApproved: false`). 3) Mandatory In-Person check + Admin Approval via `approveUser` function (`isApproved: true`, gamification fields initialized).
*   **Access Control:** Client app shows "Pending Approval" screen if `isApproved == false`. Firestore Security Rules (`isApproved` check) block feature access for unapproved users.
*   **Password Reset:** Standard Firebase flow (personal email).
*   **Admin Management:** Admins use standard Email/Password login. Specific functions/rules grant permissions.
*   **Gamification Start:** Users select initial plant via `selectInitialPlant` function *after* approval.

## Steps Completed & Code Provided

### 1. Prepared MIS Data Source (`/schoolDirectory`)

*   **Purpose:** Trusted list for ID/Name validation.
*   **Structure:** `/schoolDirectory/{id}` with `firstName`, `lastName`, `middleName?`.
*   **Security Rule:** Deny client read, allow admin write (Included in final rules below).
*   **Population Script:** Example Node.js script (`backend/scripts/populate_directory.js`) provided for initial bulk upload using Admin SDK and `serviceAccountKey.json`. **Action Required: Adapt and run this script.**
*   **Ongoing:** `addSchoolDirectoryEntry` Cloud Function for Admin App.

### 2. Defined Plant Configuration Data (`/plantConfigs`)

*   **Purpose:** Stores available plant types and growth rules.
*   **Structure:** `/plantConfigs/{plantTypeId}` with `name`, `stagePointThresholds`, `maxStage`, etc.
*   **Security Rule:** Allow authenticated read, admin write (Included in final rules below).
*   **Population:** **Action Required: Populate manually via Console or script.**

### 3. Defined User Profile Data Structure (`/users`)

*   **Purpose:** Stores individual user app data (Key: Firebase Auth UID).
*   **Structure:** Includes `uid`, `studentOrEmployeeId`, names, `email` (personal), `role`, **`isApproved` (default `false`)**, `isBanned`, `createdAt`, `peerSupportHandle`, `currentPlant` (initially `null`). Gamification fields (`points`, `plantStage`, `pointsTowardsNextStage`) initialized upon approval.
*   **Indexes:** `firestore.indexes.json` definition provided for `studentOrEmployeeId` and `email`. **Action Required: Deploy indexes.**

### 4. Implemented Backend Logic (Cloud Functions - TypeScript)

*   **Code:** Full TypeScript code for `registerUserPendingApproval`, `approveUser`, `addSchoolDirectoryEntry`, `selectInitialPlant` provided for `backend/functions/src/index.ts`.
*   **Build & Deployment:** **Action Required: Run `npm run build` and `firebase deploy --only functions`.**

### 5. Defined Security Rules (`firestore.rules`)

*   **Code:** Full `firestore.rules` file provided below, including helper functions and rules enforcing the approval workflow. **Action Required: Replace content of `backend/firestore.rules` and deploy.**
    ```firestore.rules
    rules_version = '2';
    service cloud.firestore {
      match /databases/{database}/documents {

        // --- Helper Functions ---
        function isSignedIn() { return request.auth != null; }
        function isOwner(userId) { return isSignedIn() && request.auth.uid == userId; }
        // !! IMPORTANT: Reading from DB in rules is less efficient/secure than Custom Claims. !!
        // !! Plan to implement Custom Claims and update these checks later. !!
        function getUserData(userId) { return get(/databases/$(database)/documents/users/$(userId)).data; }
        function getRole(userId) { return getUserData(userId).role; }
        function isAdmin() { return isSignedIn() && getRole(request.auth.uid) == 'admin'; }
        function isModerator() { return isSignedIn() && (getRole(request.auth.uid) == 'moderator' || isAdmin()); }
        function isApproved(userId) { return exists(/databases/$(database)/documents/users/$(userId)) && getUserData(userId).isApproved == true; }
        function isSelfApproved() { return isSignedIn() && isApproved(request.auth.uid); }
        function isNotBanned(userId) { return exists(/databases/$(database)/documents/users/$(userId)) && getUserData(userId).isBanned == false; }
        function canAccessFeatures() { return isSelfApproved() && isNotBanned(request.auth.uid); } // Key check for features

        // --- /schoolDirectory Collection ---
        match /schoolDirectory/{schoolId} {
          allow read: if false;
          allow write: if isSignedIn() && isAdmin() && isNotBanned(request.auth.uid);
        }

        // --- /plantConfigs Collection ---
        match /plantConfigs/{plantId} {
           allow read: if isSignedIn();
           allow write: if isAdmin() && isNotBanned(request.auth.uid);
        }

        // --- /users Collection ---
        match /users/{userId} {
          allow read: if isSignedIn() && isNotBanned(request.auth.uid) && (isOwner(userId) || isAdmin());
          allow create: if false;
          allow update: if isAdmin() && isNotBanned(request.auth.uid); // Primarily Admin/Function updates
          allow delete: if false; // Or admin only
        }

        // --- Rules for Core App Features (Examples) ---
        match /activityLog/{activityId} { allow read, write: if canAccessFeatures() /* && ownership checks etc. */; }
        match /moodEntries/{entryId} { allow read, write: if canAccessFeatures() && request.resource.data.userId == request.auth.uid; allow create: if canAccessFeatures() && request.resource.data.userId == request.auth.uid; }
        match /journalEntries/{entryId} { allow read, write: if canAccessFeatures() && request.resource.data.userId == request.auth.uid; allow create: if canAccessFeatures() && request.resource.data.userId == request.auth.uid; }
        match /meditationSessions/{sessionId} { allow read, write: if canAccessFeatures() && request.resource.data.userId == request.auth.uid; allow create: if canAccessFeatures() && request.resource.data.userId == request.auth.uid; }
        match /peerSupportPosts/{postId} { allow read: if canAccessFeatures() && (resource.data.status == 'approved' || isModerator()); allow write, delete: if false; } // Force via functions
        match /peerSupportPosts/{postId}/replies/{replyId} { allow read: if canAccessFeatures() && (resource.data.status == 'approved' || isModerator()); allow write, delete: if false; } // Force via functions
        match /peerSupportHelpfulVotes/{voteId} { allow read, write, delete: if false; } // Force via functions
        match /supportChats/{chatId} { allow read, write: if canAccessFeatures() /* && participant checks etc. */; } // Placeholder
        match /supportChats/{chatId}/messages/{messageId} { allow read, write: if canAccessFeatures() /* && participant checks etc. */; } // Placeholder

        // --- Admin Logs ---
        match /adminLogs/{logId} {
          allow read: if isAdmin() && isNotBanned(request.auth.uid);
          allow write: if false; // Only functions write here
        }
      }
    }
    ```
*   **Deployment:** **Action Required: Deploy rules via `firebase deploy --only firestore:rules`.**

### 6. Planned Password Reset Flow

*   Utilizes Firebase Auth's built-in `sendPasswordResetEmail`.
*   **Action Required: Implement UI/Logic on client apps.**

## Outcome

*   Complete backend setup for user sign-up with mandatory manual approval workflow.
*   Mechanisms for admins to manage school directory and approve users.
*   Standard Email/Password login flow using personal emails.
*   Security rules enforce approval status for feature access.
*   Foundation for gamification (plant selection post-approval).
*   Standard password reset capability enabled.
*   Full code provided for Functions and Security Rules.

## Next Steps

*   **Execute Prerequisite Actions:** Run population script for `/schoolDirectory`, populate `/plantConfigs`.
*   **Implement Client Logic:** Build KMP/Android and Flutter Admin apps (Sign-up, Login, Approval, Plant Selection flows).
*   **Implement Role/Claim Management:** Create `setUserRole` function, update rules helpers.
*   **Phase 3:** Implement core features (activity tracking, point system, gamification updates, forum, support chat).
