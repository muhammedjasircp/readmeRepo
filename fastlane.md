## What is Fastlane?

Fastlane allows you to automate the complete release process of your iOS and Android apps. It handles tedious tasks like generating screenshots, dealing with code signing and releasing your application.

For working with Fastlane, Mac is required. Fastlane will not work on Windows PC. All you have to do is configure a Fastfile that defines the steps needed to build and deploy your mobile application.

## Installing Fastlane

First you need to install Fastlane on your Mac. Follow these steps:

Install the latest Xcode command line tools:

```zsh
$ xcode-select --install
```

Install Ruby using Homebrew:

```zsh
$ brew install ruby
```

Install Fastlane with RubyGems:

```zsh
$ sudo gem install fastlane -NV
```

You are now ready to set up Fastlane for iOS and Android 🚀

## Fastlane for Android

### Prerequisites

Before continuing make sure you have:

- ⚠️A Google Play Console _admin_ account and its username (email) and password
- ⚠️An application created in the Google Play Console (unlike for iOS Fastlane cannot do that for you)
- ⚠️[Google Credentials JSON file](https://docs.fastlane.tools/getting-started/android/setup/#collect-your-google-credentials). Download the JSON key file, and copy it into `/android/key.json`
- ⚠️Android signing key. use the [Official documentation ](https://reactnative.dev/docs/signed-apk-android#generating-a-signing-key)

### Setting up

First you need to set up Fastlane for your android project:

```
$ cd yourApp/android
$ fastlane init
```

Fastlane will automatically detect your project and ask for any missing information.

The following questions will be asked:

- Package Name (com.krausefx.app)
  - Our answer is `com.packagename`
- Path to the json secret file
  - Type `key.json` (path to the file previously created in the Prerequisites step)
- Download existing metadata and setup metadata management?
  - `y`

Fastlane will then give you some information about git, the files it will create, etc. Just type enter to continue.

Once the setup has finished you can see a new folder inside the android folder:

- fastlane/
  - Appfile
  - Fastfile

[Appfile](https://docs.fastlane.tools/advanced/Appfile/#appfile) contains identifiers used to connect to the Google Play Console and the link to the key.json file.
[Fastfile](https://docs.fastlane.tools/actions/) contains all actions you can launch. A beta [lane](https://docs.fastlane.tools/advanced/lanes/), a deploy lane and a test lane are available by default.

```ruby
default_platform(:android)

platform :android do
  desc "Runs all the tests"
  lane :test do
    gradle(task: "test")
  end

  desc "Submit a new Beta Build to Crashlytics Beta"
  lane :beta do
    gradle(task: "clean assembleRelease")
    crashlytics

    # sh "your_script.sh"
    # You can also use other beta testing services here
  end

  desc "Deploy a new version to the Google Play"
  lane :deploy do
    gradle(task: "clean assembleRelease")
    upload_to_play_store
  end
end

```

We can customize these lanes and perform [actions](https://docs.fastlane.tools/actions/) like,

- Testing
- Building
- Screenshots
- Project
- Code Signing
- Documentation
- Beta
- Push
- Releasing your app
- Source Control
- Notifications
- App Store Connect
- Misc
- Deprecated
- Plugins

### Creating a beta build

⚠️ The first time you deploy your application, you must upload it into Google Play Console manually. Google don't allow to use theirs APIs for the first upload.

```ruby
desc "Submit a new build to Alpha channel"
lane :alpha do
    gradle(task: 'assemble', build_type: 'Release')
    upload_to_play_store(track: 'alpha')
end
```

❗ There is no official plugin to automatically upgrade android version code (unlike the iOS lane).
Before each deployment, be sure to manually upgrade the versionCode value inside android/app/build.gradle. for this we can use the action

```ruby
increment_version_code(
    gradle_file_path: "./app/build.gradle"
)
```

our lane would be

```ruby
desc "Submit a new build to Alpha channel"
lane :alpha do
    increment_version_code(
        gradle_file_path: "./app/build.gradle"
    )
    gradle(task: 'assemble', build_type: 'Release')
    upload_to_play_store(track: 'alpha')
end
```

Now we can create an Alpha build using `bundle exec fastlane alpha` command

## Fastlane for iOS

### Prerequisites

Before continuing make sure you have:

- ⚠️An Apple ID with an admin user, with its username (email) and password
- ⚠️Your app name, if not already created on the Developer Portal. Fastlane can create applications in the Developer Portal and App Store Connect, so it's recommended to let Fastlane do the job for you.
- ⚠️You also need to create an App Icon and apply it to use Fastlane or you will get an error on running fastlane beta. You can simply create one using the website [MakeAppIcon](https://makeappicon.com/)

Open your Xcode project and modify some information:

- In the `General` tab, `Identity` section, change the `Bundle Identifier` to your identifier (useful for Fastlane)
- In the `Signing & Capabilities` tab, `Signing` section, disable `Automatically manage signing`
- In the `Build Settings` tab, set view filter on top to `All` and `Combined`, then go to the `Signing` section and into `Code Signing Identity`, set `Apple Development` for the `debug` line (including `Any iOS SDK` also) and set `Apple Distribution` for the `release` line (including `Any iOS SDK` also).

Like this:

| Code Signing Identity | < Multiple values > |
| --------------------- | ------------------- |
| Debug                 | Apple Development   |
| Any iOS SDK           | Apple Development   |
| Release               | Apple Distribution  |
| Any iOS SDK           | Apple Distribution  |

#### Setting up

First you need to set up Fastlane for your iOS project:

```
$ cd my-project/ios
$ fastlane init
```

Fastlane will automatically detect your project and ask for any missing information.

The following questions will be asked:

- `What would you like to use fastlane for?`
  - For this tutorial a good answer is `2 - Automate beta distribution to TestFlight`
- `Select Scheme:`
  - Here we will select the scheme without `-tvOS` suffix
- `Apple ID Username:`

- `Password (for Apple ID Username):`

- If your account has multiple teams in the App Store Connect, you may have this question: `Multiple App Store Connect teams found, please enter the number of the team you want to use:`
  - Select the right team
- If your account has multiple teams in the Developer Portal, you may have this question: `Multiple teams found on the Developer Portal, please enter the number of the team you want to use:`
  - Select the right team
- If you haven't already created the App on the Developer Portal or App Store Connect, Fastlane can do it for you! (else you must have a message `Your app 'com.tcm.boilerplate' is available in your Apple Developer Portal / App Store Connect`)
  - It will ask `Do you want fastlane to create the App ID for you on the Apple Developer Portal / App Store Connect? (y/n)`
    - Type `y`
  - `App Name`:
    - `Your app name`

Fastlane will then give you some information about git, the files it will create, etc. Just type `enter` to continue.

**Congrats!** Fastlane has created some files.

If you are using Git, commit all generated files.

Once the setup has finished you can see a new folder inside the `ios` folder:

```
 - fastlane/
   - Appfile
   - Fastfile
```

It's not finish, you need to follow `Code Signing` part to setting up a provisioning profile.

For information:

`Appfile` contains identifiers used to connect to the Developer Portal and App Store Connect.
You can read more about this file [here](https://docs.fastlane.tools/advanced/#appfile).

`Fastfile` contains all actions you can launch.
You can read more about this file [here](https://docs.fastlane.tools/actions).
Because we previously chose `Automate beta distribution to TestFlight` on set up, a `beta` [lane](https://docs.fastlane.tools/advanced/lanes/) is available by default.
This `lane` contains 3 actions:

- increment the build number of your app
- build your app
- upload to TestFlight

```ruby
lane :beta do
    increment_build_number(xcodeproj: "yourApp.xcodeproj")
    build_app(workspace: "yourApp.xcworkspace", scheme: "yourApp")
    upload_to_testflight()
  end
```

To fetch the metadata of our generated app we can use

```zsh
bundle exec fastlane deliver init
```

All the screenshots and other metadata files can now be directly updated from our repo.

## Code Signing

If you are just starting a new project, it's important to think about how you want to handle code signing. This guide will help you choose the best approach for you.

For existing projects it might make sense to switch from a manual process to the [match approach](https://codesigning.guide/) to make it easier for new team-members to onboard.

#### Using Match

With [match](https://docs.fastlane.tools/actions/match/) you store your private keys and certificates in a git repo to sync them across machines. This makes it easy to onboard new team-members and set up new Mac machines. This approach is secure and uses technology you already use.

1. Initialize match using `bundle exec fastlane match init`. It will ask for the git repo url where you would like to store the certificates.
2. Uncomment and update the values of `app_identifier` and `username` with the corresponding values in the generated `Matchfile`
3. To generate and store new certificates and provisioning profiles run:
   - `bundle exec fastlane match appstore` for distribution certificates
   - `bundle exec fastlane match development` for development certificates
4. After certificates are created. using xcode select the appropriate provision profiles for Debug and Release configs
5. In the Info.plist add a row with key `ITSAppUsesNonExemptEncryption` with value `NO` if your app uses no encryption. Otherwise set it as `YES` and also provide a value for the `ITSEncryptionExportComplianceCode` key.
6. Finally in the Fastfile add match(),

```ruby
lane :beta do
    increment_build_number(xcodeproj: "yourApp.xcodeproj")
    match(type: "appstore")
    build_app(workspace: "yourApp.xcworkspace", scheme: "yourApp")
    upload_to_testflight()
  end
```

For other code signing approaches you can refer [Codesigning concepts](https://docs.fastlane.tools/codesigning/getting-started/)

Creating a beta build and uploading it on TestFlight is now really easy.
Just type the following:

```zsh
$ cd my-project/ios
$ fastlane beta
```

## Troubleshooting

If the `fastlane init` process is stuck when running `bundle install` or `bundle update` it may mean that bundle command is asking for root permissions.
You can stop the process and retry again with `sudo fastlane init`, however you will need to change back ownership of the generated files to your user when it finishes by running this command:

```zsh
$ sudo chown <your-user> <files>
```
