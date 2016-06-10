# electron-distribution-guide

Guide to distribute Electron apps made simple

## Development-signed Application

This document is intended for development of Mac App Store apps. If for submission to the Mac App Store, please check [Mac App Store Deployment].

> NB: The Team ID used in this document are marked either with `XXXXXXXXXX` or with `YYYYYYYYYY`. Please note the difference: Team ID `XXXXXXXXXX` appears with distribution certificates; Team ID `YYYYYYYYYY`, however, appears with development certificates which is slightly separated from a team environment. See below for a list of possible signing identities.

### Prerequisites

#### General Prerequisites

Please check [Mac General Prerequisites] and make sure to finish them before moving on.

#### Certificates

In order to test your apps on registered devices, you will have to obtain the following certificate from Apple after becoming a registered developer.

- Mac Development: `Mac Developer: * (YYYYYYYYYY)`

> NB: We do not create **installers** for testing on registered devices; basically we don't have certificates to do so.

To verify if all your codesigning certificates, including the one mentioned above, are installed and valid, you may use the following script to do so:

```sh
security find-identity -v
```

A sample output may look like the following:

```
1) ???????????????????????????????????????? "iPhone Distribution: Zhuo Lu (XXXXXXXXXX)"
2) ???????????????????????????????????????? "Mac Developer: Zhuo Lu (YYYYYYYYYY)"
3) ???????????????????????????????????????? "3rd Party Mac Developer Application: Zhuo Lu (XXXXXXXXXX)"
4) ???????????????????????????????????????? "Developer ID Application: Zhuo Lu (XXXXXXXXXX)"
5) ???????????????????????????????????????? "3rd Party Mac Developer Installer: Zhuo Lu (XXXXXXXXXX)"
6) ???????????????????????????????????????? "Developer ID Installer: Zhuo Lu (XXXXXXXXXX)"
7) ???????????????????????????????????????? "iPhone Developer: Zhuo Lu (YYYYYYYYYY)"
  7 valid identities found
```

> NB: Please note here that `Mac Developer` and `iPhone Developer` has a different Team ID associated from what appears to be on other ones. Only the `Mac Developer` certificate is required here in this document though.

##### Create your Certificates

**EITHER** from [Member Center] at Apple Developer:

In `Certificates, Identifiers & Profiles` you may find under `OS X` the `Certificates`, in which request could be made for `Mac Development` certificates. The required certificate could be generated is under `Development > Mac Development`.

After you create the necessary certification, download them and open each so that they are installed in your keychain.

**OR** in Xcode:

Under `Xcode > Preferences (⌘,) > Accounts`, you may add your Apple ID. With your team selected, the `View Details...` in the bottom right could find you the available certificates for creation/download.

#### Provisioning Profiles

> NB: This is new in Electron version 1.1.1 for use of capability around application groups. Like on iOS working with `.mobileprovision`, we here use `.provisionprofile`.

Also make sure to have obtained development profiles from Apple; an example may be named `Mac Development Provisioning Profiles: *`. However, the names may be customized. You may either use a *wildcard* or an *explicit* profile. The file extension should be `.provisionprofile`.

To verify the entitlements of the provisioning profile, you may use the following script to do so:

```sh
security cms -D -i path/to/my/development.provisionprofile
```

A sample output may look like the following with a *wildcard* profile:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>AppIDName</key>
  <string>XXXXXXXXXX.*</string>
  <key>ApplicationIdentifierPrefix</key>
  <array>
  <string>XXXXXXXXXX</string>
  </array>
  <key>CreationDate</key>
  <date>2016-06-09T14:38:49Z</date>
  <key>Platform</key>
  <array>
    <string>OSX</string>
  </array>
  <key>DeveloperCertificates</key>
  <array>
    <data>...</data>
  </array>
  <key>Entitlements</key>
  <dict>
    <key>keychain-access-groups</key>
    <array>
      <string>XXXXXXXXXX.*</string>    
    </array>
    <key>com.apple.application-identifier</key>
    <string>XXXXXXXXXX.net.mintkit.*</string>
    <key>com.apple.developer.team-identifier</key>
    <string>XXXXXXXXXX</string>
  </dict>
  <key>ExpirationDate</key>
  <date>2017-06-09T14:38:49Z</date>
  <key>Name</key>
  <string>Mac Team Provisioning Profile: *</string>
  <key>ProvisionedDevices</key>
  <array>
    <string>????????-????-????-????-????????????</string>
    ...
  </array>
  <key>TeamIdentifier</key>
  <array>
    <string>XXXXXXXXXX</string>
  </array>
  <key>TeamName</key>
  <string>Zhuo Lu</string>
  <key>TimeToLive</key>
  <integer>365</integer>
  <key>UUID</key>
  <string>????????-????-????-????-????????????</string>
  <key>Version</key>
  <integer>1</integer>
</dict>
```

Please note that there should be a key about `ProvisionedDevices` which specifies a list of devices allowed for testing your app. If such entry cannot be found, it is likely that your provisioning profile created is for distribution.

##### Create your Provisioning Profiles

> NB: Usually, Xcode manages the provisioning profiles automatically for you in a project. However, as we are doing this separately from Xcode, some of the modern techniques may not apply very well with commands.

From [Member Center] at Apple Developer:

In `Certificates, Identifiers & Profiles` you may find under `OS X` the `Provisioning Profiles`, in which request could be made for `Mac App Development` provisioning profile. You may need to register at least one device in order to complete the request.

After you create the necessary profiles,

**EITHER** download and save them to somewhere for using later.

**OR** in Xcode:

Under `Xcode > Preferences (⌘,) > Accounts`, with your team selected, the `View Details...` in the bottom right could find you the created provisioning profiles for download. After downloading, you may find those provisioning profiles under `~/Library/MobileDevice/Provisioning Profiles` the files. It is worth noting that files are named after their UUIDs.

You may use the script above to verify the provisioning profiles and to see if one is for development or distribution. Please note the one for development for that it will be used later.

### Code Signing

#### Modify Info.plist

First, please make sure to have `ElectronTeamID` key added in `Info.plist` located within the app contents, which has your Team ID as value:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  ...
  <key>ElectronTeamID</key>
  <string>XXXXXXXXXX</string>
</dict>
</plist>
```

*Please mind replacing `XXXXXXXXXX` with your Team ID, which does not appear to be the same as the one on your personal development signing identity.*

The Team ID generated by Apple could be found in the provisioning profile. Please note there is a difference of it from your personal Team ID on your Mac Developer signing identity; refer to the `Entitlements` part, you may find similar to the following:

```xml
<key>com.apple.application-identifier</key>
<string>XXXXXXXXXX.net.mintkit.*</string>
<key>com.apple.developer.team-identifier</key>
<string>XXXXXXXXXX</string>
```

The team identifier from `com.apple.developer.team-identifier` could be used here as `ElectronTeamID`.

#### Copy Embedded Provisioning Profile

Then, to within the app contents folder where `Info.plist` is placed, copy your provisioning profile file and rename it to `embedded.provisionprofile`. Please again make sure that the provision profile used is for development with registered devices.

#### Create/update Entitlements Files

**EITHER** refer below if setting up from ground start:

In case you have not set up an entitlements file which enables features from Apple like app sandbox and push notifications, please manually create a new file with extension `.entitlememts`.

> NB: Though the entitlements file itself being a property list file, it is recommended to use this extension for that Xcode has slightly better support over this file type.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>com.apple.application-identifier</key>
    <string>XXXXXXXXXX.net.mintkit.app</string>
    <key>com.apple.developer.team-identifier</key>
    <string>XXXXXXXXXX</string>
    <key>com.apple.security.app-sandbox</key>
    <true/>
    <key>com.apple.security.application-groups</key>
    <array>
      <string>XXXXXXXXXX.net.mintkit.app</string>
    </array>
  </dict>
</plist>
```

*Please mind replacing `XXXXXXXXXX` with your Team ID and `net.mintkit.app` with your app bundle identifier, and also there being a period (.) separating the two parts in the applciation identifiers.*

To clarify each of the entitlement keys:

- `com.apple.application-identifier`: The application identifier usually covered by modern versions of Xcode; we have to do this manually though.
- `com.apple.developer.team-identifier`: The Team ID as earlier.
- `com.apple.security.app-sandbox`: Enables App Sandbox.
- `com.apple.security.application-groups`: Allows access to group containers that are shared among multiple apps produced by a single development team, and allows certain additional interprocess communication between the apps.

After adding other required entitlement keys required for you app to run, please save the file as it will be used later. Here we name it after `mas.entitlements`.

Another entitlements file required here is used so that every sub components within your app could inherit the declarations made above. Create this entitlements file with the following content:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>com.apple.security.app-sandbox</key>
    <true/>
    <key>com.apple.security.inherit</key>
    <true/>
  </dict>
</plist>
```

To clarify each of the entitlement keys:

- `com.apple.security.app-sandbox`: Enables App Sandbox.
- `com.apple.security.inherit`: Child process inheritance of the parent’s sandbox.

After adding other required entitlement keys required for you app to run, please save the file as it will be used later. Here we name it after `mas.inherit.entitlements`.

**OR** make sure that the entitlement keys mentioned above are added to your existing entitlements files.

#### Sign your App

Please use the following script to code sign your product:

```sh
#!/bin/bash

# App name (excluding .app)
APP="MyApp"
# Path to the app (ending in .app)
APP_PATH="path/to/MyApp.app"

# Signing identities
# Specify the full identity name to avoid vague matching
APP_KEY="Mac Developer:"

# Path to entitlements files
APP_ENTITLEMENTS="path/to/mas.entitlements"
INHERIT_ENTITLEMENTS="path/to/mas.inherit.entitlements"

# Path to app frameworks
FRAMEWORKS_PATH="$APP_PATH/Contents/Frameworks"

# Sign components from Electron
codesign --sign "$APP_KEY" --force --entitlements "$INHERIT_ENTITLEMENTS" "$FRAMEWORKS_PATH/Electron Framework.framework/Versions/A/Electron Framework"
codesign --sign "$APP_KEY" --force --entitlements "$INHERIT_ENTITLEMENTS" "$FRAMEWORKS_PATH/Electron Framework.framework/Versions/A/Libraries/libffmpeg.dylib"
codesign --sign "$APP_KEY" --force --entitlements "$INHERIT_ENTITLEMENTS" "$FRAMEWORKS_PATH/Electron Framework.framework/Versions/A/Libraries/libnode.dylib"
codesign --sign "$APP_KEY" --force --entitlements "$INHERIT_ENTITLEMENTS" "$FRAMEWORKS_PATH/Electron Framework.framework"
codesign --sign "$APP_KEY" --force --entitlements "$INHERIT_ENTITLEMENTS" "$FRAMEWORKS_PATH/$APP Helper.app/Contents/MacOS/$APP Helper"
codesign --sign "$APP_KEY" --force --entitlements "$INHERIT_ENTITLEMENTS" "$FRAMEWORKS_PATH/$APP Helper.app/"
codesign --sign "$APP_KEY" --force --entitlements "$INHERIT_ENTITLEMENTS" "$FRAMEWORKS_PATH/$APP Helper EH.app/Contents/MacOS/$APP Helper EH"
codesign --sign "$APP_KEY" --force --entitlements "$INHERIT_ENTITLEMENTS" "$FRAMEWORKS_PATH/$APP Helper EH.app/"
codesign --sign "$APP_KEY" --force --entitlements "$INHERIT_ENTITLEMENTS" "$FRAMEWORKS_PATH/$APP Helper NP.app/Contents/MacOS/$APP Helper NP"
codesign --sign "$APP_KEY" --force --entitlements "$INHERIT_ENTITLEMENTS" "$FRAMEWORKS_PATH/$APP Helper NP.app/"
codesign --sign "$APP_KEY" --force --entitlements "$INHERIT_ENTITLEMENTS" "$APP_PATH/Contents/MacOS/$APP"

# May insert signing instructions for additional binaries

# Sign the app
codesign -s "$APP_KEY" --force --entitlements "$APP_ENTITLEMENTS" "$APP_PATH"
```

You may use the following script to validate the signing process:

```sh
codesign --verify --deep --verbose=2 "path/to/MyApp.app"
```

An example output may be similar to the following:

```
--prepared:/Users/zhuolu/Development/MyApp.app/Contents/Frameworks/MyApp Helper NP.app
--prepared:/Users/zhuolu/Development/MyApp.app/Contents/Frameworks/MyApp Helper EH.app
--validated:/Users/zhuolu/Development/MyApp.app/Contents/Frameworks/MyApp Helper NP.app
--validated:/Users/zhuolu/Development/MyApp.app/Contents/Frameworks/MyApp Helper EH.app
--prepared:/Users/zhuolu/Development/MyApp.app/Contents/Frameworks/MyApp Helper.app
--validated:/Users/zhuolu/Development/MyApp.app/Contents/Frameworks/MyApp Helper.app
--prepared:/Users/zhuolu/Development/MyApp.app/Contents/Frameworks/Electron Framework.framework/Versions/Current/.
--validated:/Users/zhuolu/Development/MyApp.app/Contents/Frameworks/Electron Framework.framework/Versions/Current/.
MyApp.app: valid on disk
MyApp.app: satisfies its Designated Requirement
```

> NB: Please do not intend to use the `spctl` tool to assess the acceptance of your app by the system as only *Developer ID-signed* apps and those downloaded from the *Mac App Store* are accepted by default.


[Mac General Prerequisites]: MacGeneralPrerequisites.md
[Development-signed Application]: DevelopmentSignedApplication.md
[Mac App Store Deployment]: MacAppStoreDeployment.md
[Mac App Store Deployment (outdated)]: MacAppStoreDeploymentOutdated.md
[Developer ID-signed Application]: DeveloperIDSignedApplication.md

[Member Center]: https://developer.apple.com/membercenter/
