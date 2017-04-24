# Maven Android
This is a Maven Artifact Repository for uploading library `aar` files from private github repositories.

## Prepare your project for deploying

### General pointers before deploying your library : 
 - Proguard your source-code preserving only the api you want to expose.

### Signing your artifacts :
In order to prove that the artifacts are uploaded by you, you need to sign them with your gpg keys. You can obtain the keys by following the steps below :

1. Download GPG Suite from [here](https://releases.gpgtools.org/GPG_Suite-2016.10_v2.dmg) to create GPG keys via easy UI. Find the latest version [here](https://gpgtools.org/).
2. Install GPG Suite.
3. Open GPG Keychain.
4. Click on the new Key Icon.
5. Enter your Full Name, email, and passphrase. Check upload key checkbox. Do not lose the **passphrase**, you will need it to sign your artifacts.
6. Click on Generate Key.
7. After your key is generated, you will be able to see it in the keychain :
8. Copy the **Key Id** from here. We will need it later.
9. Publish your keys :
	- Search for your keys using the following command:
	```
	$ gpg --keyserver hkp://pgp.mit.edu --search-keys johndoe@example.com # Use your email
	```
	- If your `keyid` is not in the list, publish it using the following commands (`YYYYYYYY` is your `keyId`:
	```
	$ gpg --keyserver hkp://keyserver.ubuntu.com --send-keys YYYYYYYY
	$ gpg --keyserver hkp://pgp.mit.edu --send-keys YYYYYYYY
	```

### Signing Information in `gradle.properties`
Create a `gradle.properties` file inside the library module of your project (Multiple files if you have multiple modules). Set the following information obtained in the previous step :

```
signing.keyId=<yourKeyId>
signing.password=<yourKeyPassphrase>
signing.secretKeyRingFile=<yourKeyRingFilePath>
```
You can find your key file at `/Users/garima-fueled/.gnupg/secring.gpg`. Make sure to add your `gradle.properties` file to `.gitignore`.

### Scripts
 - Add the `gradle-mvn-push.gradle` and `version.gradle` gradle scripts to your project from [here](https://github.com/Fueled/garage-android/tree/master/extra/gradle). (Add it to the `extra/gradle/` directory. 
 - Also add the deploy script `deploy_library.sh` from [here](https://github.com/Fueled/garage-android/tree/master/scripts/). (Add it to the `scripts/` directory.

### Project `build.gradle` configuration : 

Inside the `build.gradle` file of your library module add the following lines (If you have multiple library modules, make sure to add the following to `build.gradle` files of all the modules :  

##### Apply the uploadArchives script :
Apply the `gradle-mvn-push.gradle` script : 

```
apply from: '../extra/gradle/gradle-mvn-push.gradle'
```

##### Set the variables :

At the top level : 

 - `version` `buildVersionName()`
 - `group` `<your package name>`

Inside the `ext` closure : 

```
ext {
	POM_NAME = <your pom name>
	POM_ARTIFACT_ID = <name of artifact>
	POM_PACKAGING = "aar"
}
```
##### Publishing a default configuration (Multiple build variants):

If you have multiple build-variants in your library module, then make sure you publish a default configuration inside the `android` closure. Change this configuration to the one desired  before running the deploy script:


```
android {
	defaultPublishConfig "debug"
}
```


Refer to the garage `build.gradle` [here](https://github.com/Fueled/garage-android/blob/master/garage/build.gradle)
 
 

## Deploy

Inside your project run the deploy script :
 
```
./scripts/deploy_library.sh
```
You might need to do `chmod a+x` to `deploy_library.sh`.

#### About Deploy script
 - This script takes in the version name as `(major.sprint.build)` -> `0.1.1` and so on...
 - It clones the `maven-android` repository in the `~/.maven` folder of the user.
 - It then runs the `uploadArchives` task created using the `gradle-mvn-push.gradle` script.
 - It then copies the library `aar` files to the `maven-android` project and commits them using the default `.ssh` config. Make sure you have the ssh properly setup. You can choose to not commit the artifacts and instead go to the `~/.maven` directory to commit those yourself.
 - It also pushes a branch on the `maven-android` projects : `release-<versionName>`.
 
 
## Use the library in your project :

This project currently contains the following libraries :

### Garage
Inside your root project's `build.gradle` add the maven repository as follows :

```
def getReleaseRepositoryGithub() {
    return 'http://raw.githubusercontent.com/Fueled/maven-android/maven/releases'
}

allprojects {
    repositories {
        jcenter()
        maven { url "https://jitpack.io" } //we need to be able to remove this in future
        maven {
            url getReleaseRepositoryGithub()
        }
    }
}

```

Add the dependency : 

```
debugCompile ("com.fueled.garage:garage:0.1.2@aar") {
    	transitive = true
}
```

```
releaseCompile "com.fueled.garage:garage-no-op:0.1.2@aar"
```
 
#### That's All Folks!
