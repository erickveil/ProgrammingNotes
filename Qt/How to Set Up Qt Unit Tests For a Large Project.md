---
title: How to Set Up Qt Unit Tests For a Large Project
layout: note
tags:
  - Qt6
  - Cpp
  - UnitTest
  - TDD
---

# Create a Subdirs Project
The Subdir project will be your top level test which runs the whole test suite. All of the tests you create for your project will be run when you run from this project.

The project by itself is nothing. There is no code outside of the project file. It is like a project that contains only other projects. A Subdirs project requires other projects to run.

Create a Subdirs project like any other:

- File -> New Project -> Other Project -> Subdirs Project

## Create Your First Unit Test

Once you finish your initial Subdirs project, the Wizard will automatically re-run for you to create your subproject.

Create your Autotest project like normal. The default location will be selected for this project as a subdirectory inside the Subdir project. You will also see the Subdir project automatically selected for the "Create as Subproject" dropdown.

# Create Subsequent Unit Tests

You can now create more Autotest projects. You can create one for each class in your main project if you like, or whatever subdivision makes sense for you.

Just create an Autotest project like normal. When it comes time to select a directory for your project, set it inside your Subdirs project directory.

You will get the option to make this project a subproject of your Subdirs project during creation.

# Add Existing Unit Tests

You already made a bunch of unit tests, and now you want to organize them under a Subdir project? No problem.

Go to your open Subdir project in the **Projects** sidebar and right click the project root. Select "Add existing project". This option is greyed out if you're not right clicking a subdir project, but since we're in the correct project type, we can add new tests.

You can navigate to your existing unit test wherever it is and add it to the Subdir project.

You might be tempted to move your existing test projects inside the Subdir project to keep it neatly in place. Know that you might wind up having to change any relative paths to classes outside of the tests if you do that.