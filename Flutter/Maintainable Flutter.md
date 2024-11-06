
I like clean, clear code. I want to be able to read my code at 5:00 PM on a Friday, with my brain already half in the weekend, and still understand it because it imposes such a low cognitive load on the human reader. 

My main criticism of Flutter is not in the code itself, but in how the structure of its UI code is presented in documentation and examples. I am afraid that the antipatterns present will affect how actual developers code in this language, and the result will be less maintainable code that is difficult to read.

Here is actual code from Flutter's own [documenataion](https://docs.flutter.dev/get-started/fundamentals/widgets):

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp( // Root widget
      home: Scaffold(
        appBar: AppBar(
          title: const Text('My Home Page'),
        ),
        body: Center(
          child: Builder(
            builder: (context) {
              return Column(
                children: [
                  const Text('Hello, World!'),
                  const SizedBox(height: 20),
                  ElevatedButton(
                    onPressed: () {
                      print('Click!');
                    },
                    child: const Text('A button'),
                  ),
                ],
              );
            },
          ),
        ),
      ),
    );
  }
}
```

When encountering code examples, documentation, and tutorials online, it is common for the code itself to be structured in a way that makes the specific point that an author is trying to demonstrate. That point is most often "how this specific task is accomplished," and that is the end of it. The point is never "This is how you write maintainable code." 

Even I don't write maintainable code myself the first time out the gate when I'm programming. Most of my "first drafts" of code tend to focus on "get the job done," or "just make it work." I always have to refactor to make it maintainable. 

Here is an example of my own code for a very simple UI in Flutter. This is the "First Draft". It matches all the anitpatterns you find in Flutter's own documentation:

```dart
class ListenerPage extends StatelessWidget {
  final TextEditingController _portController =
    TextEditingController(text: '50502');

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Container(
          width: 350,
          child: Padding(
            padding: const EdgeInsets.all(16.0),
            child: Consumer<ListenerViewModel>(
              builder: (context, viewModel, child) {
                return Column(
                  mainAxisSize: MainAxisSize.min,
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Container(
                      width: 300,
                      child: TextField(
                        controller: _portController,
                        keyboardType: TextInputType.number,
                        decoration: InputDecoration(
                          labelText: 'Port Number',
                          border: OutlineInputBorder(),
                        ),
                      ),
                    ),
                    SizedBox(height: 16.0),
                    Row(
                      children: [
                        ElevatedButton(
                            onPressed: () {
                              if (_portController.text.isNotEmpty) {
                                int port = int.parse(_portController.text);
                                viewModel.toggleListener(port);
                              }
                            },
                            style: ElevatedButton.styleFrom(
                                backgroundColor:
                                    const Color.fromARGB(255, 162, 210, 234)),
                            child: Text(viewModel.isListening
                                ? 'Stop Listener'
                                : 'Start Listener')),
                        SizedBox(width: 16.0),
                        Text(
                          viewModel.isListening ? 'Listening...' : 'Stopped',
                          style: TextStyle(
                            color: viewModel.isListening
                                ? const Color.fromARGB(255, 46, 107, 48)
                                : const Color.fromARGB(255, 112, 29, 23),
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                      ],
                    ),
                    SizedBox(height: 24.0),
                    Expanded(
                      child: Container(
                        width: 300,
                        height: 150,
                        padding: EdgeInsets.all(8.0),
                        decoration: BoxDecoration(
                          color: Color(0xffe6e8fa),
                          border: Border.all(color: Colors.blueGrey),
                          borderRadius: BorderRadius.circular(20.0),
                        ),
                        child: SingleChildScrollView(
                          child: Text(
                            viewModel.receivedMessages,
                            style: TextStyle(fontSize: 16.0),
                          ),
                        ),
                      ),
                    ),
                  ],
                );
              },
            ),
          ),
        ),
      ),
    );
  }
}
```

If you took one look at that code sample, eyes glazed over, and skipped down to this text, I don't blame you. I would to.

There are obscene amounts of nesting. Whole objects are instantiated *inside the argument lists* of other objects! 

Flutter's documentation for UI development appears to encourage egregious amounts of nesting. It sort-of makes sense - we are used to nested UI code written in a DOM structure like HTML, or WPF; or in a JSON-like structure like QML. These files are written with the hierarchy implicit in their layout, and the nesting comes with that unless it's been minified into something even more unreadable.

But all of those markup languages have OOP based analogs in other languages for when you want to write more dynamic GUIs. In those cases, we stick to patterns for readable OOP code, not strictly adhere to a nested hierarchy. So why do it here?

Let's look at a section of the above code. Crtl-F and find the node for the `Column`. 

First of all, you'll have no idea what the purpose of this Column is in the design. You'll have to read a whole bunch of surrounding code (and hope it's related) to get the context. That takes *effort*. Exerting effort violates [The First Virtue](https://thethreevirtues.com/)

Suppose we look at our UI, and we realize we want to nest that `Column` inside a `SizeBox` object for the layout. We start by adding the `SizeBox` just above `Column` and assigning the `Column` to its `child` parameter:

```dart
return SizeBox (
	child: Column(
		mainAxisSize: MainAxisSize.min,
		crossAxisAlignment: CrossAxisAlignment.start,
		children: [
...
```

Now we need to put in the ending `)` parenthesis for the `SizeBox` argument list - but where does it go? Somewhere in here?:

```dart
                        ),
                      ),
                    ),
                  ],
                );
              },
            ),
          ),
        ),
      ),
    );
  }
}
```

Good luck with that!

# Refactoring the UI Code

I posit that **we do not need the UI code to be nested or to show off its hierarchy.** Maintaining the nested structure is not as important as keeping the code readable. The hierarchy will not be as useful to us in the future as is the readability of the code and the identification of what is being defined.

The process to flatten out this nesting is to define objects individually, assign them to self-describing variables, and using those as arguments in other objects. 

The naming scheme should allow the reader to make a connection from the visible element on the page to the object that is defined. Therefore, the name should not tell us that this is a "TextWidget". We already know that from the class name. It should tell us the *purpose* of that text widget.

Let's start with the innermost nested object:

```dart
TextStyle(
	color: viewModel.isListening
		? const Color.fromARGB(255, 46, 107, 48)
		: const Color.fromARGB(255, 112, 29, 23),
	fontWeight: FontWeight.bold,
	),
```

The only indication of this object's purpose is if we are able to discern it from the context of it's position of the hierarchy, and/or interpret it from its own argument values. The need to do that adds to our cognitive load as a reader. It's a little bit on just this one piece, but as we work on legacy code, it adds up quickly.

This makes use of the `viewModel` object because it uses the program's state to decide the style of the text it's attached to, so it must be define inside the `builder` method of the Padding.

Let's pull it out and refactor:

```dart
TextStyle indicatorTextStyle = TextStyle(
	color: viewModel.isListening
		? const Color.fromARGB(255, 46, 107, 48)
		: const Color.fromARGB(255, 112, 29, 23),
	fontWeight: FontWeight.bold,
);
```

Now we can assign the object `indicatorTextStyle` to the `Text` widget's `style` argument:

```dart
Text(
  viewModel.isListening ? 'Listening...' : 'Stopped',
  style: indicatorTextStyle,
),
```

And next we will pull out that same Text and make *it* into its own object, and repeat until we run out of objects:

```dart
builder: (context, viewModel, child) {
	TextStyle indicatorTextStyle = TextStyle(
		  color: viewModel.isListening
			  ? const Color.fromARGB(255, 46, 107, 48)
			  : const Color.fromARGB(255, 112, 29, 23),
		  fontWeight: FontWeight.bold,
	  );

	Text indicatorText = Text(
	  viewModel.isListening ? 'Listening...' : 'Stopped',
	  style: indicatorTextStyle,
	);

	Text listenButtonText = Text(
	  viewModel.isListening ? 'Stop Listener' : 'Start Listener'
	);

	ButtonStyle listenButtonStyle = ElevatedButton.styleFrom(
	  backgroundColor: const Color.fromARGB(255, 162, 210, 234)
	);

	ElevatedButton listenButton = ElevatedButton(
	  onPressed: () {
		if (_portController.text.isNotEmpty) {
		  int port = int.parse(_portController.text);
		  viewModel.toggleListener(port);
		}
	  },
	  style: listenButtonStyle,
	  child: listenButtonText,
	);

	Row listenButtonRow = Row(
	  children: [
		listenButton,
		SizedBox(width: 16.0),
		indicatorText,
	  ],
	);

	InputDecoration portInputHelperText = InputDecoration(
	  labelText: 'Port Number',
	  border: OutlineInputBorder(),
	);

	TextField portInputBox = TextField(
	  controller: _portController,
	  keyboardType: TextInputType.number,
	  decoration: portInputHelperText,
	);

	Container portInputSizingBox = Container(
	  width: 300,
	  child: portInputBox,
	);

	Text outputText = Text(
	  viewModel.receivedMessages,
	  style: TextStyle(fontSize: 16.0)
	);

	BoxDecoration outputBoxStyle = BoxDecoration(
	  color: Color(0xffe6e8fa),
	  border: Border.all(color: Colors.blueGrey),
	  borderRadius: BorderRadius.circular(20.0),
	);

	Container outputBox = Container(
	  width: 300,
	  height: 150,
	  padding: EdgeInsets.all(8.0),
	  decoration: outputBoxStyle,
	  child: SingleChildScrollView( child: outputText ),
	);

	Column pageRootColumn = Column(
	  mainAxisSize: MainAxisSize.min,
	  crossAxisAlignment: CrossAxisAlignment.start,
	  children: [
		portInputSizingBox,
		SizedBox(height: 16.0),
		listenButtonRow,
		SizedBox(height: 24.0),
		Expanded( child: outputBox,),
	  ],
	);

	return pageRootColumn;
}
```

This is just the builder method, and we can continue to do this with the rest of the class.

It's a large method still. You might want to offload logical groupings of objects into their own factory methods that clearly describe what's being created:

```dart
builder: (context, viewModel, child) {
	TextField portInput = createPortInputBox(viewModel);
	Row controlPanel = createListenButtonRow(viewModel);
	Expanded outputDataArea = createOutputBox(viewModel);
	SizedBox spacer = SizedBox(height: 16.0);

	Column pageRootColumn = Column(
	  mainAxisSize: MainAxisSize.min,
	  crossAxisAlignment: CrossAxisAlignment.start,
	  children: [
		portInput,
		spacer,
		controlPanel,
		spacer,
		outputDataArea,
	  ],
	);
	
	return pageRootColumn;
}
```

That is *much* easier to read! 

If you find yourself in a position where you have to care about implementation details, you can always drill down into the factory methods, but it is very clear where to go looking for things because we named them after their *purpose*. The removal of the nesting is replaced by clear communication to the **human** reader. 

- If I later want to encase the `controlPanel` in a decorative box, it will be very easy to do so. 
- If I have to debug what's going on with the `outputBox`, I will be able to visit a small, focused method and not be distracted by all the clutter around it.

Notice I don't have to get crazy about it. Objects with just a couple of arguments are easily instantiated in-line still - like those `SizeBox` spacers. They're not that important, and I can still tell which one I'm dealing with by their position in the `Column` list.

The `builder` method itself can probably be offloaded to its own lambda or method as we continue to refactor the rest of the class.

Also, the helper methods should make automated testing sections of the UI a magnitude easier.

The final, refactored whole `build` method is now as easy to read as this:

```dart
@override
  Widget build(BuildContext context) {
    Consumer pageBuilder = Consumer<ListenerViewModel>(
      builder: _buildListenerPage
    );

    Padding edgeSaver = Padding(
      padding: const EdgeInsets.all(16.0),
      child: pageBuilder,
    );

    Container pageRoot = Container(
      width: 350,
      child: edgeSaver,
    );

    Center pageAlignement = Center ( child: pageRoot );

    return Scaffold( body: pageAlignement);
}
```

You can see the original file here:

https://github.com/erickveil/FlutterListener/blob/master/lib/listener_page.dart

You can see the fully refactored file here:

https://github.com/erickveil/FlutterListener/blob/934b1045e44acfaf5a6e00b6e639f8a1130b558d/lib/listener_page.dart

The diff is here:

https://github.com/erickveil/FlutterListener/pull/1/files

