---
title: Permissions
sourceCodeUrl: 'https://github.com/expo/expo/tree/sdk-36/packages/expo-permissions'
---

import InstallSection from '~/components/plugins/InstallSection';
import PlatformsSection from '~/components/plugins/PlatformsSection';

When you are creating an app that requires access to potentially sensitive information on a user's device, such as their location or contacts, you need to ask for the user's permission first. After they granted the permission(s), you can access the data. The `expo-permissions` module makes that possible.

Please read the [permissions on iOS](#permissions-on-ios) and [permissions on Android](#permissions-on-android) before deploying your app to the stores. If you don't configure or explain the permissions properly **it may result in your app getting rejected or pulled from the stores**. Read more about deploying to the stores in the [App Store Deployment Guide](../../distribution/app-stores/#system-permissions-dialogs-on-ios).

<PlatformsSection android emulator ios simulator web />

## Installation

<InstallSection packageName="expo-permissions" />

### Permissions on iOS

To request and use permissions on iOS, you have to describe why the permissions are required. You can do that by defining `ios.infoPlist` property in your [`app.json` manifest](../../workflow/configuration/#ios) in the managed workflow. When using the bare workflow, you have to edit the `infoPlist` file directly.

See the [`Permission types`](#permission-types) below to learn about what `infoPlist` property you need to define for each permission. You can find the full list of available properties in [Apple's InfoPlistKeyReference](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW1). Apple also documented the basic guidelines about the structure of the message in the [Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/ios/app-architecture/requesting-permission/).

> **Note:** apps using permissions without descriptions _may be rejected from the App Store_. (see the [App Store Deployment Guide](../../distribution/app-stores/#system-permissions-dialogs-on-ios))

### Permissions on Android

To use permissions on Android, they have to be defined in your `AndroidManifest.xml` first. Some Expo and React Native modules imply these permissions by default. If you use `expo-location`, for example, both the `ACCESS_COARSE_LOCATION` and `ACCESS_FINE_LOCATION` are implied and added to your `AndroidManifest.xml` when building a standalone app.

You can limit these permissions by defining `android.permissions` property in your [`app.json` manifest](../workflow/configuration/#android) in the managed workflow. Only the minimum required permissions for Expo and the permissions listed in the `android.permissions` array are added. When using the bare workflow, you have to [blacklist permissions in your `AndroidManifest.xml`](#excluding-android-permissions-in-bare-workflow) manually.

See the [`Permission types`](#permission-types) below to learn about which Android permissions are added. You can find a full list of all available permissions in the [Android Manifest.permissions reference](https://developer.android.com/reference/android/Manifest.permission).

> **Note:** apps using dangerous or signature permissions without valid reasons _may be rejected by Google_. Make sure you follow the [Android permissions best practices](https://developer.android.com/training/permissions/usage-notes) when submitting your app.

> **Note:** by default, the permissions implied by the modules you installed are added to the `AndroidManifest.xml`. To exclude permissions, you have to define the `android.permissions` manifest property or [blacklist them in the bare workflow](#excluding-android-permissions-in-bare-workflow).

## Usage

### Manually testing permissions

Often you want to be able to test what happens when you reject a permission to ensure that it has the desired behavior. An operating-system level restriction on both iOS and Android prohibits an app from asking for the same permission more than once (you can imagine how this could be annoying for the user to be repeatedly prompted for permissions). So in order to test different flows involving permissions, you may need to uninstall and reinstall the Expo app. In the simulator this is as easy as deleting the app and expo-cli will automatically install it again next time you launch the project from it.

### Usage in bare workflow

`expo-permissions` includes the shared infrastructure for handling system permissions, it does not include the code specific to particular permissions. For example, if you want to use the `CAMERA_ROLL` permission, you need to install `expo-image-picker` or `expo-media-library`.

The following table shows you which permissions correspond to which packages.

| Permission type             | Packages                                  |
| --------------------------- | ----------------------------------------- |
| `NOTIFICATIONS`             | `expo-notifications`                      |
| `USER_FACING_NOTIFICATIONS` | `expo-notifications`                      |
| `LOCATION`                  | `expo-location`                           |
| `CAMERA`                    | `expo-camera`, `expo-barcode-scanner`     |
| `AUDIO_RECORDING`           | `expo-av`                                 |
| `CONTACTS`                  | `expo-contacts`                           |
| `CAMERA_ROLL`               | `expo-image-picker`, `expo-media-library` |
| `CALENDAR`                  | `expo-calendar`                           |
| `REMINDERS`                 | `expo-calendar`                           |
| `SYSTEM_BRIGHTNESS`         | `expo-brightness`                         |
| `MOTION`                    | `expo-sensors`                            |

### Excluding Android permissions in bare workflow

When adding Expo and React Native modules to your project, certail Android permissions might be implied. The modules should only add the relevant permissions required to use the module. Sometimes you want to remove some of these permissions. Unfortunately, the `android.permission` manifest property doesn't work in the bare workflow.

When you want to exclude specific permissions from the build, you have to "blacklist" them in your `AndroidManifest.xml`. You can do that with the `tools:node="remove"` attribute on the `<use-permission>` tag.

```xml
<manifest xmlns:tools="http://schemas.android.com/tools">
  <uses-permission tools:node="remove" android:name="android.permission.ACCESS_FINE_LOCATION" />
</manifest>
```

> **Note:** you have to define the `xmlns:tools` attribute on `<manifest>` before you can use the `tools:node` attribute on permissions.


# API

```js
import * as Permissions from 'expo-permissions';
```

## Hooks

### `usePermissions`

```ts
const [permission, askPermission, getPermission] = usePermissions(Permissions.CAMERA, { ... });
```

Get or ask permission for protected functionality within the app. This returns the result for the requested permissions. It also returns additional callback to interact based on user input.

#### Arguments

- **permissionTypes (_Type|Type[]_)** -- One or more types to get or ask permission for. After the permission status is fetched, you can show different UI based on the current status.
- **permissionOptions** -- Optional configuration to change the behavior of the hook.
  - **get (_boolean_)** -- Retrieves the permission status without interacting with the user, `true` by default.
  - **ask (_boolean_)** -- Prompts the user with the requested permission directly. Without using the `askPermission` callback, `false` by default.

#### Returns

- **permission (_[PermissionsResponse](#permissions-response)|undefined_)** -- An object with information about the permissions, including status, expiration, and scope (if applicable).
- **askPermission (_() => void_)** -- A callback to ask the user for permission.
- **getPermission (_() => void_)** -- A callback to get the permission status without interacting with the user.

#### Example: hook

```tsx
function App() {
  const [permission, askForPermission] = usePermissions(Permissions.CAMERA, { ask: true });

  if (!permission || permission.status !== 'granted') {
    return (
      <View>
        <Text>Permission is not granted</Text>
        <Button title="Grant permission" onPress={askForPermission} />
      </View>
    );
  }

  return (
    <View>
      <Camera />
    </View>
  );
}
```

### `Permissions.getAsync(...types)`

Determines whether your app has already been granted access to the provided permissions types.

#### Arguments

- **permissionTypes (_string_)** -- The names of the permissions types.

#### Returns

A `Promise` with a [`PermissionResponse`](#permissions-response) object.

#### Example

```javascript
async function alertIfRemoteNotificationsDisabledAsync() {
  const { status } = await Permissions.getAsync(Permissions.NOTIFICATIONS);
  if (status !== 'granted') {
    alert('Hey! You might want to enable notifications for my app, they are good.');
  }
}

async function checkMultiPermissions() {
  const { status, expires, permissions } = await Permissions.getAsync(
    Permissions.CALENDAR,
    Permissions.CONTACTS
  );
  if (status !== 'granted') {
    alert('Hey! You have not enabled selected permissions');
  }
}
```

### `Permissions.askAsync(...types)`

Prompt the user for types of permissions. If they have already granted access, response will be success.

#### Arguments

- **types (_string_)** -- The names of the permissions types.

#### Returns

A `Promise` resolving to a [`PermissionResponse`](#permissionresponse) -- an object describing the new state of the permissions, after asking the user.

#### Example

```javascript
async function getLocationAsync() {
  // permissions returns only for location permissions on iOS and under certain conditions, see Permissions.LOCATION
  const { status, permissions } = await Permissions.askAsync(Permissions.LOCATION);
  if (status === 'granted') {
    return Location.getCurrentPositionAsync({ enableHighAccuracy: true });
  } else {
    throw new Error('Location permission not granted');
  }
}
```

## Types

### `PermissionResponse`

The permission response is an object describing the current state of the requested permission(s). This response contains the top-level `status`, `granted`, `expires` and `canAskAgain` properties representing the outcome for all individual permissions.

```ts
interface PermissionResponse {
  status: 'granted' | 'undetermined' | 'denied';
  granted: boolean;
  expires: 'never' | number;
  canAskAgain: boolean;
  permissions: { // an object with an entry for each permission requested
    [permissionType: string /* PermissionType */]: PermissionInfo;
  };
}
```

#### PermissionResponse.status

This property is either `granted`, `undetermined` or `denied`, based on the requested permissions. It's reducted using the following rules.

- When one or more permissions are `undetermined`, the status is `undetermined`
- When one or more permissions are `denied`, but none of them are `undetermined`, the status is `denied`
- When all permissions are `granted`, the status is `granted`.

Here are some examples of permission statuses and the top-level status.

```
[granted, granted, granted] => granted
[granted, granted, denied] => denied
[denied, denied, denied] => denied
[granted, granted, undetermined] => undetermined
[granted, denied, undetermined] => undetermined
```

#### PermissionResponse.granted

This property is set to `true` when all requested permission are granted. If one or more are `denied` or `undetermined`, this is set to `false`.

#### PermissionResponse.expires

This property is set to the permission that expires the earliest. When none of the requested permissions expires, it's set to `never`.

#### PermissionResponse.canAskAgain

This property is set to `true` when the app can request the user to grant all requested permissions.

#### PermissionResponse.permissions

This object contains information, per requested permission, using the [`PermissionInfo`](#permissioninfo) type.


### `PermissionInfo`

This object contains information about a single requested permission, it's retuned within the `PermissionResponse` using the `permissions` property. It also may include additional platform-specific info, like the scope of the permission.

```ts
interface PermissionInfo {
  status: 'granted' | 'undetermined' | 'denied';
  granted: boolean;
  expires: 'never' | number;
  canAskAgain: boolean;
  ios?: {
    scope: 'whenInUse' | 'always';
  };
  android?: {
    scope: 'fine' | 'coarse' | 'none';
  };
}
```

## Permission types

### `Permissions.NOTIFICATIONS`

The permission type for user-facing notifications **and** remote push notifications.

> **Note:** On iOS, asking for this permission asks the user not only for permission to register for push/remote notifications, but also for showing notifications as such. At the moment remote notifications will only be received when notifications are permitted to play a sound, change the app badge or be displayed as an alert. As iOS is more detailed when it comes to notifications permissions, this permission status will contain not only `status` and `expires`, but also Boolean values for `allowsSound`, `allowsAlert` and `allowsBadge`.

> **Note:** On iOS, this does not disambiguate `undetermined` from `denied` and so will only ever return `granted` or `undetermined`. This is due to the way the underlying native API is implemented.

> **Note:** Android does not differentiate between permissions for local and remote notifications, so status of permission for `NOTIFICATIONS` should always be the same as the status for `USER_FACING_NOTIFICATIONS`.

### `Permissions.USER_FACING_NOTIFICATIONS`

The permission type for user-facing notifications. This does **not** register your app to receive remote push notifications; see the `NOTIFICATIONS` permission.

> **Note:** iOS provides more detailed permissions, so the permission status will contain not only `status` and `expires`, but also Boolean values for `allowsSound`, `allowsAlert` and `allowsBadge`.

> **Note:** Android does not differentiate between permissions for local and remote notifications, so status of permission for `USER_FACING_NOTIFICATIONS` should always be the same as the status for `NOTIFICATIONS`.

### `Permissions.LOCATION`

The permission type for location access.

<!-- TODO: Permissions.LOCATION issue (search by this phrase) -->

> **Note:** iOS is not working with this permission being not individually, `Permissions.askAsync(Permissions.SOME_PERMISSIONS, Permissions.LOCATION, Permissions.CAMERA, ...)` would throw.
> On iOS ask for this permission type individually.

> **Note (iOS):** In Expo client this permission will always ask the user for permission to access location data while the app is in use.

> **Note (iOS):** iOS provides more detailed permissions, returning `{ status, permissions: { location: { ios } } }` where `ios` which is an object containing: `{ scope: 'whenInUse' | 'always' | 'none' }`
> If you would like to access location data in a standalone app, note that you'll need to provide location usage descriptions in `app.json`. For more information see [Deploying to App Stores guide](../../distribution/app-stores/#system-permissions-dialogs-on-ios).
>
> **What location usage descriptions should I provide?** Due to the design of the location permission API on iOS we aren't able to provide you with methods for asking for `whenInUse` or `always` location usage permission specifically. However, you can customize the behavior by providing the following sets of usage descriptions:
>
> - if you provide only `NSLocationWhenInUseUsageDescription`, your application will only ever ask for location access permission "when in use",
> - if you provide both `NSLocationWhenInUseUsageDescription` and `NSLocationAlwaysAndWhenInUseUsageDescription`, your application will only ask for "when in use" permission on iOS 10, whereas on iOS 11+ it will show a dialog to the user where he'll be able to pick whether he'd like to give your app permission to access location always or only when the app is in use,
> - if you provide all three: `NSLocationWhenInUseUsageDescription`, `NSLocationAlwaysAndWhenInUseUsageDescription` and `NSLocationAlwaysUsageDescription`, your application on iOS 11+ will still show a dialog described above and on iOS 10 it will only ask for "always" location permission.

### `Permissions.CAMERA`

The permission type for photo and video taking.

### `Permissions.AUDIO_RECORDING`

The permission type for audio recording.

### `Permissions.CONTACTS`

The permission type for reading contacts.

### `Permissions.CAMERA_ROLL`

The permission type for reading or writing to the camera roll.

> **Note (iOS):** iOS provides more detailed permissions, returning `{ status, permissions: { cameraRoll: { scope } } }` where `scope` can be:
>
> - `all` if the user granted your app access to the whole photo library
> - `limited` if the user granted your app access only to selected photos (only available on **iOS 14.0+**)
> - `none` if user denied or hasn't yet granted the permission

### `Permissions.CALENDAR`

The permission type for reading or writing to the calendar.

### `Permissions.REMINDERS`

The permission type for reading or writing reminders.
(iOS only, on Android would return `granted` immediately)

### `Permissions.SYSTEM_BRIGHTNESS`

The permissions type for changing brightness of the screen

### `Permissions.MOTION`

The permission for accessing `DeviceMotion` and `DeviceOrientation` in the web browser. This can only be requested from a website using HTTPS (`expo web --https`). This permission cannot be silently retrieved, you can only request it. This permission can only be requested with a user interaction i.e. a button press.
