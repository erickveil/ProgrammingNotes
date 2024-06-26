---
title: MVC Project Development
layout: note
tags:
  - android
  - architecture
  - kotlin
  - mvc
---

MVC (Model-View-Controller) is a popular design pattern often used in web development, but it's also quite relevant in Android development. The idea behind MVC is to separate the application's concerns into three interconnected components:

- **Model**: This represents the data and the business logic of your application. It's where your database, network code, and data manipulation logic typically live. In an Android context, this might be your SQLite database, a REST API integration, or any internal logic classes that handle the data your application uses.
    
- **View**: The View is what the user sees and interacts with. It displays the data (the model) and sends user actions (like button clicks) to the controller to process. In Android, this is usually represented by the XML layout files, but also includes the Android Views (like `TextView`, `Button`, `RecyclerView`) you interact with in your Java or Kotlin code.
    
- **Controller**: This component acts as an intermediary between the Model and the View. It listens to the user actions performed on the View, processes them (possibly updating the Model), and can update the View to reflect changes in the Model. In Android, Activities and Fragments typically take on the role of the controller, managing the lifecycle of your views and responding to user input.

This is going to be much more straight forward than our MVI project. MVC is the classic method for Android development. 

### Approach Considerations

MVC is going to work best if we're going to use XML for our View, as it's a bit of a wedge to mash Jetpack Compose into this pattern. In Compose, the View and Controller aspects tend to blend due to its declarative nature. Really, you don't have Controllers in the traditional sense. The logic for responding to user interactions and updating the state is often embedded within or directly managed by the composable functions themselves or by ViewModel components that the composable functions observe. Compose fits more naturally with patterns like MVVM (Model-View-ViewModel).

# The Model

Here's my old JSON table we're going to use as the data source:

```json
{ 
	"tableName": "Dungeon Loot Table", 
	"description": "This table determines what loot you find in the dungeon.", 
	"results": [ "Gold coins", "Magic sword", "Potion of healing" ] 
}
```

As usual, I'm putting this in the `assets` directory.

Yes, putting your JSON file in the `assets` directory of your Android project is a good approach. The `assets` directory is the right place for files that you want to include in your application and access as raw byte streams. It's particularly useful for static files that you bundle with your app, like JSON files, because it allows you to read the file contents as a String, which you can then parse.

Here's how you can structure your Kotlin Model to parse the JSON using `kotlinx.serialization`, a powerful Kotlin library for serializing and deserializing JSON data into Kotlin objects. First, you'll need to add the library to your project.

1. **Add the dependencies** to your `build.gradle` file:
```kotlin
dependencies {  
	implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:+")  
}
```

And the plugin at the top of your `build.gradle`:
```kotlin
plugins {  
	kotlin("plugin.serialization") version "+"  
}
```

2. **Define your data classes** to model the JSON structure. These classes will be used by `kotlinx.serialization` to deserialize the JSON into Kotlin objects.

**LootTable.kt**:
```kotlin
@Serializable
data class LootTable(
    @SerialName("tableName") val tableName: String,
    @SerialName("description") val description: String,
    @SerialName("results") val results: List<String>
) {
    fun getRandomItem(): String = results[Random.nextInt(results.size)]
}
```

# The View

Honestly, I just use the WSYIWYG editor in Android Studio to set up my view. It's just faster than editing the XML. Even still, my view is just a Button and a Text View:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/system_accent1_100"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/but_rollOnTable"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="16dp"
        android:layout_marginTop="16dp"
        android:layout_marginEnd="16dp"
        android:text="Roll For Loot"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/tv_output"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="16dp"
        android:layout_marginTop="16dp"
        android:layout_marginEnd="16dp"
        android:background="@color/material_dynamic_neutral70"
        android:minHeight="200dp"
        android:paddingHorizontal="16dp"
        android:paddingVertical="16dp"
        android:text="Result will appear here."
        android:textAlignment="center"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/but_rollOnTable" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

# The Controller

Setting up the Controller part (in this case, your Activity) in Kotlin to interact with these UI elements will be straightforward. You'll set up an `OnClickListener` for the button that, when pressed, calls a function to roll on the loot table and updates the TextView with the selected item.

Let's create a Kotlin Activity that loads the loot table from the JSON file in the assets directory, waits for the user to press the roll button, and then displays a random item from the loot table in the TextView.

**MainActivity.kt**:
```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var lootTable: LootTable

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Load the loot table from the JSON file
        lootTable = loadLootTable(this)

        // Find the button and text view by their IDs
        val rollButton: Button = findViewById(R.id.but_rollOnTable)
        val outputTextView: TextView = findViewById(R.id.tv_output)

        // Set a click listener on the button to roll on the loot table and update the text view
        rollButton.setOnClickListener {
            val selectedItem = lootTable.getRandomItem()
            outputTextView.text = selectedItem
        }
    }

    private fun loadLootTable(context: Context): LootTable {
        val jsonString = assets.open("loot_table.json").bufferedReader().use { it.readText() }
        return Json.decodeFromString(LootTable.serializer(), jsonString)
    }
}
```

This code does the following:

1. **Loads the loot table** from the JSON file in the `assets` directory when the activity is created. Make sure your JSON file is named `loot_table.json` and is placed in the `assets` folder. If you haven't created this folder yet, you can do so by right-clicking on the `src/main` directory in Android Studio, choosing `New > Directory`, and naming it `assets`.

2. **Finds the button and text view** in the layout by their IDs.

3. **Sets a click listener on the button**. When the button is pressed, it calls `getRandomItem()` on the `lootTable` object to select a random item, then updates the text view to display this item.

# The End

This setup follows the MVC pattern with:
- **Model**: Represented by `LootTable.kt`, handling data and logic.
- **View**: The UI elements in `activity_main.xml`.
- **Controller**: The logic within `MainActivity.kt` that binds the Model and View together, responding to user interactions and updating the UI accordingly.

It's pretty simple compared to my **Eight Part** 😲 MVI walkthrough which does the same thing as this one. I think if you're going to want to put together a project that's not MVC, you're going to have to come up with a very good reason to give up the simplicity of development you've got here. 

One reason that I can think of right now might be: Maybe you're working on a more dynamic UI. The static nature of the XML does not lend itself very well to dynamically generating UI elements on the fly. It's not impossible, but I would rather do that job in Compose.

Another, more common reason: The state of your Controller is not preserved between configuration changes, such as screen rotation. In these cases, the Activity or Fragment is destroyed and rebuilt. In an app like this one, the random result would be lost. You can manually preserve state yourself, or make use of the ViewModel, but at that point, you're right back in MVI or MVVM territory. If this project grows in functionality, it's going to be easier to use one of those.

Here's the whole project on GitHub: [MVC Table Roller](https://github.com/erickveil/MVC_Table_Roller)
