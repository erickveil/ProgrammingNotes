---
title: Kotlin Jetpack Navigation
layout: note
repository:
  - https://github.com/erickveil/JetpackNavigation 
  - https://github.com/erickveil/FragmentNavigation
tags:
  - android
  - jetpack
  - kotlin
  - navigation
---

# Jetpack Compose vs Traditional XML

In Jetpack Compose, the concept of activities and fragments is replaced by **composable functions** that define UI components. When you're using Jetpack Compose, you're working at a higher level of abstraction where the entire UI is built through these composable functions, which can be assembled into complex UIs by combining simpler, reusable composable pieces.

Here's a quick rundown on the terminology and how it shifts when you move from traditional Android UI development (using XML layouts and fragments) to Jetpack Compose:

### Traditional Android Development:
- **Activity**: A single, focused thing that the user can do. Most commonly, an activity hosts fragments or serves as a container for the entire UI.
- **Fragment**: A modular section of an activity, which has its own lifecycle, receives its own input events, and can be added or removed while the activity is running.

### Jetpack Compose:
- **Composable Function**: Functions that define the UI by describing how it should look based on the current state of the application, and can react to state changes over time. These functions are the building blocks of your UI.
- **Screen or View**: In Compose, what was previously referred to as an "activity" or "fragment" could now conceptually be referred to as a "screen" or "view". Each screen or view is simply a composable function that typically represents a full screen of content.
- **Navigation Graph**: Similar to the traditional approach, you define a navigation graph in Compose. However, instead of defining navigation between fragments, you define navigation between composable destinations.

### Implementing Navigation in Jetpack Compose:
In Jetpack Compose, navigation between different screens is handled by the Navigation component specifically designed for Compose, using the following elements:

- **NavController**: Manages the navigation stack within a NavHost.
- **NavHost**: A composable that displays destinations from your navigation graph and manages navigation between them.

### Example:
When you talk about navigation in a Compose-based app, you might say:

- "Navigate from the MainScreen composable to the DetailScreen composable."
- "Define a NavHost in the main activity that manages navigation between composables."

Thus, the vocabulary shifts from using "activities" and "fragments" to simply discussing "composable functions" or "screens/views" when you describe different parts of the UI in Jetpack Compose. The focus in Compose is more on the seamless integration of UI components and less on the container that holds them, emphasizing a fluid, state-driven approach to UI design.

# A Simple Jetpack Compose Project With Navigation

Let's set up a simple Jetpack Compose application with three screens: PinkScreen, GreenScreen, and BlueScreen, each with navigation as you described. Here's how you can set it up:

### Step 1: Setup Your Project
Add the necessary dependencies in your `build.gradle` (Module: app) file to include Jetpack Compose and Navigation Compose:

```groovy
android {
    ...
    buildFeatures {
        // Enables Jetpack Compose for this module
        compose true
    }

    composeOptions {
        kotlinCompilerExtensionVersion = '1.3.1' // use the latest version available
        kotlinCompilerVersion '1.8.0' // match your Kotlin version
    }
}

dependencies {
    implementation 'androidx.core:core-ktx:1.8.0'
    implementation 'androidx.compose.ui:ui:1.3.1'
    implementation 'androidx.compose.material:material:1.3.1'
    implementation 'androidx.compose.ui:ui-tooling-preview:1.3.1'
    implementation 'androidx.lifecycle:lifecycle-runtime-ktx:2.5.1'
    implementation 'androidx.activity:activity-compose:1.6.0'
    implementation 'androidx.navigation:navigation-compose:2.5.2'
}
```

### Step 2: Define the Composable Screens
Define each screen with the specified UI components and color schemes.

```kotlin
@Composable
fun PinkScreen(navController: NavController) {
    ScreenLayout(color = Color(0xFFFFC0CB), screenLabel = "Pink Screen", navController = navController)
}

@Composable
fun GreenScreen(navController: NavController) {
    ScreenLayout(color = Color(0xFF00FF00), screenLabel = "Green Screen", navController = navController)
}

@Composable
fun BlueScreen(navController: NavController) {
    ScreenLayout(color = Color(0xFF0000FF), screenLabel = "Blue Screen", navController = navController)
}

@Composable
fun ScreenLayout(color: Color, screenLabel: String, navController: NavController) {
    Box(modifier = Modifier
        .fillMaxSize()
        .background(color))

    Column(
        modifier = Modifier.fillMaxSize(),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text(
            text = screenLabel,
            color = Color.White,
            fontSize = 24.sp,
            fontWeight = FontWeight.Bold
        )

        if (screenLabel != "Pink Screen") {
            Button(
                onClick = {
                    navController.navigate("pink")
                },
                colors = ButtonDefaults.buttonColors(containerColor = Color(0xFFFFC0CB))
            ) {
                Text("Go to Pink")
            }
        }

        if (screenLabel != "Green Screen") {
            Button(
                onClick = {
                    navController.navigate("green")
                },
                colors = ButtonDefaults.buttonColors(containerColor = Color(0xFF00FF00))
            ) {
                Text("Go to Green")
            }
        }

        if (screenLabel != "Blue Screen") {
            Button(
                onClick = {
                    navController.navigate("blue")
                },
                colors = ButtonDefaults.buttonColors(containerColor = Color(0xFF0000FF))
            ) {
                Text("Go to Blue")
            }
        }

        // Show the back button only if there is a destination to go back to
        if (navController.previousBackStackEntry != null) {
            Button(
                onClick = { navController.popBackStack() },
                colors = ButtonDefaults.buttonColors(containerColor = Color.Gray)
            ) {
                Text("Back")
            }
        }
    }
}
```

### Step 3: Set Up Navigation
Define a `NavHost` in your main activity that manages navigation between these composables.

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            val navController = rememberNavController()
            NavHost(navController = navController, startDestination = "pink") {
                composable("pink") { PinkScreen(navController) }
                composable("green") { GreenScreen(navController) }
                composable("blue") { BlueScreen(navController) }
            }
        }
    }
}
```

### Additional Notes:
- **Navigation Routes**: Each screen is identified by a route (e.g., "pink", "green", "blue"). Navigation actions are bound to these routes.
- **Back Button**: The `Back` button uses `navController.popBackStack()` to navigate back to the previous screen, leveraging the built-in back stack management of the Navigation component.

This setup should create a simple navigational flow between three color-coded screens, each with navigation and back functionality using Jetpack Compose and the Navigation component.

# Intent Based Navigation - Navigating Activities

Jetpack Navigation is primarily designed to handle navigation within a single activity using fragments, not multiple activities. The recommended modern Android architecture suggests moving towards a single activity design with multiple fragments to handle different screens in your app. This model provides a more consistent and predictable handling of the Android lifecycle and back stack.

However, if you prefer or need to use multiple activities, you can still manage navigation between them using traditional intents. Jetpack Navigation isn't necessary or particularly advantageous for navigating between different activities.

### Traditional Intent-Based Navigation
Here’s how you could implement the navigation in a traditional way using intents, keeping in line with your setup of three separate activities:

#### MainActivity
In `MainActivity`, you would have a button that, when clicked, starts `SecondActivity`. You would also have a back button that simply finishes the current activity, thereby taking you back to the previous one in the stack.

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        findViewById<Button>(R.id.buttonToSecond).setOnClickListener {
            startActivity(Intent(this, SecondActivity::class.java))
        }

        findViewById<Button>(R.id.backButton).setOnClickListener {
            finish()
        }
    }
}
```

#### SecondActivity
In `SecondActivity`, similar to `MainActivity`, you will have buttons to navigate to `ThirdActivity` and to go back to the previous activity.

```kotlin
class SecondActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_second)

        findViewById<Button>(R.id.buttonToThird).setOnClickListener {
            startActivity(Intent(this, ThirdActivity::class.java))
        }

        findViewById<Button>(R.id.backButton).setOnClickListener {
            finish()
        }
    }
}
```

#### ThirdActivity
Finally, `ThirdActivity` would have a button to go back to `MainActivity` (not typically recommended as it creates a new instance of `MainActivity` rather than navigating back through the stack) and a back button to return to the previous activity.

```kotlin
class ThirdActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_third)

        findViewById<Button>(R.id.buttonToMain).setOnClickListener {
            startActivity(Intent(this, MainActivity::class.java))
        }

        findViewById<Button>(R.id.backButton).setOnClickListener {
            finish()
        }
    }
}
```

### Using Jetpack Navigation in a Single Activity Setup
If you decide to adopt the single activity approach recommended by modern Android best practices, you would convert each of your activities into fragments. You'd then use Jetpack Navigation to manage transitions between these fragments, handle passing data between them, and respect the fragment lifecycle more cleanly than with multiple activities.

Implementing Jetpack Navigation in a single activity setup involves creating a `NavHostFragment` in your main activity and defining a navigation graph that outlines the possible navigation paths between fragments. This would generally make your app more maintainable and your navigation code simpler and more centralized.

If you need further details on how to transition your app to a single activity architecture using Jetpack Navigation, I'm here to guide you through that process!

# Fragment Based XML Jetpack Navigation

To implement the same basic navigation idea using fragments in a single activity setup with Jetpack Navigation, you'll follow these key steps:

1. **Set Up Your Project**: Include Jetpack Navigation dependencies and configure your Android project.
2. **Create the Fragments**: Each screen (MainActivity, SecondActivity, ThirdActivity) will be represented as a Fragment.
3. **Design the Navigation Graph**: Define how fragments are interconnected.
4. **Implement the Navigation**: Use NavController to handle fragment transitions.

### Step 1: Set Up Your Android Project
First, make sure your project includes the necessary dependencies for Jetpack Navigation:

1. **Add Navigation dependencies to your `build.gradle` (Module: app):**

   ```kotlin
   dependencies {
		implementation("androidx.navigation:navigation-fragment-ktx:+")  
		implementation("androidx.navigation:navigation-ui-ktx:+")
   }
   ```

### Step 2: Create Fragment Layouts
Define the layouts for each of your fragments in XML. Here’s an example layout for `PinkFragment`:

**res/layout/fragment_pink.xml**
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp"
    android:background="#FFC0CB">

    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Pink Fragment"
        android:textSize="24sp"
        android:textColor="#FFFFFF"/>

    <Button
        android:id="@+id/button_to_green"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Go to Green"
        android:layout_marginTop="16dp"/>

    <Button
        android:id="@+id/button_to_blue"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Go to Blue"
        android:layout_marginTop="8dp"/>

    <Button
        android:id="@+id/button_back"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Back"
        android:background="#808080"
        android:layout_marginTop="32dp"/>

</LinearLayout>
```

Create similar layouts for `GreenFragment` and `BlueFragment`, changing the background color and text appropriately. Note that we can use the WISIWYG in Android Studio to lay these out.

### Step 3: Create the Fragments
Define the fragment classes. Here's how you could set up `PinkFragment`:

**PinkFragment.kt**
```kotlin
class PinkFragment : Fragment() {

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.fragment_pink, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        view.findViewById<Button>(R.id.button_to_green).setOnClickListener {
            findNavController().navigate(R.id.action_pinkFragment_to_greenFragment)
        }
        view.findViewById<Button>(R.id.button_to_blue).setOnClickListener {
            findNavController().navigate(R.id.action_pinkFragment_to_blueFragment)
        }
        view.findViewById<Button>(R.id.button_back).setOnClickListener {
            findNavController().popBackStack()
        }
    }
}
```

Define `GreenFragment` and `BlueFragment` in a similar way, adjusting the navigation actions as necessary.

### Step 4: Define the Navigation Graph
Create a navigation graph in the `res/navigation` directory. Just like the actual Fragments, here we can use Android Studio's WYSIWYG editor for visually laying out the Navigation Graph. It allows us to see exactly where the actions connect us.

**nav_graph.xml**
```xml
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    app:startDestination="@id/pinkFragment">

    <fragment
        android:id="@+id/pinkFragment"
        android:name="com.example.myapp.PinkFragment"
        android:label="fragment_pink"
        tools:layout="@layout/fragment_pink">
        <action
            android:id="@+id/action_pinkFragment_to_greenFragment"
            app:destination="@id/greenFragment" />
        <action
            android:id="@+id/action_pinkFragment_to_blueFragment"
            app:destination="@id/blueFragment" />
    </fragment>

    <!-- Define GreenFragment and BlueFragment with similar actions -->
</navigation>
```

### Step 5: Set Up the NavHostFragment
Ensure your main activity's layout includes a `NavHostFragment` that references the navigation graph.

**activity_main.xml**
```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/nav_host_fragment"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:navGraph="@navigation/nav_graph"
        app:defaultNavHost="true" />

</FrameLayout>
```

Note how instead of a `<Fragment>` node, we use the `<FragmentContainerView>`. This allows Android to better set up the initial fragment when the application starts (though in practice, I don't see a difference).

This setup allows each button in the fragments to navigate to the next fragment using `NavController`. You don't need a "back" button specifically because Android handles back navigation natively when using fragments and the navigation component, respecting the back stack automatically.

This approach should set you up with a basic navigational flow between fragments in a single activity using Jetpack Navigation. It's scalable, easier to manage than navigating whole Activities, and follows more modern Android development practices.
