Zoom Service
===============

This project tries to make integrations with the [Zoom SDK](https://zoom.us/developer) on Swift easier by providing all the properly configured functions to the user to use with their project.

1. [Installation](#installation)
1. [Usage](#usage)

# Installation

## Basics
**Download the `ZoomService.swift` file and add it into your app manually.** ZoomService currently utilizes the [singleton pattern](https://sourcemaking.com/design_patterns/singleton), so it is able to be referenced from anywhere in your app.

Your app must first already have MobileRTC installed. For instructions on how to install MobileRTC into your app, Follow the `Deployment` section from the [Zoom documentation](https://github.com/zoom/zoom-sdk-ios/blob/master/doc/Zoom%20iOS%20MobileRTC.pdf).

## Sample App
There is a sample app with ZoomService integrated available on [GitHub](github.com/george-lim). Feel free to download the app and explore the usage of ZoomService.

# Usage

## MobileRTC Authentication
If you haven't already, [sign up](https://zoom.us/developer) for a Zoom Developer account which provides access to the Zoom REST API, Desktop SDKs, and most importantly, the MobileRTC stacks.

Choose the iOS MobileRTC Stack option once you have signed in, and under the credential tab, copy the APP Key and APP Secret into the `appKey` and `appSecret` into the ZoomService `ZoomAPI` struct. The direct link to the credentials page is found [here](https://zoom.us/developer/ios/credential).

Finally, in your `AppDelegate.swift` file, add the line `ZoomService.sharedInstance.authenticateSDK()` in your `didFinishLaunchingWithOptions` function and you should be set.

**Note: The APP Key and APP Secret are your credentials to authorize your use of the MobileRTC framework in your app. As of now, you still cannot start or join calls with until you are authorized as a Zoom API user or a Zoom member.**

## Zoom Authentication
Zoom provides 2 ways for you to use the MobileRTC Stack functions. You can authenticate yourself as just an API user or as also a Zoom member (which requires the user to sign in to their Zoom account). You **MUST** always first authenticate yourself as an API user. Afterward, you can provide the user with an option to authenticate as a user.

**Zoom API Authentication (Required)**

If your app requires the ability to start or create calls without the user signing in, you first need to obtain the API `userID` and `userToken` values.

Sign in to the Zoom developer site again and navigate to the [REST API playground](https://zoom.us/developer/api/playground) page. Note that your REST credentials in the credentials tab and your MobileRTC credentials from earlier are **NOT** the same. On the playground page, change the `API Endpoint` to `https://api.zoom.us/v1/user/getbyemail` and the `User Email Address` to your email you used to create your Zoom developer account.

Press the `Send API Request` and scroll down until you see the `Response` array. You want to copy the value for `id` and `token` and paste them as the `userID` and `userToken` strings in the ZoomService file respectively.

What we are doing is sending a REST API call through the Zoom developer REST API service to fetch our API ID and token values as if we are an API user. Using this data, we can authorize ourselves in our app as an API user.

**Zoom Member Authentication (Optional)**

To authenticate the user as a Zoom member, you are required to implement your own view flow to retrieve the user's username and password values. After you have received them, call the `ZoomService.sharedInstance.login(email: String, password: String)` function in your code to authenticate the user.

## Starting / Joining calls

**These 2 features are available to both authentication types.** Please note that API authenticated users can only utilize these 2 core functions in the MobileRTC Stack. This means that scheduling calls ahead of time is **IMPOSSIBLE** for an API user without using a separate Zoom REST API call which is outside the scope of MobileRTC. So to clarify, API authenticated users can only start a call (immediately) or join a call.

To join a call, simply execute `ZoomService.sharedInstance.joinMeeting(number: String = ZoomAPI.defaultName, password: String = "")` in your code. Neither parameter is required. The MobileRTC framework will then take over and present a view on top of your app which will launch the call.

Likewise, to start a call, simply execute `ZoomService.sharedInstance.startMeeting(name: String = ZoomAPI.defaultName, number: Int = -1, password: String = "")` in your code. If you specify a meeting number, ZoomService will attempt to create a call for the number specified, and will return an error if someone is already using the meeting number. None of the parameters are required.

## Meeting Scheduling / User Account Management

**The following list of functions are exclusive to users who have signed in to their own Zoom accounts in your app. Zoom's SDK does not allow API users to schedule, edit, or delete meetings. To accomplish this, refer to Zoom's REST API usage.** 

`func scheduleMeeting(topic: String, startTime: Date, timeZone: TimeZone = NSTimeZone.local, durationInMinutes: TimeInterval)` - schedules a meeting ahead of time.

**Note: the following functions require the user to have permission to edit the specified meeting. Otherwise, the functions will return an error.**

`func editMeeting(_ zoomMeeting: ZoomMeeting, topic: String? = nil, startTime: Date? = nil, timeZone: TimeZone? = nil, durationInMinutes: TimeInterval? = nil)` - edits an existing Zoom meeting by ZoomMeeting reference.

`func editMeeting(number: Int, topic: String? = nil, startTime: Date? = nil, timeZone: TimeZone? = nil, durationInMinutes: TimeInterval? = nil)` - edits an existing Zoom meeting by meeting number.

`func deleteMeeting(_ zoomMeeting: ZoomMeeting)` - deletes an existing Zoom meeting by ZoomMeeting reference.

`func deleteMeeting(number: Int)` - deletes an existing Zoom meeting by meeting number.