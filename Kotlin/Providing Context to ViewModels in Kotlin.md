---
title: Providing Context to ViewModels in Kotlin
layout: note
tags:
  - android
  - kotlin
---


In an MVI (Model-View-Intent) application architecture, especially in Android development using Kotlin, it's important to follow best practices regarding separation of concerns and lifecycle awareness. Directly passing a `Context` (like from `MainActivity`) to your `ViewModel` is generally discouraged due to potential memory leaks and the violation of the separation of concerns principle. However, there are several approaches to safely access resources or perform operations that require a `Context` in your `ViewModel`.

### Use Application Context

If your `ViewModel` needs access to the application context (which is safe and does not lead to memory leaks because it's tied to the application's lifecycle, not an activity's lifecycle), you can use `AndroidViewModel` instead of the regular `ViewModel`. `AndroidViewModel` includes a reference to the application context.

Here's how you can implement it:

```kotlin
class MyViewModel(application: Application) : AndroidViewModel(application) {
    // Now you can use application context
    val appContext = application.applicationContext

    fun someFunction() {
        // Use appContext as needed
    }
}
```

**Create/Get your ViewModel in `MainActivity`:**

In your `MainActivity` or any other Activity/Fragment where you need to use the `ViewModel`, you can obtain an instance of your `ViewModel` like this:

```kotlin
import androidx.lifecycle.ViewModelProvider

class MainActivity : AppCompatActivity() {
    private lateinit var viewModel: MyViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Initialize the ViewModel
        viewModel = ViewModelProvider(this)[MyViewModel::class.java]
    }
}
```

When you use `ViewModelProvider(this).get(MyViewModel::class.java)`, if your `ViewModel` extends `AndroidViewModel`, the Android framework automatically provides the application context to the `ViewModel`'s constructor. This is because `AndroidViewModel` expects the application context as its constructor parameter, and the `ViewModelProvider` is designed to handle this automatically.

### Repository Pattern

If your `ViewModel` needs to perform operations that require context (like accessing shared preferences, databases, or resources), a good practice is to encapsulate these operations within a repository or a similar abstraction layer. This repository can then take the `Context` as a parameter in its constructor, usually preferring the application context to avoid memory leaks.

```kotlin
class MyRepository(private val context: Context) {
    // Use context to access resources, databases, etc.
}

class MyViewModel(private val repository: MyRepository) : ViewModel() {
    // Interact with the repository
}
```

In your `MainActivity` or the respective component responsible for the creation of the ViewModel, make sure to provide the repository to the ViewModel, possibly using a ViewModel factory if necessary.

### ViewModelProvider.Factory

If you need to pass specific dependencies to your `ViewModel`, you can create a custom `ViewModelProvider.Factory`. This is a bit more involved but gives you flexibility in dependency injection without breaking the architecture principles.

```kotlin
class MyViewModelFactory(private val application: Application, private val someDependency: SomeType) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(MyViewModel::class.java)) {
            @Suppress("UNCHECKED_CAST")
            return MyViewModel(application, someDependency) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}
```

Then, when you instantiate your ViewModel in `MainActivity`, use this factory.

Remember, direct use of `Context` in the ViewModel for UI-related tasks or storing references to activity contexts can lead to memory leaks and should be avoided. Always use the application context or properly scoped dependencies.

