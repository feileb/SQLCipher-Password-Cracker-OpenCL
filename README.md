Description
===========

**PBKDF2-noHMAC-SHA1--AES-256-CBC** encrypted SQLite database (**SQLCipher v2** standard) password bruteforcing using OpenCL and Python. The code here is for cracking **password only consists of fixed-length hex chars**, but can be easily adapted for universal use. This repository is the result of many developers' effort.

Installation
=============

1. Get **python 3.7 64-Bit**
2. Download and install the **Win32 OpenCL SDK** (e.g. from [Intel](https://software.intel.com/en-us/articles/opencl-drivers) to get the CPU platform SDK(usually the new gen Intel GPUs have already contain SDK in its normal drivers), or from [Nvidia](https://developer.nvidia.com/cuda-downloads) to get CUDA SDK for GPUs)
3. Install **[pyOpenCL](https://pypi.org/project/pyopencl/)** using:
   `python -m pip install pyopencl`  
   Maybe you should manually configure something about your OpenCL SDK path, refer to pyOpenCL [wiki](https://wiki.tiker.net/PyOpenCL/Installation/)
4. Install **[pysqlcipher3](https://github.com/rigglemania/pysqlcipher3)** using the manual there

Hints for pysqlcipher3 Installation
======================

## Suggestion for Windows users ##

In the process of installing pysqlcipher, you should install **[OpenSSL x64 libraries](http://slproweb.com/products/Win32OpenSSL.html)**  and then compile **[sqlcipher](https://github.com/sqlcipher/sqlcipher)** first. I suggest "Build against amalgamation", which is tested in mycomputer, whose steps after installing OpenSSL are as following:

1. Compile the amalgamation files of sqlcipher:
Run the **Developer Command Prompt of Visual Studio**, then cded to the **sqlcipher folder**, run:

    `nmake /f Makefile.msc sqlite3.c`

2. Copy to pysqlcipher3 folder: Make a folder named ***amalgamation*** in **pysqlcipher3 folder**,  then copy the generated ***sqlite3.c*** and ***sqlite3.h*** to the folder.

3. Modify the source files of pysqlcipher3: In ***pysqlcipher3\src\python3***, change the ***"#include "pysqlcipher\sqlite3.h""*** to ***"#include "sqlite3.h""*** of ***connection.h***, ***statement.h*** and ***util.h***.

4. Modify the setup.py of pysqlcipher3: change the string ***"openssl_lib_path = os.path.join(openssl, "lib")"*** to ***"openssl_lib_path = os.path.join(openssl, "lib\VC")"*** Then, change the string ***"libeay32.lib"*** to ***"libcrypto64MD.lib"***, then add a following statement: ***ext.extra_link_args.append("libssl64MD.lib")***

5. Build and Install: cded to **pysqlcipher3 folder**:

    `python setup.py build_amalgamation`  
    `python setup.py install`  

    Finally copy the ***libcrypto-1_1-x64.dll*** and ***libssl-1_1-x64.dll*** in the OpenSSL path to **pysqlcipher3 folder**.

## Suggestion for Linux users ##

1. You can directly get sqlcipher dev package:

    `sudo apt-get install libsqlcipher-dev`

2. Then install pysqlcipher3:

    `python -m pip install pysqlcipher3`  

3. That's all.

Run
===

1. Run `python genTestDB.py [password]` to get a password encrypted SQLite database for test.
2. Modify the variables from the header of ***opencl_test.py*** to satisfy your requirments.
3. `python opencl_test.py`
4. Now you get the available platforms and their number in console output.
5. `python opencl_test.py [platform Number]`

Thanks for support
==================

- OpenCL code for PBKDF2-SHA1 and python code framework： [opencl_brute](https://github.com/bkerler/opencl_brute) by *B. Kerler*
- OpenCL code for AES roundkeys: [AES-OpenCL](https://github.com/adrianbelgun/AES-OpenCL) by *adrianbelgun*
- OpenCL code for AES-CBC-256 decrypting: [CryptoCL](https://github.com/Omegaice/CryptoCL) by *Omegaice*
- General ideas on validating the decrypted database: [EnMicroMsg.db-Password-Cracker](https://github.com/chg-hou/EnMicroMsg.db-Password-Cracker) by *chg-hou*

Ideas
=====

- Using a small trick to map the **hex string password representation** with **OpenCL numeric global id** at a low cost in performance, so this code uses little (video and main) memory.
- Reasonably divide the tasks of CPU and GPU, avoid large data transfer between devices.

Issues
======

- Tested with : Intel Core i5 8300H, NVIDIA GTX 1050 Ti on Windows 10.
- AMD APP SDK is **not** supported because the AMD OpenCL compiler will wrongly parse the marcos in opencl code, thus get mistaken result. Theoritically you can replace the marcos with real functions to solve the problem, as all OpenCL functions will get inlined so no overhead will appear, but I have not practically tested this.
- Only support maximum 16 chars for password with hex chars ranged from 0-9,a-f.
- Commandline parser and distributed cracking is being developed, will be released soon.
- More decryption algorithm of sqlcipher and its variant is considered for future development.