---
title: Adding basic sign-in code for mobile
description: Adding code to a mobile game to enable basic sign-in to Xbox Live.
ms.date: 03/14/2019
ms.topic: article
keywords: xbox live, xbox, games, mobile, sign-in
ms.localizationpriority: medium
---

# Adding basic sign-in code for mobile

Adding basic sign-in code to your game enables the user to do basic sign-in to Xbox Live on the user's device.

This article is for Managed Partners, rather than the Creators Program.


## Prerequisites

Before adding code to your game to do basic sign-in into the Xbox Live services, you must do the following:

1. Create a new game at Partner Center and note the game's Title ID, SCID, Sandbox ID, and Client ID.

2. Add the Xbox Live SDK to an IDE that targets Android or iOS.

See [Getting Started](../live-getstarted-nav.md).

Do the following steps in the order shown.
For example, you must initialize XAL before initializing XSAPI.


## Initialize Xbox Authentication Library (XAL)

XAL has two sets of arguments to pass in:
* `XalPlatformArgs` defines arguments that are needed in order to display the Sign-In window to your game.
* `XalInitArgs` defines arguments that are needed in order to link XAL to your specified game from Partner Center.

1. Add the following `XalInit` function.

```cpp
HRESULT XalInit()
{
    // Trace Debugger must be set before any XAL calls
    HCSettingsSetTraceLevel(HCTraceLevel::Verbose);
    HCTraceSetTraceToDebugger(true);
    
    std::string clientId = ; // TODO: Add your Client ID here. Make sure Client ID is all lowercase!
    std::string redirUri = "ms-xal-" + clientId + "://auth";
    
    XalPlatformArgs xalPlatformArgs = {};
    xalPlatformArgs.redirectUri = redirUri.c_str();
#if ANDROID
    xalPlatformArgs.javaVM      = getJavaVM();
    xalPlatformArgs.appContext  = getAppContext();
#endif

    XalInitArgs xalInitArgs = {};
    xalInitArgs.clientId     = clientId.c_str();
    xalInitArgs.titleId      = ; // TODO: Add your Title ID here.
    xalInitArgs.sandbox      = ; // TODO: Add your Sandbox here.
    xalInitArgs.platformArgs = &xalPlatformArgs;

    return XalInitialize(xalInitArgs, nullptr);
}
```


## Initialize Xbox Services API (XSAPI)

Set up your project to call the Xbox Live sign-in API, as described below.

Initializing Xbox Live requires a set of arguments to be passed in.

1. Add the following `XsapiInit` function.

```cpp
HRESULT XsapiInit()
{
    XblInitArgs args = {};
    args.scid = ; // TODO: Add your SCID here.
#if ANDROID
    args.javaVM = getJavaVM();
    args.applicationContext = getAppContext();
#end

    return XblInitialize(&args);
}
```


## Sign-in silently

Now that Xbox Live is initialized, set up the Sign-In and Sign-Out functionality, as follows.

Start by adding in functionality for signing-in silently.

1. Add the following `XAL_TrySignInUserSilently` function, which should be called when your game starts up, to auto sign-in the previously logged-in user.
   This function wraps the async call `XalTryAddDefaultUserSilentlyAsync`.

[!INCLUDE [Identity_TrySignInUserSilently](../../code/snippets/Identity_TrySignInUserSilently.md)]

When the `XAsyncBlock` returns from calling the server, it will run the callback function.

2. Add the following `XAL_TrySignInUserSilently_Callback` callback function, which grabs the result from the server.
After grabbing the result, pass the result to gameplay.

[!INCLUDE [Identity_TrySignInUserSilently_Callback](../../code/snippets/Identity_TrySignInUserSilently_Callback.md)]

3. Add the following code to handle sign-in gameplay.
   This code will try to create an `XblContext` from the `XalUser`.
   If an `XblContext` is created, the user has properly signed in.

[!INCLUDE [Identity_Gameplay_SignInUser](../../code/snippets/Identity_Gameplay_SignInUser.md)]


## Sign-in with UI

If sign-in silently fails, then the user will need to sign-in using XAL's web view UI.

1. Just like with "Sign-In Silently", create a `XAL_TrySignInUserWithUI` wrapper function that calls the async function `XalAddUserWithUiAsync`:

[!INCLUDE [Identity_TrySignInUserWithUI](../../code/snippets/Identity_TrySignInUserWithUI.md)]

2. Add the following `XAL_TrySignInUserWithUI_Callback` callback function, to grab the result from the server to pass on to gameplay:

[!INCLUDE [Identity_TrySignInUserWithUI_Callback](../../code/snippets/Identity_TrySignInUserWithUI_Callback.md)]

3. Add the following code to handle sign-in gameplay.
   This code will try to create an `XblContext` from the `XalUser`.
   If an `XblContext` is created, the user has properly signed in.

[!INCLUDE [Identity_Gameplay_SignInUser](../../code/snippets/Identity_Gameplay_SignInUser.md)]


## Sign-out

Now that sign-in is taken care of, implement sign-out.

1. Add the following `XAL_TrySignOutUser` function, which wraps the async function `XalSignOutUserAsync`:

[!INCLUDE [Identity_TrySignOutUser](../../code/snippets/Identity_TrySignOutUser.md)]

2. Add the following `XAL_TrySignOutUser_Callback` callback function, which grabs the `XAsyncGetStatus` result:

[!INCLUDE [Identity_TrySignOutUser_Callback](../../code/snippets/Identity_TrySignOutUser_Callback.md)]

3. Add the following code to handle sign-out gameplay.

[!INCLUDE [Identity_Gameplay_SignOutUser](../../code/snippets/Identity_Gameplay_SignOutUser.md)]


## Cleanup

Now that everything is implemented, clean it up when your game closes, as follows.

1. XAL doesn't require any cleanup, however, you will need to close your `XalUserHandle` and `XblContextHandle`; add the following:

```cpp
if (m_xblContext)
{
    XalUserHandle user = nullptr;
    HRESULT hr = XblContextGetUser(m_xblContext, &user);

    if (SUCCEEDED(hr)) { XalUserCloseHandle(user); }

    XblContextCloseHandle(m_xblContext);
}
```

2. When your game closes, cleanup Xbox Live, by adding the following:

```cpp
XblCleanup();
```

You have finished adding the basic sign-in code.


## Test basic sign-in

Test that the basic sign-in code works properly, as follows.

1. When your game starts for the first time, make sure that you're calling your "Sign-In Silently" code.

   This code fails with the result "E_XAL_UIREQUIRED", since there hasn't been a user signed-in before.

2. Call "Sign-In with UI".

   A webview window appears, displaying the Xbox Live Sign-In portal.

   ![Xbox Live Sign-In portal](live-getting-xsapi-to-sign-in-images/xboxlive-signin-window.png)

3. Sign-in by using the Xbox Live Sign-In portal.

   `XalAddUserWithUiResult` returns "S_OK".
   Thus your sign-in code will create an `XblContext`, which confirms that your user has properly signed in to your game on Xbox Live.

4. Close your game without signing-out, and then re-open your game.

   This time, your "Sign-In Silently" code succeeds, and automatically signs-in the user.

5. Call your Sign-Out code.
   Make sure you're closing your `XalUserHandle` and `XblContextHandle`.

   If cleared properly, you should be able to run your game and get the "E_XAL_UIREQUIRED" result from your "Sign-In Silently" code.

Repeat the above steps as needed.

Your game now enables the user to do basic sign-in to Xbox Live.


## Next step

Now that your game enables the user to do basic sign-in to Xbox Live on the device, you are ready to implement any of the Xbox Live features, which are provided through the Xbox Services API (XSAPI).
See [Features](../../features/live-features-nav.md).
