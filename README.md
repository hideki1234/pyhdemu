Hadoop Streaming Emulator for Python
=======

Hadoop Streaming Emulator for Python is a command line program which emulates the behavior of Hadoop when you'd run Python scripts as a mapper and a reducer. With this emulator, you can debug your mapper and reducer in Python before you actually run them on Hadoop.
  
  
### Prerequisite
* Python 2.7, 3.3, or 3.4  
(Make sure what version of Python is running on your target Hadoop platform.)

### Install
The emulator consists of files below:
* hdemu.py
* hseexceptions.py
* TextInputFormat.py
* TextOutputFormat.py
* aggregate.py

Copy these three files from [mikec964/chelmbigstock](https://github.com/mikec964/chelmbigstock)/emulator to your local directory. These files should be in the same directory.

### How to run
From command prompt, run  
python _install_dir_\hdemu.py -input _input_data_ -output _output_dir_ -mapper _python_mapper_ -reducer _python_reducer_

ex)  
`> python ..\emulator\hdemu.py -input data -output output_dir -mapper wc_mapper.py -reducer wc_reducer.py`

**Command line parameters**  

| parameter                  | required? | description                                      |  
| -------------------------- |:---------:| ------------------------------------------------ |  
| `-input input_data_or_dir` | **required** | Input data or directory. If a directory is specified, all files in the directory are used as input |
| `-output output_dir` | **required** | Output directory. The emulator creates the directory and stores result in it. If it already exists, the emulator returns error |
| `-mapper python_script` | **required** | Python script for mapper |
| `-reducer python_script` | **required** | Python script for reducer. If `aggregate` is specified, the built-in aggregator is used. |
| `-interim interim_dir` | optional | If specified, interim results (mapper input/output, reducer input/output) are saved in the directory. If the directory already exists, the emulator returns error. If the option is not specified, the interim results are deleted. |
| `-cmdenv envvar=value` | optional | Provide an environment variable to pass to mapper/reducer. To provide multiple environment variables, each variable must be preceded by `-cmdenv`. |
| `-files path1[,path2,...]` | optional | Specify comma-separated files to be copied to the emulator's working directory |

### Known problems
* the `aggregate` reducer supports `LongValueSum`, `LongValueMax`, and `LongValueMin` only.

### How to run sample mapper/reducer
A sample mapper/reducer is in the emulator\sample directory. It contains  

| file name | description |
| --------- | ----------- |
| readme.txt | About the sample |  
| wc_mapper.py | sample mapper script |  
| wc_reducer.py | sample reducer script |  
| input | data directory |  

**steps to run the sample**  
1. Copy the entire 'emulator' directory including the subdirectory (this directory) to a work directory  
2. In command prompt, go to the sample directory you just copied.  
3. Run the command below:  
`> python ..\hdemu.py -input input -output output -mapper wc_mapper.py -reducer wc_reducer.py`  
4. When the command finished, make sure the 'output' directory is created. Check out the contents.  
  
  
  
## Programming mapper/reducer in Python
### The first line of a mapper and reducer
A mapper/reducer script must start with the line  
`#!/usr/bin/env python`  
This is necessary to run your mapper/reducer on an actual Hadoop environment. The emulator works fine without the line, though.

### Mapper input
Hadoop doesn't pass a key to a Python mapper. It passes a value only. The value is a line of input files.  

### Mapper output
The format of mapper output must be `key + TAB + value`. In a Python string literal, `'key\tvalue'`.  

### Shuffle between mapper and reducer
After the mapper finishes, Hadoop puts together key-value pairs from the mapper and sort them by 'key'. For example, if the mapper returns <key, value> pairs  
`<one, 1> <two, 1> <three, 1> <two, 1> <three, 1> <four, 1> <three, 1> <four, 1> <four, 1> <four, 1>`  
Hadoop sorts them like  
`<four, 1> <four, 1> <four, 1> <four, 1> <one, 1> <three, 1> <three, 1> <three, 1> <two, 1> <two, 1>`  

### reducer input
A Python reducer receives a key-value pair one by one in the order sorted by Hadoop. The format is the same as mapper output (`key + TAB + value`). It is guaranteed that the pairs with the same key are delivered to a single reducer.  

### reducer output
The format must be the same as mapper (`key + TAB + value`).
***
