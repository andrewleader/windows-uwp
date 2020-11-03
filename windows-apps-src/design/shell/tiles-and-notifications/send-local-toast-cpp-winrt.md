---
Description: Learn how to send a local toast notification from a C++ WinRT app and handle the user clicking the toast.
title: Send a local toast notification from a C++ WinRT app
ms.assetid: E9AB7156-A29E-4ED7-B286-DA4A6E683638
label: Send a local toast notification from a C++ WinRT app
template: detail.hbs
ms.date: 05/19/2017
ms.topic: article
keywords: windows 10, uwp, send toast notifications, notifications, send notifications, toast notifications, how to, quickstart, getting started, code sample, walkthrough, cpp, c++, winrt, win32
ms.localizationpriority: medium
---
# Send a local toast notification from C++ WinRT apps


A toast notification is a message that an app can construct and deliver to the user while they are not currently inside your app. This quickstart walks you through the steps to create, deliver, and display a Windows 10 toast notification using rich content and interactive actions. These quickstart uses local notifications, which are the simplest notification to implement.

> [!IMPORTANT]
> If you're writing a C# app, please see the [C# documentation](send-local-toast.md). For other C++ languages, please see [C++ UWP](send-local-toast-cpp-uwp.md) and [C++ WRL](send-local-toast-desktop-cpp-wrl.md).



## Step 1: Install and configure WinRT

If you aren't using WinRT in your project yet...

1. **Install the [Microsoft.Windows.CppWinRT NuGet package](https://www.nuget.org/packages/Microsoft.Windows.CppWinRT/)**: This package allows you to call Windows 10 APIs from your Win32 C++ app.
1. **Ensure your target Windows SDK is 10.0.17134.0 or greater**: In your project's properties, go to **General** \> **Windows SDK Version**, and select **All Configurations** and **All Platforms**. Ensure that **Windows SDK Version** is set to 10.0.17134.0 (Windows 10, version 1803) or greater (note that the "latest installed" option doesn't work, you have to pick a specific version).
1. **Ensure you've configured `pch.h`.**

### Configuring pch.h

If you currently have a precompiled `framework.h` or `stdafx.h` header, rename it to `pch.h` (and rename the corresponding `cpp` file if you have that too). Find and replace all `#include "framework.h"` (or `#include "stdafx.h"`) with `#include "pch.h"`.

If you don't have a precompiled header, create a new `pch.h` file.

Set project property **C/C++** > **Precompiled Headers** > **Precompiled Header** to *Create (/Yc)*, and **Precompiled Header File** to *pch.h*.

Then, ensure your `pch.h` file includes `#include <unknwn.h>`.

**pch.h**

```cpp
#pragma once
#include <unknwn.h> // Needed for notifications

...
```


## Step 2: Copy compat library code

Copy the [DesktopNotificationManagerCompat.h](https://raw.githubusercontent.com/WindowsNotifications/desktop-toasts/aleader/toolkit-7/CPP-WINRT/DesktopToastsCppWinRtApp/DesktopNotificationManagerCompat.h) and [DesktopNotificationManagerCompat.cpp](https://raw.githubusercontent.com/WindowsNotifications/desktop-toasts/aleader/toolkit-7/CPP-WINRT/DesktopToastsCppWinRtApp/DesktopNotificationManagerCompat.cpp) file from GitHub into your project. The compat library abstracts much of the complexity of desktop notifications. The following instructions require the compat library.


## Step 3: Add namespace declarations

In whichever file you're looking to send notifications from, add the following declarations.

```cpp
#include "pch.h"
#include <winrt/Windows.Data.Xml.Dom.h>
#include <winrt/Windows.UI.Notifications.h>
#include "DesktopNotificationManagerCompat.h"

using namespace winrt;
using namespace Windows::Data::Xml::Dom;
using namespace Windows::UI::Notifications;
```


## Step 4: Register your app

#### [MSIX/sparse](#tab/msix+sparse)

You don't have to do anything! Your app is already registered.

#### [Unpackaged](#tab/unpackaged)

Provide a unique AUMID for your app, along with the display name and icon you would like used on your toasts. If you also distribute your app as MSIX or sparse, feel free to call this method... if running under MSIX/sparse, this method will automatically no-op.

```cpp
DesktopNotificationManagerCompat::Register(L"CompanyName.ProductName", L"Display name", L"C:\\icon.png");
```

---


## Step 5: Send a toast

In Windows 10, your toast notification content is described using an adaptive language that allows great flexibility with how your notification looks. See the [toast content documentation](adaptive-interactive-toasts.md) for more information.

We'll start with a simple text-based notification. Construct the notification content and show the notification!

<img alt="Simple text notification" src="images/send-toast-01.png" width="364"/>

```cpp
// Construct the toast template
XmlDocument doc;
doc.LoadXml(L"<toast>\
    <visual>\
        <binding template=\"ToastGeneric\">\
            <text></text>\
            <text></text>\
        </binding>\
    </visual>\
</toast>");

// Populate with text and values
doc.SelectSingleNode(L"//text[1]").InnerText(L"Andrew sent you a picture");
doc.SelectSingleNode(L"//text[2]").InnerText(L"Check this out, Happy Canyon in Utah!");

// Construct the notification
ToastNotification notif{ doc };

// And send it!
DesktopNotificationManagerCompat::CreateToastNotifier().Show(notif);
```


## Step 6: Handling activation

#### [MSIX](#tab/msix)

First, in your **Package.appxmanifest**, add:

1. Declaration for **xmlns:com**
1. Declaration for **xmlns:desktop**
1. In the **IgnorableNamespaces** attribute, **com** and **desktop**
1. **desktop:Extension** for **windows.toastNotificationActivation** to declare your toast activator CLSID (using a new GUID of your choice).
1. MSIX only: **com:Extension** for the COM activator using the GUID from step #4. Be sure to include the `Arguments="-ToastActivated"` so that you know your launch was from a notification

**Package.appxmanifest**

```xml
<!--Add these namespaces-->
<Package
  ...
  xmlns:com="http://schemas.microsoft.com/appx/manifest/com/windows10"
  xmlns:desktop="http://schemas.microsoft.com/appx/manifest/desktop/windows10"
  IgnorableNamespaces="... com desktop">
  ...
  <Applications>
    <Application>
      ...
      <Extensions>

        <!--Specify which CLSID to activate when toast clicked-->
        <desktop:Extension Category="windows.toastNotificationActivation">
          <desktop:ToastNotificationActivation ToastActivatorCLSID="replaced-with-your-guid-C173E6ADF0C3" /> 
        </desktop:Extension>

        <!--Register COM CLSID LocalServer32 registry key-->
        <com:Extension Category="windows.comServer">
          <com:ComServer>
            <com:ExeServer Executable="YourProject\YourProject.exe" Arguments="-ToastActivated" DisplayName="Toast activator">
              <com:Class Id="replaced-with-your-guid-C173E6ADF0C3" DisplayName="Toast activator"/>
            </com:ExeServer>
          </com:ComServer>
        </com:Extension>

      </Extensions>
    </Application>
  </Applications>
 </Package>
```

Then, **in your app's startup code**, **after** calling Register, subscribe to the OnActivated event.

```cpp
// Listen to notification activation
DesktopNotificationManagerCompat::OnActivated([](DesktopNotificationActivatedEventArgsCompat e)
{
    // Obtain the arguments from the notification
    std::wstring args = e.Argument();

    // Obtain any user input (text boxes, menu selections) from the notification
    auto userInput = e.UserInput();

    // TODO: Show the corresponding content
});
```

When the user clicks any of your notifications (or a button on the notification), the following will happen...

**If your app is currently running**...

1. The **DesktopNotificationManagerCompat::OnActivated** event will be invoked on a background thread.

**If your app is currently closed**...

1. Your app's EXE will be launched with a command line argument equal to `TOAST_ACTIVATED_LAUNCH_ARG` to indicate the process was started due to a modern activation and that the event handler will soon be invoked.
1. Then, the 
 **DesktopNotificationManagerCompat::OnActivated** event will be invoked on a background thread.


#### [Sparse/unpackaged](#tab/sparse+unpackaged)

When the user clicks any of your notifications (or a button on the notification), the following will happen...

**If your app is currently running**...

1. The **DesktopNotificationManagerCompat::OnActivated** event will be invoked on a background thread.

**If your app is currently closed**...

1. Your app's EXE will be launched with a command line argument equal to `TOAST_ACTIVATED_LAUNCH_ARG` to indicate the process was started due to a modern activation and that the event handler will soon be invoked.
1. Then, the 
 **DesktopNotificationManagerCompat::OnActivated** event will be invoked on a background thread.

**In your app's startup code**, **after** calling Register, subscribe to the OnActivated event.

```cpp
// Listen to notification activation
DesktopNotificationManagerCompat::OnActivated([](DesktopNotificationActivatedEventArgsCompat e)
{
    // Obtain the arguments from the notification
    std::wstring args = e.Argument();

    // Obtain any user input (text boxes, menu selections) from the notification
    auto userInput = e.UserInput();

    // TODO: Show the corresponding content
});
```

---


## Step 7: Handling uninstallation

#### [MSIX](#tab/msix)

You don't need to do anything! When MSIX apps are uninstalled, all notifications and any other related resources are automatically cleaned up.

#### [Sparse/unpackaged](#tab/sparse+unpackaged)

If your app has an uninstaller, in your uninstaller you should call `DesktopNotificationManagerCompat::Uninstall();`. If your app is a "portable app" without an installer, consider calling this method upon app exit unless you have notifications that are meant to persist after your app is closed.

The uninstall method will clean up any scheduled and current notifications, remove any associated registry values, and remove any associated temporary files that were created by the library.

---


## Activation in depth

The first step in making your notifications actionable is to add some launch args to your notification, so that your app can know what to launch when the user clicks the notification (in this case, we're including some information that later tells us we should open a conversation, and we know which specific conversation to open).

```cpp
// Construct the toast template
XmlDocument doc;
doc.LoadXml(L"<toast>\
    <visual>\
        <binding template=\"ToastGeneric\">\
            <text></text>\
            <text></text>\
        </binding>\
    </visual>\
</toast>");

// Set launch arguments
doc.DocumentElement().SetAttribute(L"launch", L"action=viewConversation&conversationId=9813");
```

Then, in your OnActivated handler...

```cpp
DesktopNotificationManagerCompat::OnActivated([](DesktopNotificationActivatedEventArgsCompat e)
    {
        if (e.Argument()._Starts_with(L"action=like"))
        {
            sendBasicToast(L"Sent like!");

            if (!_hasStarted)
            {
                exit(0);
            }
        }

        else if (e.Argument()._Starts_with(L"action=reply"))    
        {
            std::wstring msg = e.UserInput().Lookup(L"tbReply").c_str();

            sendBasicToast(L"Sent reply! Reply: " + msg);

            if (!_hasStarted)
            {
                exit(0);
            }
        }

        else
        {
            if (!_hasStarted)
            {
                if (e.Argument()._Starts_with(L"action=viewConversation"))
                {
                    std::cout << "Launched from toast, opening the conversation!\n\n";
                }

                start();
            }
            else
            {
                showWindow();

                if (e.Argument()._Starts_with(L"action=viewConversation"))
                {
                    std::cout << "\n\nOpening the conversation!\n\nEnter a number to continue: ";
                }
                else
                {
                    std::wcout << L"\n\nToast activated!\n - Argument: " + e.Argument() + L"\n\nEnter a number to continue: ";
                }
            }
        }
    });

void showWindow()
{
    HWND hwnd = GetConsoleWindow();
    WINDOWPLACEMENT place = { sizeof(WINDOWPLACEMENT) };
    GetWindowPlacement(hwnd, &place);
    switch (place.showCmd)
    {
    case SW_SHOWMAXIMIZED:
        ShowWindow(hwnd, SW_SHOWMAXIMIZED);
        break;
    case SW_SHOWMINIMIZED:
        ShowWindow(hwnd, SW_RESTORE);
        break;
    default:
        ShowWindow(hwnd, SW_NORMAL);
        break;
    }
    SetWindowPos(0, HWND_TOP, 0, 0, 0, 0, SWP_SHOWWINDOW | SWP_NOSIZE | SWP_NOMOVE);
    SetForegroundWindow(hwnd);
}
```

To properly support being launched while your app is closed, in your main function, you'll want to determine whether you're being launched from a toast or not. If launched from a toast, you'll receive an argument equal to `TOAST_ACTIVATED_LAUNCH_ARG`. In that case, you should stop performing any normal launch activation, and allow your **OnActivated** code to handle launching.

```cpp
int main(int argc, char* argv[])
{
    // Register and listen for toast activation
    DesktopNotificationManagerCompat::Register(L"Microsoft.SampleCppWinRtApp", L"Sample C++ WinRT App", L"C:\\MyIcon.png");
    DesktopNotificationManagerCompat::OnActivated(ToastActivated);

    if (argc >= 2 && strcmp(argv[1], TOAST_ACTIVATED_LAUNCH_ARG) == 0)
    {
        // Was launched from a toast, OnActivated will be called and we'll decide whether to start the app or exit
        std::cin.ignore();   
    }

    else
    {
        start();
    }
}
```


## Adding images

You can add rich content to notifications. We'll add an inline image and a profile (app logo override) image.

> [!IMPORTANT]
> Http images are only supported in MSIX/sparse apps that have the internet capability in their manifest. Win32 non-MSIX/sparse apps do not support http images; you must download the image to your local app data and reference it locally.

<img alt="Toast with images" src="images/send-toast-02.png" width="364"/>

```cpp
// Construct the toast template
XmlDocument doc;
doc.LoadXml(L"<toast>\
    <visual>\
        <binding template=\"ToastGeneric\">\
            <text></text>\
            <text></text>\
            <image/>
        </binding>\
    </visual>\
</toast>");

// Set image URL
doc.SelectSingleNode(L"//image").as<XmlElement>().SetAttribute(L"src", L"https://picsum.photos/360/202?image=883");

...
```



## Adding buttons and inputs

You can add buttons and inputs to make your notifications interactive. Buttons can launch your foreground app, a protocol, or your background task. We'll add a reply text box, a "Like" button, and a "View" button that opens the image.

<img alt="Toast with images and buttons" src="images/send-toast-03.png" width="364"/>

```cpp
// Construct the toast template
XmlDocument doc;
doc.LoadXml(L"<toast>\
    <visual>\
        <binding template=\"ToastGeneric\">\
            <text></text>\
            <text></text>\
        </binding>\
    </visual>\
    <actions>\
        <input\
            id=\"tbReply\"\
            type=\"text\"\
            placeHolderContent=\"Type a reply\"/>\
        <action\
            content=\"Reply\"\
            activationType=\"background\"/>\
        <action\
            content=\"Like\"\
            activationType=\"background\"/>\
        <action\
            content=\"View\"\
            activationType=\"background\"/>\
    </actions>\
</toast>");

doc.SelectSingleNode(L"//action[1]").as<XmlElement>().SetAttribute(L"arguments", L"action=reply&conversationId=9813");
doc.SelectSingleNode(L"//action[2]").as<XmlElement>().SetAttribute(L"arguments", L"action=like&conversationId=9813");
doc.SelectSingleNode(L"//action[3]").as<XmlElement>().SetAttribute(L"arguments", L"action=viewImage&imageUrl=https://picsum.photos/364/202?image=883");

...
```

The activation of foreground buttons are handled in the same way as the main toast body (your OnActivated will be called).



## Handling background activation

For Win32 applications, background activations are handled the same as foreground activations (your **OnActivated** event handler will be triggered). You can choose to not show any UI and close your app after handling activation.




## Provide a primary key for your toast

If you want to programmatically remove or replace the notification you send, you need to use the Tag property (and optionally the Group property) to provide a primary key for your notification. Then, you can use this primary key in the future to remove or replace the notification.

To see more details on replacing/removing already delivered toast notifications, please see [Quickstart: Managing toast notifications in action center (XAML)](https://docs.microsoft.com/previous-versions/windows/apps/dn631260(v=win.10)).

Tag and Group combined act as a composite primary key. Group is the more generic identifier, where you can assign groups like "wallPosts", "messages", "friendRequests", etc. And then Tag should uniquely identify the notification itself from within the group. By using a generic group, you can then remove all notifications from that group by using the [RemoveGroup API](https://docs.microsoft.com/uwp/api/Windows.UI.Notifications.ToastNotificationHistory#Windows_UI_Notifications_ToastNotificationHistory_RemoveGroup_System_String_).

```cpp
// Construct the notification
ToastNotification notif{ doc };

// Set tag/group
notif.Tag(L"18345");
notif.Group(L"wallPosts");

// And send it!
DesktopNotificationManagerCompat::CreateToastNotifier().Show(notif);
```



## Clear your notifications

Apps are responsible for removing and clearing their own notifications. When your app is launched, we do NOT automatically clear your notifications.

Windows will only automatically remove a notification if the user explicitly clicks the notification.

Here's an example of what a messaging app should do…

1. User receives multiple toasts about new messages in a conversation
2. User taps one of those toasts to open the conversation
3. The app opens the conversation and then clears all toasts for that conversation (by using [RemoveGroup](https://docs.microsoft.com/uwp/api/Windows.UI.Notifications.ToastNotificationHistory#Windows_UI_Notifications_ToastNotificationHistory_RemoveGroup_System_String_) on the app-supplied group for that conversation)
4. User's Action Center now properly reflects the notification state, since there are no stale notifications for that conversation left in Action Center.

To learn about clearing all notifications or removing specific notifications, see [Quickstart: Managing toast notifications in action center (XAML)](https://docs.microsoft.com/previous-versions/windows/apps/dn631260(v=win.10)).

```cpp
DesktopNotificationManagerCompat::History().Clear();
```



## Resources

* [Full code sample on GitHub](https://github.com/WindowsNotifications/quickstart-sending-local-toast-win10)
* [Toast content documentation](adaptive-interactive-toasts.md)
* [ToastNotification Class](/uwp/api/Windows.UI.Notifications.ToastNotification)
* [ToastNotificationActivatedEventArgs Class](/uwp/api/Windows.ApplicationModel.Activation.ToastNotificationActivatedEventArgs)