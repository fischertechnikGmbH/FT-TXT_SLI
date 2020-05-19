
# What is an SLI?
## Introduction
From the perspectif of RoboPro an SLI is a functional unit. RoboPro can send data to a SLI and can get data back from the SLI. But RoboPro can also start and stop more complex activities in a SLI, for example dedicated websocket or MQTT based communication with the outsife world, more complex closed control loops, etc.<br/>
The fischertechnic TXT SLI makes it possible that C/C++ functions can be access via a RoboPro elements.
For this RoboPro knows 2 elements, the `Shared library Input` element and the `Shared library Output` element. Thats is all.

The shared library input/output element allows to call functions and return/supply a value from/to shared library modules installed on the TXT controller. Such libraries are typically written in the C or C++ programming language. This allows interfacing ROBOPro with C / C++ programs, which is useful for accessing advanced sensors or for compute intensive tasks like image processing

For outputs the C functions should be declared like:<br/>
 `int setValueDouble(double v);int getValueDouble(double * v);`<br/> `int setValueShort(short v);int getValueShort(short * v); `

## The `Shared library Input` and the `Shared library Output` element.
Each input element allows to return only one numeric value. If parameters are required, the shared library output element can be used to first set parameters in the library. If multiple parameters or multiple return values are required, multiple input and/or output elements can be used. This means that it is typically required to write a small wrapper layer to interface ROBOPro to existing shared libraries. fischertechnik provides a library for the BME680 environmental sensor as example.

The input element can either retrieve a 16 bit signed short or a 64 bit double value from the shared library. Example C decalartions for such functions are: `c int getValueDouble(double* t); int getValueShort(short* t);

The function names should start with get and end with Short or Double to indicate the type, but arbitrary names can be used as well. A return value of 0 is interpreted as success, all other return values as error.

### Where to find?
THese two element can be found in `Send, receive`:

![](./docs/sli/element(both).PNG)

### What is the meaning of their properties?
This are the two:

![](./docs/sli/element(set).png)  ![](./docs/sli/element(get).png)

#### General properties
1. Library name & Extend library name<br/>
   The name of the share library consists als of 3 parts. <br/>
   [`lib`] 'NAME` [`.so`], The `NAME` is showed in the RoboPro box. <br/>
2. Function name& & Extend library name & Data type<br/>
   Under Function name the C name of the function to be called is entered. If Extend Function Name is checked, the name is prepended with get and the selected data type, either Short or Double, is appended.<br/>
   The name of the interface function consists of 3 parts in C/C++ :<br/>
   [`set` | `get`] `NAME` [`Short`|`Double`], The `NAME` is showed in the RoboPro box. <br/>

   When their is ai filter, the complete name will be visable in the RoboPro box.<br/>
   If the `Data type` is an integer, the RoboPro linker expect a function with shorts and
   if the `Data type` is a Float, the RoboPro linker expect a function with shorts.<br/>
   Hereby some examples:
   ```
   /*! Four functions from the library libExampleSLI.so, library NAME =ExampleSLI */

   /*! NAME=Value, data type= double, setter*/
   int setValueDouble(double v) {
   	value_d = v;
   	return 0;
   }

   /*! NAME=Value, data type= short, setter*/
   int setValueShort(short v) {
   	value_s = v;
   	return 0;
   }

   /*! NAME=Value, data type= double, getter*/
   int getValueDouble(double * v) {
   	*v = value_d;
   	return 0;
   }

    /*! NAME=Value, data type= short, getter*/
   int getValueShort(short * v) {
   	*v = value_s;
   	return 0;
   }
   ```
   The Data type of the value returned can be either a 16 bit signed integer or a floating point value. If the data type is floating point, the value is converted from C 64 bit double format to ROBOPro 48 bit float format. Please note that the range and precision of these types is different and conversion errors may occur, especially for out of range values. Please also note that for 16 bit integers values a value of -32768 is treated as undefined/error in ROBOPro and is usually displayed as '?'.

3. Error output<br/>
   Optional the function has the choice to continue with the normal workflow exit or with the error workflow exit.<br/>
   The error workflow exit will be follow when the function return a int value `not 0`. or when the RoboPro discover an error. This is visible in the trace info.<br/>
   ```
   /*! Example using */
   int setValueShort(short motorId) {
   	 if (!IsInit) return -1;
     if (motorId>4) return -2; if (motorId<1) return -3;//motorId out of range
   	/* user code */
   	return 0;
   }
   ```
   ![RoboPro](./docs/sli/element(error).png)<br/>
   The information in the trace on the screen.
   ```
   SharedLibraryInterface_ExecuteWriteINT16 libExampleSLI.so setValueShort 5
   SharedLibraryInterface_ExecuteReadINT16 lib 0xb3507898
   SharedLibraryInterface_ExecuteReadINT16 func 0xb1d809e9
   SharedLibraryInterface_ExecuteWriteINT16 libExampleSLI.so setValueShort RESULT -2 5

   ```
1. Lock Interface
    If Lock Interface is checked, the shared library IO is locked to the current ROBOPro thread. This allows safe combination of several shared library input and output elements without interruptions from other threads. The last element in a sequence must have this unchecked. Otherwise no other ROBOPro thread can use the shared library interface.




## How to trace/debug SLI's
  The fischertechnik RoboPro SLI run time writes enough data to the stout and sterr, in the SLI the developer can also add his own trace information or use SpdLog. In combination with een SSH-terminal (Putty), the log file from Putty and a editor like Notpad++ (search in the text) is this an important source during the development of the SLI.

# document history
- 2020-05-19 CvL 466.1.1 new