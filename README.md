[![Build Status](https://travis-ci.org/EIPStackGroup/OpENer.svg?branch=master)](https://travis-ci.org/EIPStackGroup/OpENer)<a href="https://scan.coverity.com/projects/opener">
  <img alt="Coverity Scan Build Status"
       src="https://scan.coverity.com/projects/14200/badge.svg?flat=1"/>
</a> 
[![Quality Gate](https://sonarcloud.io/api/project_badges/measure?project=OpENer&metric=alert_status)](https://sonarcloud.io/dashboard?id=OpENer)
[![Join the chat at https://gitter.im/EIPStackGroupOpENer/Lobby](https://badges.gitter.im/EIPStackGroupOpENer/Lobby.svg)](https://gitter.im/EIPStackGroupOpENer/Lobby?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)


# Documentation
    @mainpage OpENer - Open Source EtherNet/IP(TM) Communication Stack
	##  Documentation
	
	 EtherNet/IP stack for adapter devices (connection target); supports multiple
	 I/O and explicit connections; includes features and objects required by the
	 CIP specification to enable devices to comply with ODVA's conformance/
	 interoperability tests.
	
	 @section intro_sec Introduction
	
	 ## This is the introduction.
	
	 @section install_sec Building
	 How to compile, install and run OpENer on a specific platform.
	
	 @subsection build_req_sec Requirements
	 OpENer has been developed to be highly portable. The default version targets
	 PCs with a POSIX operating system and a BSD-socket network interface. To
	 test this version we recommend a Linux PC or Windows with Cygwin installed.
	  You will need to have the following installed:
	   - gcc, make, binutils, etc.
	
	 for normal building. These should be installed on most Linux installations
	 and are part of the development packages of Cygwin.
	
	 For the development itself we recommend the use of Eclipse with the CDT
	 plugin. For your convenience OpENer already comes with an Eclipse project
	 file. This allows to just import the OpENer source tree into Eclipse.
	
	 @subsection compile_pcs_sec Compile for PCs
	   -# Directly in the shell
	       -# Go into the bin/pc directory
	       -# Invoke make
	       -# For invoking opener type:\n
	          ./opener ipaddress subnetmask gateway domainname hostaddress
	 macaddress\n
	          e.g., ./opener 192.168.0.2 255.255.255.0 192.168.0.1 test.com
	 testdevice 00 15 C5 BF D0 87
	   -# Within Eclipse
	       -# Import the project
	       -# Go to the bin/pc folder in the make targets view
	       -# Choose all from the make targets
	       -# The resulting executable will be in the directory
	           ./bin/pc
	       -# The command line parameters can be set in the run configuration
	 dialog of Eclipse
	
	 @section further_reading_sec Further Topics
	   - @ref porting
	   - @ref extending
	   - @ref license
	
	 @page porting Porting OpENer
	 @section gen_config_section General Stack Configuration
	 The general stack properties have to be defined prior to building your
	 production. This is done by providing a file called opener_user_conf.h. An
	 example file can be found in the src/ports/POSIX or src/ports/WIN32 directory.
	 The documentation of the example file for the necessary configuration options:
	 opener_user_conf.h
	
	 @copydoc ./ports/POSIX/sample_application/opener_user_conf.h
	
	 @section startup_sec Startup Sequence
	 During startup of your EtherNet/IP(TM) device the following steps have to be
	 performed:
	   -# Configure the network properties:\n
	       With the following functions the network interface of OpENer is
	       configured:
	        - EIP_STATUS ConfigureNetworkInterface(const char *ip_address,
	        const char *subnet_mask, const char *gateway_address)
	        - void ConfigureMACAddress(const EIP_UINT8 *mac_address)
	        - void ConfigureDomainName(const char *domain_name)
	        - void ConfigureHostName(const char *host_name)
	        .
	       Depending on your platform these data can come from a configuration
	       file or from operating system functions. If these values should be
	       setable remotely via explicit messages the SetAttributeSingle functions
	       of the EtherNetLink and the TCPIPInterface object have to be adapted.
	   -# Set the device's serial number\n
	      According to the CIP specification a device vendor has to ensure that
	      each of its devices has a unique 32Bit device id. You can set it with
	      the function:
	       - void setDeviceSerialNumber(EIP_UINT32 serial_number)
	   -# Initialize OpENer: \n
	      With the function CipStackInit(EIP_UINT16 unique_connection_id) the
	      internal data structures of opener are correctly setup. After this
	      step own CIP objects and Assembly objects instances may be created. For
	      your convenience we provide the call-back function
	      ApplicationInitialization. This call back function is called when the
	 stack is ready to receive application specific CIP objects.
	   -# Create Application Specific CIP Objects:\n
	      Within the call-back function ApplicationInitialization(void) or
	      after CipStackInit(void) has finished you may create and configure any
	      CIP object or Assembly object instances. See the module @ref CIP_API
	      for available functions. Currently no functions are available to
	      remove any created objects or instances. This is planned
	      for future versions.
	   -# Setup the listening TCP and UDP port:\n
	      THE ETHERNET/IP SPECIFICATION demands from devices to listen to TCP
	      connections and UDP datagrams on the port AF12hex for explicit messages.
	      Therefore before going into normal operation you need to configure your
	      network library so that TCP and UDP messages on this port will be
	      received and can be hand over to the Ethernet encapsulation layer.
	
	 @section normal_op_sec Normal Operation
	 During normal operation the following tasks have to be done by the platform
	 specific code:
	   - Establish connections requested on TCP port AF12hex
	   - Receive explicit message data on connected TCP sockets and the UPD socket
	     for port AF12hex. The received data has to be hand over to Ethernet
	     encapsulation layer with the functions: \n
	      int HandleReceivedExplictTCPData(int socket_handle, EIP_UINT8* buffer, int
	 buffer_length, int *number_of_remaining_bytes),\n
	      int HandleReceivedExplictUDPData(int socket_handle, struct sockaddr_in
	 *from_address, EIP_UINT8* buffer, unsigned int buffer_length, int
	 *number_of_remaining_bytes).\n
	     Depending if the data has been received from a TCP or from a UDP socket.
	     As a result of this function a response may have to be sent. The data to
	     be sent is in the given buffer pa_buf.
	   - Create UDP sending and receiving sockets for implicit connected
	 messages\n
	     OpENer will use to call-back function int CreateUdpSocket(
	     UdpCommuncationDirection connection_direction,
	     struct sockaddr_in *pa_pstAddr)
	     for informing the platform specific code that a new connection is
	     established and new sockets are necessary
	   - Receive implicit connected data on a receiving UDP socket\n
	     The received data has to be hand over to the Connection Manager Object
	     with the function EIP_STATUS HandleReceivedConnectedData(EIP_UINT8
	 *data, int data_length)
	   - Close UDP and TCP sockets:
	      -# Requested by OpENer through the call back function: void
	 CloseSocket(int socket_handle)
	      -# For TCP connection when the peer closed the connection OpENer needs
	         to be informed to clean up internal data structures. This is done
	 with
	         the function void CloseSession(int socket_handle).
	      .
	   - Cyclically update the connection status:\n
	     In order that OpENer can determine when to produce new data on
	     connections or that a connection timed out every @ref kOpenerTimerTickInMilliSeconds
	 milliseconds the
	     function EIP_STATUS ManageConnections(void) has to be called.
	
	 @section callback_funcs_sec Callback Functions
	 In order to make OpENer more platform independent and in order to inform the
	 application on certain state changes and actions within the stack a set of
	 call-back functions is provided. These call-back functions are declared in
	 the file opener_api.h and have to be implemented by the application specific
	 code. An overview and explanation of OpENer's call-back API may be found in
	 the module @ref CIP_CALLBACK_API.
	
	 @page extending Extending OpENer
	 OpENer provides an API for adding own CIP objects and instances with
	 specific services and attributes. Therefore OpENer can be easily adapted to
	 support different device profiles and specific CIP objects needed for your
	 device. The functions to be used are:
	   - CipClass *CreateCipClass( const CipUdint class_code,
                          const int number_of_class_attributes,
                          const EipUint32 highest_class_attribute_number,
                          const int number_of_class_services,
                          const int number_of_instance_attributes,
                          const EipUint32 highest_instance_attribute_number,
                          const int number_of_instance_services,
                          const int number_of_instances,
                          char *name,
                          const EipUint16 revision,
                          InitializeCipClass initializer );
	   - CipInstance *AddCipInstances(CipClass *RESTRICT const cip_class,
                             const int number_of_instances)
	   - CipInstance *AddCipInstance(CipClass *RESTRICT const class,
                            const EipUint32 instance_id)
	   - void InsertAttribute(CipInstance *const cip_instance,
                     const EipUint16 attribute_number,
                     const EipUint8 cip_data_type,
                     void *const cip_data,
                     const EipByte cip_flags);
	   - void InsertService(const CipClass *const cip_class_to_add_service,
                   const EipUint8 service_code,
                   const CipServiceFunction service_function,
                   char *const service_name);
	
	 @page license OpENer Open Source License
	 The OpENer Open Source License is an adapted BSD style license. The
	 adaptations include the use of the term EtherNet/IP(TM) and the necessary
	 guarding conditions for using OpENer in own products. For this please look
	 in license text as shown below:
	
	 @include "../license.txt"
	
	/ 










OpENer Version 2.1.0
====================

Welcome to OpENer!
------------------

OpENer is an EtherNet/IP&trade; stack for I/O adapter devices; supports multiple 
I/O and explicit connections; includes objects and services to make EtherNet/IP&trade;-
compliant products defined in THE ETHERNET/IP SPECIFICATION and published by 
ODVA (http://www.odva.org).

Participate!
------------
Users and developers of OpENer can join the respective Google Groups in order to exchange experience, discuss the usage of OpENer, and to suggest new features and CIP objects, which would be useful for the community.

Developers mailing list: https://groups.google.com/forum/#!forum/eip-stack-group-opener-developers

Users mailing list: https://groups.google.com/forum/#!forum/eip-stack-group-opener-users

Requirements:
-------------
OpENer has been developed to be highly portable. The default version targets PCs
with a POSIX operating system and a BSD-socket network interface. To test this 
version we recommend a Linux PC or Windows with Cygwin (http://www.cygwin.com) 
installed. You will need to have the following installed:
	 CMake
* gcc
* make
* binutils
* the development library of libcap (libcap-dev or equivalient)
 
for normal building. These should be installed on most Linux installations and
are part of the development packages of Cygwin.

If you want to run the unit tests you will also have to download CppUTest via
https://github.com/cpputest/cpputest

For configuring the project we recommend the use of a CMake GUI (e.g., the 
cmake-gui package on Linux, or the Installer for Windows available at [CMake](https://cmake.org/))

Compile for Linux/POSIX:
----------------
1. Make sure all the needed tools are available (CMake, make, gcc, binutils)
2. Change to the <OpENer main folder>/bin/posix
3. For a standard configuration invoke ``setup_posix.sh``
	1. Invoke the ``make`` command
	2. Grant OpENer the right to use raw sockets via ``sudo setcap cap_net_raw+ep ./src/ports/POSIX/OpENer``
	3. Invoking OpENer:

		``./src/ports/POSIX/OpENer <interface_name>``

		e.g. ``./src/ports/POSIX/OpENer eth1``

OpENer also now has a real-time capable POSIX startup via the OpENer_RT option, which requires that the used kernel has the full preemptive RT patches applied and activated.
If you want to use OpENer_RT, instead of step 2, the  ``sudo setcap cap_net_raw,cap_ipc_lock,cap_sys_nice+ep ./src/ports/POSIX/OpENer
`` has to be run to grant OpENEr ``CAP_SYS_NICE``, ``CAP_IPC_LOCK``, and the ``CAP_NET_RAW`` capabilities, needed for the RT mode

Shared library support has been added to CMakeLists file and is enabled by setting OPENER_BUILD_SHARED_LIBS=ON. It has only been tested under Linux/POSIX platform.


Compile for Windows XP/7/8 via Visual Studio:
---------------------------------------------
1. Invoke setup_windows.bat or configure via CMake
2. Open Visual Studio solution OpENer.sln in bin/win32
3. Compile OpENer by chosing ``Build All`` in Visual Studio
4. For invoking OpENer type from the command line:
	1. Change to <OpENer main folder>\bin\win32\src\ports\WIN32\
	2. Depending if you chose the ``Debug`` or ``Release`` configuration in Visual Studio, your executable will either show up in the subfolder Debug or Release
	3. Invoke OpENer via

		``OpENer <interface_index>``

		e.g. ``OpENer 3``

In order to get the correct interface index enter the command ``route print`` in a command promt and search for the MAC address of your chosen network interface at the beginning of the output. The leftmost number is the corresponding interface index.
		
Compile for Windows XP/7/8/10 via Cygwin:
--------------------------------------
The POSIX setup file can be reused for Cygwin. Please note, that you cannot use RT mode and you will have to remove the code responsible for checking and getting the needed capabilities, as libcap is not available in Cygwin. The easier and more supported way to build OpENer for Windows is to either use MinGW or Visual Studio.

In order to run OpENer, it has to be run as privileged process, as it needs the rights to use raw sockets.

Compile for MinGW on Windows XP/7/8/10
-------------------------------
1. Make sure 64 bit mingw is installed. (Test with gcc --version, should show x86_64-posix-seh-rev1)
2. Make sure CMake is installed. (Test with cmake --version, should be version 3.xx)
3. Change to <opener install dir>/bin/mingw
4. Run the command `setup_mingw.bat` in a dos command line. (Not a bash shell). If tracing is desired, 
use the following (where the cmake parameter must be enclosed in quotes) or change the ./source/CMakeList.txt file.
    ```
    setup_mingw.bat "-DOpENer_TRACES:BOOL=TRUE"
    ```
5. Run the command "make" from the same directory (./bin/mingw)
6. The opener.exe is now found in <opener install dir>\bin\mingw\src\ports\MINGW
7. Start it like this: "opener 192.168.250.22", where the ip address is the local computer's address on the nettwork you want to use.
		
Directory structure:
--------------------
- bin ...  The resulting binaries and make files for different ports
- doc ...  Doxygen generated documentation (has to be generated for the SVN version) and Coding rules
- data ... EDS file for the default application
- source
	- src ... the production source code
		- cip ... the CIP layer of the stack
		- cip_objects ... additional CIP objects
		- enet_encap ... the Ethernet encapsulation layer
		- ports ... the platform specific code
		- utils ... utility functions
	- tests ... the test source code
		- enet_encap ... tests for Ethernet encapsulation layer
		- utils ... tests for utility functions

Documentation:
--------------
The documentation of the functions of OpENer is part of the source code. The source 
packages contain the generated documentation in the directory doc/api_doc. If you 
use the GIT version you will need the program Doxygen for generating the HTML 
documentation. You can generate the documentation by invoking doxygen from the 
command line in the opener main directory.

Porting OpENer:
---------------
For porting OpENer to new platforms please see the porting section in the 
Doxygen documentation.

Contributing to OpENer:
-----------------------
The easiest way is to fork the repository, then create a feature/bugfix branch.
After finishing your feature/bugfix create a pull request and explain your changes.
Also, please update and/or add doxygen comments to the provided code sections.
Please stick to the coding conventions, as defined in source/doc/coding_rules
The easiest way to conform to the indenting convertion is to set uncrustify as git filter in the OpENer repository, which can be done with the following to commands:

```
git config filter.uncrustify.clean "/path/to/uncrustify/uncrustify -c uncrustify.cfg --mtime --no-backup"

git config filter.uncrustify.smudge "cat"
```


