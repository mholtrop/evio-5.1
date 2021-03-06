# EVIO

This is a forked version of the EVIO 5.1, as it was originally released by JLAB.
The actual, proper, release of EVIO is found on git at: https://github.com/JeffersonLab/evio

The problem I have with the official version is that it does not build, and the C/C++ versions do not have
a functioning CMakeLists.txt. This version corresponds to the one used by GEMC for CLAS12.

In other words:

*WARNING THIS IS A HACK*

# ORIGINAL README Follow

Evio stands for event input/output and contains libraries which read & write
data in it own unique format. It was created and is maintained by the Data
Acquisition (DAQ) Group at Thomas Jefferson National Accelerator Facility
(Jefferson Lab). It was originally used in the online DAQ as part of their
CODA software, but is now being used offline and in other applications as
well.


There is a C library as well as a C++ library which is primarily a
wrapper around the C. There are a few utility programs included.
The Java version is a jar file containing all the capabilities
of the C and C++ libraries with a more extensive, user-friendly
interface. It contains a GUI for looking at evio format data.

If you only plan to run C/C++ applications you can skip the Java
installation. If you only plan to use Java applications you can
you can skip the C/C++ installation.


1) Getting & Installing evio

The evio package, including documentation, can be found on the
JLab Data Acquisition Group CODA wiki at http://coda.jlab.org.

To install all of evio, download the evio-4.x.tar.gz
file (or whatever version happens to be current) and untar it.
This will give you a full evio distribution with the top level
directory being evio-4.x. The documentation, is available on the
above-mentioned web site but also exists in the doc subdirectory
of the full distribution.

Note that for C/C++, only Linux, Solaris, and Darwin (Mac OSX)
operating systems are supported. The libraries and executables
are placed into the $CODA/<arch>/lib and bin subdirectories
(eg. ...Linux-x86_64/lib).  Be sure to change your LD_LIBRARY_PATH
environmental variable to include the correct lib directory.


2) C/C++ code

Compiling using SCons (make is no longer supported)

From the SCons website:
"SCons is an Open Source software
construction tool -- that is, a next-generation build tool.
Think of SCons as an improved, cross-platform substitute for
the classic Make utility with integrated functionality similar
to autoconf/automake and compiler caches such as ccache. In short,
SCons is an easier, more reliable and faster way to build software."

SCons is written in python, thus to use this build system with evio,
both python and SCons packages need previous installation. If your
system does not have one or the other, go to the http://www.python.org/
and http://www.scons.org/ websites for downloading.

The SCons files needed for compiling are already included as part of
the evio distribution. To compile, the user needs merely to run "scons"
in the top level evio directory. To compile and install libraries and
header files, first define the CODA environmental variable containing
the directory in which to install things and then run "scons install".

The following is a table of SCons compilation options:

scons -h              print out help
scons -c              remove all generated files

scons install         make C and C++ library code;
                      place libraries in architecture-specific lib directory
                      place binaries in architecture-specific bin directory
                      directory and headers in an include directory

scons -c install      uninstall libs, headers, and binaries

scons tar             create a tar file (evio-4.x.tgz) of the evio top level
                      directory and put in ./tar directory

scons doc             generate html documentation from doxygen
                      comments in the source code and put in ./doc directory

scons undoc           remove the doc/doxygen/C & CC
                      directories with all the generated documentation

scons --dbg           compile with debug flag
scons --32bits        compile 32bit libraries & executables on 64bit system
scons --vx5.5         cross compile for vxworks version 5.5
scons --vx6.0         cross compile for vxworks version 6.0

scons --prefix=<dir>  use base directory <dir> when doing install.
                      Defaults to CODA environmental variable.
                      Libs go in <dir>/<arch>/lib, headers in <dir>/<arch>/include
                      and executables in <dir>/<arch>/bin

scons --incdir=<dir>  copy header files to directory <dir> when doing install
                      (takes precedence over --prefix or default location)

scons --libdir=<dir>  copy library files to directory <dir> when doing install
                      (takes precedence over --prefix or default location)

scons --bindir=<dir>  copy executable files to directory <dir> when doing install
                      (takes precedence over --prefix or default location)



You can see these options by running "scons -h"


Note that currently only Linux, Solaris, and Darwin (Mac OSX)
operating systems are fully supported. The libraries and executables
are placed into the $CODA/<arch>/lib and bin subdirectories
(eg. ...Linux-x86_64/lib).  Be sure to change your LD_LIBRARY_PATH
environmental variable to include the correct lib directory.

All libraries and a limited set of executables are also supported for
vxworks versions 5.5 and 6.0 .


3) JAVA

One can download the jevio-4.x.jar file from the CODA website or it can
be generated from the general evio distribution.  In either case,
put the jar file into your classpath and run your java application.

If you're using the jar file from the CODA website, java version 1.6 or
higher is necessary since it was compiled with version 1.6. If
generating it, use java version 1.5 or higher to compile.

If you wish to recompile the java part of evio, ant must be installed
on your system (http://ant.apache.org). First extract the package files
from the general evio tar file:
  
# download evio-4.x.tar.gz into <top>
$ cd <top>
$ tar -fxz evio-4.x.tar.gz 
$ cd evio-4.x
$ ant

To get a list of options with ant, type "ant help":

 ant  help       - print out usage
 ant  env        - print out build file variables' values
 ant  compile    - compile java files
 ant  clean      - remove class files
 ant  cleanall   - remove all generated files
 ant  jar        - compile and create jar file
 ant  install    - create jar file and install into 'prefix'
 ant               if given on command line by -Dprefix=dir',
 ant               else install into CODA if defined
 ant  uninstall  - remove jar file previously installed into 'prefix'
 ant               if given on command line by -Dprefix=dir',
 ant               else installed into CODA if defined
 ant  all        - clean, compile and create jar file
 ant  javadoc    - create javadoc documentation
 ant  developdoc - create javadoc documentation for developer
 ant  undoc      - remove all javadoc documentation
 ant  prepare    - create necessary directories


To generate a new evio jar file, type "ant jar" which will
create the file and place it in <top>/build/lib .


4) Documentation

All documentation is available from http://coda.jlab.org.

However, if using the downloaded distribution, some of the documentation
needs to be generated and some already exists. For existing docs look in
doc/users_guide for pdf and Microsoft Word format documents.

Some of the documentation is in the source code itself and must be generated
and placed into its own directory.
The java code is documented with, of course, javadoc and the C/C++ code is
documented with doxygen comments (identical to javadoc comments).

To generate all the these docs, from the top level directory type:
    "scons doc"
To remove it all type:
    "scons undoc"

The javadoc is placed in the doc/javadoc directory.
The doxygen docs for C code are placed in the doc/doxygen/C/html directory,
and the doxygen docs for C++ code are placed in the doc/doxygen/CC/html directory.
To view the html documentation, just point your browser to the index.html
file in each of those directories.

Alternatively, just the java documentation can be generated by typing
    "ant javadoc"
for user-level documentation, or
    "ant developdoc"
for developer-level documentation. To remove it:
    "ant undoc"


5) Problems

Carl Timmer - timmer@jlab.org


6) Copyright

For any issues regarding use and copyright, read the NOTICE file.

