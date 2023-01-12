# xtravision-kotlin-package
Host AAR file.


# How to add AAR file into your App
To get a Git project into your build:

Step 1. Add repo in your root setting.gradle at the end of repositories:
```sh
	
    repositories {
        ...
        maven { url 'https://jitpack.io' }
    }
```
Step 2. Add the dependency into app/build.gradle
```
	dependencies {
	        implementation 'com.github.xtravision-ai:xtravision-kotlin-package:0.0.5'
	}
```

# How to Integrate with Codebase (TODO)