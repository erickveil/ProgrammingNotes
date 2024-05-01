---
title: Navigating Activities and Fragments in Kotlin
layout: note
repository: https://github.com/erickveil/MultiActivity
tags:
  - android
  - kotlin
  - xml
  - UI
---

# Switching Between Activities

To switch between `MainActivity` and `SecondActivity` in your Kotlin Android app, you need to set up the buttons in each activity to start the other activity when clicked. Here’s a step-by-step guide on how to do this:

### 1. Define the Buttons in XML

First, ensure each of your activity layouts (XML files) includes a button. Here’s an example for both:

**activity_main.xml** (for `MainActivity`):
```xml
<Button
    android:id="@+id/buttonToSecond"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Go to Second Activity" />
```

**activity_second.xml** (for `SecondActivity`):
```xml
<Button
    android:id="@+id/buttonToMain"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Go to Main Activity" />
```

### 2. Set Up the Button Listeners in Each Activity

In each activity, you need to set up the button to start the other activity when it’s clicked. 

**MainActivity.kt**:
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Initialize the button using findViewById
        val buttonToSecond = findViewById<Button>(R.id.buttonToSecond)
        buttonToSecond.setOnClickListener {
            val intent = Intent(this, SecondActivity::class.java)
            startActivity(intent)
        }
    }
}
```

**SecondActivity.kt**:
```kotlin
class SecondActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_second)

        // Initialize the button using findViewById
        val buttonToMain = findViewById<Button>(R.id.buttonToMain)
        buttonToMain.setOnClickListener {
            val intent = Intent(this, MainActivity::class.java)
            startActivity(intent)
        }
    }
}
```

### 3. Ensure Both Activities Are Declared in the AndroidManifest.xml

Both activities must be declared in `AndroidManifest.xml`. The `MainActivity` is usually set as the LAUNCHER activity, and `SecondActivity` just needs a standard declaration:

```xml
<application
    ... >
    <activity android:name=".MainActivity">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
    </activity>
    <activity android:name=".SecondActivity" />
</application>
```

### 4. Run Your App

Build and run your app. The buttons should now switch between the two activities as expected.

This setup uses explicit intents to navigate between activities, which is appropriate when you know the specific activities that should be started. Make sure you test the app to ensure the transitions are smooth and the app behaves as intended.

# About Fragments

Fragments are versatile and reusable components in Android development that you can use within an activity. They are particularly useful for creating dynamic and flexible user interfaces and managing user interactions within an activity. Here are some scenarios where using a fragment instead of a new activity might be the better choice:

### 1. Modular UI Components
Fragments are ideal for situations where you want to create modular, adaptable user interface components that can be reused across different activities. For example, if you have a navigation drawer or a common action panel that needs to appear across several activities, using a fragment for these UI parts makes it easier to maintain and manage them.

### 2. Adapting to Different Screen Sizes
Fragments are extremely useful for handling UIs that must adapt to different screen orientations and sizes, such as tablets versus smartphones. You can design one activity to host multiple fragments, and depending on the screen size and orientation, you can show or hide fragments as necessary. For example, in landscape mode on a tablet, you might show a list and details side by side using two fragments.

### 3. Managing Complex User Interfaces
In complex UIs where different parts of the screen change independently of others, fragments can be a good choice. They can each maintain their own lifecycle, state, input events, and transitions, which simplifies the management of complex interactions within an activity.

### 4. Retaining State Across Configuration Changes
Fragments can be designed to retain their state across configuration changes like screen rotations by setting `setRetainInstance(true)`. This can be advantageous when you need to retain complex transient state such as a network connection or a large data set that should not be reloaded or recreated each time the user rotates the device.

### 5. Multi-pane Layouts
For creating multi-pane layouts especially on tablets, fragments are incredibly beneficial. You can use a single activity layout that includes several fragments, each managing a different aspect of the user interface. This way, the user can interact with multiple panels on the same screen seamlessly.

### 6. Improved Lifecycle Management
Fragments also have their own lifecycle, which is closely tied to their host activity's lifecycle but is slightly more detailed because it adds additional states to manage the fragment’s view hierarchy. This makes it easier to manage UI components that need to respond dynamically to lifecycle changes without putting all the responsibility in the activity’s lifecycle methods.

### Example Use Case
Consider a news application:
- **Smartphone**: The app shows a list of articles in one fragment. When a user selects an article, it replaces the list fragment with a detail fragment that shows the article.
- **Tablet**: The app shows both the list and the detail fragments side by side on the same screen.

Using fragments in these ways helps make your app more robust, scalable, and maintainable. It allows you to build sophisticated UIs that can adapt to various device configurations and user interactions.

## Headless Fragments

You can create an invisible fragment in Android, often referred to as a headless fragment. A headless fragment doesn't have a user interface (UI) component and is not attached to any layout file. Instead, it is used to retain data, manage long-running tasks, or handle background operations across configuration changes like screen rotations.

### Purpose of Headless Fragments
Headless fragments are useful for tasks that need to be retained across the lifecycle of an activity without requiring a visual representation. These tasks include:

- Managing background tasks such as data loading or asynchronous operations.
- Handling listeners or callbacks from activities and other fragments.
- Storing complex data structures that need to persist across configuration changes.

### Creating a Headless Fragment
To create a headless fragment, you would typically:

1. **Define the Fragment Class**: Create a standard fragment class but do not associate it with any XML layout.

```kotlin
import android.os.Bundle
import androidx.fragment.app.Fragment

class DataFragment : Fragment() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Retain this fragment across configuration changes.
        retainInstance = true
    }

    // Your methods to handle background tasks, data management, etc.
}
```

2. **Add to an Activity**: You can add this fragment to your activity dynamically, making sure to not add a UI component:

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Check if the Fragment is already added to avoid adding it again on rotate.
        if (savedInstanceState == null) {
            supportFragmentManager.beginTransaction()
                .add(DataFragment(), "dataFragment")
                .commit()
        }
    }
}
```

In the above example, the `DataFragment` is added to the `MainActivity` without any UI. The fragment’s tag `"dataFragment"` is used to identify it within the fragment manager, making it easier to find and interact with if necessary.

### Benefits and Considerations
- **Lifecycle Awareness**: Since fragments have their own lifecycle, a headless fragment can handle its tasks in alignment with the activity's lifecycle, making management of processes like network calls more robust.
- **Memory Management**: The `retainInstance` property can be set to `true` to preserve the fragment during configuration changes, such as screen rotations, without needing to re-create or re-initialize the background tasks.
- **Versatility**: Headless fragments can communicate with other fragments and activities through interfaces, making them a versatile solution for managing interactions within your app.

However, with the rise of modern Android architecture components like ViewModel and LiveData, many use cases for headless fragments can now be more effectively handled using these components. They provide a cleaner, lifecycle-aware way to handle data and UI logic separation without the overhead of managing fragment transactions and lifecycle directly.

# Creating a Simple Fragment

Here's how you can create a custom fragment that behaves like a pop-up, covering the bottom third of the screen:

### 1. Define the Fragment
Create a new `Fragment` subclass. In this fragment, you'll define the UI and the functionality to dismiss the fragment.

```kotlin
import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.Button
import androidx.fragment.app.Fragment

class PopupFragment : Fragment() {

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // Inflate the layout for this fragment
        val view = inflater.inflate(R.layout.fragment_popup, container, false)

        val closeButton = view.findViewById<Button>(R.id.close_button)
        closeButton.setOnClickListener {
            // Remove the fragment
            fragmentManager?.beginTransaction()?.remove(this)?.commit()
        }

        return view
    }
}
```

### 2. Create the Fragment Layout
Create the layout XML for this fragment (`fragment_popup.xml`). This layout should be designed to cover only part of the screen, such as the bottom third.

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/popup_container"
    android:layout_width="match_parent"
    android:layout_height="250dp"
    android:layout_gravity="bottom"
    android:background="@android:color/white"
    android:elevation="4dp">

    <Button
        android:id="@+id/close_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Close"
        android:layout_gravity="center" />
</FrameLayout>
```

### 3. Add and Show the Fragment
To display this fragment over your current activity, add it to a container in your activity's layout. Ideally, you should have a `FrameLayout` that can act as a placeholder for this fragment.

**Activity Layout Example (`activity_main.xml`):**
```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fragment_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!-- Your other UI elements -->

</FrameLayout>
```

**Show the Fragment from an Activity or Another Fragment:**
```kotlin
val popupFragment = PopupFragment()
supportFragmentManager.beginTransaction()
    .add(R.id.fragment_container, popupFragment)
    .commit()
```

### 4. Adjusting Fragment Appearance
If you want the fragment to appear only as a partial overlay (e.g., covering just a portion of the screen while allowing interaction with other UI elements), you can adjust the dimensions and positioning in the layout file of the fragment (`fragment_popup.xml`). Use layout parameters like `layout_gravity` to position it at the bottom, top, or center as needed.

This setup allows you to create a fragment that functions like a pop-up, where you can control the size, position, and behavior entirely through your fragment's layout and the transaction methods used to display it. This method is more manual but gives you maximum flexibility over the appearance and functionality of the pop-up.

