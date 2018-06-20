
![../../../Pictures/Intel_Logo.png](../../../Pictures/Intel_Logo.png))

#Distributed Asynchronous Object Storage (DAOS)
Installation, Configuration and Administration Guide

High Performance Data Division

INTEL FEDERAL, LLC PROPRIETARY

May 2018

**Generated under Argonne Contract number: B609815**

    **DISTRIBUTION STATEMENT:** None Required

    **Disclosure Notice:** This presentation is bound by Non-Disclosure
    Agreements between Intel Corporation, the Department of Energy, and
    DOE National Labs, and is therefore for Internal Use Only and not
    for distribution outside these organizations or publication outside
    this Subcontract.

    **USG Disclaimer:** This report was prepared as an account of work
    sponsored by an agency of the United States Government. Neither the
    United States Government nor any agency thereof, nor any of their
    employees, makes any warranty, express or implied, or assumes any
    legal liability or responsibility for the accuracy, completeness, or
    usefulness of any information, apparatus, product, or process
    disclosed, or represents that its use would not infringe privately
    owned rights. Reference herein to any specific commercial product,
    process, or service by trade name, trademark, manufacturer, or
    otherwise does not necessarily constitute or imply its endorsement,
    recommendation, or favoring by the United States Government or any
    agency thereof. The views and opinions of authors expressed herein
    do not necessarily state or reflect those of the United States
    Government or any agency thereof.

    **Export:** This document contains information that is subject to
    export control under the Export Administration Regulations.

    **Intel Disclaimer:** Intel makes available this document and the
    information contained herein in furtherance of CORAL. None of the
    information contained therein is, or should be construed, as advice.
    While Intel makes every effort to present accurate and reliable
    information, Intel does not guarantee the accuracy, completeness,
    efficacy, or timeliness of such information. Use of such information
    is voluntary, and reliance on it should only be undertaken after an
    independent review by qualified experts.

    Access to this document is with the understanding that Intel is not
    engaged in rendering advice or other professional services.
    Information in this document may be changed or updated without
    notice by Intel.

    This document contains copyright information, the terms of which
    must be observed and followed.

    Reference herein to any specific commercial product, process or
    service does not constitute or imply endorsement, recommendation, or
    favoring by Intel or the US Government.

    Intel makes no representations whatsoever about this document or the
    information contained herein. IN NO EVENT SHALL INTEL BE LIABLE TO
    ANY PARTY FOR ANY DIRECT, INDIRECT, SPECIAL OR OTHER CONSEQUENTIAL
    DAMAGES FOR ANY USE OF THIS DOCUMENT, INCLUDING, WITHOUT LIMITATION,
    ANY LOST PROFITS, BUSINESS INTERRUPTION, OR OTHERWISE, EVEN IF INTEL
    IS EXPRESSLY ADVISED OF THE POSSIBILITY OF SUCH DAMAGES.

    Company Name: Intel Federal, LLC

    Company Address: 4100 Monument Corner Drive, Suite 540 Fairfax, VA
    22030

    Copyright © 2015 - 2016, Intel Corporation.

    Technical Lead: (Name, E-mail, Phone) \_Al Gara,
    alan.gara@intel.com, 408-765-0996

    Contract Administrator: (Name, E-mail, Phone) Aaron Matzkin,
    aaron.matzkin@intel.com, 503-712-0833

    Program Manager: (Name, E-mail, Phone) Jacob Wood,
    jacob.r.wood@intel.com, 503-264-2219

Document Revision History

Revision Number|Date|Comments
---|---|---
1.0     |      March 2018  | Initial version.
                                     

  

Contents

[1 Introduction](#introduction)

-   [1.1 Terms used in this Document](#terms-used-in-this-document)

-   [1.2 Additional Documentation](#additional-documentation)

-   [1.3 Software Requirements](#software-requirements)

-   [1.4 Hardware Compatibility](#hardware-compatibility)

[2 DAOS Architecture](#daos-architecture)

[3 Installing DAOS](#installing-daos)

-   [3.1 Getting DAOS](#_Toc515893238)

-   [3.2 Building DAOS](#building-daos)

-   [3.3 DAOS Components](#daos-components)

[4 Deploying DAOS](#deploying-daos)

-   [4.1 Configuring DAOS](#configuring-daos)

    -   [4.1.1 Introduction](#introduction-1)

    -   [4.1.2 Tuning DAOS](#tuning-daos)

    -   [4.1.3 Command Line Interface](#command-line-interface)

    -   [4.1.4 Troubleshooting](#_Toc515893246)

[5 Monitoring DAOS](#monitoring-daos)

[6 DAOS Fault Handling](#daos-fault-handling)

-   [6.1 Introduction](#introduction-2)

[7 DAOS Security Framework](#daos-security-framework)

[Appendix A. Usage Examples](#usage-examples)

-   [A.1 Usage Examples of DAOS](#usage-examples-of-daos)

1.  



Introduction
============

1.  The Distributed Asynchronous Object Storage (DAOS) is an
    open-source, software-defined object store designed from the ground
    up for massively distributed Non Volatile Memory (NVM). DAOS takes
    advantage of next generation NVM technology like Storage Class
    Memory (SCM) and NVM express (NVMe) while presenting a key-value
    storage interface and providing features such as transactional
    non-blocking I/O, advanced data protection with self healing on top
    of commodity hardware, end-to-end data integrity, fine grained data
    control and elastic storage to optimize performance and cost.

Terms used in this Document 
----------------------------

1.  The following terms and abbreviations are used in this document.

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Term                      Definition
  ------------------------- ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  1.  Active member count   1.  The number of members in a Service Process set which have not been evicted
                            

  1.  BBN                   1.  Burst Buffer Node
                            

  1.  Client Process Set    1.  A group of processes that consume services provided by one or more service processes sets.
                            

  1.  CN                    1.  Compute Node
                            

  1.  CPU                   1.  Central Processing Unit
                            

  1.  Daemon                1.  A process offering system level resources.
                            

  1.  HLD                   1.  High Level Design
                            

  1.  I/O                   1.  Input/Output
                            

  1.  ION                   1.  I/O Node (e.g., Burst Buffer node, gateway node)
                            

  1.  CaRT                  1.  The CaRT RPC library. A software library built on top of the Mercury Function Shipping library to provide distributed communication functionality.
                            

  1.  OFI                   1.  OpenFabrics Interfaces
                            

  1.  Origin/Target         1.  In the context of a single RPC, the Origin is the process calling the API and the Target is the process that executes it.
                            

  1.  OS                    1.  Operating System
                            

  1.  PSR                   1.  Preferred Service Rank. The default target rank used for RPCs by a member of a client process set to a service process set when the RPC does not require a specific target.
                            

  1.  RPC                   1.  Remote Procedure Call.
                            

  1.  RTE                   1.  Run Time Environment
                            

  1.  Service Process Set   1.  A group of processes that collaborate to provide a distributed service.
                            

  1.                        1.  
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Additional Documentation
------------------------

1.  Refer to the following documentation for architecture and
    description:

  Document   Location
  ---------- ----------
  1.         1.  
  1.         1.  

1.  

Software Requirements
---------------------

1.  DAOS requires a C99-capable compiler, a golang compiler and the
    scons build tool. Moreover, the DAOS stack leverages the following
    open source projects:

-   CaRT[^1] that relies on both Mercury[^2] and Libfabric[^3] for
    lightweight network transport and PMIx[^4] for process set
    management. See the CaRT repository for more information on how to
    build the CaRT library.

-   PMDK[^5] for persistent memory programming.

-   SPDK[^6] for userspace NVMe device access and management.

-   ISA-L[^7] for checksum and erasure code computation.

-   Argobots[^8] for thread management.

1.  The DAOS build system can be configured to download and build any
    missing dependencies automatically.

Hardware Compatibility
----------------------

1.  DAOS requires a 64-bit processor architecture and is primarily
    developed on x64-64 and AArch64 platforms. Apart from using a
    popular Linux distribution, no special considerations are necessary
    to build DAOS on 64-bit ARM processors. The same instructions that
    are used in Xeon are applicable for ARM builds as well. DAOS and its
    dependencies will make the necessary adjustments automatically in
    their respective build systems for ARM platforms.

    Storage-class memory (SCM) can be emulated with DRAM by creating
    tmpfs mountpoints on the DAOS servers.

DAOS Architecture
=================

Installing DAOS
===============

1.  DAOS is open-source and available in GitHub at
    <https://github.com/daos-stack>.

    To check out the DAOS source code, run the following command:

<!-- -->

1.  \$ git clone https://github.com/daos-stack/daos.git

<!-- -->

1.  This clones the DAOS git repository (path referred as \${daospath}
    below). To initialize the submodules, run the following commands:

<!-- -->

1.  \$ cd \${daospath}

    \$ git submodule init

    \$ git submodule update

Building DAOS:
--------------

1.  Docker is the fastest way to build, install and run DAOS on a
    non-Linux system.

### Building DAOS in a Container

1.  To build the Docker image directly from GitHub, run the following
    command:

<!-- -->

1.  \$ docker build -t daos -f Dockerfile.centos\\:7
    github.com/daos-stack/daos\#:utils/docker

<!-- -->

1.  This creates a CentOS7 image, fetches the latest DAOS version from
    GitHub and builds it in the container. For Ubuntu, replace
    Dockerfile.centos:7 with Dockerfile.ubuntu:16.04.

    To build from a local tree stored on the host, a volume must be
    created to share the source tree with the Docker container. To do
    so, execute the following command to create a docker image without
    checking out the DAOS source tree:

<!-- -->

1.  \$ docker build -t daos -f utils/docker/Dockerfile.centos\\:7
    --build-arg NOBUILD=1 .

<!-- -->

1.  And then the following command to export the DAOS source tree to the
    docker container and build it:

<!-- -->

1.  \$ docker run -v \${daospath}:/home/daos/daos:Z daos scons
    --build-deps=yes USE\_INSTALLED=all install

### Running DAOS in a Container

1.  Let's first create a container that will run the DAOS service:

<!-- -->

1.  \$ docker run -it -d --name server --tmpfs
    /mnt/daos:rw,uid=1000,size=1G -v /tmp/uri:/tmp/uri daos

<!-- -->

1.  Add "-v \${daospath}:/home/daos/daos:Z" to this command line if DAOS
    source tree is stored on the host.

    This allocates 1GB of DRAM for DAOS storage. The more, the better.

    To start the DAOS service in this newly created container, execute
    the following command:

<!-- -->

1.  \$ docker exec server orterun -H localhost -np 1 --report-uri
    /tmp/uri/uri.txt daos\_server

<!-- -->

1.  Once the DAOS server is started, the integration tests can be run as
    follows:

<!-- -->

1.  \$ docker run -v /tmp/uri:/tmp/uri daos \\

    orterun -H localhost -np 1 --ompi-server file:/tmp/uri/uri.txt
    daos\_test

<!-- -->

1.  Again, "-v \${daospath}:/home/daos/daos:Z" must be added if the DAOS
    source tree is shared with the host.

Build DAOS from Scratch
-----------------------

1.  The below instructions have been verified with CentOS. Installations
    on other Linux distributions might be similar with some variations.
    Please contact us in our forum if running into issues.

### Build Prerequisites

1.  Please install the following software packages (or equivalent for
    other distros):

    On CentOS and openSuSE:

<!-- -->

1.  \$ yum install -y epel-release

    \$ yum install -y git gcc gcc-c++ make cmake golang libtool scons
    boost-devel

    \$ yum install -y libuuid-devel openssl-devel libevent-devel
    libtool-ltdl-devel

    \$ yum install -y librdmacm-devel libcmocka libcmocka-devel
    readline-devel

    \$ yum install -y doxygen pandoc flex patch nasm yasm

    \# Additionally required SPDK packages

    \$ yum install -y CUnit-devel libaio-devel astyle-devel python-pep8
    lcov

    \$ yum install -y python clang-analyzer sg3\_utils libiscsi-devel

    \$ yum install -y libibverbs-devel numactl-devel doxygen mscgen
    graphviz

<!-- -->

1.  On Ubuntu and Debian:

<!-- -->

1.  \$ apt-get install -y git gcc golang make cmake libtool-bin scons
    autoconf

    \$ apt-get install -y libboost-dev uuid-dev libssl-dev libevent-dev
    libltdl-dev

    \$ apt-get install -y librdmacm-dev libcmocka0 libcmocka-dev
    libreadline6-dev

    \$ apt-get install -y curl doxygen pandoc flex patch nasm yasm

    \# Additionally required SPDK packages

    \$ apt-get install -y libibverbs-dev librdmacm-dev libcunit1-dev
    graphviz

    \$ apt-get install -y libaio-dev sg3-utils libiscsi-dev doxygen
    mscgen

<!-- -->

1.  Moreover, please make sure all the auto tools listed below are at
    the appropriate versions.

-   m4 (GNU M4) 1.4.16

-   flex 2.5.37

-   autoconf (GNU Autoconf) 2.69

-   automake (GNU automake) 1.13.4

-   libtool (GNU libtool) 2.4.2

### Building DAOS & Dependencies

1.  If all the software dependencies listed previously are already
    satisfied, then just type the following command in the top source
    directory to build the DAOS stack:

<!-- -->

1.  \$ scons install

<!-- -->

1.  Otherwise, the missing dependencies can be built automatically by
    invoking scons with the following parameters:

<!-- -->

1.  \$ scons --build-deps=yes USE\_INSTALLED=all install

<!-- -->

1.  By default, DAOS and its dependencies are installed under
    \${daospath}/install. The installation path can be modified by
    adding the PREFIX= option to the above command line (e.g.
    PREFIX=/usr/local).

### Environment setup

1.  Once built, the environment must be modified to search for binaries
    and header files in the installation path. This step is not required
    if standard locations (e.g. /bin, /sbin, /usr/lib, ...) are used.

<!-- -->

1.  CPATH=\${daospath}/install/include/:\$CPATH

    PATH=\${daospath}/install/bin/:\${daospath}/install/sbin:\$PATH

    export CPATH PATH

<!-- -->

1.  If using bash, PATH can be setup for you after a build by sourcing
    the script scons\_local/utils/setup\_local.sh from the daos root.
    This script utilizes a file generated by the build to determine the
    location of daos and its dependencies.

    If required, \${daospath}/install must be replaced with the
    alternative path specified through PREFIX. The network type to use
    as well the debug log location can be selected as follows:

<!-- -->

1.  export CRT\_PHY\_ADDR\_STR="ofi+sockets",

    OFI\_INTERFACE=eth0, where eth0 is the network device you want to
    use.

<!-- -->

1.  for infiniband you could use ib0 or whichever else pointing to IB
    device.

    Running DAOS

    DAOS uses orterun(1) for scalable process launch. The list of
    storage nodes can be specified in an host file (referred as
    \${hostfile}). The DAOS server and the application can be started
    seperately, but must share an URI file (referred as \${urifile}) to
    connect to each other. The \${urifile} is generated by orterun using
    (--report-uri filename) at server and used at application with
    (--ompi-server file:filename). In addition, the DAOS server must be
    started with the --enable-recovery option to support server failure.
    See the orterun(1) man page for additional options.

Starting the DAOS server
------------------------

1.  On each storage node, the DAOS server will use a storage path
    (specified by --storage or -s) that must be a directory in a tmpfs
    filesystem, for the time being. If not specified, it is assumed to
    be /mnt/daos. To configure the storage:

<!-- -->

1.  \$ mount -t tmpfs -o size=&lt;bytes&gt; tmpfs /mnt/daos

<!-- -->

1.  To start the DAOS server, run:

<!-- -->

1.  \$ orterun -np &lt;num\_servers&gt; --hostfile \${hostfile}
    --enable-recovery --report-uri \${urifile} daos\_server

<!-- -->

1.  By default, the DAOS server will use all the cores available on the
    storage server. You can limit the number of execution streams with
    the -c \#cores\_to\_use option. Hostfile used here is the same as
    the ones used by Open MPI. See
    (https://www.open-mpi.org/faq/?category=running\#mpirun-hostfile)
    for additional details.

Creating/destroy a DAOS pool
----------------------------

1.  A DAOS pool can be created and destroyed through the DAOS management
    API (see daos\_mgmt.h). We also provide an utility called dmg to
    manage storage pools from the command line.

    To create a pool:

<!-- -->

1.  \$ orterun --ompi-server file:\${urifile} dmg create --size=xxG

<!-- -->

1.  This creates a pool distributed across the DAOS servers with a
    target size on each server of xxGB. The UUID allocated to the newly
    created pool is printed to stdout (referred as \${pooluuid}).

    To destroy a pool:

<!-- -->

1.  \$ orterun --ompi-server file:\${urifile} dmg destroy
    --pool=\${pooluuid}

Building applications or I/O middleware against the DAOS library
----------------------------------------------------------------

1.  Include the daos.h header file in your program and link with -Ldaos.
    Examples are available under src/tests.

Running applications
--------------------

1.  \$ orterun -np &lt;num\_clients&gt; --hostfile \${hostfile\_cli}
    --ompi-server file:\$urifile \${application} eg., ./daos\_test

DAOS for Development
--------------------

1.  For development, it is recommended to build and install each
    dependency in a unique subdirectory. The DAOS build system supports
    this through the TARGET\_PREFIX variable. Once the submodules have
    been initialized and updated, run the following:

<!-- -->

1.  \$ scons PREFIX=\$(daos\_prefix\_path}
    TARGET\_PREFIX=\${daos\_prefix\_path}/opt install --build-deps=yes

<!-- -->

1.  Installing the components into seperate directories allow to upgrade
    the components individually replacing --build-deps=yes with
    --update-prereq={component\_name}. This requires change to the
    environment configuration from before. For automated environment
    setup, source scons\_local/utils/setup\_local.sh.

<!-- -->

1.  ARGOBOTS=\${daos\_prefix\_path}/opt/argobots

    CART=\${daos\_prefix\_path}/opt/cart

    HWLOC=\${daos\_prefix\_path}/opt/hwloc

    MERCURY=\${daos\_prefix\_path}/opt/mercury

    PMDK=\${daos\_prefix\_path}/opt/pmdk

    OMPI=\${daos\_prefix\_path}/opt/ompi

    OPA=\${daos\_prefix\_path}/opt/openpa

    PMIX=\${daos\_prefix\_path}/opt/pmix

    SPDK=\${daos\_prefix\_path}/opt/spdk

    PATH=\$CART/bin/:\$OMPI/bin/:\${daos\_prefix\_path}/bin/:\$PATH

<!-- -->

1.  With this approach DAOS would get built using the prebuilt
    dependencies in \${daos\_prefix\_path}/opt and required options are
    saved for future compilations. So, after the first time, during
    development, a mere "scons" and "scons install" would suffice for
    compiling changes to daos source code.

    If you wish to compile DAOS with clang rather than gcc, set
    COMPILER=clang on the scons command line. This option is also saved
    for future compilations.

DAOS Components:
----------------

Deploying DAOS
==============

1.  

Configuring DAOS
----------------

### Introduction

#### Configuration file

1.  

### Tuning DAOS

### Command Line Interface

1.  

Debugging
---------

1.  DAOS uses the debug system defined in
    [CaRT](https://github.com/daos-stack/cart) but more specifically the
    GURT library. Log files for both client and server are written to
    "/tmp/daos.log" unless otherwise set by D\_LOG\_FILE.

### Registered Subsystems/Facilities

1.  The debug logging system includes a series of subsystems or
    facilities which define groups for related log messages (defined per
    source file). There are common facilities which are defined in GURT,
    as well as other facilities that can be defined on a per-project
    basis (such as those for CaRT and DAOS). DD\_SUBSYS can be used to
    set which subsystems to enable logging. By default all subsystems
    are enabled ("DD\_SUBSYS=all").

-   DAOS Facilities:\
    common, tree, vos, client, server, rdb, pool, container, object,
    placement, rebuild, tier, mgmt, eio, tests

-   Common Facilities (GURT):\
    MISC, MEM

-   CaRT Facilities:\
    RPC, BULK, CORPC, GRP, LM, HG, PMIX, ST, IV

### Priority Logging

1.  All macros that output logs have a priority level, shown in
    descending order below.

-   D\_FATAL(fmt, ...) FATAL

-   D\_CRIT(fmt, ...) CRIT

-   D\_ERROR(fmt, ...) ERR

-   D\_WARN(fmt, ...) WARN

-   D\_NOTE(fmt, ...) NOTE

-   D\_INFO(fmt, ...) INFO

-   D\_DEBUG(mask, fmt, ...) DEBUG

1.  The priority level that outputs to stderr is set with DD\_STDERR. By
    default in DAOS (specific to the project), this is set to CRIT
    ("DD\_STDERR=CRIT") meaning that all CRIT and more severe log
    messages will dump to stderr. This, however, is separate from the
    priority of logging to "/tmp/daos.log". The priority level of
    logging can be set with D\_LOG\_MASK, which by default is set to
    INFO ("D\_LOG\_MASK=INFO"), which will result in all messages
    excluding DEBUG messages being logged. D\_LOG\_MASK can also be used
    to specify the level of logging on a per-subsystem basis as well
    ("D\_LOG\_MASK=DEBUG,MEM=ERR").

### Debug Masks/Streams:

1.  DEBUG messages account for a majority of the log messages, and
    finer-granularity might be desired. Mask bits are set as the first
    argument passed in D\_DEBUG(mask, ...). To accomplish this, DD\_MASK
    can be set to enable different debug streams. Similar to facilities,
    there are common debug streams defined in GURT, as well as other
    streams that can defined on a per-project basis (CaRT and DAOS). All
    debug streams are enabled by default ("DD\_MASK=all").

-   DAOS Debug Masks:

    -   md = metadata operations

    -   pl = placement operations

    -   mgmt = pool management

    -   epc = epoch system

    -   df = durable format

    -   rebuild = rebuild process

    -   daos\_default = (group mask) io, md, pl, and rebuild operations

-   Common Debug Masks (GURT):

    -   any = generic messages, no classification

    -   trace = function trace, tree/hash/lru operations

    -   mem = memory operations

    -   net = network operations

    -   io = object I/Otest = test programs

### Common Use Cases

-   Generic setup for all messages (default settings)

1.  \$ D\_LOG\_MASK=DEBUG

    \$ DD\_SUBSYS=all

    \$ DD\_MASK=all

-   Disable all logs for performance tuning

1.  \$ D\_LOG\_MASK=ERR -&gt; will only log error messages from all
    facilities

    \$ D\_LOG\_MASK=FATAL -&gt; will only log system fatal messages

-   Disable a noisy debug logging subsystem

1.  \$ D\_LOG\_MASK=DEBUG,MEM=ERR -&gt; disables MEM facility by
    restricting all

    logs from that facility to ERROR or higher priority only

-   Enable a subset of facilities of interest

1.  \$ DD\_SUBSYS=rpc,tests

    \$ D\_LOG\_MASK=DEBUG -&gt; required to see logs for RPC and TESTS
    less severe

    than INFO (majority of log messages)

-   Fine-tune the debug messages by setting a debug mask

1.  \$ D\_LOG\_MASK=DEBUG

    \$ DD\_MASK=mgmt -&gt; only logs DEBUG messages related to pool
    management

<!-- -->

1.  Refer to the DAOS Environment Variables documentation (Appendix B)
    for more information about the debug system environment.

Monitoring DAOS 
================

DAOS Fault Handling
===================

1.  

Introduction
------------

DAOS Security Framework
=======================

Usage Examples {#usage-examples .Appendix1}
==============

####### Usage Examples of DAOS

######## 

###### DAOS Environment Variables

1.  This section lists the environment variables used by DAOS. Many of
    them are used for development purposes only and may be removed or
    changed in the future.

    The description of each variable follows the following format:

-   Short description

-   Type

-   The default behavior if not set.

-   A longer description if necessary

1.  This table defines a type:

  ---------------------------------------------------------------------------
  Type          Values
  ------------- -------------------------------------------------------------
  1.  BOOL      1.  0 means false; any other value means true
                

  1.  BOOL2     1.  no means false; any other value means true
                

  1.  BOOL3     1.  set to empty or any value means true; unset means false
                

  1.  INTEGER   1.  Non-negative decimal integer
                

  1.  STRING    1.  String
                
  ---------------------------------------------------------------------------

####### Common environment variables

1.  Environment variables in this section apply to both the server side
    and the client side

<!-- -->

1.  DAOS\_IO\_BYPASS

####### Server environment variables

1.  Environment variables in this section only apply to the server side.

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Variable                     Description
  ---------------------------- ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  1.  VOS\_CHECKSUM            1.  Checksum algorithm used by VOS. STRING. Default to disabling checksums. The following checksum algorithms are supported: crc64 and crc32.
                               

  1.  VOS\_MEM\_CLASS          1.  Memory class used by VOS. STRING. Default to persistent memory. If the value is set to DRAM, all data is stored in volatile memory; otherwise, all data is stored in persistent memory.
                               

  1.  RDB\_ELECTION\_TIMEOUT   1.  Raft election timeout used by RDBs in milliseconds. INTEGER. Default to 7000 ms.
                               

  1.  RDB\_REQUEST\_TIMEOUT    1.  Raft request timeout used by RDBs in milliseconds. INTEGER. Default to 3000 ms.
                               

  1.  DAOS\_REBUILD            1.  Determines whether to start rebuilds when excluding targets. BOOL2. Default to true.
                               

  1.  DAOS\_MD\_CAP            1.  Size of a metadata pmem pool/file in MBs. INTEGER. Default to 128 MB.
                               

  1.  DAOS\_START\_POOL\_SVC   1.  Determines whether to start existing pool services when starting a daos\_server. BOOL. Default to true.
                               

  1.  DAOS\_IMPLICIT\_PURGE    1.  Whether to aggregate unreferenced epochs. BOOL. Default to false.
                               

  1.  DAOS\_PURGE\_CREDITS     1.  The number of credits for probing object trees when aggregating unreferenced epochs. INTEGER. Default to 1000.
                               
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

####### Client

1.  Environment variables in this section only apply to the client side.

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Variable                   Description
  -------------------------- -------------------------------------------------------------------------------------------------------------------------------------------
  1.  DAOS\_SINGLETON\_CLI   1.  Determines whether to run in the singleton mode, in which the client does not need to be launched by orterun. BOOL. Default to false.
                             
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------

####### Debug System (Client & Server)

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Variable           Description
  ------------------ ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  1.  D\_LOG\_FILE   1.  DAOS debug logs (both server and client) are written to /tmp/daos.log by default. The debug location can be modified by setting this environment variable ("D\_LOG\_FILE=/tmp/daos\_server").
                     

  1.  DD\_SUBSYS     1.  Used to specify which subsystems to enable. DD\_SUBSYS can be set to individual subsystems for finer-grained debugging ("DD\_SUBSYS=vos"), multiple facilities ("DD\_SUBSYS=eio,mgmt,misc,mem"), or all facilities ("DD\_SUBSYS=all") which is also the default setting. If a facility is not enabled, then only ERR messages or more severe messages will print.
                     

  1.  DD\_STDERR     1.  Used to specify the priority level to output to stderr. Options in decreasing priority level order: FATAL, CRIT, ERR, WARN, NOTE, INFO, DEBUG. By default, all CRIT and more severe DAOS messages will log to stderr ("DD\_STDERR=CRIT"), and the default for CaRT/GURT is FATAL.
                     

  1.  D\_LOG\_MASK   1.  Used to specify what type/level of logging will be present for either all of the registered subsystems or a select few. Options in decreasing priority level order: FATAL, CRIT, ERR, WARN, NOTE, INFO, DEBUG. DEBUG option is used to enable all logging (debug messages as well as all higher priority level messages). Note that if D\_LOG\_MASK is not set, it will default to logging all messages excluding debug ("D\_LOG\_MASK=INFO"). EX: "D\_LOG\_MASK=DEBUG" This will set the logging level for all facilities to DEBUG, meaning that all debug messages, as well as higher priority messages will be logged (INFO, NOTE, WARN, ERR, CRIT, FATAL) EX: "D\_LOG\_MASK=DEBUG,MEM=ERR,RPC=ERR" This will set the logging level to DEBUG for all facilities except MEM & RPC (which will now only log ERR and higher priority level messages, skipping all DEBUG, INFO, NOTE & WARN messages)
                     

  1.  DD\_MASK       1.  Used to enable different debug streams for finer-grained debug messages, essentially allowing the user to specify an area of interest to debug (possibly involving many different subsystems) as opposed to parsing through many lines of generic DEBUG messages. All debug streams will be enabled by default ("DD\_MASK=all"). Single debug masks can be set ("DD\_MASK=trace") or multiple masks ("DD\_MASK=trace,test,mgmt"). Note that since these debug streams are strictly related to the debug log messages, DD\_LOG\_MASK must be set to DEBUG. Priority messages higher than DEBUG will still be logged for all facilities unless otherwise specified by D\_LOG\_MASK (not affected by enabling debug masks).
                     
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[^1]: https://github.com/daos-stack/cart

[^2]: https://mercury-hpc.github.io/

[^3]: https://ofiwg.github.io/libfabric/

[^4]: https://github.com/pmix/master

[^5]: https://github.com/pmem/pmdk.git

[^6]: http://spdk.io/

[^7]: https://github.com/01org/isa-l

[^8]: https://github.com/pmodels/argobots
