---
title: Using Context in the ViewModel in Kotlin
layout: note
tags:
  - android
  - architecture
  - kotlin
---


In Android development, ViewModel plays a crucial role in managing UI-related data in a lifecycle-conscious way. However, using context within ViewModel can be tricky. 

### Understanding ViewModel

A ViewModel is part of Android's Architecture Components, designed to store and manage UI-related data in a lifecycle-conscious way. It allows data to survive configuration changes such as screen rotations. The primary advantage is that it keeps the UI controllers (like Activities and Fragments) lean by delegating the data handling responsibility to the ViewModel.

### Why Context in ViewModel Can Be Problematic

Contexts in Android are references to the application's state and environment. Using a context in ViewModel can be risky due to potential memory leaks. The lifecycle of a context tied to an Activity or Fragment is often shorter than that of a ViewModel, leading to situations where the ViewModel holds a reference to a destroyed context, causing a memory leak.

### When You Need Context in ViewModel

There are scenarios where you need a context in your ViewModel, such as:
- Accessing resources like strings, dimensions, or images.
- Using system services like `ClipboardManager`, `ConnectivityManager`, etc.
- Interacting with the database, which might require a context.

### Best Practices for Using Context in ViewModel

1. **Application Context**:
   Use the application context instead of the activity or fragment context to avoid memory leaks. The application context is tied to the lifecycle of the entire application, not just an individual activity or fragment.

   ```kotlin
   class MyViewModel(application: Application) : AndroidViewModel(application) {
       private val appContext: Context = getApplication<Application>().applicationContext

       fun getResourceString(id: Int): String {
           return appContext.getString(id)
       }
   }
   ```

2. **Dependency Injection**:
   Use dependency injection (DI) frameworks like Dagger or Hilt to provide dependencies that require a context. This keeps your ViewModel free from direct context references and makes it easier to test.

   ```kotlin
   @HiltViewModel
   class MyViewModel @Inject constructor(
       private val someRepository: SomeRepository
   ) : ViewModel() {

       fun doSomething() {
           someRepository.performAction()
       }
   }
   ```

3. **Context Wrapper**:
   If you must use the context directly in ViewModel, create a context wrapper class that can be injected. This wrapper can handle the context-related operations.

   ```kotlin
   class ContextProvider @Inject constructor(private val appContext: Context) {
       fun getString(id: Int): String {
           return appContext.getString(id)
       }
   }

   class MyViewModel @Inject constructor(
       private val contextProvider: ContextProvider
   ) : ViewModel() {

       fun getResourceString(id: Int): String {
           return contextProvider.getString(id)
       }
   }
   ```

While using context in ViewModel is generally discouraged due to the risk of memory leaks, there are safe ways to handle it when necessary. By using the application context, leveraging dependency injection, or creating a context wrapper, you can keep your ViewModel clean and maintainable. Following these best practices ensures that your application remains robust and leak-free, leading to a better user experience and easier maintenance.
