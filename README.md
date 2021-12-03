# Friendly_chat_app
Version 1.0 2021/12/3

This repository contains code for the FriendlyChat project.

## Overview

FriendlyChat is an app that allows users to send and receive text and photos in realtime across platforms.

<img src="https://user-images.githubusercontent.com/73274912/144553710-05094b43-879a-4d16-bb64-800b6c233fa5.jpeg" width=45% height=25% alt="Sign In">        <img src="https://user-images.githubusercontent.com/73274912/144561726-751f85a9-dada-4336-88b9-28f1ee17b155.jpeg" width=45% height=25% alt="Notification">


https://user-images.githubusercontent.com/73274912/144553612-4c0280c2-7caf-4aee-b814-55a715b731e0.mp4

# Device permissions
*App needs the following user's permissions to provide the featured functionality*
* Request user's permission to read and write to external storage
```xml
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
```
```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```
* Request user's permission to access the internet
```xml
<uses-permission android:name="android.permission.INTERNET"/>
```
```gradle
defaultConfig {
        applicationId "com.google.firebase.udacity.friendlychat"
        minSdkVersion 19
        targetSdkVersion 30
        versionCode 1
        versionName "1.0"
    }
```
* The app prompts the user to log in using email and google signIn
* Upon successfull authentication chat screen is loaded
* App is build upon the following features:
    * send and receive text in realtime across platforms
    * send and receive photos in realtime across platforms
* Some of the words used in a conversation are replaced by emojis 😂 😸
```gradle
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    testImplementation 'junit:junit:4.12'
    implementation 'com.google.android.material:material:1.0.0'
    implementation 'androidx.appcompat:appcompat:1.0.0'
    implementation 'com.google.firebase:firebase-database:20.0.2'
    implementation 'com.google.firebase:firebase-auth:21.0.1'
    implementation 'com.firebaseui:firebase-ui-auth:7.2.0'
    implementation 'com.google.firebase:firebase-messaging:23.0.0'
    implementation 'com.google.firebase:firebase-config:21.0.1'
    implementation 'com.github.bumptech.glide:glide:3.6.1'
    implementation platform('com.google.firebase:firebase-bom:29.0.0')
    implementation 'com.google.firebase:firebase-storage'
    implementation 'com.google.firebase:firebase-analytics'
    androidTestImplementation 'org.testng:testng:6.9.6'
    androidTestImplementation 'org.junit.jupiter:junit-jupiter'
    androidTestImplementation 'androidx.test.ext:junit:1.1.1'
}
```
## Features
* SignIn Intent implemented with FirebaseUI
```java
startActivityForResult(
                            AuthUI.getInstance()
                                    .createSignInIntentBuilder()
                                    .setIsSmartLockEnabled(true, true)
                                    .setAvailableProviders(Arrays.asList(              
                                            new AuthUI.IdpConfig.EmailBuilder().build(),
                                            new AuthUI.IdpConfig.GoogleBuilder().build()))
                                    .build(),
                            RC_SIGN_IN); //Array List initialized and stored in variable named providers in MainActivity

```
* Uploading choosen photo to Firebase Storage and make a reference to it from the Realtime Database
```java
// Upload file to Firebase Storage
           Uri selected_photo = data.getData();
            StorageReference photoRef = mChatPhotosStorageReference.child("chat_photos/"+selected_photo.getLastPathSegment());
            //upload file to firebase strorage
            photoRef.putFile(selected_photo)
                    .addOnSuccessListener(
                            new OnSuccessListener<UploadTask.TaskSnapshot>() {
                                @Override
                                public void onSuccess(
                                        UploadTask.TaskSnapshot taskSnapshot) {
                                    photoRef.getDownloadUrl().addOnSuccessListener(new OnSuccessListener<Uri>() {
                                        @Override
                                        public void onSuccess(Uri uri) {
                                            Uri my_uri = uri;
                                            FriendlyMessage current_message = new FriendlyMessage(null,mUsername,my_uri.toString());
                                            mMessagesDatabaseReference.push().setValue(current_message);

                                        }
                                    });
                                }
                            });
```
* Configured Remote Configs from Firebase to adjust the variable of the exchanged message length
```java
public void fetchConfig() {
        mfirebaseRemoteConfig.fetchAndActivate()
                .addOnCompleteListener(this, new OnCompleteListener<Boolean>() {
                    @Override
                    public void onComplete(@NonNull Task<Boolean> task) {
                        if (task.isSuccessful()) {
                            boolean updated = task.getResult();
                            Log.d(TAG, "Config params updated: " + updated);
                            Toast.makeText(MainActivity.this, "Fetch and activate succeeded",
                                    Toast.LENGTH_SHORT).show();
                            applyRetrievedLengthLimit();

                        } else {
                            Toast.makeText(MainActivity.this, "Fetch failed",
                                    Toast.LENGTH_SHORT).show();
                        }
                        applyRetrievedLengthLimit();
                    }
                });
    }
```
* JavaScript function converting some of the text to emoji
```java
const functions = require('firebase-functions'); //cloud functions

// replaces keywords with emoji in the "text" key of messages
// pushed to /messages
exports.emojify =
    functions.database.ref('/messages/{pushId}/text')
    .onWrite((change, context) => {
      
        // Now we begin the emoji transformation
        console.log("emojifying!");

        // Get the value from the 'text' key of the message
        const originalText = change.after.val();
        const emojifiedText = emojifyText(originalText);

        // Return a JavaScript Promise to update the database node
        return change.after.ref.set(emojifiedText);
    });

// Returns text with keywords replaced by emoji
// Replacing with the regular expression /.../ig does a case-insensitive
// search (i flag) for all occurrences (g flag) in the string
function emojifyText(text) {
    var emojifiedText = text;
    emojifiedText = emojifiedText.replace(/\blol\b/ig, "😂");
    emojifiedText = emojifiedText.replace(/\bcat\b/ig, "😸");
    return emojifiedText;
}
```
<img src="https://user-images.githubusercontent.com/73274912/144559610-f6cd57a2-99de-43d4-a07f-4966a52892aa.jpeg" width=45% height=25% alt="LOL to 😂"> 

