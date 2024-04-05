---
title: MVI Jetpack Project Development
layout: post
date: 24-04-05
tags:
  - kotlin
  - mvi
  - android
  - jetpack
---
For this project, we're going to use a JSON file as a simple data source and write a program that rolls for the random results on the table defined in the JSON.
# Setting Up a New Project

- File -> New -> New Project -> Empty Activity

# Set Up the Project Structure

- **`   src/main/java` or `src/main/kotlin`**: Root directory for your Kotlin source files.
    - **`com/yourapplicationname`**: Your application's package.
        - **`data`**: Directory for data handling and network operations.
            - **`model`**: Contains model classes that represent the structure of your data. Your Kotlin class for the `Model` fits here.
            - **`repository`**: Contains repository classes that abstract the logic of data access and manipulation. They might interact with your model classes and data sources like databases, web services, or your JSON files.
        - **`ui`**: Directory for user interface related classes.
            - **`view`**: Contains activities, fragments, and any other views.
            - **`viewmodel`**: Contains ViewModel classes which hold the logic to manage the UI's data in response to user actions.
        - **`util`**: Utility classes that provide common functionalities across the application.
        - **`assets`**: Where we will keep data sources and other assets.

# The JSON File

In the `assets` directory, I add `lootTable.json` that looks like this:

```json
{
  "tableName": "Dungeon Loot Table",
  "description": "This table determines what loot you find in the dungeon.",
  "results": [
    "Gold coins",
    "Magic sword",
    "Potion of healing"
  ]
}
```

# The Model

### 1. Set Up the Dependencies

We will use `kotlinx` to parse our JSON eventually, so we will have to edit the gradle files.

Under Gradle Scripts, there are two `build.gradle.kts` scripts.  The one you want is marked with `(Module :app)`.

Add the following to the existing sections:

```groovy
plugins {  
id("org.jetbrains.kotlin.plugin.serialization") version "1.5.21"  
}
```


```groovy
dependencies {  
implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.2.2")  
}
```

Be sure to sync the project when you're done with editing this file.

### 2. Create the Model Class

In the `data/model` directory, I add the Kotlin class, `LootTable.kt` that looks like this:

```kotlin
package net.erickveil.mvi_table_roller.data.model  
import kotlinx.serialization.Serializable 

@Serializable
data class LootTable (  
	val tableName: String,  
	val description: String,  
	val results: List<String>  
)
```

In this model:

- `@Serializable` annotation is from `kotlinx.serialization`, indicating that this data class can be serialized and deserialized to and from JSON.
- `LootTable` is the main data class that represents your table, with properties matching the JSON keys: `tableName`, `description`, and `results`.
- `tableName` and `description` are of type `String`.
- `results` is a list of strings, each representing a possible outcome from the table.
# The View

### 1. Set Up the Gradle for Compose

We might need to add the necessary dependencies and enable Compose in the build.gradle (module-level) file.

To enable Jetpack Compose, ensure you have the following in your module-level `build.gradle`:

1. **Add the Compose Compiler dependency:** This is required for Compose to work. Ensure you're using a version compatible with your Kotlin version.

    ```groovy
    buildFeatures {
        compose = true
    }
    composeOptions {
        kotlinCompilerExtensionVersion = "1.5.1" // Use the version compatible with your setup
    }
    ```

2. **Include Compose dependencies:** You'll need the core Compose libraries, as well as any additional ones you plan to use (like Material Design components, which your code snippet indicates you are using).

    ```groovy
    dependencies {
        implementation 'androidx.compose.ui:ui:1.0.0' // Update versions as needed
        implementation 'androidx.compose.material:material:1.0.0'
        implementation 'androidx.compose.ui:ui-tooling-preview:1.0.0'
        // Other dependencies...
    }
    ```

These might already be set up correctly because of the type of new project we chose.

### 2. The View File

Now I'm going to create a basic view. It won't do anything right off the bat. We're going to use Jetpack Compose for the UI.

Let's design a simple UI with a button that the user can press to see a result from the loot table. This interface will contain a button and a text display area where the result can be shown. The functionality to display the actual result will be added later, but for now, we'll set up the basic UI elements.

We will be using [Material Design 3](https://developer.android.com/develop/ui/compose/designsystems/material3) for various UI components.

```kotlin
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material3.Button
import androidx.compose.material3.ButtonDefaults
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp

@Composable
fun LootTableUIEnhanced(onRollTable: () -> Unit, resultText: String = "Result appears here") {
    // Screen background color
    Box(modifier = Modifier
        .fillMaxWidth()
        .background(Color(0xFFEDE7F6))) {
        Column(
            modifier = Modifier.padding(16.dp),
            verticalArrangement = Arrangement.spacedBy(10.dp)
        ) {
            // Button with rounded corners
            Button(
                onClick = onRollTable,
                modifier = Modifier.fillMaxWidth(),
                shape = RoundedCornerShape(8.dp),
                colors = ButtonDefaults.buttonColors(Color(0xFF673AB7))
            ) {
                Text("Roll on Loot Table", color = Color.White)
            }

            // Result text within a rounded corner rectangle
            Surface(
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(vertical = 4.dp),
                shape = RoundedCornerShape(8.dp),
                color = Color(0xFFD1C4E9)
            ) {
                Text(
                    text = resultText,
                    modifier = Modifier
                        .padding(16.dp)
                        .fillMaxWidth(),
                    color = Color.Black,
                    style = MaterialTheme.typography.bodyLarge
                )
            }
        }
    }
}

@Preview(showBackground = true)
@Composable
fun PreviewLootTableUIEnhanced() {
    LootTableUIEnhanced(onRollTable = {})
}

```

In this code snippet:

- `LootTableUI` is a composable function that creates the UI for the loot table interaction. It includes a button for rolling on the table and a text area for displaying the result.
- `onRollTable` is a lambda function that will be triggered when the button is pressed. The actual logic for rolling on the loot table and updating the result will be implemented later.
- `resultText` is a placeholder text that will display the result of rolling on the loot table. Initially, it's set to "Result appears here," but it will be updated dynamically once you implement the functionality.
- `@Preview` is used to see a preview of the UI in Android Studio's design tool.
- The entire screen is given a light purple background using a `Box` composable with a background modifier set to a custom color.
- The button now has a custom color (a deeper shade of purple) and rounded corners, achieved through the `shape` and `colors` parameters in the `Button` composable.
- The result text is placed inside a `Surface` composable, which allows for custom shape and color. This gives the text's container rounded corners and a distinct background color for differentiation.
- The `fillMaxWidth` modifier is applied to both the button and the text container, ensuring they span the width of the screen with some margin on each side for padding.
- The `padding` inside the text container ensures the text is left-justified and has space to breathe, while the `fillMaxWidth` modifier inside the `Text` composable ensures proper text wrapping.

### 3. Look at the View

So now we have written all this code and we haven't run any of it, which is a little bit of nonsense. So let's try and get a look at the View we created. Unfortunately, we're going to have to write just a little more code before that can happen, because this is modern programming, and all that bloat and boilerplate that was in COBOL is still around after all these years for some reason.

To have your `MainActivity.kt` display the UI defined in your `LootTableView.kt` instead of the default `Greeting` Composable, you need to modify the `setContent` block within `MainActivity`. This involves calling your `LootTableUIEnhanced` Composable function directly inside `setContent`, instead of `Greeting`.

Here's how you update `MainActivity`:

1. **Import `LootTableUIEnhanced` in `MainActivity`**: Make sure you import the `LootTableUIEnhanced` Composable function at the top of your `MainActivity.kt` file.

2. **Update `setContent` Block**: Replace the call to `Greeting` within the `setContent` block with a call to `LootTableUIEnhanced`. You'll provide implementations for any parameters `LootTableUIEnhanced` expects, such as `onRollTable` and optionally `resultText` if you want to specify a default value other than "Result appears here".

Here's how the updated `MainActivity.kt` might look:

```kotlin
package net.erickveil.mvi_table_roller

import android.os.Bundle
import android.util.Log
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import net.erickveil.mvi_table_roller.ui.theme.MVITableRollerTheme
import net.erickveil.mvi_table_roller.ui.view.LootTableUIEnhanced 

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MVITableRollerTheme {
                // A surface container that uses the 'background' color from the theme
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    // Now using LootTableUIEnhanced instead of Greeting
                    LootTableUIEnhanced(onRollTable = {
                        // Implement what should happen when the button is pressed
                        Log.d("@LOOT", "Button Pressed.")
                    })
                }
            }
        }
    }
}
```

This change makes it so that when your application starts, `MainActivity` will set up the UI to display the layout defined in `LootTableUIEnhanced` instead of the default greeting message. For now, the `onRollTable` lambda can be left empty or with a placeholder implementation until you integrate the full MVI functionality to handle the button's action.

# Adjust the Style

The UI looks pretty bad. And we're using constants to define how awful it looks. Let's create a style class.

## 1. Color

Find the `Color.kt` file and add the new colors you want to it. We're not going to fuss with themes for now:

```kotlin
val pageBGColor = Color(0xff556f75)  
val buttonColor = Color(0xff9cb6b9)  
val descriptoinBGColor = Color(0xffbd9b6e)  
val textColor = Color.Black  
val buttonTextColor = textColor
```

Then go back to `LootTableView.kt` and replace the hard coded colors with these values. If you begin to type the name and press tab to autocomplete, Android Studio will also add the import statement for the color for you at the top.

## 2. Minimum Height of the Output Box

The output box now changes its height to fit the text contents. But I want it to be a minimum height if there's not that much text (such as at the start) so that it doesn't just look like another button. 

To do that, I add the `heightIn()` method tot he `Modifier` chain for that box in LootTableView.kt:

```kotlin
Surface(
    modifier = Modifier
        .fillMaxWidth()
        .heightIn(min = 300.dp) // Set the minimum height to 100.dp
        .padding(vertical = 4.dp),
    shape = RoundedCornerShape(8.dp),
    color = descriptionBGColor // This is a predefined color in our Color.kt
) {
    Text(
        text = resultText,
        modifier = Modifier
            .padding(16.dp)
            .fillMaxWidth(),
        color = Color.Black,
        style = MaterialTheme.typography.bodyLarge
    )
}

```

## 3. Replace the Contents for the Dimensions of Things

For managing UI styling elements such as dimensions, typography, and more in a centralized and reusable manner across your Jetpack Compose application, you can adopt a similar approach to how you handle colors—by defining them in a separate file. This approach not only keeps your code organized and clean but also makes it easier to update your app's look and feel by changing values in just one place.

### Defining Dimensions in a Centralized Way

1. **Create a `Dimens.kt` File**: Just like you have a `Color.kt` for colors, create a `Dimens.kt` file for dimension constants. This file will hold constants for margins, padding, sizes, etc.

    ```kotlin
    // In Dimens.kt
    package net.erickveil.mvi_table_roller.ui.theme

    import androidx.compose.ui.unit.dp

    val controlPadding = 16.dp  
	val outputBoxMinHeight = 100.dp  
	val columnSpacing = 10.dp  
	val cornerRadius = 8.dp  
	val internalPadding = 4.dp
    // Add more dimension constants as needed
    ```

2. **Use Dimensions in Composables**: Import and use these dimensions in your Composable functions.

    ```kotlin
    // In your Composable
    import net.erickveil.mvi_table_roller.ui.theme.SurfaceMinHeight

    Surface(
        modifier = Modifier
            .fillMaxWidth()
            .heightIn(min = SurfaceMinHeight) // Use the constant here
            .padding(vertical = DefaultPadding),
        shape = RoundedCornerShape(cornerRadius),
        color = descriptionBGColor
    ) {
        Text(
            text = resultText,
            modifier = Modifier
                .padding(all = DefaultPadding)
                .fillMaxWidth(),
            color = Color.Black,
            style = MaterialTheme.typography.bodyLarge
        )
    }
    ```

### Benefits of This Approach

- **Consistency**: Ensures that your UI maintains a consistent look and feel across different screens and components.
- **Maintainability**: Makes it easier to update and manage UI dimensions since they're centralized. For example, changing the default padding or the minimum height of elements across your app can be done in one place.
- **Readability**: Improves the readability of your Composable functions by abstracting away the raw values and giving them descriptive names.

This method aligns with best practices for managing theme-related properties and ensures that your application's UI is scalable and easy to maintain. Similar to how you would manage colors and typography, handling dimensions in a centralized manner aids in achieving a consistent design language throughout your app.

# Define the Intent

In the Model-View-Intent (MVI) architecture, after setting up the Model and the View, the next step indeed involves defining Intents. Intents in MVI are responsible for representing the user's intentions, essentially the actions that the user can perform on the UI that might require some data processing or logic execution, such as fetching data, updating the UI, etc.

For your application, where users interact with a "Roll on Loot Table" button to get a result, you'd define an Intent that captures this user action. Here's how you can approach it:

### Defining Intents

1. **Identify User Actions**: Start by identifying all the possible actions users can perform. In your case, it's primarily the action of "rolling" to get a loot result.

2. **Create an Intent Sealed Class**: A sealed class in Kotlin is a good way to define a restricted class hierarchy that represents all possible Intents your View can emit. For your loot table application, this might look something like:

    ```kotlin
    sealed class LootTableIntent {
        object RollLootTable : LootTableIntent()
        // Add more Intents here as your application grows
    }
    ```
#### About Sealed Classes
A `sealed` class cannot be instantiated on its own and can only be subclassed within the file in which it is declared. This feature allows you to define a closed set of subclasses that are known at compile time, making it easier to handle them when you're using them in, for example, `when` expressions. This is particularly useful in cases where a function or method can only return a limited, fixed number of types.
### Handling Intents in Your Architecture

Once you have your Intents defined, the next steps in the MVI architecture involve creating a ViewModel that listens to these Intents, performs the necessary data processing (fetching the loot results in this case), and then publishes a new state to the UI.

### Implementing the ViewModel

Your ViewModel will act as the intermediary that handles Intents, interacts with the Model to retrieve or manipulate data, and then generates a new ViewState that reflects the result of these operations. Here's a simplified version of what this might look like:

1. **Define a ViewState**: First, define a data class that represents the state of your UI at any given time. This class will include all the necessary information to render your UI.

    ```kotlin
    data class LootTableViewState(
        val isLoading: Boolean = false,
        val resultText: String = "Result appears here",
        val error: String? = null
    )
    ```

2. **Create a ViewModel**: Implement a ViewModel that handles the Intents and updates the ViewState accordingly.

    ```kotlin
    class LootTableViewModel : ViewModel() {
        private val _state = MutableStateFlow(LootTableViewState())
        val state: StateFlow<LootTableViewState> = _state.asStateFlow()

        fun processIntent(intent: LootTableIntent) {
            when (intent) {
                is LootTableIntent.RollLootTable -> rollLootTable()
            }
        }

        private fun rollLootTable() {
            // Simulate fetching data and updating the state
            _state.value = _state.value.copy(
                isLoading = true,
                // Simulate fetching a random result
                resultText = "Magic sword"
            )
            // Here, you would actually fetch the data from your model and update the state accordingly
        }
    }
    ```

3. **Connect ViewModel with UI**: Finally, in your Composable function (`LootTableUIEnhanced`), observe the `LootTableViewState` from your `LootTableViewModel` and update the UI accordingly. You'll also need to modify the `onRollTable` lambda to send the `RollLootTable` Intent to your ViewModel.

This approach keeps your UI code clean and focused on displaying data, while your ViewModel handles the logic and state management, adhering to the principles of MVI.