﻿# `simple-add-dpc++` Sample

`simple-add-dpc++` provides the simplest example of DPC++ while providing an
example of using both buffers and Unified Shared Memory.

For comprehensive instructions see the [DPC++ Programming](https://software.intel.com/en-us/oneapi-programming-guide) and search based on relevant terms noted in the comments.


| Optimized for                     | Description
|:---                               |:---
| OS                                | Linux* Ubuntu* 18.04, Windows 10
| Hardware                          | Skylake with GEN9 or newer, Intel&reg; Programmable Acceleration Card with Intel&reg; Arria&reg; 10 GX FPGA
| Software                          | Intel&reg; oneAPI DPC++/C++ Compiler



## Purpose

The `simple-add-dpc++` is a simple program that adds two large vectors of
integers and verifies the results. This program is implemented using C++ and
Data Parallel C++ (DPC++) for Intel&reg; CPU and accelerators.

In this sample, you can learn how to use the most basic code in C++ language
that offloads computations to a GPU using the DPC++ language. This includes
using Unified Shared Memory (USM) and buffers. USM requires an explicit wait
for the asynchronous kernel's computation to complete. Buffers, at the time
they go out of scope, bring main memory in sync with device memory implicitly;
the explicit wait on the event is not required as a result. This sample
provides examples of both implementations for simple side by side review.

The code will attempt to execute on an available GPU and fallback to the
system's CPU if a compatible GPU is not detected. If successful, the name of
the offload device and a success message is displayed. Confirming your
development environment is set up correctly!

## Key Implementation Details

The basic DPC++ implementation explained in the code includes device selector,
USM, buffer, accessor, kernel, and command groups.

## License
Code samples are licensed under the MIT license. See
[License.txt](https://github.com/oneapi-src/oneAPI-samples/blob/master/License.txt) for details.

Third party program Licenses can be found here: [third-party-programs.txt](https://github.com/oneapi-src/oneAPI-samples/blob/master/third-party-programs.txt)

## Known Issues

With oneAPI 2021.4 the argument for accessors was changed from 'noinit' to
'no_init'. The change was derived from a change between the SYCL 2020
provisional spec and that of the 2020Rev3 spec

If running this sample and it fails, do one of the following
- Update the Intel® oneAPI Base Toolkit to 2021.4
- Change the 'no_init' argument  to 'noinit'



## Setting Environment Variables


For working at a Command-Line Interface (CLI), the tools in the oneAPI toolkits
are configured using environment variables. Set up your CLI environment by
sourcing the ``setvars`` script every time you open a new terminal window. This
will ensure that your compiler, libraries, and tools are ready for development.


### Linux
Source the script from the installation location, which is typically in one of
these folders:


For root or sudo installations:


  ``. /opt/intel/oneapi/setvars.sh``


For normal user installations:

  ``. ~/intel/oneapi/setvars.sh``

**Note:** If you are using a non-POSIX shell, such as csh, use the following command:

     ``$ bash -c 'source <install-dir>/setvars.sh ; exec csh'``

If environment variables are set correctly, you will see a confirmation
message.

If you receive an error message, troubleshoot the problem using the
Diagnostics Utility for Intel® oneAPI Toolkits, which provides system
checks to find missing dependencies and permissions errors.
[Learn more](https://www.intel.com/content/www/us/en/develop/documentation/diagnostic-utility-user-guide/top.html).


**Note:** [Modulefiles scripts](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-programming-guide/top/oneapi-development-environment-setup/use-modulefiles-with-linux.html)
    can also be used to set up your development environment.
    The modulefiles scripts work with all Linux shells.


**Note:** If you wish to fine
    tune the list of components and the version of those components, use
    a [setvars config file](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-programming-guide/top/oneapi-development-environment-setup/use-the-setvars-script-with-linux-or-macos/use-a-config-file-for-setvars-sh-on-linux-or-macos.html)
    to set up your development environment.

### Windows

Execute the  ``setvars.bat``  script from the root folder of your
oneAPI installation, which is typically:


  ``"C:\Program Files (x86)\Intel\oneAPI\setvars.bat"``


For Windows PowerShell* users, execute this command:

  ``cmd.exe "/K" '"C:\Program Files (x86)\Intel\oneAPI\setvars.bat" && powershell'``


If environment variables are set correctly, you will see a confirmation
message.

If you receive an error message, troubleshoot the problem using the
Diagnostics Utility for Intel® oneAPI Toolkits, which provides system
checks to find missing dependencies and permissions errors.
[Learn more](https://www.intel.com/content/www/us/en/develop/documentation/diagnostic-utility-user-guide/top.html).

## Building the `simple add DPC++` Program for CPU and GPU

> **Note**: If you have not already done so, set up your CLI
> environment by sourcing  the `setvars` script located in
> the root of your oneAPI installation.
>
> Linux Sudo: . /opt/intel/oneapi/setvars.sh
>
> Linux User: . ~/intel/oneapi/setvars.sh
>
> Windows: C:\Program Files(x86)\Intel\oneAPI\setvars.bat
>
>For more information on environment variables, see Use the setvars Script for [Linux or macOS](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-programming-guide/top/oneapi-development-environment-setup/use-the-setvars-script-with-linux-or-macos.html), or [Windows](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-programming-guide/top/oneapi-development-environment-setup/use-the-setvars-script-with-windows.html).

### Include Files
The include folder is located at "%ONEAPI_ROOT%\dev-utilities\latest\include" on your development system.

### Running Samples in DevCloud
If running a sample in the Intel DevCloud, remember that you must specify the compute node (cpu, gpu, fpga_compile, or fpga_runtime) and whether to run in batch or interactive mode. For more information, see the Intel® oneAPI Base Toolkit Get Started Guide ([https://devcloud.intel.com/oneapi/get-started/base-toolkit/](https://devcloud.intel.com/oneapi/get-started/base-toolkit/)).


### Using Visual Studio Code*  (Optional)

You can use Visual Studio Code (VS Code) extensions to set your environment, create launch configurations,
and browse and download samples.

The basic steps to build and run a sample using VS Code include:
 - Download a sample using the extension **Code Sample Browser for Intel oneAPI Toolkits**.
 - Configure the oneAPI environment with the extension **Environment Configurator for Intel oneAPI Toolkits**.
 - Open a Terminal in VS Code (**Terminal>New Terminal**).
 - Run the sample in the VS Code terminal using the instructions below.

To learn more about the extensions and how to configure the oneAPI environment, see
[Using Visual Studio Code with Intel® oneAPI Toolkits](https://www.intel.com/content/www/us/en/develop/documentation/using-vs-code-with-intel-oneapi/top.html).

After learning how to use the extensions for Intel oneAPI Toolkits, return to this readme for instructions on how to build and run a sample.

### On a Linux* System
Perform the following steps:
1. Build the `simple-add-dpc++` program using the following make commands (default uses USM):
    ```
    make all
    ```
> Note! To build with buffers use: `make build_buffers`

2. Run the program using:
    ```
    make run
    ```
> Note! To run with buffers use: `make run_buffers`

3. Clean the program using:
    ```
    make clean
    ```

### On a Windows* System Using a Command Line Interface
1. Select **Programs** > **Intel oneAPI 2021** > **Intel oneAPI Command Prompt** to launch a command window.
2. Build the program using the following `nmake` commands (Windows supports USM only):

    ```
    nmake -f Makefile.win
    ```

3. Run the program using:
    ```
    nmake -f Makefile.win run
    ```

4. Clean the program using:
    ```
    nmake -f Makefile.win clean
    ```


### On a Windows* System Using Visual Studio* Version 2017 or Newer
Perform the following steps:
1. Launch the Visual Studio* 2017.
2. Select the menu sequence **File** > **Open** > **Project/Solution**.
3. Locate the `simple-add` folder.
4. Select the `simple-add.sln` file.
5. Select the configuration 'Debug' or 'Release'
6. Select **Project** > **Build** menu option to build the selected configuration.
7. Select **Debug** > **Start Without Debugging** menu option to run the program.


## Building the `simple-add` Program for Intel&reg; FPGA

### On a Linux* System

Perform the following steps:

1. Clean the `simple-add` program using:
    ```
    make clean -f Makefile.fpga
    ```

2. Based on your requirements, you can perform the following:
   * Build and run for FPGA emulation using the following commands:
    ```
    make fpga_emu -f Makefile.fpga
    make run_emu -f Makefile.fpga
    ```
    * Build and run for FPGA hardware.
      **NOTE:** The hardware compilation takes a long time to complete.
    ```
    make hw -f Makefile.fpga
    make run_hw -f Makefile.fpga
    ```
    * Generate static optimization reports for design analysis. Path to the reports is `simple-add_report.prj/reports/report.html`
    ```
    make report -f Makefile.fpga
    ```

### On a Windows* System Using a Command Line Interface
Perform the following steps:

**NOTE:** On a Windows* system, you can only compile and run on the FPGA
emulator. Generating an HTML optimization report and compiling and running on
the FPGA hardware is not currently supported.

1. Select **Programs** > **Intel oneAPI 2021** > **Intel oneAPI Command Prompt** to launch a command window.
2. Build the program using the following `nmake` commands:
   ```
   nmake -f Makefile.win.fpga clean
   nmake -f Makefile.win.fpga
   nmake -f Makefile.win.fpga run
   ```

### On a Windows* System Using Visual Studio* Version 2017 or Newer
Perform the following steps:
1. Launch the Visual Studio* 2017.
2. Select the menu sequence **File** > **Open** > **Project/Solution**.
3. Locate the `simple-add` folder.
4. Select the `simple-add.sln` file.
5. Select the configuration 'Debug-fpga'
6. Select **Project** > **Build** menu option to build the selected configuration.
7. Select **Debug** > **Start Without Debugging** menu option to run the program.

## Running the Sample
### Application Parameters
There are no editable parameters for this sample.

### Example of Output
<pre>simple-add output snippet changed to:
Running on device:        Intel(R) Gen9 HD Graphics NEO
Array size: 10000
[0]: 0 + 100000 = 100000
[1]: 1 + 100000 = 100001
[2]: 2 + 100000 = 100002
...
[9999]: 9999 + 100000 = 109999
Successfully completed on device.</pre>

## Troubleshooting
If an error occurs, troubleshoot the problem using the Diagnostics Utility for Intel® oneAPI Toolkits.
[Learn more](https://software.intel.com/content/www/us/en/develop/documentation/diagnostic-utility-user-guide/top.html)