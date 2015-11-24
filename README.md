# Android-SDK

## Integration steps

Add the following repository.

```groovy
    defaultConfig {
        ...

        repositories {
            maven { url "https://github.com/Medable/Android-SDK/raw/master/" }
        }
    }
```

Add the following dependency.

```groovy
dependencies {
    compile 'com.medable:AndroidSDK:0.1.+'    // or use the last version number instead of 0.1.+
}
```

Synch Gradle, and you're set!
