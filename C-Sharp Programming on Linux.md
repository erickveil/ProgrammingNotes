---
title: C# Programming on Linux
layout: note
tags:
  - CSharp
  - Linux
---

> [!tldr]
> The insane hoops and telemetry make distribution of C# on a Linux machine unfeasable unless you are making a tool for yourself, or want to use Docker. No simple support able for any GUI on Linux development, either out of the box or in an internet search, so we're stuck with command line only. So maybe make your back ends in C# and tape it to a QT front end on linux.
> 
> It's just not there yet.

> [!todo]
> Since writing this, [Avalonia](https://github.com/AvaloniaUI/Avalonia) looks like it has come out as a nice GUI option for multi-platform work with C#. It is possible that this is worth looking into again.

# Installing

(make sure you are on the right version!)

Make sure Prerequisites are intalled:
https://docs.microsoft.com/en-us/dotnet/core/linux-prerequisites?tabs=netcore2x

Install .NET Core
https://dotnet.microsoft.com/download/linux-package-manager/ubuntu16-04/sdk-current

You may have problems when installing:

```Bash
sudo apt-get install dotnet-sdk-2.2
```
Output:
```
...The following packages have unmet dependencies...
```


Then trying to get the dependencies, you get to:

```Bash
sudo apt-get install libicu55
```
Output:
```
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Package libicu55 is not available, but is referred to by another package.
This may mean that the package is missing, has been obsoleted, or
is only available from another source

E: Package 'libicu55' has no installation candidate
```


The Fix:

```Bash
wget http://security.ubuntu.com/ubuntu/pool/main/i/icu/libicu55_55.1-7ubuntu0.4_amd64.deb -P /tmp/

sudo dpkg -i /tmp/libicu55_55.1-7ubuntu0.4_amd64.deb
```

Then try to install agian.

# Disable telemetry

https://unix.stackexchange.com/questions/117467/how-to-permanently-set-environmental-variables

vim ~/.profile

Add these lines:

```C#
# Disable Telemetry
export DOTNET_CLI_TELEMETRY_OPTOUT=1
```

The command line command is `dotnet`

Create a boilerplate hello world project (or just use IDE such as [Ryder](https://www.jetbrains.com/rider/) to create):

`dotnet new console -o helloWorld`

To run the project, cd into the directory and run

`dotnet run`

or provide the project name:

`dotnet run --projcet /path/to/project.csproj`

# Building:

To create a standalone version that doesn't need the `dotnet run` command, build from Rider (or `man dotnet` to see the build commands).
The executable will be at the bottom of the bin directory (follow it all the way down).
It will inexplicably be created with the `.dll` suffix, even though it is an executable and not a windows library. Maybe it can be used as a windows library, and dotnet doesn't make the distinction. Who knows with Microsoft.

Once built in this way, you can use proper argument options with the program.
If you try to run, say `dotnet run myProject.cproj -v` then the `-v` option will be incorrectly associated with the dotnet program, not your project (you will get the help file because dotnet doesn't have such an option).

But if you create a build, you can run `./myProject.dll -v` and the option will correctly be associated with your program.

Note that you may have to pass your compiled dll file to `chmod` before you can execute it. 


