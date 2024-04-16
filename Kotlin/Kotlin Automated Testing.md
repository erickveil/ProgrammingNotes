---
title: Kotlin Automated Testing
layout: note
tags:
  - android
  - kotlin
  - tests
---

For testing, we're going to use JUnit. When it comes to UI testing, we're going to use Espresso, and when it comes to mocks, we're going to use Mokito.

### Test Directory Structure

We're going to create one test file for each class we test, and mirror the application directory structure to make it more organized.

Our root directory for tests will be: `src/test/java/projectName/...` and from there we will re-create the directory structure of the project.

Only if we run into tests that require the Android environment will we use `src/androidTest/java/projectName/...`

There are example tests in each directory.

### Running Tests

You can run all unit tests at once by right-clicking the `test` folder in Android Studio and selecting `Run 'Tests in 'test...''`.

You can also run individual tests in the same way by right clicking each file, or clicking the run button on the left in the file itself.

# Writing Tests

## The Model

Testing a data model which primarily holds data without complex logic or behavior, might not always be necessary. However, there are scenarios where writing tests for such a model could be beneficial:

1. **Serialization and Deserialization**: If a class is annotated with `@Serializable`, you might want to ensure that it can be correctly serialized into JSON and deserialized back into a Kotlin object. This is particularly relevant if the structure of your JSON might change or if you're using custom serialization logic.

2. **Validation Logic**: If your model includes any validation logic (for example, ensuring that `tableName` is not empty or that `results` contains a specific set of items), testing these validations would be crucial.

### Writing a Test for Serialization and Deserialization

To write a test for serialization and deserialization, you can use the Kotlin serialization library's `Json` object to serialize the `LootTable` object into a JSON string and then deserialize it back. 

Create a test class for `LootTable`:

```kotlin
class LootTableTest {

    @Test
    fun serializeAndDeserialize() {
        val lootTable = LootTable(
            tableName = "Test Table",
            description = "A test loot table.",
            results = listOf("Gold", "Sword", "Potion")
        )

        // Serialize
        val jsonString = Json.encodeToString(lootTable)
        assertNotNull(jsonString)

        // Deserialize
        val deserialized = Json.decodeFromString<LootTable>(jsonString)
        assertEquals(lootTable, deserialized)
    }
}
```

### What This Test Does

1. **Creates an Instance of `LootTable`**: This instance is set up with known values for `tableName`, `description`, and `results`.

2. **Serializes the Object**: Using `Json.encodeToString()`, the `LootTable` object is converted into a JSON string. The test then checks that the resulting JSON string is not `null`.

3. **Deserializes the JSON String**: Using `Json.decodeFromString<LootTable>()`, the JSON string is converted back into a `LootTable` object. The test checks that this newly deserialized object matches the original object.

## The Repository - Mocking with Mockito

The Repository interacts with the Android framework (for reading assets) and involves serialization/deserialization logic. The goal is to ensure that your repository correctly loads and parses the JSON data into a `LootTable` object.

Testing this class involves dealing with:
- The Android `Context` to access assets.
- File I/O operations.
- Serialization/deserialization logic.

### Approach to Testing a Repository

1. **Mock the Context and Assets**: Use a testing framework like Mockito to mock the `Context` and the `AssetManager`. This allows you to control the responses from these components without accessing real files.
2. **Isolate File Reading and Parsing Logic**: Since `loadJsonFromAssets` and `parseJsonToLootTable` are private and tightly coupled with file I/O and serialization logic, consider testing `getLootTable` as a whole. Alternatively, you could make these methods internal or package-private to directly test them, but ensure it doesn't compromise your app's architecture or security.
3. **Use a Sample JSON File in Test Resources**: Place a sample JSON file that matches your `lootTable.json` structure in your test resources directory (`src/test/resources`), allowing you to use it during tests without relying on the actual assets folder.

### Sample Test for `LootRepository`

First, add Mockito and the Kotlin serialization testing dependencies to your `build.gradle` file:

```kotlin
dependencies {  
	testImplementation("org.mockito:mockito-core:+")  
	testImplementation("org.mockito.kotlin:mockito-kotlin:+")  
	testImplementation("io.mockk:mockk:+")  
}
```

Then, create a test class for `LootRepository`. Here's an example illustrating how to mock the `Context` and set up a basic test environment:

```kotlin
@RunWith(MockitoJUnitRunner::class)
class LootRepositoryTest {

    @Mock
    private lateinit var mockContext: Context

    @Mock
    private lateinit var mockAssets: AssetManager

    private lateinit var repository: LootRepository

    @Before
    fun setUp() {
        // Set up to use our mockAssets to load files from instead of actual files.
        `when`(mockContext.assets).thenReturn(mockAssets)

        // Simulate "assets.open(fileName)" to return a ByteArrayInputStream based on a test JSON string
        val testJsonString = """
	        {
		        "tableName":"Test Table",
		        "description":"A test description",
		        "results":["Gold","Magic Sword"]
		    }""".trimIndent()
        
        `when`(mockAssets.open("lootTable.json")).
	        thenReturn(ByteArrayInputStream(testJsonString.toByteArray()))

        repository = LootRepository(mockContext)
    }

    @Test
    fun getLootTable_returnsExpectedData() {
        val result = repository.getLootTable()

        assertNotNull(result)
        assertEquals("Test Table", result?.tableName)
        assertEquals("A test description", result?.description)
        assertTrue(result?.results?.contains("Gold") == true)
        assertTrue(result?.results?.contains("Magic Sword") == true)
    }
}
```

- **MockitoJUnitRunner**: Used to initialize mocks and simplify the setup of test cases.
- **Mocking `Context` and `AssetManager`**: Allows you to simulate the reading of assets without accessing real files.
- **Setting Up Test Conditions**: The `setUp` method configures Mockito to return a `ByteArrayInputStream` when `assets.open` is called, simulating the reading of a JSON file from assets.
- **Testing the `getLootTable` Method**: Checks if the method correctly loads and parses the JSON into a `LootTable` object, then validates the content of the object.

## The ViewModel and Testing Coroutines

### A Testable ViewModel

The ViewModel should not be a complex beast. It is tempting to get out of hand with it since it is the hub where everything connects: 

- The View references the ViewModel's immutable public state.
- It should have just the one publicly accessible interface for `processIntent()` in an MVI design.
- The constructor should only accept the injection of what `ViewModel` or `ApplicationViewModel` require: That is nothing or `ApplicationViewModel`
- Then all of our business should be offloaded into testable classes that provide *their* public interfaces to the `processIntent` switch.

The Repository should not be injected, but instantiated within the ViewModel itself, along with any of the other utility or business classes it needs. If you require it's injection, then all you're doing is offloading the business of instantiation to the View, which should not have to take on that responsibility. It is also going to make setting up a test more difficult.

So all that you are testing on the ViewModel is the `processIntent` and examining the public `state` for acceptable changes.

Here is my ViewModel:

```kotlin
open class LootTableViewModel ( application: Application )  
: AndroidViewModel(application) {  
  
	private val _state = MutableStateFlow(LootTableViewState())  
	val state: StateFlow<LootTableViewState> = _state.asStateFlow()  
	
	private var lootTable: LootTable? = null  
	private var repository: LootRepository  
	  
	init {  
		repository = LootRepository(application.applicationContext)  
		loadLootTable()  
	}  
	  
	private fun loadLootTable() {  
	  
		// This block puts the file loading in a coroutine so that it is an 
		// asynchronous operation.  
		viewModelScope.launch {  
			lootTable = repository.getLootTable()   
			_state.value = _state.value.copy(  
				isLoaded = true  
			)  
		}  
	}  
	  
	fun processIntent(intent: LootTableIntent) {  

		when (intent) {  
			is LootTableIntent.RollLootTable -> rollLootTable()  
		}  
	}  
	  
	private fun rollLootTable() {  
		val randomItem = getRandomLootItem()  
			_state.value = _state.value.copy(  
			resultText = randomItem ?: "No loot found"  
		)  
	}  
	  
	private fun getRandomLootItem(): String? {  
		return lootTable?.results?.takeIf { it.isNotEmpty() }?.random()  
	}  
}
```

Even this is a bit much for a ViewModel. The private business logic here should be offloaded to their own classes and get accessed from `processIntent`. If this were a real project, the Intents would grow fast, and we don't want this class expanding into a gigantic God-class, which it could very easily do.

But this will do for this demonstration, particularly because I want to also demonstrate handling coroutines in a test.

Here is the test:

```kotlin
@ExperimentalCoroutinesApi  
@RunWith(MockitoJUnitRunner::class)  
class LootTableViewModelTest {  
	  
	@Mock  
	private lateinit var mockApplication: Application  
	private lateinit var lootTableViewModel:LootTableViewModel  

	// The dispatcher will allow us to handle coroutines
	private val testDispatcher = StandardTestDispatcher()  
	  
	@Before  
	fun setup() {  
		// Set the main dispatcher to the one we instantiated
		Dispatchers.setMain(testDispatcher)  
		lootTableViewModel = LootTableViewModel(mockApplication)   
	}  
	  
	@After  
	fun tearDown() {
		Dispatchers.resetMain()  
	}  
	  
	@Test  
	fun processIntent_rollLootTable() = runTest {  
	  
		// run out the init method which calls a coroutine  
		advanceUntilIdle()  
		  
		lootTableViewModel.processIntent(LootTableIntent.RollLootTable)  
		val expected = "No loot found"  
		assertEquals(expected, lootTableViewModel.state.value.resultText)  
	  
	}  
}
```

We set up a `StandardTestDispatcher` so that we can call `advanceUntilIdle` before checking the values in our test. We make this call at the start of our `processIntent_rollLootTable()` test because when our ViewModel was instantiated, it ran its `init()` method, and that is where the coroutine for loading external data happens. 

The `advanceUntilIdle` allows us to wait until all coroutines are finished, and we can proceed with the test.

Since our ViewModel is an `ApplicationViewModel` we mock the `Application`object using Espresso for this test.

## Testing the View

Tests for the View where we have Jetpack Compose, are going to need the Android environment, so the test is going to go in the `androidTest` directory instead of the normal `test`.

It's worth noting that in your gradle file, the `testImpementation` libraries are only going to be available in the `test` directory. That means Mockito and Mockk. For the `androidTest` you will want to add the following to the gradle file:

```kotlin
dependencies {  
	// For compose testing  
	androidTestImplementation("androidx.test.espresso:espresso-core:+")  
	androidTestImplementation("androidx.test:runner:+")  
	androidTestImplementation("androidx.test:rules:+")  
	androidTestImplementation("androidx.compose.ui:ui-test-junit4")  
	androidTestImplementation("org.mockito:mockito-android:+")  
	androidTestImplementation("org.mockito.kotlin:mockito-kotlin:+")  
	androidTestImplementation("io.mockk:mockk:+")  
	androidTestImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:+")  
}
```

Remember as always to use Android Studio to assign the latest version numbers.

For Compose, we want to test the following:

- Did the element draw properly on the screen?
- Does the View reflect any changes required by a user's interaction with a composable?

### Referencing Composables

If you've got a classic XML type View, then you'll be grabbing controls from the screen using the familiar `R.id.myIdValue` inside an `onView(withId())` call. But with Jetpack Compose, you don't necessarily have IDs on everything. You *could* grab composables by their visible text, but what if the text is dynamic?

It turns out that composables have a `description` property. It is intended for screen readers for the visually impaired, but we can use that text to identify specific elements on the screen.

For these values, I want to reference them, not only to assign them to the composables, but in the test as well. I'd rather not rely on my ability to accurately copy and paste strings that may change over time. So I'm going to create an object to hold these values:

```kotlin
object ComposableDescription {  
	const val OUTPUT_BOX = "Output Display Box"  
	const val ROLL_BUTTON = "Table Roller Button"  
	const val OUTPUT_TEXT = "Output Box Text"
}
```

Now in the Composable function, I assign this value in the `modifier.semantics` property of each composable:

```kotlin
Text(  
	text = state.value.resultText,  
	modifier = Modifier  
	.padding(controlPadding)  
	.fillMaxWidth()  
	.semantics {  
		this.contentDescription = ComposableDescription.OUTPUT_TEXT  
	},  
	color = Color.Black,  
	style = TextStyle(fontSize = textFontSize)  
)
```

Now here are the tests:

```kotlin
@ExperimentalCoroutinesApi  
@RunWith(AndroidJUnit4::class)  
class LootTableUIEnhancedTest {  

	// Set up an environment to test in  
	@get:Rule  
	val composeTestRule = createComposeRule()  
	  
	@Mock  
	private lateinit var mockApp:Application  
	  
	@Before  
	fun setup() {  
		MockitoAnnotations.openMocks(this)  
	}  
	  
	@After  
	fun tearDown() {  
	}  
	  
	@Test  
	fun isButtonExists() = runTest {  
		composeTestRule.setContent {  
			LootTableUIEnhanced(  
				LootTableViewModel( mockApp )  
			)  
		}  
		// This is how we find a node by its text:
		composeTestRule.onNodeWithText("Roll on Loot Table").assertExists()  
	}  
	  
	@Test  
	fun isOutputBoxDisplayed() = runTest {  
		composeTestRule.setContent {  
			LootTableUIEnhanced(viewModel = LootTableViewModel(mockApp))  
		}  
		// Here we find the node on it's description instead:
		composeTestRule
			.onNodeWithContentDescription(ComposableDescription.OUTPUT_BOX)  
			.assertIsDisplayed()  
	}  
	  
	@Test  
	fun testButtonClick() = runTest {  
		val mockViewModel = LootTableViewModel(mockApp)  
		composeTestRule.setContent {  
			LootTableUIEnhanced(mockViewModel)  
		}  
	  
		val initialText = "Result goes here."  
	  
		// Check the state's starting text  
		assertEquals(initialText, mockViewModel.state.value.resultText)  
		  
		// Check the view's starting text  
		composeTestRule
			.onNodeWithContentDescription(ComposableDescription.OUTPUT_TEXT)  
			.assertTextEquals(initialText)  
		  
		// Click the button  
		composeTestRule
			.onNodeWithContentDescription(ComposableDescription.ROLL_BUTTON)  
			.performClick()  
		  
		// Make sure that the state's text has changed  
		assertNotEquals(initialText, mockViewModel.state.value.resultText)  
		  
		// Test that the displayed text matches the state text  
		// This sounds like "DUH!" but I did find an error while writing 
		// this text:  
		// My OUTPUT_BOX description was on the wrong Text object!  
		composeTestRule.onNodeWithContentDescription(
			ComposableDescription.OUTPUT_TEXT)
				.assertTextEquals(mockViewModel.state.value.resultText)  
	}  
}
```

