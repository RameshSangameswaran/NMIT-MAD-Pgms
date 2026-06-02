---
title: "GPS Location Android Application - Lab Manual"
author: "Mobile Application Development Lab"
date: "May 2026"
documentclass: article
geometry: margin=1in
fontsize: 11pt
---

# GPS Location Android Application - Lab Manual

**Package:** `com.example.gpslocation`  
**Application Name:** GPSLocation  
**Minimum SDK:** Android 8.0 (API 26)

---

## 1. Introduction

This lab manual documents the **GPSLocation** Android application developed as part of the Mobile Application Development course. The application demonstrates how to retrieve and display the device's GPS coordinates (latitude and longitude) using Google Play Services Location API.

### 1.1 Aim

To understand how an Android application accesses device location, handles runtime permissions, and displays GPS data on the user interface.

### 1.2 Objectives

After completing this lab, students will be able to:

- Identify the folder structure of an Android Studio project
- Understand the role of `AndroidManifest.xml` and Gradle configuration
- Design UI using XML layout files
- Implement GPS location retrieval using `FusedLocationProviderClient`
- Request and handle runtime location permissions
- Trace the complete application flow from launch to location display
- Use Android Navigation Component for fragment-based navigation

### 1.3 Technologies Used

| Technology | Purpose |
|------------|---------|
| Java | Application logic |
| XML | UI layouts and resources |
| View Binding | Type-safe view access |
| Google Play Services Location | GPS/location API |
| Android Navigation Component | Fragment navigation |
| Material Design 3 | UI components (Toolbar, FAB) |

---

## 2. Project Structure (Structural View)

The following tree shows the important files and folders in the GPSLocation project:

```
GPSLocation/
|
|-- app/
|   |-- build.gradle.kts                    # App module build configuration
|   |
|   +-- src/main/
|       |-- AndroidManifest.xml             # Permissions and launcher activity
|       |
|       |-- java/com/example/gpslocation/
|       |   |-- MainActivity.java           # Main activity (app host)
|       |   |-- FirstFragment.java          # GPS location screen (core logic)
|       |   +-- SecondFragment.java         # Secondary screen (navigation demo)
|       |
|       +-- res/
|           |-- layout/
|           |   |-- activity_main.xml       # Main activity layout
|           |   |-- content_main.xml        # Fragment container layout
|           |   |-- fragment_first.xml      # First fragment layout
|           |   +-- fragment_second.xml     # Second fragment layout
|           |
|           |-- navigation/
|           |   +-- nav_graph.xml           # Navigation graph
|           |
|           |-- menu/
|           |   +-- menu_main.xml           # Toolbar menu
|           |
|           +-- values/
|               |-- strings.xml             # String resources
|               |-- colors.xml              # Color definitions
|               |-- dimens.xml              # Dimension values
|               +-- themes.xml              # App theme
|
|-- build.gradle.kts                        # Root build file
+-- settings.gradle.kts                     # Project settings
```

### 2.1 File Summary

| File | Type | Description |
|------|------|-------------|
| `AndroidManifest.xml` | XML | Declares permissions and launcher activity |
| `MainActivity.java` | Java | Entry point; hosts fragments and toolbar |
| `FirstFragment.java` | Java | Fetches and displays GPS coordinates |
| `SecondFragment.java` | Java | Secondary screen with back navigation |
| `activity_main.xml` | XML | Root layout with toolbar and FAB |
| `fragment_first.xml` | XML | UI for GPS screen |
| `nav_graph.xml` | XML | Defines fragment navigation paths |
| `strings.xml` | XML | All text displayed in the app |

---

## 3. Application Flow (Top to Bottom)

When the user launches the app, execution proceeds in the following order:

```
+------------------------------------------------------------------+
| STEP 1: Android OS reads AndroidManifest.xml                     |
|         Finds MainActivity as LAUNCHER activity                  |
+----------------------------+-------------------------------------+
                             |
                             v
+------------------------------------------------------------------+
| STEP 2: MainActivity.onCreate() is called                        |
|         Loads activity_main.xml                                  |
|         Includes content_main.xml                                |
|         Sets up Toolbar, FAB, and Navigation Controller          |
+----------------------------+-------------------------------------+
                             |
                             v
+------------------------------------------------------------------+
| STEP 3: NavHostFragment loads nav_graph.xml                      |
|         startDestination = FirstFragment                         |
+----------------------------+-------------------------------------+
                             |
                             v
+------------------------------------------------------------------+
| STEP 4: FirstFragment.onCreateView() loads fragment_first.xml    |
|         Displays "Get Location" button and TextView              |
+----------------------------+-------------------------------------+
                             |
                             v
+------------------------------------------------------------------+
| STEP 5: User taps "Get Location" button                          |
|         FirstFragment.getLastLocation() is called                |
|         -> Checks location permissions                           |
|         -> If not granted: requests permissions from user        |
|         -> If granted: calls FusedLocationProviderClient         |
+----------------------------+-------------------------------------+
                             |
                             v
+------------------------------------------------------------------+
| STEP 6: Location result received                                 |
|         Latitude and Longitude displayed in TextView             |
|         OR "Location not found" message if GPS unavailable       |
+------------------------------------------------------------------+
```

### 3.1 Permission Flow

```
User taps "Get Location"
        |
        v
Are FINE or COARSE location permissions granted?
    |                    |
   NO                   YES
    |                    |
    v                    v
Show permission      Call fusedLocationClient.getLastLocation()
dialog to user              |
    |                       v
    v                 Location object returned?
User grants?           |            |
    |                 YES           NO
   YES                 |            |
    |                  v            v
    v            Display lat/long   Show "Location not found"
getLastLocation()   in TextView
called again
```

### 3.2 Navigation Flow (Optional)

```
FirstFragment  ----(action_FirstFragment_to_SecondFragment)---->  SecondFragment
               <---(action_SecondFragment_to_FirstFragment)-----
```

The **Previous** button in `SecondFragment` navigates back to `FirstFragment`.

---

## 4. Configuration Files

Configuration files define how the app is built, what permissions it needs, and how screens are connected.

### 4.1 AndroidManifest.xml

**Path:** `app/src/main/AndroidManifest.xml`

**Purpose:** Declares location permissions, application settings, and the launcher activity.

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.GPSLocation">
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:theme="@style/Theme.GPSLocation">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

| Element | Explanation |
|---------|-------------|
| `ACCESS_FINE_LOCATION` | Permission for precise GPS location |
| `ACCESS_COARSE_LOCATION` | Permission for approximate network-based location |
| `android.intent.action.MAIN` | Identifies the main entry point |
| `android.intent.category.LAUNCHER` | Shows app icon in device app drawer |
| `android:exported="true"` | Allows Android system to launch this activity |

### 4.2 build.gradle.kts (App Module)

**Path:** `app/build.gradle.kts`

**Purpose:** Configures SDK versions, enables View Binding, and declares project dependencies.

```kotlin
plugins {
    alias(libs.plugins.android.application)
}

android {
    namespace = "com.example.gpslocation"
    compileSdk {
        version = release(36) {
            minorApiLevel = 1
        }
    }

    defaultConfig {
        applicationId = "com.example.gpslocation"
        minSdk = 26
        targetSdk = 36
        versionCode = 1
        versionName = "1.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_11
        targetCompatibility = JavaVersion.VERSION_11
    }
    buildFeatures {
        viewBinding = true
    }
}

dependencies {
    implementation(libs.activity.ktx)
    implementation(libs.appcompat)
    implementation(libs.constraintlayout)
    implementation(libs.material)
    implementation(libs.navigation.fragment)
    implementation(libs.navigation.ui)
    implementation(libs.play.services.location)
    testImplementation(libs.junit)
    androidTestImplementation(libs.espresso.core)
    androidTestImplementation(libs.ext.junit)
}
```

| Setting / Dependency | Purpose |
|---------------------|---------|
| `minSdk = 26` | Minimum Android 8.0 required |
| `viewBinding = true` | Auto-generates binding classes for layouts |
| `play.services.location` | Google Fused Location Provider API |
| `navigation.fragment` | Fragment-based navigation support |

### 4.3 nav_graph.xml

**Path:** `app/src/main/res/navigation/nav_graph.xml`

**Purpose:** Defines all app screens (fragments) and navigation actions between them.

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/nav_graph"
    app:startDestination="@id/FirstFragment">

    <fragment
        android:id="@+id/FirstFragment"
        android:name="com.example.gpslocation.FirstFragment"
        android:label="@string/first_fragment_label"
        tools:layout="@layout/fragment_first">

        <action
            android:id="@+id/action_FirstFragment_to_SecondFragment"
            app:destination="@id/SecondFragment" />
    </fragment>
    <fragment
        android:id="@+id/SecondFragment"
        android:name="com.example.gpslocation.SecondFragment"
        android:label="@string/second_fragment_label"
        tools:layout="@layout/fragment_second">

        <action
            android:id="@+id/action_SecondFragment_to_FirstFragment"
            app:destination="@id/FirstFragment" />
    </fragment>
</navigation>
```

| Attribute | Explanation |
|-----------|-------------|
| `app:startDestination` | First screen shown when app opens (`FirstFragment`) |
| `android:name` | Fully qualified Java class for the fragment |
| `action` | Defines navigation path from one fragment to another |

### 4.4 menu_main.xml

**Path:** `app/src/main/res/menu/menu_main.xml`

**Purpose:** Defines items shown in the Toolbar overflow menu.

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    tools:context="com.example.gpslocation.MainActivity">
    <item
        android:id="@+id/action_settings"
        android:orderInCategory="100"
        android:title="@string/action_settings"
        app:showAsAction="never" />
</menu>
```

---

## 5. Resource Files (XML - values/)

Resource files in `res/values/` store reusable strings, colors, dimensions, and themes.

### 5.1 strings.xml

**Path:** `app/src/main/res/values/strings.xml`

**Purpose:** Stores all text strings used throughout the application.

```xml
<resources>
    <string name="app_name">GPSLocation</string>
    <string name="action_settings">Settings</string>
    <!-- Strings used for fragments for navigation -->
    <string name="first_fragment_label">First Fragment</string>
    <string name="second_fragment_label">Second Fragment</string>
    <string name="next">Next</string>
    <string name="previous">Previous</string>

    <string name="lorem_ipsum">
        Lorem ipsum dolor sit amet, consectetur adipiscing elit. ...
        (placeholder text for Second Fragment)
    </string>
    <string name="get_location">Get Location</string>
    <string name="location_not_found">Location not found. Make sure GPS is enabled.</string>
    <string name="permission_denied">Permission denied</string>
    <string name="location_format">Latitude: %1$.6f\nLongitude: %2$.6f</string>
    <string name="location_placeholder">Location will appear here</string>
</resources>
```

| String Resource | Used In | Description |
|----------------|---------|-------------|
| `app_name` | Manifest | Application name shown on device |
| `get_location` | `fragment_first.xml` | Button label |
| `location_format` | `FirstFragment.java` | Formats latitude and longitude (6 decimal places) |
| `location_placeholder` | `fragment_first.xml` | Default TextView text before location is fetched |
| `location_not_found` | `FirstFragment.java` | Shown when GPS returns no location |
| `permission_denied` | `FirstFragment.java` | Toast when user denies location permission |
| `previous` | `fragment_second.xml` | Back navigation button label |
| `lorem_ipsum` | `fragment_second.xml` | Sample scrollable text on second screen |

### 5.2 themes.xml

**Path:** `app/src/main/res/values/themes.xml`

**Purpose:** Defines the visual theme applied to the entire application.

```xml
<resources xmlns:tools="http://schemas.android.com/tools">
    <!-- Base application theme. -->
    <style name="Base.Theme.GPSLocation" parent="Theme.Material3.DayNight.NoActionBar">
        <!-- Customize your light theme here. -->
        <!-- <item name="colorPrimary">@color/my_light_primary</item> -->
    </style>

    <style name="Theme.GPSLocation" parent="Base.Theme.GPSLocation" />
</resources>
```

Uses **Material Design 3** with no built-in ActionBar (custom Toolbar is used instead).

### 5.3 dimens.xml

**Path:** `app/src/main/res/values/dimens.xml`

**Purpose:** Defines reusable dimension values.

```xml
<resources>
    <dimen name="fab_margin">16dp</dimen>
</resources>
```

The `fab_margin` value sets the right margin for the Floating Action Button.

### 5.4 colors.xml

**Path:** `app/src/main/res/values/colors.xml`

**Purpose:** Defines color constants for the application.

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="black">#FF000000</color>
    <color name="white">#FFFFFFFF</color>
</resources>
```

---

## 6. Layout Files (XML - res/layout/)

Layout files define the visual structure of each screen.

### 6.1 activity_main.xml

**Path:** `app/src/main/res/layout/activity_main.xml`  
**Used by:** `MainActivity.java`

**Purpose:** Root layout containing the Toolbar, main content area, and Floating Action Button.

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    tools:context=".MainActivity">

    <com.google.android.material.appbar.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:fitsSystemWindows="true">

        <com.google.android.material.appbar.MaterialToolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize" />

    </com.google.android.material.appbar.AppBarLayout>

    <include layout="@layout/content_main" />

    <com.google.android.material.floatingactionbutton.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|end"
        android:layout_marginEnd="@dimen/fab_margin"
        android:layout_marginBottom="16dp"
        app:srcCompat="@android:drawable/ic_dialog_email" />

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

**View Hierarchy:**

```
CoordinatorLayout (id: main)
|-- AppBarLayout
|   +-- MaterialToolbar (id: toolbar)
|-- content_main.xml (included)
+-- FloatingActionButton (id: fab)
```

| View | ID | Role |
|------|----|------|
| `CoordinatorLayout` | `main` | Root container |
| `MaterialToolbar` | `toolbar` | Top app bar / action bar |
| `include content_main` | - | Embeds fragment container |
| `FloatingActionButton` | `fab` | Floating button (bottom-right) |

### 6.2 content_main.xml

**Path:** `app/src/main/res/layout/content_main.xml`  
**Used by:** Included in `activity_main.xml`

**Purpose:** Hosts the Navigation Component fragment container where fragments are displayed.

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layout_behavior="@string/appbar_scrolling_view_behavior">

    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/nav_host_fragment_content_main"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:defaultNavHost="true"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:navGraph="@navigation/nav_graph" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

| Attribute | Explanation |
|-----------|-------------|
| `NavHostFragment` | Manages which fragment is currently visible |
| `app:defaultNavHost="true"` | Handles the device Back button |
| `app:navGraph` | Links to `nav_graph.xml` |

### 6.3 fragment_first.xml

**Path:** `app/src/main/res/layout/fragment_first.xml`  
**Used by:** `FirstFragment.java`

**Purpose:** UI layout for the GPS location screen.

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp"
    tools:context=".FirstFragment">

    <Button
        android:id="@+id/button_first"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/get_location"
        app:layout_constraintBottom_toTopOf="@id/textview_first"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_chainStyle="packed" />

    <TextView
        android:id="@+id/textview_first"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="24dp"
        android:text="@string/location_placeholder"
        android:textSize="18sp"
        android:textAlignment="center"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/button_first" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

**Screen Layout:**

```
+----------------------------------+
|                                  |
|       [ Get Location ]           |  <- button_first
|                                  |
|   Location will appear here      |  <- textview_first
|                                  |
+----------------------------------+
```

| View | ID | Role |
|------|----|------|
| `Button` | `button_first` | Triggers GPS location fetch |
| `TextView` | `textview_first` | Displays latitude and longitude |

### 6.4 fragment_second.xml

**Path:** `app/src/main/res/layout/fragment_second.xml`  
**Used by:** `SecondFragment.java`

**Purpose:** UI layout for the secondary screen with a back button and scrollable text.

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.core.widget.NestedScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".SecondFragment">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:padding="16dp">

        <Button
            android:id="@+id/button_second"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/previous"
            app:layout_constraintBottom_toTopOf="@id/textview_second"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

        <TextView
            android:id="@+id/textview_second"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:text="@string/lorem_ipsum"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/button_second" />
    </androidx.constraintlayout.widget.ConstraintLayout>
</androidx.core.widget.NestedScrollView>
```

| View | ID | Role |
|------|----|------|
| `NestedScrollView` | - | Enables scrolling for long content |
| `Button` | `button_second` | Navigates back to FirstFragment |
| `TextView` | `textview_second` | Displays placeholder lorem ipsum text |

---

## 7. Java Source Files (Top-to-Bottom Flow)

This section presents the Java source code in the order it executes during application runtime.

### 7.1 MainActivity.java

**Path:** `app/src/main/java/com/example/gpslocation/MainActivity.java`

**Role:** Application entry point. Loads the main layout, sets up the Toolbar, Navigation Controller, and FAB.

```java
package com.example.gpslocation;

import android.os.Bundle;

import androidx.activity.EdgeToEdge;

import com.google.android.material.snackbar.Snackbar;

import androidx.appcompat.app.AppCompatActivity;

import android.view.View;

import androidx.core.graphics.Insets;
import androidx.core.view.ViewCompat;
import androidx.core.view.WindowInsetsCompat;
import androidx.navigation.NavController;
import androidx.navigation.fragment.NavHostFragment;
import androidx.navigation.ui.AppBarConfiguration;
import androidx.navigation.ui.NavigationUI;

import com.example.gpslocation.databinding.ActivityMainBinding;

import android.view.Menu;
import android.view.MenuItem;

public class MainActivity extends AppCompatActivity {

    private AppBarConfiguration appBarConfiguration;
    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        EdgeToEdge.enable(this);

        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        ViewCompat.setOnApplyWindowInsetsListener(binding.main, (v, insets) -> {
            Insets systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars());
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom);
            return insets;
        });
        setSupportActionBar(binding.toolbar);

        NavHostFragment navHostFragment = (NavHostFragment) getSupportFragmentManager()
                .findFragmentById(R.id.nav_host_fragment_content_main);
        if (navHostFragment != null) {
            NavController navController = navHostFragment.getNavController();
            appBarConfiguration = new AppBarConfiguration.Builder(navController.getGraph()).build();
            NavigationUI.setupActionBarWithNavController(this, navController, appBarConfiguration);
        }

        binding.fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Snackbar.make(view, "Replace with your own action", Snackbar.LENGTH_LONG)
                        .setAnchorView(R.id.fab)
                        .setAction("Action", null).show();
            }
        });
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        int id = item.getItemId();

        if (id == R.id.action_settings) {
            return true;
        }

        return super.onOptionsItemSelected(item);
    }

    @Override
    public boolean onSupportNavigateUp() {
        NavHostFragment navHostFragment = (NavHostFragment) getSupportFragmentManager()
                .findFragmentById(R.id.nav_host_fragment_content_main);
        if (navHostFragment != null) {
            NavController navController = navHostFragment.getNavController();
            return NavigationUI.navigateUp(navController, appBarConfiguration)
                    || super.onSupportNavigateUp();
        }
        return super.onSupportNavigateUp();
    }
}
```

**Method-by-Method Explanation:**

| Method | When Called | What It Does |
|--------|-------------|--------------|
| `onCreate()` | Activity starts | Inflates layout, sets toolbar, initializes navigation, sets FAB listener |
| `onCreateOptionsMenu()` | Menu created | Inflates `menu_main.xml` into toolbar |
| `onOptionsItemSelected()` | Menu item tapped | Handles Settings menu click |
| `onSupportNavigateUp()` | Up arrow tapped | Navigates back in fragment stack |

**onCreate() Execution Steps:**

| Step | Statement | Result |
|------|-----------|--------|
| 1 | `super.onCreate(savedInstanceState)` | Initializes activity |
| 2 | `EdgeToEdge.enable(this)` | Enables full-screen display |
| 3 | `ActivityMainBinding.inflate(...)` | Loads `activity_main.xml` |
| 4 | `setContentView(binding.getRoot())` | Displays layout on screen |
| 5 | `setSupportActionBar(binding.toolbar)` | Activates toolbar |
| 6 | Find `NavHostFragment` | Loads `FirstFragment` as start destination |
| 7 | `setupActionBarWithNavController(...)` | Syncs toolbar title with current fragment |
| 8 | `fab.setOnClickListener(...)` | Shows Snackbar on FAB click |

---

### 7.2 FirstFragment.java

**Path:** `app/src/main/java/com/example/gpslocation/FirstFragment.java`

**Role:** Core GPS logic. Requests location permissions and displays coordinates.

```java
package com.example.gpslocation;

import android.Manifest;
import android.content.pm.PackageManager;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Toast;

import androidx.activity.result.ActivityResultLauncher;
import androidx.activity.result.contract.ActivityResultContracts;
import androidx.annotation.NonNull;
import androidx.core.content.ContextCompat;
import androidx.fragment.app.Fragment;

import com.example.gpslocation.databinding.FragmentFirstBinding;
import com.google.android.gms.location.FusedLocationProviderClient;
import com.google.android.gms.location.LocationServices;

public class FirstFragment extends Fragment {

    private FragmentFirstBinding binding;
    private FusedLocationProviderClient fusedLocationClient;

    private final ActivityResultLauncher<String[]> requestPermissionLauncher =
            registerForActivityResult(new ActivityResultContracts.RequestMultiplePermissions(), isGranted -> {
                if (isGranted.containsValue(true)) {
                    getLastLocation();
                } else {
                    Toast.makeText(getContext(), R.string.permission_denied, Toast.LENGTH_SHORT).show();
                }
            });

    @Override
    public View onCreateView(
            @NonNull LayoutInflater inflater, ViewGroup container,
            Bundle savedInstanceState
    ) {
        binding = FragmentFirstBinding.inflate(inflater, container, false);
        fusedLocationClient = LocationServices.getFusedLocationProviderClient(requireActivity());
        return binding.getRoot();
    }

    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);

        binding.buttonFirst.setText(R.string.get_location);
        binding.buttonFirst.setOnClickListener(v -> getLastLocation());
    }

    private void getLastLocation() {
        if (ContextCompat.checkSelfPermission(requireContext(), Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED &&
                ContextCompat.checkSelfPermission(requireContext(), Manifest.permission.ACCESS_COARSE_LOCATION) != PackageManager.PERMISSION_GRANTED) {

            requestPermissionLauncher.launch(new String[]{
                    Manifest.permission.ACCESS_FINE_LOCATION,
                    Manifest.permission.ACCESS_COARSE_LOCATION
            });
            return;
        }

        fusedLocationClient.getLastLocation()
                .addOnSuccessListener(requireActivity(), location -> {
                    if (location != null) {
                        String locationText = getString(R.string.location_format,
                                location.getLatitude(), location.getLongitude());
                        binding.textviewFirst.setText(locationText);
                    } else {
                        binding.textviewFirst.setText(R.string.location_not_found);
                    }
                });
    }

    @Override
    public void onDestroyView() {
        super.onDestroyView();
        binding = null;
    }

}
```

**Method-by-Method Explanation:**

| Method | When Called | What It Does |
|--------|-------------|--------------|
| `onCreateView()` | Fragment UI created | Inflates `fragment_first.xml`, initializes location client |
| `onViewCreated()` | View ready | Sets button text and click listener |
| `getLastLocation()` | Button clicked | Checks permissions, fetches GPS location, updates TextView |
| `onDestroyView()` | Fragment view destroyed | Clears binding to prevent memory leaks |

**getLastLocation() Logic:**

| Step | Action |
|------|--------|
| 1 | Check if `ACCESS_FINE_LOCATION` or `ACCESS_COARSE_LOCATION` is granted |
| 2 | If not granted, launch permission request dialog |
| 3 | If granted, call `fusedLocationClient.getLastLocation()` |
| 4 | On success: if location exists, format and display lat/long; else show error message |

**Key Classes Used:**

| Class | Package | Purpose |
|-------|---------|---------|
| `FusedLocationProviderClient` | `com.google.android.gms.location` | Retrieves device location |
| `ActivityResultLauncher` | `androidx.activity.result` | Handles runtime permission result |
| `FragmentFirstBinding` | `com.example.gpslocation.databinding` | View Binding for `fragment_first.xml` |

---

### 7.3 SecondFragment.java

**Path:** `app/src/main/java/com/example/gpslocation/SecondFragment.java`

**Role:** Secondary screen demonstrating fragment navigation back to FirstFragment.

```java
package com.example.gpslocation;

import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;

import androidx.annotation.NonNull;
import androidx.fragment.app.Fragment;
import androidx.navigation.fragment.NavHostFragment;

import com.example.gpslocation.databinding.FragmentSecondBinding;

public class SecondFragment extends Fragment {

    private FragmentSecondBinding binding;

    @Override
    public View onCreateView(
            @NonNull LayoutInflater inflater, ViewGroup container,
            Bundle savedInstanceState
    ) {

        binding = FragmentSecondBinding.inflate(inflater, container, false);
        return binding.getRoot();

    }

    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);

        binding.buttonSecond.setOnClickListener(v ->
                NavHostFragment.findNavController(SecondFragment.this)
                        .navigate(R.id.action_SecondFragment_to_FirstFragment)
        );
    }

    @Override
    public void onDestroyView() {
        super.onDestroyView();
        binding = null;
    }

}
```

**Method-by-Method Explanation:**

| Method | When Called | What It Does |
|--------|-------------|--------------|
| `onCreateView()` | Fragment UI created | Inflates `fragment_second.xml` |
| `onViewCreated()` | View ready | Sets Previous button to navigate to FirstFragment |
| `onDestroyView()` | Fragment view destroyed | Clears binding reference |

When the user taps **Previous**, `NavController.navigate(R.id.action_SecondFragment_to_FirstFragment)` returns to the GPS screen.

---

## 8. Complete Lifecycle Flow (Summary)

The following diagram summarizes the entire application lifecycle from launch to location display:

```
APP LAUNCH (User taps app icon)
    |
    v
[AndroidManifest.xml]
    |-- Declares ACCESS_FINE_LOCATION, ACCESS_COARSE_LOCATION
    |-- Registers MainActivity as LAUNCHER
    |
    v
[MainActivity.onCreate()]
    |-- Inflate activity_main.xml
    |   |-- MaterialToolbar (toolbar)
    |   |-- content_main.xml (included)
    |   |   +-- NavHostFragment (nav_host_fragment_content_main)
    |   +-- FloatingActionButton (fab)
    |-- Setup Navigation Controller
    |-- Load nav_graph.xml
    |   +-- startDestination = FirstFragment
    |
    v
[FirstFragment.onCreateView()]
    |-- Inflate fragment_first.xml
    |-- Initialize FusedLocationProviderClient
    |
    v
[FirstFragment.onViewCreated()]
    |-- Set button text: "Get Location"
    |-- Set button click listener -> getLastLocation()
    |
    v
[USER ACTION: Tap "Get Location"]
    |
    v
[FirstFragment.getLastLocation()]
    |-- Check runtime permissions
    |   |-- NOT granted -> Request permissions -> User response
    |   +-- GRANTED -> Continue
    |-- fusedLocationClient.getLastLocation()
    |-- addOnSuccessListener callback
    |   |-- location != null -> Display lat/long in textview_first
    |   +-- location == null -> Display "Location not found"
    |
    v
[RESULT DISPLAYED ON SCREEN]
    Latitude: 12.345678
    Longitude: 77.654321
```

---

## 9. File Relationship Diagram

The diagram below shows how all project files connect to each other:

```
                    AndroidManifest.xml
                    (permissions + launcher)
                            |
                            v
                    MainActivity.java
                            |
            +---------------+---------------+
            |                               |
            v                               v
    activity_main.xml              menu_main.xml
            |
    +-------+-------+-------+
    |       |       |       |
    v       v       v       v
 toolbar  content  fab   strings.xml
            |
            v
    content_main.xml
            |
            v
 nav_host_fragment_content_main
            |
            v
      nav_graph.xml
       /            \
      v              v
FirstFragment    SecondFragment
    .java            .java
      |                |
      v                v
fragment_first    fragment_second
    .xml               .xml
      |
      v
 play-services-location (Google GPS API)
 strings.xml (location_format, etc.)
```

**Dependency Table:**

| Java File | Layout File | Resource Files | External API |
|-----------|-------------|----------------|--------------|
| `MainActivity.java` | `activity_main.xml`, `content_main.xml` | `menu_main.xml`, `strings.xml`, `dimens.xml` | Navigation Component |
| `FirstFragment.java` | `fragment_first.xml` | `strings.xml` | FusedLocationProviderClient |
| `SecondFragment.java` | `fragment_second.xml` | `strings.xml` | Navigation Component |

---

## 10. Lab Exercises for Students

Complete the following exercises to reinforce your understanding of the GPS Location application.

| # | Exercise | Expected Outcome | Hint |
|---|----------|------------------|------|
| 1 | Open the project in Android Studio and run it on an emulator or physical device | App launches and shows "Get Location" button | Enable GPS/Location in device settings |
| 2 | Tap "Get Location" and grant location permission when prompted | Latitude and longitude appear on screen | Use emulator Extended Controls -> Location to set coordinates |
| 3 | Deny location permission and tap "Get Location" again | Toast message "Permission denied" appears | Revoke permission in App Settings if already granted |
| 4 | Disable GPS and tap "Get Location" | Message "Location not found. Make sure GPS is enabled." appears | Turn off location in device settings |
| 5 | Add a "Next" button in `FirstFragment` to navigate to `SecondFragment` | Tapping Next opens the second screen | Use `NavController.navigate(R.id.action_FirstFragment_to_SecondFragment)` |
| 6 | Modify `location_format` in `strings.xml` to also display location accuracy | TextView shows latitude, longitude, and accuracy in meters | Use `location.getAccuracy()` in `FirstFragment.java` |
| 7 | Change the FAB action in `MainActivity` to show the current app version | Snackbar displays version name from `BuildConfig` | Replace Snackbar message text in `onCreate()` |
| 8 | Draw the project folder structure from memory | Correct tree with all Java and XML files | Refer to Section 2 |
| 9 | Explain the purpose of each line in `AndroidManifest.xml` | Written explanation of permissions and intent-filter | Refer to Section 4.1 |
| 10 | Trace the complete flow when the user taps "Get Location" | Step-by-step written trace from button click to TextView update | Refer to Sections 3 and 7.2 |

---

## 11. Prerequisites Checklist

Before starting this lab, ensure the following requirements are met:

**Software Requirements:**

- [ ] Android Studio (latest stable version) installed
- [ ] Java Development Kit (JDK 11 or higher) configured
- [ ] Android SDK Platform API 26 or higher installed
- [ ] Google Play Services SDK available (for location API)

**Hardware / Emulator Requirements:**

- [ ] Android device with GPS **OR** Android Emulator with Google Play image
- [ ] Location/GPS enabled on the test device
- [ ] Internet connection for Gradle dependency download

**Knowledge Prerequisites:**

- [ ] Basic understanding of Java programming
- [ ] Familiarity with XML syntax
- [ ] Understanding of Android Activity lifecycle
- [ ] Awareness of mobile app permissions concept

**Project Setup Steps:**

1. Open Android Studio
2. Select **Open** and navigate to the GPSLocation project folder
3. Wait for Gradle sync to complete
4. Connect a device or start an emulator
5. Click **Run** (green play button) to build and install the app

---

## 12. Troubleshooting

Use this table to diagnose and fix common problems encountered during the lab.

| # | Problem | Possible Cause | Solution |
|---|---------|----------------|----------|
| 1 | App shows "Location not found" | GPS is disabled on device | Enable Location in device Settings; in emulator use Extended Controls -> Location |
| 2 | Toast shows "Permission denied" | User denied location permission | Go to Settings -> Apps -> GPSLocation -> Permissions -> Allow Location |
| 3 | Gradle sync fails | Missing SDK or network issue | Check internet connection; install required SDK via SDK Manager |
| 4 | `play-services-location` not found | Dependency not downloaded | File -> Sync Project with Gradle Files |
| 5 | Blank white screen on launch | Navigation graph misconfigured | Verify `startDestination` in `nav_graph.xml` points to `FirstFragment` |
| 6 | App crashes on "Get Location" | Permission not declared in manifest | Verify `ACCESS_FINE_LOCATION` and `ACCESS_COARSE_LOCATION` in `AndroidManifest.xml` |
| 7 | Build error: View Binding | View Binding not enabled | Confirm `viewBinding = true` in `app/build.gradle.kts` |
| 8 | Emulator has no Google Play | Wrong emulator image selected | Create AVD with a system image that includes Google Play Store |
| 9 | Coordinates show 0.0, 0.0 | Default emulator location not set | Set a valid latitude/longitude in emulator Extended Controls |
| 10 | Toolbar title not updating | Navigation not linked to ActionBar | Verify `NavigationUI.setupActionBarWithNavController()` in `MainActivity` |

### 12.1 Emulator Location Setup

To set a test location in the Android Emulator:

1. Run the app on the emulator
2. Click the **...** (Extended Controls) button on the emulator toolbar
3. Select **Location** from the left panel
4. Enter latitude and longitude values (e.g., 12.9716, 77.5946 for Bangalore)
5. Click **Send**
6. Return to the app and tap **Get Location**

### 12.2 Permission Reset

To test permission flow again:

1. Open device **Settings**
2. Go to **Apps** -> **GPSLocation**
3. Tap **Permissions** -> **Location**
4. Select **Don't allow** or **Ask every time**
5. Relaunch the app and tap **Get Location**

---

*End of Lab Manual*
