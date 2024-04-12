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



