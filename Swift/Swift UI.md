---
title: Swift UI
layout: note
tags:
  - swift
  - iOS
  - UI
---

To create an iOS app in Swift that displays a button and a text label, and updates the label when the button is tapped, you can use UIKit to set up the user interface. Here’s a simple example on how you can implement this:

1. **Setup Your Xcode Project**: Start a new project in Xcode and select an iOS App template. Make sure to choose "Storyboard" for the user interface.

2. **Design the Interface**:
   - Open `Main.storyboard`.
   - Drag a Button and a Label from the object library to your view controller.
   - Arrange the button and label in the view as desired.
   - Set constraints for the button and label to ensure they are positioned correctly on different device sizes.

3. **Create Outlets and an Action**:
   - Open the Assistant Editor to view the `ViewController.swift` file alongside your storyboard.
   - Control-drag from the label to the `ViewController.swift` file to create an outlet for the label. Name it `label`.
   - Control-drag from the button to the `ViewController.swift` file and choose "Action" under Connection. Name the action something like `changeText`, and make sure the event is set to "Touch Up Inside".

4. **Implement the Action Method**:
   - In the `ViewController.swift`, you should now have an action method that looks something like this:

     ```swift
     @IBAction func changeText(_ sender: UIButton) {
         label.text = "Goodbye"
     }
     ```

   - This method will change the text of the label to "Goodbye" whenever the button is tapped.

Here is the full code for the `ViewController.swift` that includes both the outlet and the action:

```swift
import UIKit

class ViewController: UIViewController {

    @IBOutlet weak var label: UILabel!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // You can initialize anything else here if needed
    }

    @IBAction func changeText(_ sender: UIButton) {
        label.text = "Goodbye"
    }
}
```

5. **Run Your App**: Select an iOS simulator and run your app. You should see the label and button. When you tap the button, the label should change from its initial text to "Goodbye".

This setup is very basic and you can expand upon it by adding more UI elements and functionality as needed.

# Dynamic UI

You can create and display the button and label dynamically in your iOS app using Swift without using a storyboard. Here's how you can set up the user interface entirely in code:

1. **Create a new iOS App Project**: Start by creating a new project in Xcode. Choose the option to not use storyboards.

2. **Setup Your ViewController**: In your `ViewController.swift` file, you will add the code to create a button and a label programmatically. You will also set up constraints or layout parameters to position your UI elements.

3. **Write the Code**:
   - First, add properties to your `ViewController` class for the label and the button.
   - In the `viewDidLoad()` method, initialize these properties, set their attributes, and add them to the view.
   - Create an action method for handling button taps and changing the label text.

Here’s a full example of how you might write this:

```swift
import UIKit

class ViewController: UIViewController {
    
    let label = UILabel()
    let button = UIButton(type: .system) // using .system to get a button with default styling
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Setup label
        label.text = "Hello"
        label.textAlignment = .center
        view.addSubview(label)
        
        // Setup button
        button.setTitle("Change Text", for: .normal)
        button.addTarget(self, action: #selector(changeText), for: .touchUpInside)
        view.addSubview(button)
        
        label.translatesAutoresizingMaskIntoConstraints = false
        button.translatesAutoresizingMaskIntoConstraints = false
        
        // Constraints for label
        NSLayoutConstraint.activate([
            label.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            label.centerYAnchor.constraint(equalTo: view.centerYAnchor, constant: -20),
            label.widthAnchor.constraint(equalToConstant: 200),
            label.heightAnchor.constraint(equalToConstant: 20)
        ])
        
        // Constraints for button
        NSLayoutConstraint.activate([
            button.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            button.topAnchor.constraint(equalTo: label.bottomAnchor, constant: 20),
            button.widthAnchor.constraint(equalToConstant: 200),
            button.heightAnchor.constraint(equalToConstant: 50)
        ])
    }
    
    @objc func changeText() {
        label.text = "Goodbye"
    }
}
```

**Explanation**:
- **UILabel and UIButton Initialization**: Both the label and the button are initialized and added to the view’s subview.
- **Auto Layout Constraints**: Constraints are added programmatically to position the label and the button. We ensure the label is centered and place the button just below the label.
- **Button Action**: The button’s action is set to call the `changeText()` method when a "Touch Up Inside" event occurs, which changes the label's text.

4. **Disable the Use of Storyboards**:
   - Ensure that the "Main Interface" entry in your project settings under "Deployment Info" is empty to indicate that the app does not use a storyboard.
   - Update the `SceneDelegate.swift` to make sure your `ViewController` is used at launch:

```swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        guard let windowScene = (scene as? UIWindowScene) else { return }
        
        window = UIWindow(windowScene: windowScene)
        window?.rootViewController = ViewController()
        window?.makeKeyAndVisible()
    }
}
```

5. **Run the App**: Build and run your app. The UI will show your dynamically created label and button, with functionality to change the text when the button is tapped.

This approach gives you full control over the UI elements and their layout without relying on the Interface Builder.

