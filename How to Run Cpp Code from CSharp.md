---
title: How to Run C++ Code from C#
layout: note
tags:
  - Cpp
  - CSharp
---

You can create a C# GUI application that opens a separate C++ program as a sub-window. This can be achieved by embedding the C++ application window within your C# application. Here's a general outline of the steps involved:

1. **Start the C++ Application**: Use the `Process` class in C# to start the C++ executable.
2. **Find the Window Handle**: Retrieve the window handle (HWND) of the C++ application.
3. **Set the Parent Window**: Use Windows API functions to set the C++ application's window as a child of the C# application's window.

Here is an example of how you might achieve this in C#:

1. **Include necessary namespaces:**
   ```csharp
   using System;
   using System.Diagnostics;
   using System.Runtime.InteropServices;
   using System.Windows.Forms;
   ```

2. **Declare Windows API functions:**
   ```csharp
   public class NativeMethods
   {
       [DllImport("user32.dll", SetLastError = true)]
       public static extern IntPtr FindWindow(string lpClassName, string lpWindowName);

       [DllImport("user32.dll")]
       public static extern IntPtr SetParent(IntPtr hWndChild, IntPtr hWndNewParent);

       [DllImport("user32.dll", SetLastError = true)]
       public static extern bool MoveWindow(IntPtr hWnd, int X, int Y, int nWidth, int nHeight, bool bRepaint);

       [DllImport("user32.dll", SetLastError = true)]
       public static extern bool SetWindowLong(IntPtr hWnd, int nIndex, uint dwNewLong);

       [DllImport("user32.dll", SetLastError = true)]
       public static extern uint GetWindowLong(IntPtr hWnd, int nIndex);

       public const int GWL_STYLE = -16;
       public const uint WS_VISIBLE = 0x10000000;
   }
   ```

3. **Start the C++ application and embed its window:**
   ```csharp
   public partial class MainForm : Form
   {
       private Process cppProcess;
       private IntPtr cppWindowHandle;

       public MainForm()
       {
           InitializeComponent();
           this.Load += MainForm_Load;
       }

       private void MainForm_Load(object sender, EventArgs e)
       {
           StartCppApplication();
           EmbedCppWindow();
       }

       private void StartCppApplication()
       {
           cppProcess = new Process();
           cppProcess.StartInfo.FileName = "path_to_your_cpp_exe";
           cppProcess.Start();
           cppProcess.WaitForInputIdle(); // Wait for the process to be ready
       }

       private void EmbedCppWindow()
       {
           cppWindowHandle = NativeMethods.FindWindow(null, "Title of C++ Window");

           if (cppWindowHandle != IntPtr.Zero)
           {
               NativeMethods.SetParent(cppWindowHandle, this.Handle);

               // Optionally, remove the window border
               uint style = NativeMethods.GetWindowLong(cppWindowHandle, NativeMethods.GWL_STYLE);
               style &= ~NativeMethods.WS_VISIBLE;
               NativeMethods.SetWindowLong(cppWindowHandle, NativeMethods.GWL_STYLE, style);

               // Move the window to the desired location and size
               NativeMethods.MoveWindow(cppWindowHandle, 0, 0, this.ClientSize.Width, this.ClientSize.Height, true);
           }
       }

       protected override void OnResize(EventArgs e)
       {
           base.OnResize(e);
           if (cppWindowHandle != IntPtr.Zero)
           {
               NativeMethods.MoveWindow(cppWindowHandle, 0, 0, this.ClientSize.Width, this.ClientSize.Height, true);
           }
       }
   }
   ```

In this example, the `StartCppApplication` method starts the C++ executable, and the `EmbedCppWindow` method finds the C++ application's window handle and sets it as a child of the C# application's main form. The `OnResize` method ensures the embedded window resizes correctly when the main form is resized.

### Important Considerations:
- Ensure the title of the C++ window is unique and known, as it's used to find the window handle.
- This approach assumes the C++ application has a visible window title and doesn't create additional child windows that might interfere with the embedding process.
- Handling the termination of the C++ application when the C# application closes or vice versa requires additional cleanup code.

This solution uses P/Invoke to call native Windows API functions, which can be powerful but requires careful handling to avoid resource leaks or crashes.

## NativeMethods Class

The Windows API methods don't exist in C#, so we need to create a wrapper to use them.

The `NativeMethods` class in the example is used to declare and import functions from the Windows API that are necessary to interact with the windows of other processes. These functions allow you to manipulate window handles, set parent-child relationships between windows, and modify window properties. Here’s a breakdown of what each part of the `NativeMethods` class does:

1. **Declaring Windows API Functions:**

    - **FindWindow**:
      ```csharp
      [DllImport("user32.dll", SetLastError = true)]
      public static extern IntPtr FindWindow(string lpClassName, string lpWindowName);
      ```
      This function searches for a window by its class name or window name (title). It returns a handle to the window (HWND). If either parameter is `null`, it matches any window with the other parameter.

    - **SetParent**:
      ```csharp
      [DllImport("user32.dll")]
      public static extern IntPtr SetParent(IntPtr hWndChild, IntPtr hWndNewParent);
      ```
      This function sets a new parent window for a specified child window. It takes two parameters: the handle to the child window and the handle to the new parent window. It returns the handle to the previous parent window.

    - **MoveWindow**:
      ```csharp
      [DllImport("user32.dll", SetLastError = true)]
      public static extern bool MoveWindow(IntPtr hWnd, int X, int Y, int nWidth, int nHeight, bool bRepaint);
      ```
      This function changes the position and size of a specified window. It takes the handle to the window and the new position (X, Y) and size (width, height). The `bRepaint` parameter specifies whether the window should be repainted after moving.

    - **SetWindowLong** and **GetWindowLong**:
      ```csharp
      [DllImport("user32.dll", SetLastError = true)]
      public static extern bool SetWindowLong(IntPtr hWnd, int nIndex, uint dwNewLong);

      [DllImport("user32.dll", SetLastError = true)]
      public static extern uint GetWindowLong(IntPtr hWnd, int nIndex);
      ```
      These functions change and retrieve the attributes of a specified window. `SetWindowLong` changes an attribute and `GetWindowLong` retrieves an attribute. The `nIndex` parameter specifies which attribute to set or get, and `dwNewLong` is the new value to set.

2. **Constants:**

    - **GWL_STYLE**:
      ```csharp
      public const int GWL_STYLE = -16;
      ```
      This constant is used to indicate that we want to change or retrieve the window style attribute. It's used with `SetWindowLong` and `GetWindowLong`.

    - **WS_VISIBLE**:
      ```csharp
      public const uint WS_VISIBLE = 0x10000000;
      ```
      This constant represents the visible window style. It's a bitmask used to modify the window style to make it visible or invisible.

Here's a more detailed explanation of how each part fits into the overall process:

1. **Finding the C++ Window:**
   - You use `FindWindow` to get the handle of the C++ application's window by its title. The returned handle allows you to manipulate the window.

2. **Setting the Parent Window:**
   - `SetParent` is used to embed the C++ application's window inside your C# application's window by setting the C# window as the parent of the C++ window.

3. **Modifying Window Styles:**
   - `GetWindowLong` and `SetWindowLong` are used to retrieve and modify the window styles of the C++ application's window. In this case, they are used to ensure the C++ window is visible and possibly to remove window decorations like borders and title bars.

4. **Positioning and Resizing the Window:**
   - `MoveWindow` is used to position and resize the C++ application's window to fit inside the C# application's client area. This function is called when the C# window is resized to keep the embedded window in sync.

This approach leverages the Windows API to allow a C# application to control and embed another application's window, creating the appearance of a unified program.