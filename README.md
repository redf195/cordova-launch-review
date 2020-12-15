Cordova Launch Review plugin [![Latest Stable Version](https://img.shields.io/npm/v/cordova-launch-review.svg)](https://www.npmjs.com/package/cordova-launch-review) [![Total Downloads](https://img.shields.io/npm/dt/cordova-launch-review.svg)](https://npm-stat.com/charts.html?package=cordova-launch-review)
============================

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [Overview](#overview)
- [Installation](#installation)
  - [Using the Cordova/Phonegap CLI](#using-the-cordovaphonegap-cli)
  - [PhoneGap Build](#phonegap-build)
- [Usage](#usage)
  - [launch()](#launch)
  - [rating()](#rating)
  - [isRatingSupported()](#isratingsupported)
- [Example project](#example-project)
- [License](#license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


# Overview

Cordova/Phonegap plugin for iOS and Android to assist in leaving user reviews/ratings in the App Stores.

- Launches the platform's App Store page for the current app in order for the user to leave a review.
- On iOS 10.3 and above, invokes the [native in-app rating dialog](https://developer.apple.com/documentation/storekit/skstorereviewcontroller/2851536-requestreview) which allows a user to rate your app without needing to open the App Store.
- On Android, invokes the [native in-app review dialog](https://developer.android.com/guide/playcore/in-app-review)

The plugin published to [npm](https://www.npmjs.com/package/cordova-launch-review) as `cordova-launch-review`

<!-- DONATE -->
[![donate](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG_global.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=ZRD3W47HQ3EMJ&source=url)

I dedicate a considerable amount of my free time to developing and maintaining this Cordova plugin, along with my other Open Source software.
To help ensure this plugin is kept updated, new features are added and bugfixes are implemented quickly, please donate a couple of dollars (or a little more if you can stretch) as this will help me to afford to dedicate time to its maintenance. Please consider donating if you're using this plugin in an app that makes you money, if you're being paid to make the app, if you're asking for new features or priority bug fixes.
<!-- END DONATE -->


# Installation

## Using the Cordova/Phonegap [CLI](http://docs.phonegap.com/en/edge/guide_cli_index.md.html)

    $ cordova plugin add cordova-launch-review

# Usage

The plugin is exposed via the `LaunchReview` global namespace.

## launch()

Platforms: Android and iOS

Launches the App Store page for the current app in order for the user to leave a review.

- On Android, opens the app's in the Play Store where the user can leave a review by pressing the stars to give a rating.
- On iOS, opens the app's page in the App Store and automatically opens the dialog for the user to leave a rating or review.

    LaunchReview.launch(success, error, appId);

### Parameters

- {function} success - (optional) function to execute on successfully launching store app.
- {function} error - (optional) function to execute on failure to launch store app. Will be passed a single argument which is the error message string.
- {string} appID - (optional) the platform-specific app ID to use to open the page in the store app
    - If not specified, the plugin will use the app ID for the app in which the plugin is contained.
    - On Android this is the full package name of the app. For example, for Google Maps: `com.google.android.apps.maps`
    - On iOS this is the Apple ID of the app. For example, for Google Maps: `585027354`


### Simple usage

    LaunchReview.launch();
    
### Advanced usage

    var appId, platform = device.platform.toLowerCase();

    switch(platform){
        case "ios":
            appId = "585027354";
            break;
        case "android":
            appId = "com.google.android.apps.maps";
            break;
    }

    LaunchReview.launch(function(){
        console.log("Successfully launched store app");
    },function(err){
        console.log("Error launching store app: " + err);
    }, appId);

## rating()

Platforms: Android and iOS

- On iOS 10.3 and above, invokes the [native in-app rating dialog](https://developer.apple.com/documentation/storekit/skstorereviewcontroller/2851536-requestreview) which allows a user to rate your app without needing to open the App Store.
- On Android, invokes the [native in-app review dialog](https://developer.android.com/guide/playcore/in-app-review) which allows a user to rate/review your app without needing to open the Play Store.


    LaunchReview.rating(success, error);
      
**iOS notes** 
- The Rating dialog will not be displayed every time `LaunchReview.rating()` is called - iOS limits the frequency with which it can be called ([see here](https://daringfireball.net/2017/01/new_app_store_review_features)).
- The Rating dialog may take several seconds to appear while iOS queries the Apple servers before displaying the dialog.
    
**Android notes**
- Be sure to follow the [Android guidelines on when to request an in-app review](https://developer.android.com/guide/playcore/in-app-review#when-to-request)
- Google Play [enforces a quota](https://developer.android.com/guide/playcore/in-app-review#quotas) on how often a user can be shown the review dialog which means the dialog might not display after you call this method. 
- The user must first rate your app in the native dialog before being shown the review textarea input.
- Neither `success` or `error` will not be called if dialog was not shown due to rate limiting, etc.
    - If you need to know the outcome it's therefore best to set timeout after which you assume the dialog has failed to show - see the example project for an example of this.
- If you're having problems with getting the native rating dialog to appear, make sure you've followed all the steps in the [Android guidelines on testing in-app reviews](https://developer.android.com/guide/playcore/in-app-review/test).
  

### Parameters

- {function} success - (optional) function to execute on successfully requesting (note: this does not guarantee it will be displayed) the launch rating dialog
    - iOS 
        - Will be passed a single string argument which indicates the result: `requested`
        - Will be called the first time after `LaunchReview.rating()` is called and the request to show the dialog is successful with value `requested`.
- {function} error - (optional) function to execute on failure to launch rating dialog. 
    - Will be passed a single argument which is the error message string.


### Simple usage

    LaunchReview.rating();
    
### Advanced usage

    //max time to wait for rating dialog to display on iOS
    var MAX_DIALOG_WAIT_TIME_IOS = 5*1000; 
    
    //max time to wait for rating dialog to display on Android and be submitted by user
    var MAX_DIALOG_WAIT_TIME_ANDROID = 60*1000; 
    
    var ratingTimerId;
    
    function ratingDialogNotShown(){
        var msg;
        if(cordova.platformId === "android"){
            msg = "Rating dialog outcome not received (after " + MAX_DIALOG_WAIT_TIME_ANDROID + "ms)";
        }else if(cordova.platformId === "ios"){
            msg = "Rating dialog was not shown (after " + MAX_DIALOG_WAIT_TIME_IOS + "ms)";
        }
        console.warn(msg);
    }

    function rating(){
        if(cordova.platformId === "android"){
            ratingTimerId = setTimeout(ratingDialogNotShown, MAX_DIALOG_WAIT_TIME_ANDROID);
        }
    
        LaunchReview.rating(function(status){
            if(status === "requested"){
                if(cordova.platformId === "android"){
                    console.log("Displayed rating dialog");
                    clearTimeout(ratingTimerId);
                }else if(cordova.platformId === "ios"){
                    console.log("Requested rating dialog");
                    ratingTimerId = setTimeout(ratingDialogNotShown, MAX_DIALOG_WAIT_TIME_IOS);
                }
            }else if(status === "shown"){
                console.log("Rating dialog displayed");
                clearTimeout(ratingTimerId);
            }else if(status === "dismissed"){
                console.log("Rating dialog dismissed");
                clearTimeout(ratingTimerId);
            }
        }, function (err){
            console.error("Error launching rating dialog: " + err);
            clearTimeout(ratingTimerId);
        });
    }
    
    rating(); // app invokes eg. via UI action
   

## isRatingSupported()

Platforms: Android and iOS

Indicates if the current platform/version supports in-app ratings dialog, i.e. calling `LaunchReview.rating()`.
Will return `true` if current platform is Android or iOS 10.3+.

    var isSupported = LaunchReview.isRatingSupported();

### Example usage

    if(LaunchReview.isRatingSupported()){
        LaunchReview.rating();
    }else{
        LaunchReview.launch();
    }

# Example project

An example project illustrating use of this plugin can be found here: [https://github.com/dpa99c/cordova-launch-review-example](https://github.com/dpa99c/cordova-launch-review-example)


# License
================

The MIT License

Copyright (c) 2015-2020 Working Edge Ltd.

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
