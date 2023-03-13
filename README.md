# xtravision-kotlin-package
Host AAR file.


### Add Xtravision-Kotlin SDK and SDK Dependencies:  

**Step-1:** Add repo in your root setting.gradle at the end of repositories:
```gradle
    repositories {
        ...
        maven { url 'https://jitpack.io' }
    }
```
**Step-2:** Add all required dependencies in the app ``build.gradle``  file.
```gradle
	dependencies {
		// Xtra-Vision:Pose-Detection
		implementation 'com.github.xtravision-ai:xtravision-kotlin-package:3.3.2'
		
		// Android Camera Preview
		implementation "androidx.camera:camera-view:1.1.0-beta01"

		//Websocket Utility, to connect to Websocket server
		implementation 'com.squareup.okhttp3:okhttp:3.12.6'
		implementation 'com.squareup.okhttp3:mockwebserver:3.12.1'
		implementation 'com.google.code.gson:gson:2.8.8'

		// CameraX Configuration
		implementation "androidx.camera:camera-core:1.1.0-beta01"
		implementation "androidx.camera:camera-camera2:1.1.0-beta01"
		implementation "androidx.camera:camera-lifecycle:1.1.0-beta01"
		implementation "androidx.camera:camera-video:1.1.0-beta01"
		implementation "androidx.camera:camera-view:1.1.0-beta01"
		implementation "androidx.camera:camera-extensions:1.1.0-beta01"

		// Required google ml-kit and required utilities
		implementation 'com.google.mlkit:pose-detection-accurate:18.0.0-beta3'
		implementation "androidx.window:window:1.0.0-beta02"
		implementation("com.google.guava:guava:31.0.1-android")
		implementation 'com.google.code.gson:gson:2.8.6'
	}
```

#### Integration with Codebase:
1. Add android PreView in your activity layout XML file, where you required a Camera view.

	```XML
	<androidx.camera.view.PreviewView
			android:layout_width="match_parent"
			android:layout_height="match_parent"
			android:id="@+id/xtraVisionPreviewView"/> 
	```
2.  Import the below classes into your Activity class.

	```java
	import ai.xtravision.*
	import ai.xtravision.util.*
	import androidx.camera.view.PreviewView
    import android.graphics.Color
	```
3.  Prepare SDK-required data:
	1. XtraVision-Connection-Data: contain all connection-specific information.
	2. XtraVision-Request-Data: request specific information for WebSocket.
	3. XtraVision-Lib-Data: contain all Lib/SDK specific information/configurations. like Camera-Selector, Current Context, 2d Skeleton view, etc
    ```kotlin
    // KOTLIN Code-base

    // STEP-1:: Set Required Variable
    val authToken = "_AUTH-TOKEN_"
    val assessmentName = "SQUATS" 
    val isPreJoin = false
    val myPreviewView : PreviewView = findViewById<PreviewView>(R.id.xtraVisionPreviewView)
    val selectedCamera = 0  //0: Front Camera, 1: BACK Camera
    // Suppose your  current class is inherited with 
    // XtraVisionAIListener interface
    val myResponseListener: XtraVisionAIListener = this 

    val assessmentConfig = XtraVisionAssessmentConfig(
                repsThreshold = 5,
                graceTimeThreshold = 20)

    // Step-2:: Prepare Initial Object
    val connectionData = XtraVisionConnectionData(
                authToken = authToken,
                assessmentName = assessmentName,
                assessmentConfig = assessmentConfig
                )
    val requestData = XtraVisionRequestData(
                isPreJoin = isPreJoin
                )
    val skeletonConfig = XtraVisionSkeletonConfig(
                dotRadius = 8.0f,
                dotColor = Color.WHITE,
                leftLineWidth = 15f,
                leftLineColor = Color.RED,
                rightLineWidth = 15f,
                rightLineColor = Color.BLUE
                )
    val libData = XtraVisionLibData(
                context = this,
                previewView = myPreviewView,
                responseListener = myResponseListener,
                selectedCamera = mySelectedCamera,
                enableSkeletonView = true,
                skeletonConfig = skeletonConfig 
                )
    ```

4. Check Camera Permission and Initiate SDK processing.
	```kotlin
	// Check and Ask for Camera permission
	XtraVisionPermissionHelper.checkAndRequestCameraPermissions(this)

	// STEP-3:: if permission is granted, then let's start an assessment
	if (XtraVisionPermissionHelper.cameraPermissionsGranted(this)) {
		// create an object and initiate the Assessment process
		val xtraVisionAIManger = XtraVisionAIManager(connectionData,reqData,libData)
		xtraVisionAIManger!!.initiate()
	} else {
		XtraVisionPermissionHelper.checkAndRequestCameraPermissions(this)
	}
	```

5. To clean up SDK cached data, use the following activity lifecycle method. (Always clear data when you completed assessment).
	```kotlin
	override fun onPause() {
		super.onPause()
		xtraVisionAIManger?.onPause()
	}

	override fun onDestroy() {
		super.onDestroy()
		xtraVisionAIManger?.onDestroy()
	}

	// on back-press destroy current activity, to prevent conflict with previous assessment
	override fun onBackPressed() {
		super.onBackPressed()
		finish()
	}

	```
---

#### The final code for the Activity Class looks like this:

```kotlin
package com.xtra.kotlin.example

import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle

import ai.xtravision.*
import ai.xtravision.util.*
import androidx.camera.view.PreviewView
import android.graphics.Color

import android.util.Log

class MainActivity : AppCompatActivity(), XtraVisionAIListener {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // STEP-1:: Set Required Variable
        val authToken = "__AUTH_TOKEN__"
        val assessmentName = "HALF_SQUAT"
        val isPreJoin = false
        val previewView : PreviewView = 
                    findViewById<PreviewView>(R.id.xtraVisionPreviewView)
        val selectedCamera = 0  //0: Front Camera, 1: Back Camera

        // STEP-2:: create initial 
        //Objects: connectionData, requestData, and libData
        val connectionData = XtraVisionConnectionData(authToken, assessmentName)
        val reqData = XtraVisionRequestData(isPreJoin)
        val skeletonConfig = XtraVisionSkeletonConfig(
            dotRadius = 8.0f,
            dotColor = Color.WHITE,
            leftLineWidth = 15f,
            leftLineColor = Color.RED,
            rightLineWidth = 15f,
            rightLineColor = Color.BLUE
		)
        val libData = XtraVisionLibData(
            this,
            previewView,
            this,
            selectedCamera,
            enableSkeletonView = true
            skeletonConfig = skeletonConfig
        )

        // Check and Ask for Camera permission
        XtraVisionPermissionHelper.checkAndRequestCameraPermissions(this)

        // STEP-3:: if permission is granted, then let's start an assessment
        if (XtraVisionPermissionHelper.cameraPermissionsGranted(this)) {
            // create an object and initiate the Assessment process
            val xtraVisionAIManger = XtraVisionAIManager(
                connectionData, reqData, libData)
            xtraVisionAIManger!!.initiate()
        } else {
            XtraVisionPermissionHelper.checkAndRequestCameraPermissions(this)
        }
    }
    
    override fun onPause() {
		super.onPause()
		xtraVisionAIManger?.onPause()
	    }

    override fun onDestroy() {
	super.onDestroy()
	xtraVisionAIManger?.onDestroy()
    }

    // on back-press destroy current activity, to prevent conflict with previous assessment
    override fun onBackPressed() {
	super.onBackPressed()
	finish()
    }

    /**
     * ==============================
     * Response Listener: Start
     * ==============================
     */
    override fun onConnectSuccess() {
        Log.d("CustomAIListener","onConnectSuccess")
    }

    override fun onConnectFailed() {
        Log.e("CustomAIListener","onConnectFailed")
    }

    override fun onConnectClose() {
        Log.d("CustomAIListener","onConnectClose")
    }

    override fun onResponse(resp: String?) {
        Log.d("CustomAIListener", resp.toString())
    }
    /**
     * ==============================
     * Response Listener: End
     * ==============================
     */
}
```


### Sample App:
https://github.com/xtravision-ai/XtraVision-KotlinSampleApp
