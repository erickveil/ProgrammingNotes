---
title: MVVM Project  Development
layout: note
tags:
  - android
  - architecture
  - jetpack
  - kotlin
  - mvvm
---

We're going to continue exploring architectures with the MVVM pattern. It's very similar to the MVI pattern, but we're more casual about the mutability of the state object, and we don't process intents - we just call the ViewModel methods directly. The data flow is also considered bi-directional as opposed to MVI's unidirectional flow. 

It's a good model to use for single page applications where you're generating dynamic composables on the page, such as in a chat app where chat bubbles are constantly being generated with text.

### Why MVVM is Seen as Bidirectional:

- **Event handling**: The View directly interacts with the ViewModel by calling its methods. This is in contrast to MVI, where user actions are wrapped in Intents and processed in a more streamlined, unidirectional flow.
- **Data binding**: MVVM often uses data binding to automatically update the UI based on ViewModel changes, but it can also allow for UI events to directly modify ViewModel properties without a formalized Intent or action system.
- **State management**: Unlike MVI, where the state is a single, immutable object that represents the entire UI state at any moment, MVVM typically manages state in a more fragmented manner, with LiveData or StateFlow objects representing various pieces of UI data. This can lead to a scenario where changes in the ViewModel can directly result from specific UI actions, and vice versa, reinforcing the perception of bidirectionality.

### Understand the Components

- **Model**: Represents the data and business logic layer of your application. It's responsible for handling the database, network operations, or any data sources.
- **View**: Deals with the visual part of your application. In Android, this includes activities, fragments, and anything related to displaying UI elements.
- **ViewModel**: Acts as a link between the Model and the View. It handles the UI-related data and logic but doesn't contain any knowledge of the View itself. ViewModel survives configuration changes like screen rotations.

# The Model

Let's start with the Model. Here's the JSON file I'll use for my data source:

```json
{ 
	"tableName": "Dungeon Loot Table", 
	"description": "This table determines what loot you find in the dungeon.",
	"results": [ "Gold coins", "Magic sword", "Potion of healing" ] 
}
```

As usual, this goes in the `assets` directory.

Placing your JSON file in the assets directory is appropriate because:
- The file is read-only and doesn't need to be modified at runtime.
- It's easily accessible from your application's code.
- Assets can be bundled with your application, making distribution simpler.

### Model Class
In the context of MVVM, the Model represents the data layer of your application. It's responsible for handling the business logic and data operations, such as fetching, parsing, and preparing data to be displayed by the UI. For your application, the Model will be responsible for loading the JSON file, parsing it, and providing an interface to access the dungeon loot table.

Here's a simple example of what your Model class might look like in Kotlin, using `kotlinx.serialization` for JSON parsing. First, you need to add the dependency for `kotlinx.serialization` in your `build.gradle` file:

```kotlin
plugins {  
	id("org.jetbrains.kotlin.plugin.serialization") version "+"  
}

dependencies { 
	implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:+")  
}
```

Remember to replace the "+" with the actual latest version number. Once you've entered the "+" Android Studio should highlight the line. You can right click and select the specific version from there.

Then, define your data classes to represent the structure of your JSON:

```kotlin
@Serializable
data class DungeonLootTable(
    @SerialName("tableName") val tableName: String,
    @SerialName("description") val description: String,
    @SerialName("results") val results: List<String>
)
```

### Repository Class
The Repository pattern is not exclusive to the MVI architecture; it's also a commonly used design pattern in MVVM and other architectures. Its primary role is to abstract the data layer, providing a clean API for the rest of the application to access the data needed for its operations. The repository acts as a mediator between different data sources, such as a local database (Room), remote API (Retrofit), or in your case, static files from the assets directory.

In MVVM, the Model layer often includes repositories, domain models, and data source implementations. The repository can fetch data from the appropriate source, cache it if necessary, and provide it to the ViewModel. The ViewModel then works with this data to prepare it for presentation in the View.

```kotlin
class DungeonLootRepository(private val context: Context) {  
  
	fun getLootTable(): DungeonLootTable? {  
		val jsonString = loadDungeonLootTable()  
		return jsonString?.let { parseJsonToLootTable(it) }  
	}  
	private fun loadDungeonLootTable(): String? {  
		return try {  
			context.assets.open("lootTable.json").bufferedReader().use {
				it.readText() 
			}  
		} catch (ioException: IOException) {  
			ioException.printStackTrace()  
			null  
		}  
	}  
	  
	private fun parseJsonToLootTable(jsonString: String): DungeonLootTable? {  
		return try {  
			Json.decodeFromString(serializer(), jsonString)  
		} catch (exception: SerializationException) {  
			exception.printStackTrace()  
			null  
		}  
	}  
}
```

# The View

Switching to Jetpack Compose for the View layer is a great choice, especially for a modern Android project like yours. Jetpack Compose simplifies UI development with a declarative approach, allowing you to create dynamic and reactive UIs efficiently. Since you're building a simple app with a Button and a TextView, let's outline how you might structure this UI with Compose, including support for light and dark themes.

### Creating the UI
With Jetpack Compose, you define your UI in Kotlin using composable functions. 
```kotlin
@Composable  
fun DungeonGeneratorScreen() {  
  
// Using MaterialTheme for light and dark themes support  
	MaterialTheme (  
		colorScheme = 
			if (isSystemInDarkTheme()) darkColorScheme() 
			else lightColorScheme()  
	)  
	{  
		Surface(  
			modifier = Modifier  
				.fillMaxSize()  
				.padding(16.dp),  
			color = MaterialTheme.colorScheme.background  
	) {  
		Column(  
			horizontalAlignment = Alignment.CenterHorizontally,  
			modifier = Modifier.fillMaxWidth()  
		) {  
				// Output Text Container with Rounded Corners  
				Surface(  
					modifier = Modifier  
						.fillMaxWidth()  
						.padding(16.dp)  
						.heightIn(min = 200.dp),  
					shape = RoundedCornerShape(8.dp),  
					// Lighter shade for the text background  
					color = 
						MaterialTheme.colorScheme.surface.copy(
							red = 0.5f, blue = 0.2f, alpha = 0.3f)  
				) {  
					Text(  
						text = "Roll Table Result",  
						modifier = Modifier.padding(16.dp),  
						style = MaterialTheme.typography.bodyLarge  
					)  
				}  
				  
				// Button with Rounded Corners  
				Button(  
					onClick = {  
					/* Action to generate dungeon content */  
					},  
					modifier = Modifier  
						.fillMaxWidth()  
						.padding(16.dp),  
					shape = RoundedCornerShape(8.dp),  
					colors = ButtonDefaults.buttonColors(containerColor =
						MaterialTheme.colorScheme.secondary)  
				) {  
					Text("Generate Dungeon Loot")  
				}  
			}  
		}  
	}  
}
```

We will need to define the callback for the Button's onClick and then replace "Roll Table Result" with the actual result.
### Dark and Light Themes
The example uses `MaterialTheme` with `darkColorScheme()` and `lightColorSceme()` to automatically support dark and light themes based on the system settings. Jetpack Compose handles theme switching for you, making it straightforward to support both themes in your app.

### Styling Details:

- **Rounded Corners**: Both the text rectangle and the button utilize `RoundedCornerShape(8.dp)` to achieve rounded corners.
- **Margins and Padding**: The use of `Modifier.padding(16.dp)` around components and `Modifier.fillMaxWidth().padding(16.dp)` for the button and text rectangle ensures there's a 16dp margin around them. The text within the rectangle has additional padding from its edges.
- **Minimum Height for Text Rectangle**: The `.heightIn(min = 200.dp)` modifier ensures the rectangle containing the text has a minimum height of 200dp but can expand to fit more content if necessary.

### Displaying the View
We will edit the `MainActivity` to display the View we just created:

```kotlin
class MainActivity : ComponentActivity() {  
	override fun onCreate(savedInstanceState: Bundle?) {  
		super.onCreate(savedInstanceState)  
		setContent {  
			MVVMTableRollerTheme {  
			// A surface container using the 'background' color from the theme  
				Surface(  
					modifier = Modifier.fillMaxSize(),  
					color = MaterialTheme.colorScheme.background  
				) {  
					DungeonGeneratorScreen()  
				}  
			}  
		}  
	}  
}
```

# The ViewModel

The ViewModel will act as an intermediary between your View (UI) and Model (data). It'll handle the logic to load your dungeon loot table from the JSON file, pick a random loot item, and prepare this data to be displayed by the UI. Since we're using Jetpack Compose for the UI, the ViewModel can take advantage of Kotlin's `State` and `LiveData` to update the UI reactively.

Here’s a basic outline of what your `DungeonViewModel` might look like, using Kotlin Coroutines for asynchronous operations:
### Implementing the ViewModel

```kotlin
class DungeonViewModel(private val repository: DungeonLootRepository) : ViewModel() {
    // Using StateFlow to hold and emit dungeon loot content
    private val _dungeonContent = MutableStateFlow<String>("Press the button to generate loot!")
    val dungeonContent: StateFlow<String> = _dungeonContent
	private var lootTable: DungeonLootTable? = null  
	  
	init {  
		lootTable = repository.getLootTable()  
	}  
	  
	fun generateDungeonLoot() {  
		// Load the dungeon loot table from the repository  
		// Pick a random item from the results and update the StateFlowo  
		lootTable?.results?.let {  
		_dungeonContent.value = 
			if (it.isNotEmpty()) it.random() 
			else "No loot found."  
		}  
	  
	}
}
```

### Key Components:

- **Coroutines and viewModelScope**: Using Kotlin Coroutines within the ViewModel's `viewModelScope` to perform data loading operations asynchronously.
- **StateFlow for State Management**: `StateFlow` is a type of `Flow` that provides a way to emit state updates. `StateFlow` always holds a value and emits the current and new state updates to its collectors. It's a great fit for Jetpack Compose, as Compose can collect these updates in a composable function reactively.
- **Repository Usage**: The ViewModel interacts with the `DungeonLootRepository` to load the dungeon loot table. This decouples the data source logic (e.g., loading and parsing the JSON file) from the UI logic.

### Connecting ViewModel with Compose UI:
To observe `dungeonContent` from your Compose UI and trigger `generateDungeonLoot` upon button press, you can collect the `StateFlow` in a Composable using `collectAsState()`, and use a button click to call `generateDungeonLoot`.

First we set up the lifecycle-viewmodel in our gradle file:

```kotlin
implementation("androidx.lifecycle:lifecycle-viewmodel-compose:+")
```

Modify part of the Compose UI to integrate the ViewModel:

```kotlin
@Composable
fun DungeonGeneratorScreen(viewModel: DungeonViewModel = viewModel()) {
    // Collecting the StateFlow as state
    val dungeonContent = viewModel.dungeonContent.collectAsState()

    Button(onClick = {
        viewModel.generateDungeonLoot()
    }) {
        Text("Generate Dungeon Loot")
    }

    // Display the dungeon content
    Text(text = dungeonContent.value, ...)
}
```

This makes sure that your UI updates when the dungeon loot content changes, thanks to the reactive nature of `StateFlow` and Compose's ability to collect and react to these changes.

Notice that the state we set up is basically a string, where in MVI, we created an Immutable State Object that carried the information we needed. MVVM does not put a hyperfocus on having an immutable state like MVI does, but you can still use a state object if you want. In fact, it's probably a better route to go. If you application is simple like this one, then you don't need to worry about it. But as it grows in complexity, it's worth looking into.

In MVVM, while it's common to see simpler state management approaches (like using individual LiveData or StateFlow properties for each piece of UI data), adopting a consolidated ViewState approach can offer several benefits:

### Benefits of Using a ViewState in MVVM:
- **Immutability**: By using a data class for your ViewState, you embrace immutability, a practice that helps prevent unintended side effects and makes your UI state easier to reason about. Kotlin's `copy` method on data classes makes it straightforward to update the state in an immutable fashion.
- **Centralized State Management**: Having a single source of truth for your UI state simplifies debugging and testing since all state changes go through this centralized state object.
- **Scalability**: As your application grows and the UI becomes more complex, having a unified state object becomes increasingly valuable for managing changes without cluttering your ViewModel with numerous state properties.

### Implementing ViewState in MVVM:
To apply this pattern in MVVM with StateFlow, you'd define a ViewState data class that encapsulates all the relevant pieces of state for your UI. Here's how you might adjust your `DungeonViewModel` to use a `ViewState`:

```kotlin
data class DungeonViewState(
    val dungeonContent: String = "Press the button to generate loot!"
)
```

In your Composable function, you'd collect `viewState` as a state and use its properties to update the UI:

```kotlin
@Composable
fun DungeonGeneratorScreen(viewModel: DungeonViewModel = viewModel()) {
    val viewState by viewModel.viewState.collectAsState()

    Button(onClick = {
        val context = LocalContext.current
        viewModel.generateDungeonLoot(context)
    }) {
        Text("Generate Dungeon Loot")
    }

    Text(text = viewState.dungeonContent, ...)
}
```

# ViewModel Factory

Now when we run, we get the following exception:

> `java.lang.RuntimeException: Cannot create an instance of class net.erickveil.mvvmtableroller.viewmodel.DungeonViewModel > Caused by: ``
> 
> `java.lang.NoSuchMethodException: net.erickveil.mvvmtableroller.viewmodel.DungeonViewModel.<init> []`

The exception you're encountering is typically thrown when the Android system tries to create an instance of your `DungeonViewModel` but can't find an appropriate constructor. This is a common issue when your ViewModel has a non-default constructor but you haven't provided a way for the framework to know how to instantiate it.

In Android's ViewModel architecture, a ViewModel is usually expected to have a default (parameterless) constructor. If your ViewModel has a constructor with parameters (like in your case, where you're likely passing a `DungeonLootRepository` instance to it), you need to use a `ViewModelProvider.Factory` to tell the Android framework how to create an instance of your ViewModel.

### Solving the ViewModel Creation Issue

You can solve this issue by creating a custom `ViewModelProvider.Factory` that knows how to instantiate your ViewModel. Here's how you can do it:

1. **Define a ViewModel Factory**

```kotlin
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import android.content.Context

class DungeonViewModelFactory(private val repository: DungeonLootRepository): ViewModelProvider.Factory {
    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(DungeonViewModel::class.java)) {
            @Suppress("UNCHECKED_CAST")
            return DungeonViewModel(repository) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}
```

2. **Use the Factory When Retrieving Your ViewModel**

When you retrieve your ViewModel from a `Fragment` or `Activity`, you need to pass an instance of your factory to the `ViewModelProvider`. Here's an example of how you might do this in a Composable function:

```kotlin
@Composable
fun DungeonGeneratorScreen() {
    // Assuming you have a way to obtain your repository instance here
    // For example, using Dependency Injection, or creating it directly
    val repository = DungeonLootRepository(/* context or other dependencies */)
    val viewModel: DungeonViewModel = viewModel(factory = DungeonViewModelFactory(repository))

    // Your Composable UI code that uses viewModel
}
```

### Important Notes

- **Passing Context to Repositories**: Be careful with passing a `Context` or any other object that can lead to memory leaks into your ViewModel or Repository. Instead, consider using application context or other lifecycle-aware components.
- **Dependency Injection**: If your project uses a Dependency Injection (DI) framework like Dagger or Hilt, you can simplify ViewModel creation and dependency provisioning. DI frameworks can automatically take care of providing the necessary instances to your ViewModel without manually defining a factory. See how I do this in my MVI series of articles.

# The End

MVVM (Model-View-ViewModel) is a powerful architectural pattern that has gained popularity for structuring Android applications (and applications in other frameworks and languages as well) due to its clear separation of concerns, improved testability, and support for responsive and dynamic user interfaces.

This project can be found hosted on my [GitHub Page here](https://github.com/erickveil/MVVM-Table-Roller/tree/master).

