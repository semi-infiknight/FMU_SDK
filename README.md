Installing the FMU SDK

Download the FMU SDK from here, and unzip the file in a directory where you have write access(Preferably in the C:/ directory). Name that directory FMUSDK_HOME. The FMU SDK contains only the C sources of the FMUs and the simulators, not the executables.

Download the Microsoft Visual Studio 2013 Express Version(VS12) as the 2015 ver (VS14) web Providers have been deprecated.(The SDK Installation only works in VS14 or lower versions.)
We need to download the build tools from here (2017 version as the msbuild is not being able to configure in 2013).The VS12 should be fully configured before the installation(You can install additional build files for VS12 is the install.bat does not work in the next step)
Download and install CMake 3.2 or higher from here.

To build Windows 32 bit versions of all FMUs and simulators of the FMU SDK, double click on `FMUSDK_HOME/install.bat`. This should create fmus in `FMUSDK_HOME/fmu10/fmu and FMUSDK_HOME/fmu20/fmu`, as well as four simulators in `FMUSDK_HOME/fmu10/bin/win32` and `FMUSDK_HOME/fmu20/bin/win32`.
 
![image](https://github.com/semi-infiknight/FMU_SDK/assets/97100765/99d33347-29e0-4ea5-8868-28858eb49691)


To build Windows 64 bit versions of all FMUs and simulators, open a command shell in the FMUSDK_HOME folder and run `install -win64`. This creates additional fmus in the `win64` subdirectories in `FMUSDK_HOME/fmu10/fmu` and `FMUSDK_HOME/fmu20/fmu`, as well as additional simulators in `FMUSDK_HOME/fmu10/bin/win64` and `FMUSDK_HOME/fmu20/bin/win64`. Building these 64 bit versions works also on 32 bit Windows platforms. Execution of the 64 bit simulators and fmus requires however a 64 bit version of Windows.
 

![image](https://github.com/semi-infiknight/FMU_SDK/assets/97100765/600cc19f-1f4b-4884-aec5-78b38345f4ce)



Open Terminal and navigate to FMUSDK_HOME

Then Enter the following commands:-
```
mkdir build
cd build
cmake -G "Visual Studio 12 2013 Win64" ..

```
This will generate the build files in the build directory.

Now navigate to C:/Program Files/MSbuild/12.0/bin or wherever the msbuild executable is on your system.

```
msbuild.exe C:\C:\FMUSDK_HOME\build\FMUSDK.sln

```
This will hard-run the msbuild command and build all FMUs into the `dist` folder.

 

 ![image](https://github.com/semi-infiknight/FMU_SDK/assets/97100765/1a639f16-739e-44a5-bec0-2baeae3da8da)
![image](https://github.com/semi-infiknight/FMU_SDK/assets/97100765/4e07de0b-139d-403a-b311-363785253df5)





{ To get a list of the available generators enter

```
cmake --help
```
 

}




![image](https://github.com/semi-infiknight/FMU_SDK/assets/97100765/f5833c26-9b8c-4ce3-8a56-ed33328afbe2)













 Simulating FMU

to run a given FMU with one of the FMU simulators, open a command shell in directory FMUSDK_HOME and run the command fmusim

```
fmusim simulator model.fmu [tEnd [h [loggingOn [csvSeparator]]]] [-win64]

```
Here:-
  simulator ..... cs10 or cs20 for co-simulation, me10 or me20 for model exchange, required
  model.fmu ..... path to FMU, relative to current dir or absolute, required
  tEnd .......... end  time of simulation, optional, defaults to 1.0 sec
  h ............. step size of simulation, optional, defaults to 0.1 sec
  loggingOn ..... 1 to activate logging,   optional, defaults to 0
  csvSeparator .. c for comma, s for semicolon, optional, defaults to c
  -win64 ........ to use a 64 bit simulator. By default, the 32 bit version is



This unzips the given FMU, parses the contained modelDescription.xml file, simulates the FMU from t=0 to t=tEnd, and writes the solution to file `result.csv`. The file is written in CSV format (comma-separated values), using `;` to separate columns and using `,` instead of `.` as decimal dot to print floating-point numbers. To change the result file format, use the `CSV separator` option. The logging option activates logging of the simulated FMU. The FMI specification does not specify what exactly to log in this case. However, when logging is switched on, the sample FMUs of the FMU SDK log every single FMU function call. Moreover, the fmusim simulators log every step and every event that is detected.

Example command:

```
fmusim me10 fmu10/fmu/me/win32/bouncingBall.fmu 5 0.1 0 s

parse C:\Users\tirea\AppData\Local\Temp\fmu\modelDescription.xml
fmiModelDescription
  fmiVersion=1.0
  modelName=bouncingBall
  modelIdentifier=bouncingBall
  guid={8c4e810f-3df3-4a00-8276-176fa3c9f003}
  numberOfContinuousStates=2
  numberOfEventIndicators=1
FMU Simulator: run 'fmu10/fmu/me/win32/bouncingBall.fmu' from t=0..5 with step size h=0.1, loggingOn=0, csv separator=';'
Simulation from 0 to 5 terminated successful
  steps ............ 51
  fixed step size .. 0.1
  time events ...... 0
  state events ..... 10
  step events ...... 0
CSV file 'result.csv' written
```


 ![image](https://github.com/semi-infiknight/FMU_SDK/assets/97100765/1ce10395-6be4-4d29-b996-63d33be362fe)
![image](https://github.com/semi-infiknight/FMU_SDK/assets/97100765/2805d28e-6a3a-4f09-99c7-ce3fad4fd3a7)

 

To plot the result file, open it, e.g. in a spreadsheet program, such as Microsoft Excel or OpenOffice Calc. The figure below shows the result of the above simulation when plotted using OpenOffice Calc 3.0. Note that the height h of the bouncing ball as computed by fmusim becomes negative at the contact points, while the true solution of the FMU does actually not contain negative height values. This is not a limitation of the FMU, but of fmusim_me, which does not attempt to locate the exact time of state events. To improve this, either reduce the step size or add your own procedure for state-event location to fmusim_me.
 




![image](https://github.com/semi-infiknight/FMU_SDK/assets/97100765/c66b809c-a251-48d3-baf8-56de823e10a4)





Creating FMU

To implement your own FMU using the FMU SDK, create a directory - say xy - in `FMUSDK_HOME/fmu10/src/models`, or `FMUSDK_HOME/fmu20/src/models`, and create files xy.c there. The name of the new directory and of the .c file must be the same. The content of the .c file should follow the existing FMU examples, see the comments in the example code. Add file modelDescription_cs.xml used for co-simulation and modelDescription_me.xml used for model-exchange. When done with editing xy.c and the xml files, open a command shell in `FMUSDK_HOME/fmu10/src/models` or in `FMUSDK_HOME/fmu20/src/models` to run the build command.

On Windows, run command build_fmu me xy to build an FMU for model-exchange, or build_fmu cs xy to build an FMU for co-simulation. This should create a 32 bit FMU file xy.fmu in the corresponding subdirectory of FMUSDK_HOME/fmu10 or FMUSDK_HOME/fmu20. To build a 64-bit FMU, append option -win64 to the build command.

The figure below might help to create or process the XML file modelDescription.xml. It shows all XML elements (without attributes) used in the schema files (XSD) for model exchange and co-simulation 1.0.


 ![image](https://github.com/semi-infiknight/FMU_SDK/assets/97100765/b854859e-1b58-4d6d-88a2-699e0953ddc7)


