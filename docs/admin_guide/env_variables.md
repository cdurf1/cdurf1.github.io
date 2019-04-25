DAOS Environment Variables {#daos-environment-variables .Appendix1}
==========================

This section lists the environment variables used by DAOS. Many of them
are used for development purposes only and may be removed or changed in
the future.

The description of each variable follows the following format:

-   Short description

-   Type

-   The default behavior if not set.

-   A longer description if necessary

This table defines a type:

  ---------------------------------------------------------------------------
  Type          Values
  ------------- -------------------------------------------------------------
  1.  BOOL      1.  0 means false; any other value means true
                

  1.  BOOL2     1.  no means false; any other value means true
                

  1.  BOOL3     1.  set to empty or any value means true; unset means false
                

  1.  INTEGER   1.  Non-negative decimal integer
                

  1.  STRING    1.  String
                
  ---------------------------------------------------------------------------

Common environment variables {#common-environment-variables .Appendix2}
============================

Environment variables in this section apply to both the server side and
the client side

1.  DAOS\_IO\_BYPASS

Server environment variables {#server-environment-variables .Appendix2}
============================

Environment variables in this section only apply to the server side.

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

Client {#client .Appendix2}
======

Environment variables in this section only apply to the client side.

  Variable               Description
  ---------------------- ---------------------------------------------------------------------------------------------------------------------------------------
  DAOS\_SINGLETON\_CLI   Determines whether to run in the singleton mode, in which the client does not need to be launched by orterun. BOOL. Default to false.

Debug System (Client & Server) {#debug-system-client-server .Appendix2}
==============================

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Variable           Description
  ------------------ ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  1.  D\_LOG\_FILE   1.  DAOS debug logs (both server and client) are written to /tmp/daos.log by default. The debug location can be modified by setting this environment variable ("D\_LOG\_FILE=/tmp/daos\_server").
                     

  1.  DD\_SUBSYS     1.  Used to specify which subsystems to enable. DD\_SUBSYS can be set to individual subsystems for finer-grained debugging ("DD\_SUBSYS=vos"), multiple facilities ("DD\_SUBSYS=eio,mgmt,misc,mem"), or all facilities ("DD\_SUBSYS=all") which is also the default setting. If a facility is not enabled, then only ERR messages or more severe messages will print.
                     

  1.  DD\_STDERR     1.  Used to specify the priority level to output to stderr. Options in decreasing priority level order: FATAL, CRIT, ERR, WARN, NOTE, INFO, DEBUG. By default, all CRIT and more severe DAOS messages will log to stderr ("DD\_STDERR=CRIT"), and the default for CaRT/GURT is FATAL.
                     

  1.  D\_LOG\_MASK   1.  Used to specify what type/level of logging will be present for either all of the registered subsystems or a select few. Options in decreasing priority level order: FATAL, CRIT, ERR, WARN, NOTE, INFO, DEBUG. DEBUG option is used to enable all logging (debug messages as well as all higher priority level messages). Note that if D\_LOG\_MASK is not set, it will default to logging all messages excluding debug ("D\_LOG\_MASK=INFO"). EX: "D\_LOG\_MASK=DEBUG" This will set the logging level for all facilities to DEBUG, meaning that all debug messages, as well as higher priority messages will be logged (INFO, NOTE, WARN, ERR, CRIT, FATAL) EX: "D\_LOG\_MASK=DEBUG,MEM=ERR,RPC=ERR" This will set the logging level to DEBUG for all facilities except MEM & RPC (which will now only log ERR and higher priority level messages, skipping all DEBUG, INFO, NOTE & WARN messages)
                     

  1.  DD\_MASK       1.  Used to enable different debug streams for finer-grained debug messages, essentially allowing the user to specify an area of interest to debug (possibly involving many different subsystems) as opposed to parsing through many lines of generic DEBUG messages. All debug streams will be enabled by default ("DD\_MASK=all"). Single debug masks can be set ("DD\_MASK=trace") or multiple masks ("DD\_MASK=trace,test,mgmt"). Note that since these debug streams are strictly related to the debug log messages, DD\_LOG\_MASK must be set to DEBUG. Priority messages higher than DEBUG will still be logged for all facilities unless otherwise specified by D\_LOG\_MASK (not affected by enabling debug masks).
                     
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


