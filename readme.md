# C++ project templates to construct projects for test automation

This guide describes how to quickly construct C++ projects for test automation on Windows or Linux.

## Contents

- [C++ project templates to construct projects for test automation](#c-project-templates-to-construct-projects-for-test-automation)
  - [Contents](#contents)
  - [Folder Structure](#folder-structure)
  - [Getting Started on Windows](#getting-started-on-windows)
    - [Prerequisite](#prerequisite)
    - [Quick-start Steps](#quick-start-steps)
    - [Code coverage analysis by Visual Studio Enterprise](#code-coverage-analysis-by-visual-studio-enterprise)
  - [Getting Started on Linux](#getting-started-on-linux)
    - [Installations](#installations)
    - [Tips of constructing unit test projects](#tips-of-constructing-unit-test-projects)
    - [Code coverage analysis by gcov and gcovr](#code-coverage-analysis-by-gcov-and-gcovr)
  - [Unit test on Jenkins](#unit-test-on-jenkins)
  - [Future of Work](#future-of-work)
  - [Reference](#reference)

## Folder Structure

Giving a quick review of the folder structure used in this repository here, and hopefully it could be helpful for managing cross platform C++ projects.

```
$(RepoDir)          // Local path of this repository
|-- bin             // Put helper tools here
|-- cc              // Put .cc or .h files here
|   |-- src         // Put source files here
|   `-- test        // Put test files here
|-- include         // Put module headers here
|-- prj             // Put project files here
|   |-- pycharm     // PyCharm projects
|   |-- qmake       // qmake projects
|   `-- v141        // Visual Studio 2017 projects
|-- profile         // Scripts to instrument and profiling binaries are put here.
|-- submod          // Put submodule here
|-- build.py        // A python script to build project and run test cases.
```

## Getting Started on Windows

### Prerequisite

- [Python](https://www.python.org/): 3.6
- [Visual Studio Enterprise](https://visualstudio.microsoft.com/): 2013 or later is required, and we are using 2017 here.
- Test Adapter
  - [Test Adapter for Google Test](https://marketplace.visualstudio.com/items?itemName=VisualCPPTeam.TestAdapterforGoogleTest): This is an extension bundled with Visual Studio since 2015, and check it while you are installing Visual Studio.
  - [Google Test Adapter](https://marketplace.visualstudio.com/items?itemName=ChristianSoltenborn.GoogleTestAdapter): Try this if you are using Visual Studio 2013 or earlier.

### Quick-start Steps

The first 4 commits in the Git log messages of this repository are the steps to construct C++ projects for test automation.

1. [756bdfd7404](https://git.cpgswtools.com/users/veawor_liu/repos/gtest_template/commits/756bdfd74041c1d5561b73246878cc68bc8199b5) is to give a simple DLL project which include one simple API as an target binary for testing and profiling.
2. [7e724d75ae2](https://git.cpgswtools.com/users/veawor_liu/repos/gtest_template/commits/7e724d75ae24915eb67acb26f2cfa917adba9671) is to add unit tests. We use [Google Test](https://github.com/google/googletest.git) as C++ test framework. It only needs few steps to setup a unit test project.
   1. Create a Visual C++ project of an empty console application named, for example, `$(TestProject)`.
   2. Git clone [Google Test](https://github.com/google/googletest.git) as a submodule at `$(RepoDir)submod/googletest`.
   3. (Optional) Git switch to an official release commit, for example, `release-1.8.1`.
   4. Add the property sheet `googletest.props` to `$(TestProject)`.
   5. Use `main.cc` and `test_division.cc` as template for main function and test cases, then add them to `$(TestProject)`.
3. (Optional) [9ab1e63cce8](https://git.cpgswtools.com/users/veawor_liu/repos/gtest_template/commits/9ab1e63cce8ad15679b2cfffb7d82b0f910214a9) is to configure projects being profiled. Open up project property and:
   1. Property -> Linker -> General -> Enable Incremental Linking -> No
   2. Property -> Linker -> Advanced -> Profile -> Yes
   3. Now we could click Test -> Analyze Code Coverage -> All Tests for running unit test
   4. Results would be shown on Code Coverage Results window.
4. [070fb8cdc78](https://git.cpgswtools.com/users/veawor_liu/repos/gtest_template/commits/070fb8cdc78da2fc6614fdbff2e7579667317796) is to add a `build.py` to build modules and validate modules.

### Code coverage analysis by Visual Studio Enterprise

- Analyze code coverage for Windows service.
  1. Build binaries (DLL or EXE) with following linker options.
     - Property -> Linker -> General -> Enable Incremental Linking -> No
     - Property -> Linker -> Advanced -> Profile -> Yes
  2. Instrument binaries. `$(RepoDir)/profile/instrument.cmd` is an example of instrumentation.
  3. Start Profile Monitor. `$(RepoDir)/profile/monitor_start.cmd` is an example to start `VSPerf`.
     - 2 options are needed to monitor Windows service.
       - `/CrossSession` Allows client access to the monitor from another session.
       - `/USER` Allows client access to the monitor from the specified account
  4. Launch and test services with instrumented binaries.
  5. Stop services.
  6. Stop Profile Monitor. `$(RepoDir)/profile/monitor_stop.cmd` is an example to stop `VSPerf`
  7. Open .coverage file with Visual Studio Enterprise.
- Multiple results could be imported and merged. This is useful when some cases could be only tested on specific platform.
- Customize code coverage analysis.
  - Compose run setting file to customize code coverage analysis. An example is `$(RepoDir)prj/v141/test_division/test_division.runsettings`.
  - Customize by `VSInstr /INCLUDE:funcspec[;funcspec]*`.

## Getting Started on Linux

### Installations

This template project is developed on [Ubuntu](https://www.ubuntu.com/download/desktop) 18.04. However, any other Linux distribution should be fine. The sample commands to install the applications as listed as below.

- `> sudo apt install g++`
  - The version of [g++](https://gcc.gnu.org/) used in this project is 7.3.0, and 5.2 or later should be good enough to compile this project.
- `> sudo apt install gcovr`
  - The version of [gcovr](http://www.gcovr.com/en/stable/index.html) used in this project is 3.4.
- (Optional) `> sudo apt install python3`
  - The version of [Python](https://www.python.org/) used in this project is 3.6. This is required for running the Python scripts in this project. This is not required if you are not interested in running those scripts.
- Qt Creator - It's recommended to use official installer to install Qt Creator. Download install from [here](https://www.qt.io/download) and make sure one of Qt libraries is selected. This is for compiler kit for save us time to configure development environment. The version of Qt library used in this project is 5.12.2, and the version of Qt Creator

### Tips of constructing unit test projects

Before going forward, we are assuming audiences are already familiar with [Quick-start Steps](#quick-start-steps) for Windows. Especially how we configure Google Test.

1. Open `division.pro` and `test_division.pro` in Qt Creator.
2. `division.pro`: The qmake project to build our demonstrating project which include a application binary interface.
3. `test_division.pro`: The qmake project to build a test project.
   - Ensure `INCLUDEPATH` is well set, meaning that headers, source, and the dependencies, such as Google Test are well included.
   - Ensure `-lpthread` is included in `LIBS`. `pthread` is a dependency of Google Test.
   - `-ldl` is for `dlopen/dlsym/dlcose` used in test code.
4. [3fa4588798b](https://git.cpgswtools.com/users/veawor_liu/repos/gtest_template/commits/3fa4588798bd6a669b51d5856a28f0d73fbf54ab) is to revised `division.h` and `test_division.cc` for making sources could be built cross Visual C++ and G++.
5. Build all projects. Tests are now listed in the Test Explorer of Qt Creator.
   - In case you don't see Test Explorer, click `Window -> Show Right Sidebar -> Tests` to open it.

### Code coverage analysis by gcov and gcovr

[gcov](https://gcc.gnu.org/onlinedocs/gcc/Gcov.html) is a test coverage program bundle with gcc compiler. If compiler and linker options are well configured, the code coverage data could be collected during the execution of program. It's recommended to use [gcovr](https://github.com/gcovr/gcovr) for analyzing code coverage results since it support to generate XML file for [Cobertura Plugin](https://wiki.jenkins.io/display/JENKINS/Cobertura+Plugin). It's more useful if we'd like to integrate code coverage analysis into Jenkins continuous integration system.

1. Ensure below options are well configured for the projects you want to collect code coverage data.
   - `QMAKE_CXXFLAGS += --coverage`
   - `QMAKE_LFLAGS += --coverage`
2. Runt test case or manually test programs.
3. Analyze the data with tools like [lcov](https://github.com/linux-test-project/lcov) or [gcovr](https://github.com/gcovr/gcovr).
4. `$(RepoDir)/profile/gcovr.sh` is a shell script to use [gcovr](https://github.com/gcovr/gcovr) to analyze code coverage results.
5. `$(RepoDir)/profile/lcov.sh` is a shell script to use [lcov](https://github.com/linux-test-project/lcov) to analyze code coverage results.

## Unit test on Jenkins

In this section, few quick steps would be given to make Jenkins to take results of Google Test to generate test reports. We don't describe how to use Jenkins here, or we'll be distracted.

- Jenkins ver. 2.164.1 is used in this guide.
- Install [xUnit plugin](https://jenkins.io/doc/pipeline/steps/xunit/).
- Project -> Configure -> Post-build Actions -> Publish xUnit test result report.
- Refer to [4dcaf1f2a28](https://git.cpgswtools.com/users/veawor_liu/repos/gtest_template/commits/4dcaf1f2a28d215f9039001702541e014f26e4bf) for how we generate report here.
- Set relative path to the XML file generated by Google Test.
- Click "Build Now" and see if the report is generated.

## Future of Work

- Google Test on Bamboo or Jenkins.
  - [xUnit plugin](https://jenkins.io/doc/pipeline/steps/xunit/) could be used to work with Google Test to have test results on Jenkins.
- Code coverage analysis on Bamboo or Jenkins.
  - [Cobertura Plugin](https://wiki.jenkins.io/display/JENKINS/Cobertura+Plugin) could be used as receiving reports generated from [gcovr](http://www.gcovr.com/en/stable/).
- Google Mock - A framework for writing and using C++ mock classes.

## Reference

- [Measure app performance in Visual Studio](https://docs.microsoft.com/en-us/visualstudio/profiling/?view=vs-2017)
  - [VSInstr](https://docs.microsoft.com/en-us/visualstudio/profiling/vsinstr?view=vs-2017)
  - [VSPerfCmd](https://docs.microsoft.com/en-us/visualstudio/profiling/vsperfcmd?view=vs-2017)
  - [VSPerfMon](https://docs.microsoft.com/en-us/visualstudio/profiling/vsperfmon?view=vs-2017)
- [Testing tools in Visual Studio](https://docs.microsoft.com/en-us/visualstudio/test/improve-code-quality?view=vs-2017)
  - [How to use Google Test for C++ in Visual Studio](https://docs.microsoft.com/en-us/visualstudio/test/how-to-use-google-test-for-cpp?view=vs-2017)
  - [Customize code coverage analysis](https://docs.microsoft.com/en-us/visualstudio/test/customizing-code-coverage-analysis?view=vs-2017)
- [gcov](https://gcc.gnu.org/onlinedocs/gcc/Gcov.html) - A coverage testing tool.
- [gcovr](https://github.com/gcovr/gcovr) - Generate code coverage reports with gcc/gcov
- [lcov](https://github.com/linux-test-project/lcov) - An extension of GCOV to generate code coverage reports.
- [Qt](https://www.qt.io/) and [qmake Manual](https://doc.qt.io/qt-5/qmake-manual.html)
