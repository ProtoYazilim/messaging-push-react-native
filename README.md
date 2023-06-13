# Proton Push Notification Developer Guide with React Native - CLI

Following steps should be performed to implement push notifications with Proton messaging platform and React Native.

Instructions below are tested and performed in a MacOS environment with M1 chip. Most of the steps should be similar with other platforms.

## Software Versions (as of 06.2023):

1. ruby: 2.7.6

- `brew install rbenv ruby-build`
- `rbenv install 2.7.6`
- After you create the React Native project with `npx` in 'Setting up the development environment' section below
  - `cd <PROJECT_HOME>`
  - `rbenv local 2.7.6`

2. node: v14.20.1
3. npm: 6.14.17
4. JDK: zulu-11.jdk (As suggested by the react native installation guide)

   node and npm versions may be higher in your platform.

## Setting up the development environment

Follow the instructions on [react native development guide](https://reactnative.dev/docs/environment-setup) to have a running development environment. Make sure you follow the instructions for your operating system. Follow the steps for **React Native CLI Quickstart**. If you would like to use **Expo Go**, visit [Proton Push Notification Developer Guide with React Native - Expo]()

Do not proceed until you make sure that you have a running react native app.

### Push Notifications on Android

#### Add Firebase SDK to the Android app

1. Register for a google development account.
2. Open [firebase development console](https://console.firebase.google.com)
3. Create a project (Enable firebase analytics when you are asked)
4. Add app and Select Android as the platform
5. Open <PROJECT_HOME>/<REACT_NATIVE_PROJECT_DIR>/android/app/build.gradle and copy the **applicationId** under **defaultConfig**
6. Enter the **applicationId** at step 5 as **Android package name** for **Register App** section on firebase console
7. Enter a nickname to be shown on firebase console for **App nickname**
8. For **Debug signing certificate SHA-1 (optional)** find the hash with the following commands
   - `cd <PROJECT_HOME>/<REACT_NATIVE_PROJECT_DIR`
   - `brew install pcregrep` or `brew install pcre2grep`
   - Use `pcregrep` or `pcre2grep` after the pipe in command below
   - `keytool -list -v -keystore ./android/app/debug.keystore -storepass android |pcre2grep -o1 "SHA1: (.+)"`
9. Copy the hash in step 8 for **Debug signing certificate SHA-1 (optional)**
10. Register app
    If you get an error at this step, Refresh the page and Go to Project Overview. Click the grid icon to show all apps and make your added project visible.
11. Continue with the Next step or 'See SDK Instructions' in the Project Page when you select your project at step 10 if you faced an error
12. Download google-services.json file and place it under `cd <PROJECT_HOME>/<REACT_NATIVE_PROJECT_DIR/android/app`
13. Click Next and make the changes shown in relevant files to **Add Firebase SDK**. Make sure that you select **Java** if you are asked for Kotlin or Java code.

- For **Module (app-level) Gradle file** change, to add the plugin for google-services, append
  - apply plugin: "com.google.gms.google-services" after other plugins at top of the file

14. Sync your project with Gradle files
15. Click 'Next' and then 'Continue to the console'

#### React Native Changes for Push Notification Support

1. `cd <PROJECT HOME>`
2. Add following packages to package.json with `npm i --save package_name_below`
   - react-native-push-notification
   - @react-native-firebase/app
   - @react-native-firebase/messaging
   - @react-native-community/push-notification-ios
   - @types/react-native-push-notification

#### Sample code to find the firebase device token

- We need to find the firebase token of the device. Most probably you would find and save this token along with user information in your backend store. You would need this token to uniquely specify the device to send the notification.

- You can copy and paste the following code in your App.tsx file for printing the token to the metro bundler terminal.

```
import React, {useEffect} from 'react';
import firebase from '@react-native-firebase/app';
import messaging from '@react-native-firebase/messaging';
const App = () => {
  useEffect(() => {
    messaging()
      .getToken(firebase.app().options.messagingSenderId)
      .then(x => console.log(x))
      .catch(e => console.log(e));
  }, []);
return <></>;
};
export default App;
```

- Once you run the code above, copy the device token from metro bundler terminal for the next steps.

#### Sample code to show a push notification

- You can place the following code in your App.tsx file to test if you can receive a push notification from proton messaging platform.

```
import React, {useEffect} from 'react';
import firebase from '@react-native-firebase/app';
import '@react-native-firebase/messaging';
import PushNotification from 'react-native-push-notification';
import {Platform} from 'react-native';
import PushNotificationIOS from '@react-native-community/push-notification-ios';
import {FirebaseMessagingTypes} from '@react-native-firebase/messaging';
const App = () => {
  if (Platform.OS !== 'ios') {
    PushNotification.createChannel(
      {
        channelId: 'protonPushChannel', // (required)
        channelName: 'protonPushChannel', // (required)
        channelDescription: 'A channel to categorise your notifications', // (optional) default: undefined.
        soundName: 'default', // (optional) See `soundName` parameter of `localNotification` function
        importance: 4, // (optional) default: 4. Int value of the Android notification importance
        vibrate: true, // (optional) default: true. Creates the default vibration patten if true.
      },
      created => console.log(`createChannel returned '${created}'`), // (optional) callback returns whether the channel was created, false means it already existed.
    );
  }

  useEffect(() => {
    firebase.messaging().onMessage(response => {
      console.log(JSON.stringify(response));
      if (Platform.OS !== 'ios') {
        showNotification(response.notification!);
        return;
      }
      PushNotificationIOS.requestPermissions().then(() =>
        showNotification(response.notification!),
      );
    });
  }, []);
  const showNotification = (
    notification: FirebaseMessagingTypes.Notification,
  ) => {
    PushNotification.localNotification({
      title: notification.title,
      message: notification.body!,
      channelId: 'protonPushChannel',
    });
  };
  return <></>;
};
export default App;
```

#### Sending a test push notification from firebase console

- Make sure that you run the Metro bundler terminal (`npx react-native start`)
- Make sure that you run the android application in another terminal (`npx react-native run-android`)

1. Click left menu item on firebase console -> Engage -> Messaging
2. Click on button with title 'Create your first campaign'
3. Select 'Firebase Notification messages' from 'Message type and platform' dialog and click Create
4. In 'Notification' section enter title and text for your test notification
5. On the right of the screen, click on 'Send test message' button
6. Paste the FCM token copied from the metro bundler terminal in previous steps to the text field with placeholder text 'Add an FCM registration token' and click on '+' Sign on the right of the text field.
7. Click on Test button
8. Validate that you receive the test push notification either in the simulator or in your physical device

#### Sending a push notification from proton API

1. Generating the service account role from firebase console
2. Download the service account role key and putting it to vault
3. Restart the push consumer
