---
title: Kotlin REST API
layout: note
tags:
  - android
  - kotlin
  - rest
---

[Project Link](https://github.com/erickveil/Kotlin-ChatGPT-REST)
# Creating the POST

For this project, I'm going to use OpenAI's API, because it's pretty simple. The only authorization it requires is a valid account key, which is a hash string.

Let's start with a utility function that creates the API payload for the POST request. This function accepts an inputMsg string and returns a string in JSON format. So if you put in: "This is the Input Message." You will get returned: 

``` json
{ 
	"model": "gpt-3.5-turbo", 
	"messages": [ 
		{ "role": "system", "content": "You are a helpful assistant." }, 
		{ "role": "user", "content": "This is the Input Message." } 
	] 
} 
```

To create the utility function in Kotlin that formats your input message into the specified JSON structure, you'll need to use a library like `Gson` or `Moshi` to serialize the data into JSON format. Here’s how you could do it using Kotlin and the Gson library:

First, you would add the Gson library to your project by including it in your `build.gradle` file:

```kotlin
dependencies {
    implementation ("om.google.code.gson:gson:+")
}
```

Next, you would write the utility function to create the API payload. Here's how you might implement this in Kotlin:

```kotlin
data class Message(val role: String, val content: String)
data class ApiPayload(val model: String, val messages: List<Message>)

fun createPayload(inputMsg: String): String {
    val systemMessage = Message("system", "You are a helpful assistant.")
    val userMessage = Message("user", inputMsg)
    val payload = ApiPayload("gpt-3.5-turbo", listOf(systemMessage, userMessage))
    
    return Gson().toJson(payload)
}

// Usage example
val jsonPayload = createPayload("This is the Input Message.")
println(jsonPayload)
```

This function does the following:
- Defines data classes `Message` and `ApiPayload` to represent the structure of your JSON.
- Creates instances of `Message` for the system and user messages.
- Bundles these into an `ApiPayload` instance with the specified model string.
- Converts the `ApiPayload` instance into a JSON string using Gson.

You can now use the `createPayload` function to generate JSON payloads for your POST requests.

# Performing the REST Call

Now let's create a class that is in charge of performing the REST call using Retrofit. It is basically going to perform the following, but in Kotlin: 

``` 
curl https://api.openai.com/v1/chat/completions \ 
	-H "Content-Type: application/json" \ 
	-H "Authorization: Bearer $OPENAI_API_KEY" \ 
	-d '{ "model": "gpt-3.5-turbo", "messages": [ { "role": "system", "content": "You are a helpful assistant." }, { "role": "user", "content": "This is the Input Message." } ] }' 
``` 

There will be one public function. This function will require as arguments: - The InputMessage string (our class will use our utility function to generate the payload) - A string for the api key - A lambda that will define what happens with the response from the server when we receive it. (This will let us allow the method caller to decide what to do with the response and how to parse it).

To create a class in Kotlin using Retrofit for making REST calls as described, you need to set up Retrofit to handle the network communication, along with creating a data model and a service interface for the API calls.

First, add the necessary dependencies for Retrofit and Gson in your `build.gradle` file:

```kotlin
dependencies {
    implementation ("com.squareup.retrofit2:retrofit:+")
    implementation ("com.squareup.retrofit2:converter-gson:+")
    implementation ("com.google.code.gson:gson:+")
}
```

Next, let’s define the service interface, the Retrofit setup, and the class to perform the API calls:

### 1. Service Interface
Create an interface for your API calls with Retrofit annotations:

```kotlin
interface OpenAIApiService {
    @POST("v1/chat/completions")
    @Headers("Content-Type: application/json")
    fun createChatCompletion(
        @Header("Authorization") authHeader: String,
        @Body payload: ApiPayload
    ): Call<String>
}

```

### 2. API Client Class
This class will handle the communication with the OpenAI API:

```kotlin
class OpenAIClient {
    private val baseUrl = "https://api.openai.com/"
    
    private val retrofit = Retrofit.Builder()
        .baseUrl(baseUrl)
        .addConverterFactory(GsonConverterFactory.create())
        .build()
    
    private val service = retrofit.create(OpenAIApiService::class.java)

    fun postChatCompletion(inputMessage: String, apiKey: String,
                       onResponse: (Response<String>) -> Unit,
                       onError: (Throwable) -> Unit) {
	    val systemMessage = Message(role = "system", 
		    content = "You are a helpful assistant.")
	    val userMessage = Message(role = "user", content = inputMessage)
	    val payload = ApiPayload(model = "gpt-3.5-turbo", 
		    messages = listOf(systemMessage, userMessage))
	    
	    val authHeader = "Bearer $apiKey"
	    val call = service.createChatCompletion(authHeader, payload)
	    call.enqueue(object : Callback<String> {
	        override fun onResponse(call: Call<String>, 
		        response: Response<String>) {
	            onResponse(response)
	        }
	        override fun onFailure(call: Call<String>, t: Throwable) {
	            onError(t)
	        }
	    })
	}


    companion object {
        fun createPayload(inputMsg: String): String {
            val systemMessage = Message("system", "You are a helpful assistant.")
            val userMessage = Message("user", inputMsg)
            val payload = ApiPayload("gpt-3.5-turbo", 
	            listOf(systemMessage, userMessage))
            
            return Gson().toJson(payload)
        }
    }
}
```

### 3. Usage Example
This example shows how you might use the `OpenAIClient` class:

```kotlin
fun postChatCompletion(inputMessage: String, apiKey: String,
                           onResponse: (Response<String>) -> Unit,
                           onError: (Throwable) -> Unit) {
	val jsonPayload = createPayload(inputMessage)
	val call = service.createChatCompletion(jsonPayload)
	call.enqueue(object : Callback<String> {
		override fun onResponse(call: Call<String>, response: Response<String>) {
			onResponse(response)
		}
		override fun onFailure(call: Call<String>, t: Throwable) {
			onError(t)
		}
	})
}
```

### Explanation:
- **OpenAIClient** handles creating the Retrofit instance and defining the method to post the chat completion.
- **postChatCompletion** constructs the JSON payload using the `createPayload` method, sends the POST request, and allows the caller to specify a lambda to handle the response.
- Retrofit's `enqueue` method is used for making the API call asynchronously, and the results are handled via callback methods, which invoke the lambda passed to `postChatCompletion`.

This setup makes it easy to modify the response handling externally, providing flexibility in how the data is processed or displayed.

# Handling the Response

To create a utility function in Kotlin that handles the response from your REST call, you'll need to parse the JSON response and extract the `message.content` from the first choice in the choices array. This function will be designed to work with Retrofit's asynchronous callback mechanism.

Here’s how you might implement this utility function:

### 1. Define Data Classes
First, define data classes that reflect the structure of the JSON response. This helps in parsing the JSON directly into Kotlin objects using Gson.

```kotlin
data class ChatCompletionResponse(
    val choices: List<Choice>,
    val created: Long,
    val id: String,
    val model: String,
    val `object`: String,
    val usage: Usage
)

data class Choice(
    val finish_reason: String,
    val index: Int,
    val message: Message,
    val logprobs: Any?
)

data class Message(
    val content: String,
    val role: String
)

data class Usage(
    val completion_tokens: Int,
    val prompt_tokens: Int,
    val total_tokens: Int
)
```

### 2. Utility Function
Create the utility function that parses the response and extracts the message content.

```kotlin
fun handleResponse(response: Response<String>): String {
    if (response.isSuccessful) {
        response.body()?.let {
            val gson = Gson()
            val completionResponse = 
	            gson.fromJson(it, ChatCompletionResponse::class.java)
            val firstMessageContent =
	            completionResponse.choices.firstOrNull()?.message?.content
            
            return firstMessageContent ?: "No message content available."
        } ?: return "Response body is null."
    } else {
        return "Error: ${response.errorBody()?.string()}"
    }
}
```

### Explanation:
- **ChatCompletionResponse** and other data classes model the structure of the JSON response. This makes it easy to use Gson to parse the JSON string into Kotlin objects.
- **handleResponse** function checks if the response is successful. If it is, it parses the body using Gson into a `ChatCompletionResponse` object.
- It attempts to retrieve the `content` of the first `message` in the `choices`. If there are no choices or messages, it returns a fallback message "No message content available."
- If the response is not successful or the body is null, appropriate error messages are returned.

This function can be used in the onResponse lambda of your Retrofit call to process the received data:

```kotlin
client.postChatCompletion("This is the Input Message.", apiKey) { response ->
    val messageContent = handleResponse(response)
    println(messageContent)
}
```

This will print the content of the message if available or an error message accordingly.

# The View

Now here is my View:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <EditText
        android:id="@+id/et_apiKey"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="16dp"
        android:layout_marginTop="16dp"
        android:layout_marginEnd="16dp"
        android:ems="10"
        android:inputType="text"
        android:text="API Key"
        android:textAlignment="center"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/tv_response"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="16dp"
        android:layout_marginTop="16dp"
        android:layout_marginEnd="16dp"
        android:background="@color/material_dynamic_secondary80"
        android:minHeight="200dp"
        android:paddingHorizontal="16dp"
        android:paddingVertical="16dp"
        android:text="Response"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/et_apiKey" />

    <EditText
        android:id="@+id/etm_input"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="16dp"
        android:layout_marginTop="16dp"
        android:layout_marginEnd="16dp"
        android:background="@color/material_dynamic_tertiary95"
        android:ems="10"
        android:gravity="start|top"
        android:inputType="textMultiLine"
        android:minHeight="100dp"
        android:paddingHorizontal="16dp"
        android:paddingVertical="16dp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/tv_response" />

    <Button
        android:id="@+id/but_send"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="16dp"
        android:layout_marginTop="16dp"
        android:layout_marginEnd="16dp"
        android:text="Send"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/etm_input" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

And here is the Controller, the MainActivity:

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}
```

We need to make it so that when the user presses `but_send` that we gather the api key from `et_apiKey` and the input message from `etm_input` and uses those as arguments to send along on our REST API call.

When the response comes back, we process it with our utility function, then make `tv_response` show the resulting string.

To achieve the behavior you described in your Android app, you will need to add functionality to the `MainActivity` that handles button clicks, retrieves text from the EditText fields, makes the API call, processes the response, and updates the TextView with the result. Here's how you can set it up:

### Update MainActivity
You'll integrate with the `OpenAIClient` you created earlier and modify the `MainActivity` as follows:

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var apiKeyEditText: EditText
    private lateinit var inputMessageEditText: EditText
    private lateinit var responseTextView: TextView
    private lateinit var sendButton: Button

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Initialize views
        apiKeyEditText = findViewById(R.id.et_apiKey)
        inputMessageEditText = findViewById(R.id.etm_input)
        responseTextView = findViewById(R.id.tv_response)
        sendButton = findViewById(R.id.but_send)

        // Set up the button click listener
        sendButton.setOnClickListener {
            val apiKey = apiKeyEditText.text.toString().trim()
            val inputMessage = inputMessageEditText.text.toString().trim()
            makeApiCall(apiKey, inputMessage)
        }
    }

    private fun makeApiCall(apiKey: String, inputMessage: String) {
	    val client = OpenAIClient()
	
	    // Format the API key as a bearer token
	    val authHeader = "Bearer $apiKey"
	
	    // Define onResponse callback
	    val onResponseCallback: (Response<String>) -> Unit = { response ->
	        val messageContent = handleResponse(response)
	        responseTextView.text = messageContent
	    }
	
	    // Define onError callback
	    val onErrorCallback: (Throwable) -> Unit = { throwable ->
	        responseTextView.text = "Failed to load data: ${throwable.message}"
	        Log.e("APIError", "Error calling API", throwable)
	    }
	
	    // Call the API with the auth header
	    client.postChatCompletion(inputMessage, authHeader, 
		    onResponseCallback, onErrorCallback)
	}
}
```

### Explanation
1. **Initialize Views**: The EditTexts and TextView are retrieved from the layout using their IDs and stored in member variables.
2. **Set Click Listener**: A click listener is set on the `sendButton` that retrieves the text from `apiKeyEditText` and `inputMessageEditText`.
3. **API Call**: The `makeApiCall` method uses the `OpenAIClient` to send the API request. The `apiKey` and `inputMessage` are passed as arguments.
4. **Process and Display Response**: The `handleResponse` utility function processes the response. The result is then displayed in the `responseTextView`.

### Required Modifications in `handleResponse`
Ensure that `handleResponse` can handle a `Response<String>` as intended, or adapt it as necessary to match the actual data structure and types returned by Retrofit.

### Handling API Keys Securely
Be cautious with how you handle API keys in your application. Avoid hard-coding them into your source code. Consider more secure options like using environment variables or secure storage mechanisms provided by Android.

# Manifest Permissions

You don't need to request the internet access permission at runtime as it is not considered a "dangerous" permission by Android. Instead, you simply need to declare it in your app's manifest file.

Here’s how you can add the necessary permission:

### 1. Update the AndroidManifest.xml
Open your `AndroidManifest.xml` file and add the following line within the `<manifest>` element but outside the `<application>` tag:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

This line should look like this in context:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="your.package.name">

    <uses-permission android:name="android.permission.INTERNET" />

    <application
        ...
        ...
        >
        ...
    </application>
</manifest>
```

### Explanation
- **Internet Permission**: This permission is necessary for any app that needs to make network requests. Since it's not a runtime permission, you don't need to request it explicitly from the user while the app is running.

After adding this permission to your manifest, your app should be able to make network requests without encountering the `SecurityException` related to missing internet permissions.

# Logging

The change you've made to the `postChatCompletion` method in your `OpenAIClient` class is appropriate and keeps the functionality encapsulated within the class. This approach not only simplifies the external API of your `OpenAIClient` but also adheres to good software design principles by hiding the details of how the authentication header is assembled. Here’s a brief review and possible enhancement for your implementation:

### Review of Your Updated Implementation

- **Auth Header Assembly**: Moving the assembly of the `authHeader` to within the `postChatCompletion` method is a good practice. It ensures that the API key is always formatted correctly and keeps the API call process streamlined.
- **Logging**: You've added logging for the JSON payload, which is helpful for debugging purposes. Make sure to manage these logs appropriately to avoid exposing sensitive information in production environments.

### Suggested Enhancement

If your application grows or requires additional configuration for network requests, consider further enhancements:

1. **Logging Interceptor**: For debugging network requests, consider adding a logging interceptor to Retrofit. This can help you log requests and responses without manually inserting log statements around payload constructions or API calls.

2. **Error Handling**: Enhance error handling by interpreting common HTTP errors and providing more detailed feedback to the user or to your logging system.

Here’s an example of how to integrate a logging interceptor:

```kotlin
import okhttp3.OkHttpClient
import okhttp3.logging.HttpLoggingInterceptor

class OpenAIClient {
    private val baseUrl = "https://api.openai.com"

    private val loggingInterceptor = HttpLoggingInterceptor().apply {
        level = HttpLoggingInterceptor.Level.BODY
    }

    private val httpClient = OkHttpClient.Builder()
        .addInterceptor(loggingInterceptor)
        .build()

    private val retrofit = Retrofit.Builder()
        .baseUrl(baseUrl)
        .client(httpClient)
        .addConverterFactory(GsonConverterFactory.create())
        .build()

    private val service = retrofit.create(OpenAIApiService::class.java)

    fun postChatCompletion(inputMessage: String, apiKey: String,
                           onResponse: (Response<String>) -> Unit,
                           onError: (Throwable) -> Unit) {
        val jsonPayload = createPayload(inputMessage, apiKey)
        val authHeader = "Bearer $apiKey"
        val call = service.createChatCompletion(authHeader, jsonPayload)
        call.enqueue(object : Callback<String> {
            override fun onResponse(call: Call<String>, response: Response<String>) {
                onResponse(response)
            }
            override fun onFailure(call: Call<String>, t: Throwable) {
                onError(t)
            }
        })
    }
}
```


To use `OkHttpClient` and its related features like the `HttpLoggingInterceptor` in your Android project, you need to add the corresponding dependencies to your `build.gradle.kts` (Kotlin DSL build script). Here's how you can do this:

### Step 1: Add Dependencies
Open your `build.gradle.kts` file and ensure you include the necessary dependencies for `OkHttpClient` and the logging interceptor. Below is the typical setup:

```kotlin
import org.jetbrains.kotlin.gradle.plugin.KotlinDependencyHandler

fun KotlinDependencyHandler.okhttp() {
    implementation("com.squareup.okhttp3:okhttp:4.9.0") // Replace with the latest version as necessary
    implementation("com.squareup.okhttp3:logging-interceptor:4.9.0") // Same version as okhttp
}

dependencies {
    // Retrofit dependencies
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation("com.squareup.retrofit2:converter-gson:2.9.0")
    // Gson
    implementation("com.google.code.gson:gson:2.8.6")

    // OkHttp and Logging Interceptor
    okhttp()

    // Other dependencies
}
```

### Step 2: Sync Project
After updating the build file, sync your project with the new configuration. This can usually be done by clicking on the "Sync Now" prompt in the IDE or manually triggering a sync from the Gradle tool window.

### Explanation
- **Dependencies**: Adding `OkHttpClient` and `HttpLoggingInterceptor` allows you to configure your HTTP client for advanced functionalities like logging network requests and responses, which is extremely useful for debugging API calls.
- **Versions**: Make sure to check for the latest versions of these libraries to keep your application up to date with security patches and new features.

### Usage Example
Once the dependencies are added and the project is synced, you can use `OkHttpClient` with `HttpLoggingInterceptor` as shown in the previous example. This setup will enable you to log detailed information about every network interaction, which aids significantly during development and troubleshooting.

### Final Note
Keep your dependencies updated and review them periodically to ensure compatibility and security. Using the logging interceptor only in debug builds is also a good practice to avoid exposing sensitive information in production releases. You can achieve this by conditionally adding the interceptor like so:

```kotlin
if (BuildConfig.DEBUG) {
    val loggingInterceptor = HttpLoggingInterceptor().apply {
        level = HttpLoggingInterceptor.Level.BODY
    }
    httpClientBuilder.addInterceptor(loggingInterceptor)
}
```

This ensures that logging is only enabled during development, maintaining the security of network communications in your production app.

