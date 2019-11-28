<h1 align="center">
  Continuous Integration With GitHub, Fastlane & Jenkins
</h1>

<h3 align="center">
  <img src="https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/header_logo.png?raw=true" alt="fastlane Logo" width="500">
  </a>
</h3>



###### Machine environments

- Java 11

- Homebrew 2.1.16
- Ruby 2.6.5
- Fastlane 2.135.2
- firebase_app_distribution 0.1.4
- Jenkins 2.206

###### Prerequisites

- [*Codesigning*](https://developer.apple.com/support/code-signing/)
- [*Authenticating with GitHub from Git*](https://help.github.com/en/github/getting-started-with-github/set-up-git#next-steps-authenticating-with-github-from-git)
- [*Bitcode*](https://developer.apple.com/library/archive/technotes/tn2151/_index.html)
- [*Upload an app to App Store Connect*](https://help.apple.com/xcode/mac/current/#/dev442d7f2ca)

###### References

- https://fastlane.tools/
- https://github.com/fastlane/fastlane
- https://www.raywenderlich.com/233168-fastlane-tutorial-getting-started
- https://www.raywenderlich.com/1774995-continuous-integration-with-github-fastlane-jenkins
- https://jenkins.io/doc/
- https://docs.fastlane.tools/best-practices/continuous-integration/jenkins/

# 1. Fastlane:

_fastlane_ is a tool for iOS and Android developers to automate tedious tasks like generating screenshots, dealing with provisioning profiles, and releasing your application.

## 1.2 Getting Started:  

### 1.2.1 Setup Fastlane:

The tool fastlane is a collection of *Ruby scripts*, so you must have the correct version of Ruby installed. Chances are that your OS comes with *Ruby 2.0* by default, but you can confirm whether this is the case by opening Terminal and entering:

```bash
ruby -v
```

If it’s not installed, the easiest way to do so is via [*Homebrew*](http://brew.sh/) a package manager for macOS.

```bash
/usr/bin/ruby -e \
  "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Then, install Ruby using:

```bash
brew update && brew install ruby
```

You’ll also need *Xcode Command Line Tools (CLT)*. To ensure that they’re installed, enter into Terminal:

```bash
xcode-select --install
```

Now you’re ready to install fastlane! Enter the following command to do so:

```bash
sudo gem install fastlane -NV
```

Navigate to your iOS app and run

```bash
fastlane init
```

*fastlane* will automatically detect your project, and ask for any missing information.

> *Notes*: If you get a “permission denied” error, prefix this command with `sudo`.

After some output, fastlane will ask: "What would you like to use fastlane for?"

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/fastlane_init.png?raw=true)

You should config fastlane file by ourself. Input **4** and tap **Enter**.

Back to project folder, you’ll see a few new things: 

+ **Gemfile**: which includes the fastlane gem as a project dependency

- **Appfile**: stores the app identifier, your Apple ID and any other identifying information that fastlane needs to set up your app.
- **Fastfile**: manages the *lanes* you’ll create to invoke fastlane actions.

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/fastlane_file.png?raw=true)

### 1.2.2 Setup environment variables:

*fastlane* requires some environment variables set up to run correctly. In particular, having your locale not set to a UTF-8 locale will cause issues with building and uploading your build. In your shell profile add the following lines:

```bash
export LC_ALL=en_US.UTF-8
export LANG=en_US.UTF-8
```

## 1.3 Configuration fastlane:

### 1.3.1 Appfile

```ruby
# app_identifier("[[APP_IDENTIFIER]]") # The bundle identifier of your app
# apple_id("[[APPLE_ID]]") # Your Apple email address

# For more information about the Appfile, see:
#     https://docs.fastlane.tools/advanced/#appfile

```

Remove the **#** in the beginning of the line to enable the options

In case your account has multiple teams, add the following lines:

```ruby
itc_team_id("120184686") # App Store Connect Team ID
team_id("N4Q7U8RR8X") # Developer Portal Team ID
```

### 1.3.2 Fastfile - Lane configuration

Open **Fastfile**, you will see something like this:

```ruby
default_platform(:ios)

platform :ios do
  desc "Description of what the lane does"                    # 1
  lane :custom_lane do                                        # 2
    # add actions here: https://docs.fastlane.tools/actions
  end
end
```

Here’s what this code does:

1. Provides a description for the **lane**. A lane is a workflow of sequential tasks.
2. Provides the **lane name** and the **actions**(tasks) of it.

To get the most up-to-date information from the command line on your current version you can also run:

```bash
fastlane actions # list all available fastlane actions
fastlane action [action_name] # more information for a specific action
```

Or you can refer in [available actions](https://docs.fastlane.tools/actions/)

For more action, check out the [fastlane plugins](https://docs.fastlane.tools/plugins/available-plugins/) page. If you want to create your own action, check out the [local actions](https://docs.fastlane.tools/create-action/#local-actions) page.

### 1.3.3 Build your app

*fastlane* takes care of building your app using an **action** called [*build_app*](https://docs.fastlane.tools/actions/build_app/) (alias for ***build_ios_app***, or ***gym***), just add the following to your **Fastfile**:

```bash
build_app(scheme: "FastlanePlayground")
```

Additionally you can specify more options for building your app, for example:

```bash
build_app(scheme: "FastlanePlayground",
          export_method: "development")
```

For example, you will have below config for build scheme *FastlanePlayground* and export  as *ad-hoc* method

```bash
default_platform(:ios)

platform :ios do
  desc "Build scheme FastlanePlayground"
  lane :lane_dev do
    build_app(scheme: "FastlanePlayground",
              export_method: "ad-hoc")
  end
end
```

And then, in **Terminal**, run:

```bash
fastlane lane_dev
```

If everything works, you should have a `FastlanePlayground.ipa` file in the current directory. If you see any codesigning error, don't worry, you will go to the next part.

### 1.3.4 Codesigning (Matchfile)

Chances are that something went wrong because of code signing at the previous step. [Code Signing Guide](https://docs.fastlane.tools/codesigning/getting-started/) will helps you setting up the right code signing approach for your project.

As the link above, i will suggest to use [*match*](https://fastlane.tools/match) (alias for ***sync_code_signing***) for codesigning. The concept of [*match*](https://fastlane.tools/match) is described in the [codesigning guide](https://codesigning.guide/).

With [*match*](https://fastlane.tools/match) you will **store** your private keys and certificates in a **git repo** to sync them across machines. This makes it easy to onboard new team-members and set up new Mac machines. This approach [is secure](https://docs.fastlane.tools/actions/match/#is-this-secure) and uses technology you already use.

Getting started with [*match*](https://fastlane.tools/match) requires you to **<u>revoke your existing certificates</u>**.

To get started, create a new private Git repo and run:

```bash
fastlane match init
```

It will ask for storage mode. Select *git* and enter *URL of the Git Repo*. After all, [*match*](https://fastlane.tools/match) will create a file in *./fastlane/Matchfile*. Open it by code editor, you you see something like this:

```bash
git_url("https://github.com/hdwebsoft-mobile/ios-certificates.git")

storage_mode("git")

type("development") # The default type, can be: appstore, adhoc, enterprise or development

# app_identifier(["tools.fastlane.app", "tools.fastlane.app2"])
# username("user@fastlane.tools") # Your Apple Developer Portal username
```

Remove **#** from *app_identifier* and *username* and update those values, so [*match*](https://fastlane.tools/match) can fetch provisioning and certificates of app 

To generate and store new certificates and provisioning profiles run:

```bash
fastlane match development
```

<center>or</center>

```bash
fastlane match appstore
```

[*match*](https://fastlane.tools/match) will ask for passphrase for Git Repo. After enter passphrase, it will be store in keychain. To avoid this prompt, you can define *environment variable* in **Matchfile** by using `MATCH_PASSWORD`:

```ruby
ENV["MATCH_PASSWORD"] = "hdwebsoft"
```

In **Fastfile**, just add this line:

```ruby
match(type: "development")
```

If success, you will see provisioning file info. You should save this **Environment Variable**, because it may needed for **future use**.

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/fastlane_match_complete.png?raw=true)

### 1.3.5 Uploading app

After building your app, it's ready to be uploaded to a beta testing service of your choice. The beauty of **fastlane** is that you can easily switch beta provider, or even upload to multiple at once, without any extra work.

#### 1.3.5.1 Firebase Crashlytics

[Firebase App Distribution](https://github.com/fastlane-community/fastlane-plugin-firebase_app_distribution) makes distributing your apps to trusted testers painless. By getting your apps onto testers' devices quickly, you can get feedback early and often. To learn more about Firebase App Distribution, go [here](https://firebase.google.com/docs/app-distribution).

Because [Firebase App Distribution](https://github.com/fastlane-community/fastlane-plugin-firebase_app_distribution) is not a part of fastlane, so you need to install it as a [*Plugins*](https://docs.fastlane.tools/plugins/using-plugins/). Run this command:

```bash
fastlane add_plugin firebase_app_distribution
```

All you have to do is putting the config of Firebase Crashlytics project into **firebase_app_distribution**, after **build_app**:

```ruby
lane :lane_dev do
  desc "Build scheme FastlanePlayground"
  match(type: "development") #1
  disable_automatic_code_signing #2
  update_project_provisioning(profile:ENV["sigh_co.hdwebsoft.FastlanePlayground_development_profile-path"])    #3
  build_app(scheme: "FastlanePlayground", #4
            export_method: "development", #5
            export_options:{
              compileBitcode: false #6
            })
  firebase_app_distribution(app: "1:434647080927:ios:6c65db07c49a490ca92e6a", #7
                            release_notes: "Fastlane release 15",
                            groups: "tester")
  upload_symbols_to_crashlytics(gsp_path: "./FastlanePlayground/Config/GoogleService-Info.plist") #8
end
```

Here’s what this code does:

1. Codesigning, fetch provisioning and certificates
2. Disable Automatically Code Signing, so you can use provisioning profile
3. In case provisioning profile has not been set, **build_app** can not do its job because of missing provisioning profile. Because [*match*](https://fastlane.tools/match) has fetched provisioning already, so you can use that profile and update to project
4. Build scheme FastlanePlayground
5. This is method of distribution. Select export method is development
6. (Optional) For export_method `development`, you should disable **<u>Rebuild from bitcode</u>**. Because you will deploy to Crashlytics, so if bitcode is enable in Build Settings, **Rebuild from bitcode** will re-compile and make another build. In that case, old dSYM will not match with new build. In case deploy to AppStore, you can remove this option because it can be downloaded from App Store Connect, after upload the build. For more understanding, you will need to dig deeper about **bitcode**
7. Upload build to Firebase Crashlytics
8. Upload dsym to Firebase Crashlytics

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/fastlane_beta_distribute_options.png?raw=true)

#### 1.3.5.2 App Store deployment

<h1>
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
</h1>



# 2. Jenkins

## 2.1 Setup Jenkins

Jenkin require Java version 8-11. If device has multiple Java version, you can set the version of <u>**current terminal window**</u> with this command in MacOS.

```bash
# List Java versions installed
/usr/libexec/java_home -V

# Java 11
export JAVA_HOME=$(/usr/libexec/java_home -v 11)

# Java 1.8
export JAVA_HOME=$(/usr/libexec/java_home -v 1.8)

# Java 1.7
export JAVA_HOME=$(/usr/libexec/java_home -v 1.7)

# Java 1.6
export JAVA_HOME=$(/usr/libexec/java_home -v 1.6)
```

If machine does not has required java version, check Java installation [in here](https://medium.com/w-logs/installing-java-11-on-macos-with-homebrew-7f73c1e9fadf)

Then, the recommended way to install [Jenkins](http://jenkins-ci.org/) is through [homebrew](http://brew.sh/):

```bash
brew update && brew install jenkins
```

From now on start `Jenkins` by running:

```bash
jenkins
```

When the installer finishes, it should open **localhost:8080** in your browser — it wants a password:

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/jenkins-unlock.png?raw=true)

Follow the instruction and open the log file **initialAdminPassword**. Copy the password and paste it into the **Administrator password** field. Click **Continue** to load the **Customize Jenkins** page:

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/jenkins-install-plugins.png?raw=true)

Select **Install suggested plugins**, this takes several minutes.

Now create your **admin account** — entering **admin** for both **Username** and **Password**. Fill remain others areas and Continue

## 2.2 Configuration Jenkins

### 2.2.1 Configuration build

You’ll see a welcome page, prompting you to **New Item** in the side menu:

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/jenkins-welcome.png?raw=true)

On the next page, fill **item name** and select **Freestyle project**:

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/jenkins-new-item.png?raw=true)

Tap OK and **Configuration** screen will appear. In General tab, check into GitHub project and fill **Project url**

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/jenkins-project-general.png?raw=true)

Next, scroll down or click across to **Source Code Management**: Select **Git** and fill your **Repository URL** again. To config **Credentials**, tap into **Add**

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/jenkins-project-source-code.png?raw=true)

Scroll down past **Build Triggers** — you’ll come back to this tab later.

The next section is **Build Environment**: Sometimes a build keeps going, even after the console log says it’s finished, so check **Abort the build if it’s stuck** and select **No Activity** as the **Time-out strategy**. If you like to know when things happened, also check **Add timestamps to the Console Output**. 

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/jenkins-project-build-env.png?raw=true)

> **Color ANSI Console Output** is an option of [AnsiColor](https://wiki.jenkins.io/display/JENKINS/AnsiColor+Plugin) plugin, which make Console output easier for reading.

Finally, the **Build** section is where it all happens! In the **Add build step** menu, select **Execute shell**:

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/jenkins-project-build-1.png?raw=true)

Because you have configured **fastlane**, so you will know which command line that using for start building your app

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/jenkins-project-build-2.png?raw=true)

Click the **Save** button to return to the project page, then select **Build Now** from the side menu:

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/jenkins-project-build-now.png?raw=true)

Something starts happening in the **Build History** section — your first build appears! Hover your cursor over **#1** to make the menu button appear, then select **Console Output** from the menu:

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/jenkins-project-console-output.png?raw=true)

### 2.2.2 Notifying Jenkins With GitHub Webhook

Jenkins pulled your project, built it and ran the test. But you had to tell it to *Build Now* — that’s not very automated

You need to create a *GitHub webhook* to notify your Jenkins server whenever you push a new commit. A webhook is a GitHub mechanism for POSTing a notification to the webhook’s URL whenever certain events happen in your GitHub repository — for example, whenever you push a commit.

First, set up your Jenkins project to expect notifications from GitHub. Select **Configure** in menu side:

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/jenkins-project-configure.png?raw=true)

Select the **Build Triggers** tab and check **GitHub hook trigger for GITScm polling**:

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/jenkins-project-build-trigger.png?raw=true)

Scroll to the bottom and press *Save*.

Now go to your GitHub page to set up the webhook.

To add a webhook for your Jenkins server, go to your project repository page and click **Settings**. Select **Webhooks** from the side menu, then click **Add webhook**:

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/github-webhooks.png?raw=true)

The first thing you need is the **Payload URL** — the URL of your Jenkins server. But, unless you set up your Jenkins with a proper external, GitHub needs a *real* URL to send notifications to — what to do?

[ngrok](https://ngrok.com/product) to the rescue! It’s a free app that uses a secure tunnel to expose `localhost` to the internet. [GitHub’s webhooks tutorial](https://developer.github.com/webhooks/configuring/) uses it, so it must be OK. Go ahead and [download ngrok for Mac OS X](https://ngrok.com/download):

In *Terminal*, `cd` to where *ngrok* is, then run this command:

```bash
./ngrok http 8080
```

Your output will look similar to this:

```bash
ngrok by @inconshreveable                                       (Ctrl+C to quit)
                                                                                
Session Status                online                                            
Session Expires               7 hours, 59 minutes                               
Version                       2.2.8                                             
Region                        United States (us)                                
Web Interface                 http://127.0.0.1:4040                             
Forwarding                    http://6796b2b9.ngrok.io -> localhost:8080        
Forwarding                    https://6796b2b9.ngrok.io -> localhost:8080       
                                                                                
Connections                   ttl     opn     rt1     rt5     p50     p90       
                              0       0       0.00    0.00    0.00    0.00 

```

Copy your *Forwarding URL* — mine is *http://6796b2b9.ngrok.io*, for the next almost-8 hours.

Back on your GitHub *Add webhook* page, paste this URL in the *Payload URL* field, and add **/github-webhook/** to the end of the URL. This is the endpoint on your Jenkins server that responds to pushes from GitHub:

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/github-webhooks-config.png?raw=true)

Click **Add Webhook**. GitHub sends a test *POST* request to the URL.

Commit and push to GitHub, then check your Jenkins page — Side menu now has an item for **GitHub Hook Log**!

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/jenkins-project-webhook.png?raw=true)

Select **GitHub Hook Log**: An event from your ngrok URL caused Jenkins to poll your GitHub repository for changes, which it found:

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/jenkins-project-webhook-log.png?raw=true)

Now, in **Build History**, Jenkins has triggered build job #2.



# Questions and Answers

Question 1: What is export_method action in fastlane?

Answer: export_method is method of distribution. You can see it in Xcode when tap Distribute App archives build

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/fastlane_export_method.png?raw=true)

------

Question 2: Why dSYM file does not work when bitcode is enable and caused **__hidden#1234** in crashlytics?

Answer: When you archive an application with bitcode enabled, the compiler produces binaries containing bitcode(A) rather than machine code. Once the binary has been uploaded to the App Store, the bitcode is compiled down to machine code(B). So the dSYM(s) in machine is related to (A), not (B), and has been **obfuscated**. 

The mappings to **de-obfuscate** them also reside in the xcarchive folder in a folder names "BCSymbolMaps".

To **de-obfuscate** such a dSYM one would use the following command:

```bash
dsymutil --symbol-map <bcSymbol-file> <obfuscated-dsym-file>
```

------

Question 3: Why should we need to disable enable_bitcode option when export_method is `development`?

Answer: When perform action `build_app`, you can see in conlose log that dSYM(s) has been de-obfuscated. But, please look closer! Only some of them has been de-obfuscated, some others is not. Some de-obfuscate command has the wrong name of BCSymbol files. So it is **not unobfuscating** as below.

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/fastlane_deobfuscate_failed.png?raw=true)

------

Question 4: Why dSym(s) has been mapped with BCSymbolMaps (Console log in Terminal), but we still need to disable bitcode (`compileBitcode: false`)? Mapping has been failed because the name not match

Answer: Same answer with **Question 3**

------

Quesiton 5: Is there any others way to upload valid dSYM(s)?

Answer: Yes, you can tell Xcode to upload dSYM(s) automatically by add build script into build phase.

------

Question 6: What should i do when provisioning/certificate in git storage has been revoked?

Answer: Because fastlane will use provisioning/certificate in git storage if exists, so it will not generate the new one. The easiest way is remove all invalid provisioning/certificate in **git storage**.

**Note**: You should **NOT** use their **nuke**: ~~fastlane match **nuke** development~~. This command will delete all.  You cannot choose a particular cert or application identifier to kill. Like a nuclear bomb, it is pretty destructive.

------

Question 7: Why I can not install fastlane plugin?

Answer: In setup step, you may receive an error that plugin version is **undefined**. Updating ruby version may fix that issue.

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/fastlane_plugin_undefined.png?raw=true)

------

Question 8: How should i do when there are too many projects from different customer in 1 CI/CD machine?

Answer: In CI/CD machine, you should only use 1 github/firebase account for all project. Just setting the right permission for that account and access into all that projects. If you still want to use multiple accounts, you will need to generate different private key, config github host, etc...

------

Question 9: I have configed webhooks in github, but Jenkins still not trigger a new build when receive push action?

Answer: You should check the repository URL in **Source Code Management** section. Jenkins will not trigger action when it is not match

![](https://github.com/phhai1710/Fastlane-GitHub-Jenkins/blob/master/Resources/jenkins-project-source-code.png?raw=true)
