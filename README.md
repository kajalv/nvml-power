# NVIDIA GPU average power measurement

This module helps to measure the average power level attained by an NVIDIA GPU during the execution of a CUDA kernel.

The module is suited for use in powerful GPUs which execute most kernels in the order of a few milliseconds. If the GPU in use is slower to execute and such precision is not required, polling can be done less frequently. On running, this creates a power file
*Power_data.txt* that lists all the power values obtained by polling nvml APIs during kernel execution. The output is in the form of one column of power measurements. The average power level can be computed from the generated power data.

The module is intended to be used as a basic template for measuring power. Sample code is to be modified as needed, such as specific error handling, time period modification, etc.

## Before using the module

Verify that nvcc and nvml are installed and running on the system. In certain devices, CUDA SDK installations are not complete, as nvml installation fails.

## Using the module

### Modifications to CUDA code

- Include the *nvmlPower.hpp* header.
- Surround the kernel call statement with *nvmlAPIRun()* and *nvmlAPIEnd()*. Ensure the API calls are executed only once, otherwise the power data file will be overwritten repeatedly.

Example:
```sh
nvmlAPIRun();
kernel_call<<<>>>();
nvmlAPIEnd();
```

### Makefile for the CUDA program

* Add the following to the Makefile:

```sh
INCLUDECUDA = -I/usr/local/cuda-7.0/samples/common/inc/
HEADERNVMLAPI = -L/usr/lib64/nvidia -lnvidia-ml -L/usr/lib64 -lcuda -I/usr/include -lpthread -I/(**Path_to_the_nvmlPower_files**)
INCLUDEPROG = (**Path_to_the_nvmlPower_files**)/nvmlPower.cpp
```

Here, *Path_to_the_nvmlPower_files* refers to the directory containing the *nvml-power* module. It can be set in an environment variable, if desired.

* Add $(HEADERNVMLAPI) at the end of *nvcc* compilation commands.

* Include the *.cpp* file referred to by *INCLUDEPROG* while compiling the program, along with *INCLUDECUDA*.

Example:
```sh
radixSortThrust: radixSortThrust.cu $(INCLUDEPROG)
	nvcc $(INCLUDECUDA) -o $@ $+ $(HEADERNVMLAPI)
```

## Error handling

In order to handle errors, the programmer must decide what needs to be the next step in each scenario. The function *getNVMLError()* returns an error code with a specific meaning.
