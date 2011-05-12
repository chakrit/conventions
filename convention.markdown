# Coding Convention

This file defines coding standards and convention to be used for C# and other related languages (e.g. MSBuild files) for projects at 2nitedesign! Co., Ltd. This file is public domain, you may freely copy, reuse, redistribute or modify it provided that all identifiable names be removed.

For clarity, we'll use snippets from the [Sider](http://github.com/chakrit/sider) project with modification as a reference for demonstration purposes as it has been built according to this guideline.

If a situtation arises such that there is no defined convention for the writing code, you should fallback to Microsoft's official [C# style guide] and/or your own judgement as the default. 

**Familiarity with C# is assumed.**

## Additional Software

With each software coming with its own convention, it is useful to use the same software where possible. However, using the right tool for the right job should be prioritized over this list.

* **[NuGet](http://nuget.org)** - For managing package references. NuGet will install package to the `/packages/` folder by default.
* **[NUnit](http://nunit.org)** - As the testing framework. This is preferred over the built-in MSTest suite due to portability and compatibility with other 3rd-party software.
* **[MSBuild](http://msdn.microsoft.com/en-us/library/0k6kkbsd.aspx)** - As the building tool. This is to ease development with new developer/new project. Also external scripts for each specific development tasks are preferred over one complex build scripts. [NAnt](http://nant.sourceforge.net/) may be used as an alternative but unless it adds a lot of value, it should be avoided.

## File Organization

We make extensive uses of the [NuGet package manager](http://nuget.org) so only a single solution file should suffice in most cases.

Files inside a solution should be structured, with root folder being the folder checked out from source control software, like so:

    \Sider.sln
    \.gitignore
    \package.bat
    \test.bat
    \build\
    \lib\
    \packages\Moq.4.0.10827
    \packages\NUnit.2.5.9.10348
    \packages\Rx-Core.1.0.2856.0
    \packages\Rx-Main.1.0.2856.0
    \samples\Sider.Samples\
    \samples\Sider.Samples.Chat\
    \src\SiderAssemblyInfo.cs
    \src\Sider\
    \src\Sider.Benchmark\
    \tests\Sider.Tests
    \tools\NuGet.exe
    \tools\7za.exe

Where...

* **`\`** - The root folder should contains the main solution file that has references to all the projects inside the solution. Build scripts and other top-level development files such as test metadata or `.gitignore` source control metadata file should also be kept at the root.
* **`\build`** - Contains the binaries and other artifacts from the build process. **This folder should never be checked into source contorl.** It should always be safe to completely wipe out this folder and start over.
* **`\lib`** - Contains all the manually-reference binaries and/or sources. When possible, source reference should be done using the source control's proper submodule support such as `git submodule` or `svn submodule` so any changes can be reflected back up-stream.
* **`\packages`** - Contains all the packages installed via the package manager. Unless necessary, this folder should not be touched by the developer, it should be left as the package manager makes it to be.
* **`\samples`** - (optional) For library projects and/or projects which are to be used inside other projects. This folder should contains sample usage of such library projects. In the example, 2 sample projects are provided namely `Sider.Samples` and `Sider.Samples.Chat`.
* **`\src`** - Contains all the compilable projects inside the solution. Each project should have its own folder. There are 3 projects in the above example, namely `Sider`, `Sider.Benchmark` and `Sider.Tests`. Files shared by all projects should also be inside this folder where possible.
* **`\tests`** - (optional) For projects which require complex testing, all testing code should be put inside this folder.
* **`\tools`** - Contains all the necessary tools that is required for development such as test runners, package managers, file compressors, compilers, build runners etc. Subfolders should be used if a tool requires more than a few files to function.

### Projects

Projects should be named the same as the root namespace of the code being built. For `Sider`, the root namespace is `Sider` thus `Sider` should be used as the main project name. Except if the root namespace are split into multiple projects, a dot notation maybe used to clarify the use of each. i.e. a project maybe named `Sider.Common` if the root namespace is `Sider` but it contains common stuff shard by other projects. This should be used sparingly and the first rule should be prioritized.

### Classes and interfaces

Classes and interfaces should be put inside their own files named the same as the class or interface except where **multiple related classes and/or related interfaces or one-off classes** have similar uses or would produce a lot of files if not grouped, for example:

    src\Sider\Exceptions.cs
    src\Sider\IRedisClient.cs
    src\Sider\RedisClient.cs

Where **`Exceptions.cs`** - Contains the following code:

    using System;

    namespace Sider
    {
      public sealed class IdleTimeoutException : TimeoutException
      {
        public IdleTimeoutException(Exception inner) :
          base("Disconnection detected, possibly due to idle timeout.", inner) { }
      }

      public sealed class ResponseException : Exception
      {
        public ResponseException(string msg) :
          base(msg) { }

        public ResponseException(string msg, Exception ex) :
          base(msg, ex) { }
      }
    }

With both `IdleTimeoutException` and `ResponseException` class containing less than 10 lines of code, it make sense to group them together according to their use -- being `Exceptions`.

Whereares `IRedisClient.cs` contains > 100 lines of Redis interface method so it make sense to put it inside its own file.

Another common case is with generic classes such as `RedisClient.cs` which contains:

    public class RedisClient : RedisClient<string>
    {
      public RedisClient(
        string host = RedisSettings.DefaultHost,
        int port = RedisSettings.DefaultPort) :
        base(host, port) { }

      internal RedisClient(Stream incoming, Stream outgoing) :
        base(incoming, outgoing) { }

      public RedisClient(RedisSettings settings) : base(settings) { }
    }

    public partial class RedisClient<T> : RedisClientBase, IRedisClient<T>
    {
      public RedisClient(string host = RedisSettings.DefaultHost,
        int port = RedisSettings.DefaultPort) :
        this(RedisSettings.New().Host(host).Port(port)) { }

      public RedisClient(Func<RedisSettings.Builder, RedisSettings> settingsFunc) :
        this(settingsFunc(RedisSettings.New())) { }

      public RedisClient(RedisSettings settings) :
        base(settings)
      {

    // snipped

With the non-generic version being there simply for convenience and it is only ~ 10 lines long, thus it is placed at the top of the `RedisClient.cs` file instead of having its own file.

### Namespaces

Each namespace inside a project should have its own folder.

Where it is appropriate to put a lot of classes inside the same namespace, a special folder `_` (single underscore) maybe used to store common base classes or interfaces such as this viewmodel example:

    Web.ViewModels\_\ViewModelBase.cs
    Web.ViewModels\_\ListModel.cs
    Web.ViewModels\HomeViewModel.cs
    Web.ViewModels\StoreViewModel.cs
    Web.ViewModels\ShopViewModel.cs

    // 20 more classes ...

In this case, the `ViewModelBase` and `ListModel` class deserve special attention since they are the base class for all viewmodels but since they are still in the same `Web.ViewModels` namespace the special underscore folder is used to denote their specialty.

## Indentation

Use two whitespace character for indentation. **This is non-negotiable.** This allows for longer lines to be written where appropriate and shorter code to be more concise and easier for eye-scanning.

An extra indentation level should be added whenever there is a new scope such as with control blocks or class or a object creation, for example:

    using System.Collections.Generic;

    class Foo
    {
      public bool Method()
      {
        for (var i = 0; i < int.MaxValue; i++)
          if ((1 + 1) == 2)
            return true;

        throw new Exception("Reality distortion detected.");

        var stuff =
          from n in Enumerable.Range(0, 10)
          let p2 = n * n
          select new {
            Number = n,
            PowerOfTwo = p2
          };

        var dict = new Dictionary<string, string> {
          { "hello", "world" },
          { "abc", "def" }
        };

        Func<int, string> printer = n =>
        {
          return n.ToString();
        };
      }
    }

One special case worth mentioning is the `switch` statement. Only a single indentation level should be used inside `case` blocks and each block should also have a an extra block definition where possible:

    switch (invocation.Command) {
    case "SUBSCRIBE":
    case "PSUBSCRIBE":
    case "UNSUBSCRIBE":
    case "PUNSUBSCRIBE": {
      var result = ExecuteImmediate(invocation);
      if (!StillActive && _onDone != null) {
        _done = true;
        _onDone();
      }

      return result;
    }

    case "QUIT": {
      var result = ExecuteImmediate(invocation);

      _observable.Dispose();
      return result;
    }

    default: {
      _done = false;
    } break;
    }

Note that `break`s are put *outside* the block. And that there are 2 `result` variables inside the switch block which are possible due to the use of extra scopes for each `case`s. For one-liners `switch`, no indentation or block definition is required.

    switch (value) {
    case "message": return MessageType.Message;
    case "pmessage": return MessageType.PMessage;
    case "subscribe": return MessageType.Subscribe;
    case "psubscribe": return MessageType.PSubscribe;
    case "unsubscribe": return MessageType.Unsubscribe;
    case "punsubscribe": return MessageType.PUnsubscribe;

    default: return MessageType.Unknown;
    }

## New lines

Lines should generally be no longer than 80 characters and *MUST NOT* exceeds 90 characters. Think of 80 chars as a "soft limit" and 90 as the "hard limit".

I recommend that you use the "Editor Guidelines" plugin for Visual Studio which greatly helps keeps lines short.

### Blank lines

Blank lines are *REQUIRED* at the start of each code file and before namespace declarations.

Other than this, you may use

* single blank line - to separate blocks of code or methods and...
* double blank lines - to separate large groups of code or groups of methods (which each method separated by single line)

More than 2 blank lines should never be used. Usually if you need that level of grouping, the current entity is supposedly too large.

### Braces

Open braces should be put on a new line only when there is a new scope that executes separately from each other such as methods, classes, namespaces or lambdas.

For control blocks, open braces should be kept on the same line as the starting statement. Example:

    public class Foo
    {
      public void Bar()
      {
        execute(() => "hello world!");
        execute(() =>
        {
          var a = "hello";
          var b = "world";

          return a + b;
        });

        // same line for control blocks
        if (true) {
          do {
            while (true) {
              for (var i = 0; i < 10; i++) {
                execute(() => i.ToString());
              }
            }
          } while (true);
        }
      }


      private void execute(Func<string> func)
      {
        Console.WriteLine(func());
      }
    }

Inside a single statement, if there is a need to put code onto a new line, the new line character should be inserted *after* a parenthesis, comma or an operator to indicates that the current line is not finished like so:

    var world = num + num == 2 * num ? "Real" :
      num == Math.Sqrt(-1) ? "Imaginary" :
      "Unknown";

    int x = 2, y = 3;
    num = x * x + 3 * x + 10 +
      4 * y * y + 2 * y + 7;

    LongMethod("short arg", "short arg", "short arg",
      "more short arg", "more short arg");

    LongMethod(
      "the first argument is really really really really really really really long.");

    LongMethod(
      "the first arguemnt is suuuuuuuuuuuuuuuuuuuuuuuuuuppppppeeeeeerrrrrrr " + 
      "loooooooooonnnnnnnnnngggggggggggggggggggggg this time.");

### White Space & Parenthesis

Use of white space and parenthesis should follows standard Microsoft C# style guide.

Relying on operator precendence is totally acceptable as long as it is obvious -- such as that multiplication is always done before addition. For a more complex operations and/or long expressions, explicit parenthesises is still preferred.

    var num  = 3 * 10 + 10;
    var expr = 1 * 1 + 2 * 2 + 3 * 3;
    var longExpr =
      (2 * x * x + x) +
      (3 * x * y + y) +
      (4 * y * y + x);

## Feature usages

* `var` should be used wherever possible. The type of the variable is almost always easily deducable from the initializing expression.
* Automatic properties should be used where there is minimal references/usage inside the classes. Otherwise a proper backing field should be used.

## Codefile Organization

In a single file, code elements should be ordered according to the following:

1. `using`s statements.
2. `namespace` declaration.
3. `class` or `interface` declaration.
4. `const` or `public static` declaration.
5. `private` declaration.
6. property declaraction.
7. constructor declaration.
8. additional initializers.
9. additional way to construct objects. (i.e. factory functions or `implicit operator`s)
10. main group of methods or interface implementations usually `public`s.
11. helper group of methods usually `protected` or `private`.
12. `Dispose()` or other de-initors/destructors.

Special helper functions which are only called from one method maybe placed immediately following that method inspite of this ordering.

In other cases, locality of code is prioritized.

## Naming Convention

Otherwise stated here, naming conventions should follow the standard Microsoft's C# style guide.

Special note:

* `private` - members should starts with `_` and use camel case.
* `const` and `static readonly` - members should use `PascalCase`s.
* `interface` - should starts with `I` .. e.g. `IInterface`

## Example
