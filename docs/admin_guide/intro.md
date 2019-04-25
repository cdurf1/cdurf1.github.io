Introduction
============

The Distributed Asynchronous Object Storage (DAOS) is an open-source
object store designed from the ground up for massively distributed Non
Volatile Memory (NVM). DAOS takes advantage of next-generation NVM
technology, like Storage Class Memory (SCM) and NVM express (NVMe),
while presenting a key-value storage interface on top of commodity
hardware that provides features, such as, transactional non-blocking
I/O, advanced data protection with self-healing, end-to-end data
integrity, fine-grained data control, and elastic storage, to optimize
performance and cost.

This initial administration guide version is associated with DAOS v0.4.

Terms used in this Document 
----------------------------

The following terms and abbreviations are used in this document.

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Term            Definition
  --------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  1.  ACLs        1.  Access Control Lists
                  

  1.  CaRT        1.  Collective and RPC Transport (CaRT) library. A software library built on top of the Mercury Function Shipping library to support distributed communication functionality.
                  

  1.  CGO         1.  Go tools that enable creation of Go packages that call C code
                  

  1.  CN          1.  Compute Node
                  

  1.  CPU         1.  Central Processing Unit
                  

  1.  COTS        1.  Commercial off-the-shelf
                  

  1.  Daemon      1.  A process offering system-level resources.
                  

  1.  DCPM        1.  Intel Optane DC Persistent Memory
                  

  1.  DPDK        1.  Data Plane Development Kit
                  

  1.  dRPC        1.  DAOS Remote Procedure Call
                  

  1.  BIO         1.  Blob I/O
                  

  1.  gRPC        1.  gRPC[^1] Remote Procedure Calls
                  

  1.  GURT        1.  A common library of Gurt Useful Routines and Types provided with CaRT.
                  

  1.  HLD         1.  High-Level Design
                  

  1.  I/O         1.  Input/Output
                  

  1.  ISA-L       1.  Intel® Intelligent Storage Acceleration Library
                  

  1.  libfabric   1.  A user-space library that exports the Open Fabrics Interface
                  

  1.  Mercury     1.  A user-space RPC library that can use libfabrics as a transport
                  

  1.  NVM         1.  Non-Volatile Memory
                  

  1.  NVMe        1.  Non-Volatile Memory express
                  

  1.  OFI         1.  OpenFabrics Interfaces
                  

  1.  OS          1.  Operating System
                  

  1.  PMDK        1.  Persistent Memory Development Kit
                  

  1.  PMIx        1.  Process Management Interface for Exascale
                  

  1.  Raft        1.  Raft is a consensus algorithm used to distribute state transitions among DAOS server nodes.
                  

  1.  RDB         1.  Replicated Database, containing pool metadata and maintained across DAOS servers using the Raft algorithm.
                  

  1.  RPC         1.  Remote Procedure Call.
                  

  1.  SPDK        1.  Storage Performance Development Kit
                  

  1.  SWIM        1.  Scalable Weakly-consistent Infection-style process group Membership protocol
                  

  1.  UPI         1.  Intel® Ultra Path Interconnect
                  

  1.  UUID        1.  Universal Unique Identifier
                  

  1.  VOS         1.  Versioned Object Store
                  
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Additional Documentation
------------------------

Refer to the following documentation for architecture and description:

  ----------------------------------------------------------------------------------------------------
  Document                 Location
  ------------------------ ---------------------------------------------------------------------------
  1.  DAOS Internals       1.  https://github.com/daos-stack/daos/blob/master/src/README.md
                           

  1.  DAOS Storage Model   1.  <https://github.com/daos-stack/daos/blob/master/doc/storage_model.md>
                           

  1.  Community Roadmap    1.  https://wiki.hpdd.intel.com/display/DC/Roadmap
                           
  ----------------------------------------------------------------------------------------------------

[^1]: <https://grpc.io/>
