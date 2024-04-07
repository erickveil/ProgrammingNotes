---
title: MVI Jetpack Project Development
layout: note
date: 24-04-05
tags:
  - kotlin
  - mvi
  - android
  - jetpack
  - architecture
---
For this project, we're going to use a JSON file as a simple data source and write a program that rolls for the random results on the table defined in the JSON.
# Setting Up a New Project

- File -> New -> New Project -> Empty Activity

# Set Up the Project Structure

- **`src/main/java` or `src/main/kotlin`**: Root directory for our Kotlin source files.
    - **`com/ourapplicationname`**: Our application's package.
        - **`data`**: Directory for data handling and network operations.
            - **`model`**: Contains model classes that represent the structure of our data. Our Kotlin class for the `Model` fits here.
            - **`repository`**: Contains repository classes that abstract the logic of data access and manipulation. They might interact with our model classes and data sources like databases, web services, or our JSON files.
        - **`ui`**: Directory for user interface related classes.
            - **`view`**: Contains activities, fragments, and any other views.
            - **`viewmodel`**: Contains ViewModel classes which hold the logic to manage the UI's data in response to user actions.
        - **`util`**: Utility classes that provide common functionalities across the application.
- **`src/main/assets`**: Where we will keep data sources and other assets.

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

```kotlin
plugins {  
id("org.jetbrains.kotlin.plugin.serialization") version "+"  
}
```


```kotlin
dependencies {  
implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:+")  
}
```

Be sure to sync the project when you're done with editing this file.

Note that I use "+" when setting the version number. This will most often highlight in the editor with an option to use the latest version. You'll notice when looking at a lot of online documentation (such as blogs like this) that they tend to list specific version numbers. You want the latest version though. So use this trick.

Also note that I use Kotlin syntax for my gradle files. Lots of online references will use groovy syntax.

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
- `LootTable` is the main data class that represents our table, with properties matching the JSON keys: `tableName`, `description`, and `results`.
- `tableName` and `description` are of type `String`.
- `results` is a list of strings, each representing a possible outcome from the table.
# The View

### 1. Set Up the Gradle for Compose

We might need to add the necessary dependencies and enable Compose in the build.gradle (module-level) file.

To enable Jetpack Compose, ensure you have the following in our module-level `build.gradle`:

1. **Add the Compose Compiler dependency:** This is required for Compose to work. Ensure you're using a version compatible with our Kotlin version.

    ```groovy
    buildFeatures {
        compose = true
    }
    composeOptions {
        kotlinCompilerExtensionVersion = "1.5.1" // Use the version compatible with our setup
    }
    ```

2. **Include Compose dependencies:** You'll need the core Compose libraries, as well as any additional ones we plan to use (like Material Design components, which our code snippet indicates we are using).

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

To have our `MainActivity.kt` display the UI defined in our `LootTableView.kt` instead of the default `Greeting` Composable, we need to modify the `setContent` block within `MainActivity`. This involves calling our `LootTableUIEnhanced` Composable function directly inside `setContent`, instead of `Greeting`.

Here's how you update `MainActivity`:

1. **Import `LootTableUIEnhanced` in `MainActivity`**: Make sure you import the `LootTableUIEnhanced` Composable function at the top of our `MainActivity.kt` file.

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

This change makes it so that when our application starts, `MainActivity` will set up the UI to display the layout defined in `LootTableUIEnhanced` instead of the default greeting message. For now, the `onRollTable` lambda can be left empty or with a placeholder implementation until you integrate the full MVI functionality to handle the button's action.

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

For managing UI styling elements such as dimensions, typography, and more in a centralized and reusable manner across our Jetpack Compose application, you can adopt a similar approach to how you handle colors—by defining them in a separate file. This approach not only keeps our code organized and clean but also makes it easier to update our app's look and feel by changing values in just one place.

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

2. **Use Dimensions in Composables**: Import and use these dimensions in our Composable functions.

    ```kotlin
    // In our Composable
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

- **Consistency**: Ensures that our UI maintains a consistent look and feel across different screens and components.
- **Maintainability**: Makes it easier to update and manage UI dimensions since they're centralized. For example, changing the default padding or the minimum height of elements across our app can be done in one place.
- **Readability**: Improves the readability of our Composable functions by abstracting away the raw values and giving them descriptive names.

This method aligns with best practices for managing theme-related properties and ensures that our application's UI is scalable and easy to maintain. Similar to how you would manage colors and typography, handling dimensions in a centralized manner aids in achieving a consistent design language throughout our app.

# Define the Intent

In the Model-View-Intent (MVI) architecture, after setting up the Model and the View, the next step indeed involves defining Intents. Intents in MVI are responsible for representing the user's intentions, essentially the actions that the user can perform on the UI that might require some data processing or logic execution, such as fetching data, updating the UI, etc.

For our application, where users interact with a "Roll on Loot Table" button to get a result, you'd define an Intent that captures this user action. Here's how you can approach it:

### Defining Intents

1. **Identify User Actions**: Start by identifying all the possible actions users can perform. In our case, it's primarily the action of "rolling" to get a loot result.

2. **Create an Intent Sealed Class**: A sealed class in Kotlin is a good way to define a restricted class hierarchy that represents all possible Intents our View can emit. For our loot table application, this might look something like:

    ```kotlin
    sealed class LootTableIntent {
        object RollLootTable : LootTableIntent()
        // Add more Intents here as our application grows
    }
    ```

So, think of this in terms of setting up the **state** objects for a state pattern:
- We have a list of states (actually a list of intents) called `LootTableIntent`.
- Every **intent** (every action a user might take with a UI, intending things to happen) is one of the state objects.
- Each subclass of `LootTableIntent` is going to be used like each **state** would be.
- So here we just have one intent/state, because this is a simple example.
- That intent is `RollLootTable`
- The notation `object RollLootTable : LootTableIntent()` is basically a shorthand for:
	- Subclass the `LottTableIntent`, creating a new intent/state.
	- Do it in a way that makes this a singleton.
So it's basically here a supper powered `enum` value that better follows the state pattern than an enum would (which is why we have the state pattern).

The `sealed` class ensures that all of the values for this state machine are going to be right here in this file only.
### Handling Intents in Our Architecture

Once you have our Intents defined, the next steps in the MVI architecture involve creating a ViewModel that listens to these Intents, performs the necessary data processing (fetching the loot results in this case), and then publishes a new state to the UI.

### Implementing the ViewModel

Our ViewModel will act as the intermediary that handles Intents, interacts with the Model to retrieve or manipulate data, and then generates a new ViewState that reflects the result of these operations. Here's a simplified version of what this might look like:

#### 1. Define a ViewState 
First, define a data class that represents the state of our UI at any given time. This class will include all the necessary information to render our UI.

```kotlin
data class LootTableViewState(
	val isLoading: Boolean = false,
	val resultText: String = "Result appears here",
	val error: String? = null
)
```

#### 2. Create a ViewModel
Implement a ViewModel that handles the Intents and updates the ViewState accordingly.

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
		// Here, you would actually fetch the data from our model and update
		// the state accordingly
	}
}
```

Breakdown:

Here we set up the state machine:
```kotlin
 private val _state = MutableStateFlow(LootTableViewState())
 val state: StateFlow<LootTableViewState> = _state.asStateFlow()
```

Here we look at the current intent/state and call a method depending on what it is.
If we had different intents, they would each be handled by a separate `is` statement.

```kotlin
when (intent) {
	is LootTableIntent.RollLootTable -> rollLootTable()
}
```

Now here, we would be interacting with the Model and rolling on the table. We don't actaully do that yet. This is just a palceholder:

```kotlin
private fun rollLootTable() { ...
```

# Connect ViewModel with UI
Finally, in our Composable function (`LootTableUIEnhanced`), we observe the `LootTableViewState` from our `LootTableViewModel` and update the UI accordingly. 

Let's lay this out in order of the flow of MVI:

### 1.  Give the Composable Function for the View Access to the ViewModel

First, our Button and our Text are the user's Input and Output for this Intent. The ViewModel will be their connection to the business logic.

In `LootTableUIEnhanced` we're going to get rid of the placeholder arguments and give it a `viewModel` object. 
- We're going to use that to get the current state of the UI as represented by the `ViewModel`. 
- The `StateFlow` in the `ViewModel` holds the current state, which includes all the data our UI might need to render itself. 
- When you collect this state within our Composable function, you're telling Compose to re-compose (re-draw) our UI any time the state changes. 
- This state could be the result of processing various intents, but it's not the intent itself.

Change the function arguments:
```kotlin
fun LootTableUIEnhanced(viewModel: LootTableViewModel) {  
```

### 2. Capture The Intent From the User

Change the Button Code:
```kotlin
Button( 
	onClick = { 
		viewModel.processIntent(LootTableIntent.RollLootTable) 
	}, 
```

The `Button` Composable within `LootTableUIEnhanced` captures the user's intent to "roll the loot table" through its `onClick` listener. 
When the user presses the button, this listener invokes the `processIntent` method on the `ViewModel` with an argument that signifies the specific action or intent (`LootTableIntent.RollLootTable` in this case).

In the `processIntent` function, we basically have our "switch", the `when(intent) { is... }` statement where we look at the intent, then call the appropriate business logic.

```kotlin
fun processIntent(intent: LootTableIntent) {
    when (intent) {
        is LootTableIntent.RollLootTable -> rollLootTable()
    }
}
```

So now we have our Intent, `RollLootTable` and we call `rollLootTable()`.

```kotlin
private fun rollLootTable() {  
	_state.value = _state.value.copy(  
	isLoading = true,  
	resultText = "Magic sword"  
	)   
}
```

Here, we do some work (in this case, just some example values are set) and the result is given to the state. 
- **`_state.value`**: This accesses the current value of the `_state` `MutableStateFlow`.
- **`.copy(isLoading = true, resultText = "Magic sword")`**: Since `LootTableViewState` is a data class, you can use the `copy` method to create a new instance of it with some properties changed. This line creates a new `LootTableViewState` where `isLoading` is set to `true`, and `resultText` is set to "Magic sword", while keeping all other properties the same as in the current state.
- **Assignment**: The new state is then assigned back to `_state.value`, effectively updating the state.

We had defined the state in this `ViewModel` class using our class, `LootTableViewState`:


```kotlin
private val _state = MutableStateFlow(LootTableViewState())  
val state: StateFlow<LootTableViewState> = _state.asStateFlow()
```

- Here, `_state` is a private mutable state flow that you'll use within our `ViewModel` to update the state. 
- Meanwhile, `state` is the public, read-only version of this state flow that the UI components can collect and observe for changes.

### 3. Update the View With the State Changes

Now, back at our View's Composable function, `LootTableUIEnhanced` where we want the value to update, we use the public access to the state to change the text that the user sees.

Change the Text source:
```kotlin
Text(  
	text = state.value.resultText,
```
This is the point where we observe the changes in the state stored in the ViewModel. It's "declarative" in that the UI automatically updates in response to this change (without "imperative" commands like `setText()`).

We had defined the `state` before we started defining **Composables** (the visual UI elements usually referred to as "Controls" in other languages) which has access to the same ViewModel as the one we used in the Button. 

Add at the top of the function:
```kotlin
val state = viewModel.state.collectAsState()
```

This brings us full circle from button click to setting Intent, to doing the work with the Model required by that Intent, and then updating the View with the changes.

### 4. Update the MainActivity So That It Uses the View Correctly

Finally, we need to make sure that `MainActivity` is calling `LootTableUIEnhanced` with the correct argument, now that we changed the signature of that function.

Change the object passed to the function:
```kotlin
Surface() {  
	LootTableUIEnhanced(viewModel())  
}
```

The `viewModel()` method is part of Compose, and in this case it creates a new ViewModel to use.
It automatically infers the type of `ViewModel` thanks to that function requiring a specific subclass - in this case `LootTableViewModel`. It also scopes that object to the activity it is created in (`MainActivity` in this case) so that it is preserved.

In situations where type inference might not work as expected or when you want to be explicit, you can specify the `ViewModel` class directly:

```kotlin
val myViewModel: LootTableViewModel = viewModel()
```

Or, when passing it directly as a parameter without assigning it to a variable:

```kotlin
LootTableUIEnhanced(viewModel = viewModel())
```

# Working With the Actual Data Model

At this point, we display a constant string when the user pushes the button. But we want to:

- Load the JSON into the Model.
- Chose a random item from the List of options.
- Use that as the string displayed in our output box.

### Create the Repository

The repository is the class that we will use to manage our Data Model. From here, we will handle loading the JSON into a Model class.

```kotlin
class LootRepository(private val context: Context) {  

	fun getLootTable(): LootTable? {  
		val jsonString = loadJsonFromAssets("lootTable.json")  
		return jsonString?.let { parseJsonToLootTable(it) }  
	}  
	  
	private fun loadJsonFromAssets(fileName: String): String? {  
		return try {  
			context.assets.open(fileName).bufferedReader().use { it.readText() }  
		} catch (ioException: IOException) {  
			ioException.printStackTrace()  
			null  
		}  
	}  
	  
	private fun parseJsonToLootTable(jsonString: String): LootTable? {  
		return try {  
			Json.decodeFromString(serializer(), jsonString)  
		} catch (exception: SerializationException) {  
			exception.printStackTrace()  
			null  
		}  
	}  
}
```

Since we set up the `LootTable` data model as serializable, we can just continue using kotlinx to parse.

We will need the `Context` object so that we can load the JSON resource from the `assets` directory.

### Use the Repository in the ViewModel

Next we modify the ViewModel to load the JSON using the Repository. Since we only need to load it once and reference it, we can do that in the class's `init` method:

First, we will need to modify our ViewModel to accept the `Context`.
```kotlin
class LootTableViewModel(context: Context): ViewModel() {
```

Then add some class members to instantiate the Repository and hold our Data Model where we will be referencing it.
```kotlin
private val repository = LootRepository(context)  
private var lootTable: LootTable? = null
```

Next we create a method to load it.

We will use `viewModelScope.launch { }` to put the file loading into a coroutine. This will load the file asynchronously. It's possible that you'd want to create a state where `isLoading` is manipulated so the rest of the program knows it's safe to access the data. If the data source is particularly large, you might want to do this on a background thread. 

We're going to keep this simple here for this case though:
```kotlin
private fun loadLootTable() {  
	viewModelScope.launch {  
		lootTable = repository.getLootTable()  
	}  
}
```

Finally, we just run this in the `init`.
```kotlin
init {
	loadLootTable()
}
```

### Roll on the Table

Let's put the actual roll in its own method.

If we were certain of the presence of data (we are not, since the JSON can have *anything*, but let's suppose) then we can just select a random item from the list with
```kotlin
lootTable.results.random()
```

But we want to make sure we're not performing operations on null objects, so we will use a lambda with `takeIf { it }` which will return `null` if any of the optional values are missing, or execute the `random()` method if all is well.
```kotlin
private fun getRandomLootItem(): String? {  
	return lootTable?.results?.takeIf { it.isNotEmpty() }?.random()  
}
```

Now we can remove the placeholder code and select a random item from the table instead:
```kotlin
private fun rollLootTable() {  
	val randomItem = getRandomLootItem()  
	_state.value = _state.value.copy(  
		isLoading = false,  
		resultText = randomItem ?: "No loot found"  
	)  
}
```

This function updates the `_state` with a new value, setting `isLoading` to `false` (assuming you're using it to show a loading indicator) and `resultText` to the randomly selected item. If no item is found (for example, if `results` is empty or `lootTable` is not loaded), it sets `resultText` to "No loot found".
#### Note on Safety and Randomness

- **Null Safety**: The use of the safe call operator (`?.`) and the Elvis operator (`?:`) ensures that the function gracefully handles cases where `lootTable` might not be initialized or `results` is empty.
    
- **Randomness**: The `random()` function provides an easy and concise way to get a random element. Ensure our list has elements before calling `.random()` to avoid exceptions. The check `it.isNotEmpty()` ensures this safety.

# Dependency Injection

## Hilt

For injecting `Context` into our `ViewModel`, the most appropriate and modern approach in Android development, especially within the scope of recommended practices by Google, is to use **Hilt** for dependency injection. Hilt simplifies the process, ensures safe handling of the `Context`, and keeps our code clean and manageable. Here's why Hilt is particularly suited for this task and how to do it:

## Why Hilt?

- **Lifecycle Awareness**: Hilt is designed to be aware of Android lifecycles, ensuring that the correct type of `Context` (application, activity, etc.) is provided where needed, reducing the risk of memory leaks.
- **Simplicity**: Hilt abstracts away much of the boilerplate code needed for dependency injection, making our codebase simpler and more readable.
- **Scalability**: As our project grows, Hilt makes it easier to manage dependencies across different components of our Android app.
- **Integration with Jetpack**: Hilt works seamlessly with other Jetpack components, including `ViewModels`, and follows best practices set by the Android team.

## Add Hilt to our Project

Ensure you have Hilt set up in our project by adding the necessary dependencies and plugins. In our `build.gradle` (project level), you need to include the Hilt Gradle plugin:

```groovy
plugins { 
	...
	id("com.google.dagger.hilt.android") version "2.44" apply false
}

buildscript {  
	dependencies {  
		classpath("com.google.dagger:hilt-android-gradle-plugin:2.44")  
	}  
}
```

And in our `build.gradle` (app/module level), apply the plugin and add Hilt dependencies:

```groovy
plugins {  
	id("kotlin-kapt")  
	id("dagger.hilt.android.plugin")  
}

dependencies {
	implementation("com.google.dagger:hilt-android:2.44")  
	kapt("com.google.dagger:hilt-android-compiler:2.44")
}
```

## The Application Class

In Android development, the `Application` class in Android serves as a base class for maintaining global application state before any activity, service, or receiver objects (components) are created. It's like the entry point to our application, but not exactly the "main" as you would find in other types of programs, like a Java console application or C++ program. Instead, Android decides which component (Activity, Service, etc.) to launch based on the intent it receives and the app's manifest declarations.

### Why You Might Not See an Application Class

It's perfectly normal for small or simple Android projects not to have a custom `Application` class if they don't need to initialize global state or libraries at the application start. our `MainActivity` can indeed seem like the "main" part of our app since it's often the entry point for user interaction.

### Purpose of an Application Class

However, when you start needing to initialize global libraries, handle application-wide state, or work with frameworks that require initialization (like Hilt for dependency injection), defining a custom `Application` class becomes necessary. It allows you to:

- Initialize libraries at the start of our app's lifecycle, before any activity is started.
- Maintain global application state or resources.
- Use services like dependency injection frameworks more effectively.

### How to Create and Use an Application Class with Hilt

1. **Create a Custom Application Class**: The `@HiltAndroidApp` annotation is important here; it sets up Hilt for dependency injection across the application. This is all we really need at this point.
    ```kotlin
    import android.app.Application
    import dagger.hilt.android.HiltAndroidApp

    @HiltAndroidApp
    class TableRollerApp : Application() {
        // Initialization code here
    }
    ```


2. **Declare It in our AndroidManifest.xml**: Once you've created our custom `Application` class, you need to declare it in our `AndroidManifest.xml` so the Android system knows to use it:

    ```xml
    <application
        android:name=".TableRollerApp"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name">
        <!-- our activities and other components -->
    </application>
    ```
   
We would replace `".TableRollerApp"` with the actual path to the custom `Application` class if it's not in the root package. In this case, we've put it on the same level as `MainActivity` in our file tree. You can see in the manifest that MainActivity is still in the `LAUNCHER` category.

### Injecting Context into the ViewModel

Use `@HiltViewModel` on our ViewModel and inject `Context` using `@ApplicationContext` to ensure you're using the application context, which is safe:
```kotlin
@HiltViewModel
class LootTableViewModel @Inject constructor(
    @ApplicationContext private val context: Context) 
    : ViewModel() {
    // Use context as needed
}
```

The `@ApplicationContext` is going to get flagged as "This field leaks Context." This warning, when you're using `@ApplicationContext` in a `ViewModel` with Hilt, can be a bit misleading because using `@ApplicationContext` is actually the recommended practice to avoid context leaks. This annotation ensures that you are provided with the application context, not an activity context, which is safe to retain across the configuration changes since it's tied to the lifecycle of the entire application, not just a single activity or view.
##### Understanding the Context
- **Application Context**: It's a context tied to the lifecycle of the application, not an activity. It's safe to hold onto this context for long periods, or even for the lifetime of our app, which is why it's recommended for use in `ViewModels` and other long-lived components.
- **Activity Context**: It's tied to the lifecycle of an activity. Holding a reference to it in a `ViewModel` or any component that outlives an activity instance can lead to memory leaks.

### Prepare our Activity or Fragment

Ensure our `Activity` or `Fragment` is ready to work with Hilt by annotating them with `@AndroidEntryPoint`:

```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    // Now Hilt can inject dependencies into MainActivity, and you can obtain ViewModels via Hilt
}
```

This setup ensures that our ViewModel receives the `Context` in a safe, efficient, and scalable manner, adhering to best practices recommended by the Android team.

# MVI and the Files We Created

Now we have a working table roller, where the table can be described using a JSON file.

Our Android project structure, organized according to the Model-View-Intent (MVI) architecture, aligns each component with one of the MVI layers. Here's how each file fits into the MVI pattern:

### Model
The Model is responsible for the data-related logic, such as fetching, storing, and manipulating data.

- **`/data/model/LootTable.kt`**: Represents the data structure of our loot table. It's the data model part of the Model layer.
- **`/data/repository/LootRepository.kt`**: Acts as the data source layer that would fetch, cache, and manage our loot data (e.g., reading the `lootTable.json` file). It's part of the Model layer, serving as an intermediary between the raw data and the ViewModel.

### View
The View is responsible for presenting data to the user and capturing user inputs.

- **`/ui/view/LootTableView.kt`**: Defines the UI components (Composables) that display the loot table and button to the user. It's part of the View layer in MVI.
- **`/MainActivity.kt`**: While not a Composable itself, it sets up the content view for our app and could be considered part of the View layer since it's where our Composables are hosted.
- **`/TableRollerApp.kt`**: Likely the application class. While not directly part of the MVI pattern, it's essential for initializing app-wide components, like Hilt for dependency injection.

### Intent
The Intent layer captures the intentions to change the state of the app based on user input or other actions.

- **`/ui/intent/LootTableIntent.kt`**: Defines the actions (intents) that can be taken in the app, like rolling the loot table. It directly corresponds to the Intent layer in MVI.

### ViewModel
Though not traditionally separate in the MVI acronym, the ViewModel acts as a bridge between the View and Model, handling state and intents.

- **`/ui/viewmodel/LootTableViewModel.kt`**: Contains the business logic responding to user intents, fetching data from the repository, and preparing it to be displayed by the View. It processes intents, updates the state, and thus serves as a crucial part of both the Intent and Model layers.

### ViewState
ViewState represents the current state of the UI, which is a concept closely tied to both the View and the ViewModel.

- **`/ui/viewstate/LootTableViewState.kt`**: Defines the various states of the UI (e.g., loading, success, error) based on the data from the ViewModel. It's closely related to both the View (as it dictates what the View displays) and the ViewModel (as it's shaped by the data processing logic within the ViewModel).

# Conclusion
This project structure reflects a clear separation of concerns as prescribed by the MVI architecture, with each component playing a specific role:

- **Model**: `LootTable.kt`, `LootRepository.kt`
- **View**: `LootTableView.kt`, `MainActivity.kt`
- **Intent**: `LootTableIntent.kt`
- **Bridge (ViewModel and ViewState)**: `LootTableViewModel.kt`, `LootTableViewState.kt`

This organization facilitates understanding, maintaining, and scaling our application by clearly delineating responsibilities within our codebase.