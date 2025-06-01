---
title: How to Parse Subcommands and Args With Qt
layout: note
tags:
  - command
  - Cpp
  - Qt6
---

You **can** implement this type of command-style argument handling (like `git push ...` or `apt update ...`) using `QCommandLineParser`, but Qt’s parser is not as directly geared for subcommands as some CLI libraries in other ecosystems (like Python’s `argparse` or C++’s [CLI11](https://cliutils.github.io/CLI11/book/)). However, with some careful structure, you can achieve what you want.

Here's **how you would approach this** with `QCommandLineParser`:

### 1. Parse the main (global) options and extract the subcommand

- The first non-option argument after the program name is your subcommand (`push`, `commit`, etc.).
- After that, you can parse arguments specific to the subcommand.

### 2. Example `main.cpp`

Here's a simple structure:

```cpp
#include <QCoreApplication>
#include <QCommandLineParser>
#include <QDebug>

int main(int argc, char *argv[])
{
    QCoreApplication app(argc, argv);

    // Global options
    QCommandLineParser parser;
    parser.setApplicationDescription("Git-like CLI example");
    parser.addHelpOption();
    parser.addVersionOption();

    // Add global options if any
    QCommandLineOption verboseOption(QStringList() << "v" << "verbose", "Verbose output");
    parser.addOption(verboseOption);

    // First parse only the global options
    parser.parse(QCoreApplication::arguments());

    // Find the subcommand (first non-option argument)
    QStringList args = parser.positionalArguments();
    if (args.isEmpty()) {
        qWarning() << "No subcommand provided.";
        parser.showHelp(1);
    }
    QString subcommand = args.first();

    // Remove the subcommand from the argument list to pass the rest
    QStringList subArgs = args;
    subArgs.removeFirst();

    // Now handle each subcommand with its own QCommandLineParser
    if (subcommand == "push") {
        QCommandLineParser pushParser;
        QCommandLineOption dryRunOption(QStringList() << "n" << "dry-run", "Do not actually push");
        pushParser.addOption(dryRunOption);

        // If you want to allow --help for the subcommand:
        pushParser.addHelpOption();

        // Rebuild argv for the subcommand parser (simulate "program push ...")
        QStringList pushArgv = QStringList() << "push" << subArgs;
        pushParser.parse(pushArgv);

        bool dryRun = pushParser.isSet(dryRunOption);
        QStringList positional = pushParser.positionalArguments();

        qDebug() << "Subcommand: push";
        qDebug() << "dry-run:" << dryRun;
        qDebug() << "Other args:" << positional;
        // ... handle the push logic
    }
    else if (subcommand == "commit") {
        // Similar logic for commit
        qDebug() << "Subcommand: commit";
        // ... handle commit subcommand
    }
    else {
        qWarning() << "Unknown subcommand:" << subcommand;
        parser.showHelp(1);
    }

    return 0;
}
```

### 3. Key Points

- **QCommandLineParser** does not natively support subcommands. You create this by parsing global options first, then branching based on the subcommand, and then parsing subcommand-specific options.
- You can use a separate `QCommandLineParser` for each subcommand if you want to provide specific options and help text.
- The global parser processes things like `-v` or `--version` that apply to the whole app.
- For subcommand-specific options (like `push -n`), you manually parse the remaining arguments using a new parser instance.

### 4. Limitations

- You must manually handle splitting the arguments and dispatching to the right subcommand.
- Some more advanced CLI features may be easier in other libraries, but for many Qt-based CLI utilities, this pattern is sufficient.
