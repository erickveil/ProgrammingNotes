

> [!note]
> I was doing this build from Visual Studio Code, where I do most of my Flutter development. However, it's possible that things might be a bit easier if you try using Android Studio with all the Flutter plugins to perform the build.



**1. Create a keystore file** (if you don't have one):
```bash
keytool -genkey -v -keystore ~/my-release-key.keystore -keyalg RSA -keysize 2048 -validity 10000 -alias my-key-alias
```

Remember to set the `-keystore` path and the `-alias` value for where you want the key and its name.

The answers to the questions aren't important, but do note them down, especially the password.


**2. Create a key properties file** at `android/key.properties`:
```properties
storePassword=your_store_password
keyPassword=your_key_password
keyAlias=my-key-alias
storeFile=/path/to/my-release-key.keystore
```

If your password has spaces, do not use quotes. Just type the password as-is.

The `storePassword` and the `keyPassword` are typically the same unless you somehow ran `keytool` to ask you to set both.


**3. Update build.gradle** to reference the key properties:
````gradle
// ...existing code...

def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

// ...existing code...

android {
    // ...existing code...
    
    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }
    
    buildTypes {
        release {
            signingConfig signingConfigs.release
        }
    }
}

// ...existing code...
````

**4. Build the signed AAB**:
```bash
flutter build appbundle --release
```

The signed AAB will be created at: `build/app/outputs/bundle/release/app-release.aab`

**Note:** Add `android/key.properties` to your .gitignore to keep your keystore credentials private!

# Create an apk for sideloading

It may be just easiest to build an apk directly:

```sh
flutter build apk --release
```

# Final Notes

Pay attention to the errors and warnings that you'll inevitably get when trying to build. The Java/Gradle/Android and all the other moving parts update their minimum and compatible versions frequently, and you will have to manually keep up with it, unlike with Kotlin Development in Android Studio. The `build.gradle`, `gradle-wrapper.properties`, and `settings.gradle` files might need changes, and then packages might need to be downloaded to reflect that.