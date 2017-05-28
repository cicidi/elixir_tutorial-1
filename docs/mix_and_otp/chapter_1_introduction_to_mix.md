# 第一章 Mix简介

1.我们的第一个项目
2.项目编译
3.运行测试
4.环境
5.继续探索

在这个教程中，我们将学习如何构建一个完整的Elixir应用程序，它具有自己的监控树、配置和测试。

The application works as a distributed key-value store. We are going to organize key-value pairs into buckets and distribute those buckets across multiple nodes. We will also build a simple client that allows us to connect to any of those nodes and send requests such as:

CREATE shopping
OK

PUT shopping milk 1
OK

PUT shopping eggs 3
OK

GET shopping milk
1
OK

DELETE shopping eggs
OK
In order to build our key-value application, we are going to use three main tools:

OTP (Open Telecom Platform) is a set of libraries that ships with Erlang. Erlang developers use OTP to build robust, fault-tolerant applications. In this chapter we will explore how many aspects from OTP integrate with Elixir, including supervision trees, event managers and more;

Mix is a build tool that ships with Elixir that provides tasks for creating, compiling, testing your application, managing its dependencies and much more;

ExUnit is a test-unit based framework that ships with Elixir;

In this chapter, we will create our first project using Mix and explore different features in OTP, Mix and ExUnit as we go.

Note: this guide requires Elixir v1.2.0 or later. You can check your Elixir version with elixir --version and install a more recent version if required by following the steps described in the first chapter of the Getting Started guide.

If you have any questions or improvements to the guide, please reach discussion channels such as the Elixir Forum or the issues tracker. Your input is really important to help us guarantee the guides are accessible and up to date!

The final code for this guide is in this repository and can be used as a reference.
Our first project

When you install Elixir, besides getting the elixir, elixirc and iex executables, you also get an executable Elixir script named mix.

Let’s create our first project by invoking mix new from the command line. We’ll pass the project name as argument (kv, in this case), and tell Mix that our main module should be the all-uppercase KV, instead of the default, which would have been Kv:

$ mix new kv --module KV
Mix will create a directory named kv with a few files in it:

* creating README.md
* creating .gitignore
* creating mix.exs
* creating config
* creating config/config.exs
* creating lib
* creating lib/kv.ex
* creating test
* creating test/test_helper.exs
* creating test/kv_test.exs
Let’s take a brief look at those generated files.

Note: Mix is an Elixir executable. This means that in order to run mix, you need to have Elixir’s executable in your PATH. If not, you can run it by passing the script as argument to elixir:

$ bin/elixir bin/mix new kv --module KV
Note that you can also execute any script in your PATH from Elixir via the -S option:

$ bin/elixir -S mix new kv --module KV
When using -S, elixir finds the script wherever it is in your PATH and executes it.
Project compilation

A file named mix.exs was generated inside our new project folder (kv) and its main responsibility is to configure our project. Let’s take a look at it (comments removed):

defmodule KV.Mixfile do
  use Mix.Project

  def project do
    [app: :kv,
     version: "0.1.0",
     elixir: "~> 1.3",
     start_permanent: Mix.env == :prod,
     deps: deps()]
  end

  def application do
    [extra_applications: [:logger]]
  end

  defp deps do
    []
  end
end
Our mix.exs defines two public functions: project, which returns project configuration like the project name and version, and application, which is used to generate an application file.

There is also a private function named deps, which is invoked from the project function, that defines our project dependencies. Defining deps as a separate function is not required, but it helps keep the project configuration tidy.

Mix also generates a file at lib/kv.ex with a simple module definition:

defmodule KV do
end
This structure is enough to compile our project:

$ cd kv
$ mix compile
Will output:

Compiling 1 file (.ex)
Generated kv app
The lib/kv.ex file was compiled, an application manifest named kv.app was generated and all protocols were consolidated as described in the Getting Started guide. All compilation artifacts are placed inside the _build directory using the options defined in the mix.exs file.

Once the project is compiled, you can start an iex session inside the project by running:

$ iex -S mix
Running tests

Mix also generated the appropriate structure for running our project tests. Mix projects usually follow the convention of having a <filename>_test.exs file in the test directory for each file in the lib directory. For this reason, we can already find a test/kv_test.exs corresponding to our lib/kv.ex file. It doesn’t do much at this point:

defmodule KVTest do
  use ExUnit.Case
  doctest KV

  test "the truth" do
    assert 1 + 1 == 2
  end
end
It is important to note a couple things:

the test file is an Elixir script file (.exs). This is convenient because we don’t need to compile test files before running them;

we define a test module named KVTest, use ExUnit.Case to inject the testing API and define a simple test using the test/2 macro;

Mix also generated a file named test/test_helper.exs which is responsible for setting up the test framework:

ExUnit.start()
This file will be automatically required by Mix every time before we run our tests. We can run tests with mix test:

Compiled lib/kv.ex
Generated kv app
[...]
.

Finished in 0.04 seconds (0.04s on load, 0.00s on tests)
1 test, 0 failures

Randomized with seed 540224
Notice that by running mix test, Mix has compiled the source files and generated the application file once again. This happens because Mix supports multiple environments, which we will explore in the next section.

Furthermore, you can see that ExUnit prints a dot for each successful test and automatically randomizes tests too. Let’s make the test fail on purpose and see what happens.

Change the assertion in test/kv_test.exs to the following:

assert 1 + 1 == 3
Now run mix test again (notice this time there will be no compilation):

  1) test the truth (KVTest)
     test/kv_test.exs:5
     Assertion with == failed
     code: 1 + 1 == 3
     lhs:  2
     rhs:  3
     stacktrace:
       test/kv_test.exs:6

Finished in 0.05 seconds (0.05s on load, 0.00s on tests)
1 test, 1 failure
For each failure, ExUnit prints a detailed report, containing the test name with the test case, the code that failed and the values for the left-hand side (lhs) and right-hand side (rhs) of the == operator.

In the second line of the failure, right below the test name, there is the location where the test was defined. If you copy the test location in this full second line (including the file and line number) and append it to mix test, Mix will load and run just that particular test:

$ mix test test/kv_test.exs:5
This shortcut will be extremely useful as we build our project, allowing us to quickly iterate by running just a specific test.

Finally, the stacktrace relates to the failure itself, giving information about the test and often the place the failure was generated from within the source files.

Environments

Mix supports the concept of “environments”. They allow a developer to customize compilation and other options for specific scenarios. By default, Mix understands three environments:

:dev - the one in which Mix tasks (like compile) run by default
:test - used by mix test
:prod - the one you will use to run your project in production
The environment applies only to the current project. As we will see later on, any dependency you add to your project will by default run in the :prod environment.

Customization per environment can be done by accessing the Mix.env function in your mix.exs file, which returns the current environment as an atom. That’s what we have used in the :start_permanent options:

def project do
  [...,
   start_permanent: Mix.env == :prod,
   ...]
end
When you compile your source code, Elixir compiles artifacts to the _build directory. However, in many occasions to avoid unnecessary copying, Elixir will create filesystem links from _build to actual source files. When true, :build_embedded disables this behaviour as it aims to provide everything you need to run your application inside _build.

Similarly, when true, the :start_permanent option starts your application in permanent mode, which means the Erlang VM will crash if your application’s supervision tree shuts down. Notice we don’t want this behaviour in dev and test because it is useful to keep the VM instance running in those environments for troubleshooting purposes.

Mix will default to the :dev environment, except for the test task that will default to the :test environment. The environment can be changed via the MIX_ENV environment variable:

$ MIX_ENV=prod mix compile
Or on Windows:

> set "MIX_ENV=prod" && mix compile
Mix is a build tool and, as such, it is not always expected to be available in production, especially if your team uses explicit build steps. Therefore, it is recommended to access Mix.env only in configuration files and inside mix.exs, never in your application code (lib).
Exploring

There is much more to Mix, and we will continue to explore it as we build our project. A general overview is available on the Mix documentation.

Keep in mind that you can always invoke the help task to list all available tasks:

$ mix help
You can get further information about a particular task by invoking mix help TASK.

Let’s write some code!
