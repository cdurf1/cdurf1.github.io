[]{#_Toc46119879 .anchor}

[]{#_Toc430433419 .anchor}Contents

[MS6 Declustered RAIDZ 1](#_Toc430433417)

[Solution Architecture 1](#_Toc430433418)

[Contents 3](#_Toc430433419)

[Revision History 5](#_Toc430433420)

[1 Milestone Overview – MS6 Declustered RAIDZ Solution Architecture
6](#milestone-overview-ms6-declustered-raidz-solution-architecture)

[1.1 Description 6](#description)

[1.2 Terms 7](#terms)

[1.3 Introduction 7](#introduction)

[2 Requirements Description 9](#requirements-description)

[3 Use Cases 12](#use-cases)

[4 Solution Proposal 14](#solution-proposal)

[4.1 Declustered RAIDZ 14](#declustered-raidz)

[4.2 Distributed spare space 15](#distributed-spare-space)

[4.3 The ZFS Event Daemon 16](#the-zfs-event-daemon)

[4.4 The resilver algorithm 16](#the-resilver-algorithm)

[5 Acceptance Criteria 19](#acceptance-criteria)

[6 Testing 20](#testing)

[6.1 Example workloads 20](#example-workloads)

[6.2 Unit Tests 20](#unit-tests)

[6.3 Integration Tests 20](#integration-tests)

[6.4 System Tests 21](#system-tests)

[6.5 Stability Tests 21](#stability-tests)

***\
***

***Figures***

[Figure 2‑1 – Severity Levels for Resilvering 9](#_Toc430433439)

[Figure 2‑2 – Mirror Severity Levels 10](#_Toc430433440)

[Figure 4‑0‑1: Declustered 2-way mirror over 5 drives
15](#_Toc430433441)

[Figure 4‑2: ZFS resilver performance 17](#_Toc430433442)

[]{#_Toc430433420 .anchor}Revision History

  ---------- ----------------------------------------- ------------ -------------
  Revision   Description                               Date         Author
  0.1        First draft                               2015-06-23   Isaac Huang
  1.1        Revisions following Argonne review        2015-08-21   Isaac Huang
  1.2        Final revision following Argonne review   2015-09-19   Isaac Huang
  ---------- ----------------------------------------- ------------ -------------

[[[[]{#_Ref46905290 .anchor}]{#_Ref46905265 .anchor}]{#_Toc46134343
.anchor}]{#_Toc82841952 .anchor}**Author: Isaac Huang**

**Contributors: Eric Barton, John Salinas, Don Brady, Andreas Dilger,
John Carrier**

Milestone Overview – MS6 Declustered RAIDZ Solution Architecture
================================================================

Description
-----------

The following solution architecture applies to the CORAL project within
the CFS Aggregate Scalable Storage Unit Performance as defined by MS6 in
the NRE Contract.

MS6 Description:

The Subcontractor shall develop declustered parity and distributed hot
sparing for RAIDZ. The Subcontractor shall implement this by providing
new pseudo-random data and parity mapping schemes for RAIDZ, RAIDZ2 and
RAIDZ3 storage pools. The Subcontractor shall extend the ZFS monitoring
and administration utilities to manage these new pools and provide
policies to trigger automatic recovery and throttle performance impact
of recovery and rebalancing on application I/O. I/O load during recovery
will therefore be distributed to all drives and so can be throttled back
to minimize impact on application I/O while keeping recovery time within
acceptable limits.

The Subcontractor shall provide an analysis of the I/O performance
obtained from RAIDZ with declustered parity using the same SSU (or
similar) to that used on the Argonne Aurora system as well as an
analysis of the impact of RAIDZ on MTTDL for scenarios with multiple
disk failures, as compared to original proposed solution. The
Subcontractor shall provide an analysis of the rebuild rate of the
parity declustered RAIDZ implementation under both idle I/O load and
heavy I/O load.

The Subcontractor shall provide all source changes to the ZFS on Linux
upstream source repository and work with the ZFS on Linux community to
ensure the code lands successfully.

The Subcontractor shall deliver a Scope Statement that documents the
goals to be satisfied and specifies in-scope and out-of-scope elements
of work, assumptions and constraints and the key deliverables and
development milestones. Following completion of the Scope Statement, the
Subcontractor shall deliver a Solution Architecture that documents the
detailed solution requirements and outlines the solution broken down by
subsystem.

Terms
-----

The following terms are used throughout the document:

-   RAIDZ: a generic term to refer to ZFS RAIDZ1, RAIDZ2, RAIDZ3, and
    mirror, when there is no need to distinguish between them. Otherwise
    the more specific terms are used. The generic term RAID is also used
    when there is no need to distinguish between traditional RAID and
    RAIDZ.

-   Redundancy group: the set of drives and blocks in a RAIDZ that store
    the data and redundancy of a ZFS block, e.g. the redundancy group of
    a 16M block in a 8+2 RAIDZ2 consists of 10 drives each having 2M of
    data or parity.

-   Spare blocks/space: spare space distributed among all disks in a
    RAIDZ group.

-   VDEV (virtual device): A "virtual device" describes a single device
    or a collection of devices organized according to certain
    performance and fault characteristics. ZFS supports the following
    VDEV types: disk, file, mirror, RAIDZ, spare, log, and cache.

-   Resilver: the process of reconstructing data/parity on a failed disk
    to spare blocks in a RAIDZ group.

-   Scrub: the process of examining all in-use data blocks in a pool
    to verify that it checksums correctly. For VDEVs with redundancy
    (mirror or RAIDZ), ZFS automatically repairs any damage discovered.

-   Spare rebalance: the process of copying reconstructed data/parity
    from previous spare blocks to a replacement disk so that distributed
    spare blocks become available again.

-   COW (copy on write) : ZFS never writes in place.

-   TXG (Transaction group) : a group of smaller transactions which are
    committed to stable storage together to amortize commit overhead.

-   ZED (ZFS Event Daemon): monitors events generated by the ZFS
    kernel module. When a zevent (ZFS Event) is posted, ZED will run any
    ZEDLETs (ZFS Event Daemon Linkage for Executable Tasks) that have
    been enabled for the corresponding zevent class.

Introduction
------------

In large-scale storage configurations needed to meet the IO requirements
of future HPC systems, disk failures are inevitable and must be viewed
as a normal incident rather than an exceptional event.

When disk failures happen, it is important that RAID parity
reconstruction completes as quickly as possible. Shorter rebuild times
significantly reduce exposure to multiple cascading disk failures, which
would lead to data loss. It is also important that RAID rebuilding
process minimally impacts the application IO. A drop in performance
would cause applications to run longer or even fail to complete in their
allotted time window.

The process of rebuilding a “traditional” RAID array, when replacing a
failing/failed disk by a new one, consists of reading all the data on
all the surviving disks in the array, reconstructing the original data
of the replaced disk, and writing it to the replacement disk. After this
process is complete, the array is restored to its original full
redundancy. In ZFS, there is a similar and equivalent process, called
resilvering, that rebuilds the redundant data on the replacement disk,
since volume management is a built-in part of ZFS. This process starts
by traversing the ZFS block pointer tree to discover all the blocks of
the ZFS pool that were located on the failed disk. Upon reaching one of
these blocks, it is read, reconstructed if necessary from the
redundant/parity information, its checksum is verified and the missing
data or parity from the failed disk is written to the new disk.

The speed of rebuilding or resilvering is bounded by the write
throughput of a single replacement disk, thus the total time would grow
at least linearly (often much worse) with drive capacity. As drive
capacity continues to grow with little increase in throughput, rebuild
time can increase significantly. For example, it would take about 27
hours to fully rebuild a 10TB drive at 100MB/s. Since idle time is rare,
a drive failure and subsequent rebuild process can significantly affect
a system’s performance.

Parity declustered RAIDZ distributes data, parity, and spare
capacity across all disks in a pool so that all of them participate in
the rebuild process nearly evenly. Since the pool is many times larger
than the redundancy group size, aggregate reconstructing read
performance is correspondingly increased.  And since lost data is
reconstructed across all drives, the bottleneck of having to write a
single replacement drive is eliminated.

Requirements Description
========================

The High Level Design will discuss CORAL specific configurations, which
will depend on several design choices not detailed here in the Solutions
Architecture, such as, for example, the segregation of small metadata
blocks to reduce fragmentation.

1.  **Automatic replace**: Once drive failure is detected, ZED should
    automatically replace failed drive with distributed spare space
    (rather than using a dedicated spare drive, if any), and start the
    resilver process to rebuild redundancy. New writes that would have
    otherwise ended up on the failed drive are also directed to the
    distributed spare space.

    1.  Resilver restores redundancy. Once resilver is complete, the
        RAIDZ has the same level of protection against drive failures,
        despite that some drive(s) has failed already. For example, a
        declustered RAIDZ1 which has lost one drive and completed
        resilvering can survive another drive failure without losing
        data. To achieve this, in choosing a spare block to use, the
        resilvering algorithm must not use any drive that is already a
        member of the redundancy group for the ZFS block being
        resilvered.

    2.  As soon as the failed drive is replaced by an administrator and
        the resilver completes, the rebalance process can start to
        reclaim distributed spare space.

2.  **Dynamic IO throttling**: During resilvering or rebalancing, a
    tunable percentage of total throughput should be guaranteed to be
    available to application IO. The resilver/rebalance IO should be
    throttled to guarantee application IO expectations while making full
    use of IO when there is a lack of application IO. Note that:

    1.  During resilvering, application read throughput may be lower if
        the data has to be reconstructed, i.e. it has not been
        reconstructed yet.

    2.  Multiple severity levels for resilvering. For example, when
        there is not enough redundancy to survive additional drive
        failure without losing data, the RAIDZ is said to be in critical
        state, e.g. a RAIDZ2 that has 2 failed drives. Resilvering in
        the critical state should have highest IO priority. The
        following three levels are defined: 

  **\# of failed drives **   **RAIDZ1**   **RAIDZ2**   **RAIDZ3**
  -------------------------- ------------ ------------ ------------
  **1 **                     critical     medium       Normal
  **2**                      NA           critical     Medium
  **3**                      NA           NA           Critical

[]{#_Toc430433439 .anchor}Figure ‑ – Severity Levels for Resilvering

1.  ZFS supports arbitrary N-way mirrors,  so severity levels for
    mirrors are defined in terms of the number of healthy copies (i.e. N
    minus \# failed drives):

  **\# of healthy copies**
  -------------------------- -------- -------------
  **1**                      **2**    **&gt;= 3**
  critical                   medium   normal

[]{#_Toc430433440 .anchor}Figure ‑ – Mirror Severity Levels

1.   

    1.  A separate tunable controls the percentage of IO available to
        resilvering for each severity level.

    2.  For RAIDZ1, all resilvering is critical.

    3.  Priority escalation. If another drive fails during resilvering,
        the resilvering should proceed (or restart) with higher IO
        priority.

    4.  Rebalance has lower priority than *normal* severity
        of resilvering.

    5.  Scrub has the lowest priority and cannot be started if rebalance
        or resilvering is in progress.

<!-- -->

1.  **Declustered block layout**: Data, parity, and spare blocks should
    be distributed across all disks in a declustered RAIDZ, in a way
    such that after a drive failure all surviving drives would
    participate in the resilver process nearly evenly - i.e. all drives
    will be read from and written to nearly evenly during any interval
    of time in the resilver process.

    1.  It is important that all drives share the IO load in a balanced
        way during any interval, rather than during the whole resilver
        process. In other words, a drive or a subset of drives cannot
        become the hot spot during any interval in the whole process.

    2.  All RAIDZ levels are supported: mirror, RAIDZ1, RAIDZ2,
        and RAIDZ3.

    3.  Declustered RAIDZ provides equivalent level of
        protection against data loss. For example, declustered RAIDZ1
        can survive 1 drive failure without losing data, just like
        conventional RAIDZ1.

2.  **Dirty Time Logging (DTL) support for recovery from transient drive
    outages**: If a drive is offline temporarily, e.g. due to a
    transient error, it should be resilvered to catch up with TXGs it
    has missed. If resilvering to spare space has been started already,
    it will be cancelled.

3.  **Spare space rebalance**: When the administrator has replaced a
    failed drive, data and parity blocks should be moved back to the
    replacement drive in order to restore distributed spare space. Since
    it is a manual operation to physically replace a drive, the spare
    rebalance process should be started by the administrator.

    1.  Spare space rebalance cannot start if resilver is still in
        progress, because the priority is to restore redundancy and it
        is much faster to do so by resilvering to distributed spare
        space.

    2.  Resilver preempts rebalance. If a drive failure happens while
        rebalance is in progress *and* there is spare space available
        for resilvering, the rebalance is canceled and the resilvering
        takes place immediately. It is more urgent to restore redundancy
        than to restore spare space.

> The administrator should be able to tell the system to restart the
> rebalance as soon as an in-progress resilver completes. Running the
> replace command will block until the resilver completes, or ZED can
> start the rebalance after the resilver completes.

1.  **2M IO for 16M blocks**: For large 16M blocks, declustered RAIDZ
    should make sure each disk gets a contiguous chunk of data/parity of
    2M, while at the same time meeting the declustered block layout
    requirement.

2.  **Compatibility with all DMU consumers**: Declustered RAIDZ should
    work transparently with all DMU users, i.e. ZPL, ZVOL, and Lustre
    osd-zfs.

Use Cases
=========

1.  A drive in a declustered RAIDZ2 (configured with spare space of 2
    drive capacity) begins to return IO errors, ZED detects the errors
    and offlines the drive, which puts the RAIDZ2 group in degraded
    state. Then it immediately begins reconstructing data/parity
    originally on the offline drive to the spare space in the
    declustered RAIDZ2. New writes are also directed to the spare space.
    One drive worth of the spare space becomes unavailable once
    resilvering starts. Once resilvering completes the redundancy is
    restored, and the RAIDZ2 should return to healthy state.

    1.  The drive quickly recovers from a transient error and becomes
        online again. ZED notices it, and stops the resilvering to spare
        space. Then ZED starts an incremental resilver to bring the
        drive up to date by reconstructing only the missed TXGs, from
        information recorded on the DTL. 

2.  The administrator has configured 80% of storage throughput to be
    reserved for application IO. During resilvering, it will be
    demonstrated that: a) application IO can get sustained throughput of
    as much as 80% of backend storage throughput; b) Once application IO
    activity decreases or stops, the resilvering process quickly begins
    to make full use of the available throughput.

3.  During the resilver process, all IO generated by the resilvering is
    nearly balanced among all the surviving drives.

4.  A 2nd drive fails during the resilver. The resilver proceeds with
    higher priority as critical resilver. In this case, the
    administrator has configured 50% of throughput to be available for
    critical resilver.

    1.  Once the 1st failed drive has been resilvered, the resilver
        process continues to resilver the 2nd failed drive under normal
        resilver IO priority (80% in this example).

5.  Before the resilver of the 1st failed drive completes, the
    declustered RAIDZ2 can satisfy any read request by reconstructing
    lost data blocks, i.e. it has not lost any data from the double
    drive failure.

6.  Full redundancy has been restored once the resilver to spare space
    completes. The RAIDZ2 can survive two additional drive failures
    without losing data.

7.  The administrator locates and replaces the failed drive, and runs a
    command to begin the spare rebalance. The RAIDZ stays in healthy
    state as full redundancy has been restored already by resilvering.
    Once the rebalance completes, the spare space becomes available.
    Note that if the resilver is still in progress, rebalancing will be
    started after the resilvering completes.

8.  The administrator has configured 90% of storage throughput to be
    reserved for application IO. During rebalancing, it will be
    demonstrated that: a) application IO can get sustained throughput of
    as much as 90% of backend storage throughput; b) Once application IO
    activity decreases or stops, the rebalancing process quickly begins
    to make full use of the available throughput.

    1.  Another drive fails during rebalancing, and spare space is still
        available - the RAIDZ2 was configured with spare space of 2
        drive capacity. ZED detects the drive failure, stops the ongoing
        rebalancing and starts resilvering immediately. Once resilvering
        is complete, ZED restarts the rebalancing.

9.  Lustre osd-zfs sets block size to be 16M for large objects. This can
    be shown by using the zdb utility to dump object blocks. Read/write
    on such objects generates IO of 2M on member disks in a declustered
    RAIDZ. It can be demonstrated by gathering stats of IO sizes on the
    drives.

10. A user can create a ZPL filesystem, a ZVOL volume, or a Lustre OST
    on top of a declustered RAIDZ, and use it without any code change or
    semantic difference (from standard RAIDZ):

    1.  POSIX compliance tests pass

    2.  Lustre sanity tests pass

Solution Proposal
=================

Declustered RAIDZ
-----------------

The key is to find a declustered parity scheme that is deterministic,
run-time efficient, and provides a nearly balanced distribution of
rebuild workloads among all surviving drives. It has been shown that
perfectly balanced schemes only exist for a small class of drive
configurations. Moreover, ZFS RAIDZ breaks two assumptions of all
existing research on declustered RAID:

-   Fixed redundancy group size: ZFS redundancy groups are variable
    sizes and correspond to blocks from 512 bytes to 16MB. It is totally
    up to the block allocator (and current fragmentation of the pool)
    where the new blocks are allocated.

-   Sequential drive rebuild: ZFS resilvers blocks as they are found
    during the block tree scan. The IO pattern is often random.

Therefore Intel's goal is to find an approximation which is reasonably
balanced for the following workloads:

-   Random read: Reads always randomized since parallel streams of
    sequential file I/O interleave non-deterministically at the server.

-   Sequential write: Regardless of application write IO pattern
    (sequential or random), due to COW nature and block allocation
    policy ZFS writes are often sequential as seen at the drive level,
    unless the pool is highly fragmented. Note that fragmentation is
    addressed by the Lustre streaming performance improvement project.

-   Random read/write during resilver. Note that there is some effort in
    the OpenZFS community to prototype a sequential resilver algorithm,
    but it is too early to tell whether it will be fully implemented or
    available for this project.

Intel experimented with a prototype which declustered a 2-way mirror
over 5 drives, based on a simple complete balanced block table; and
found the IO during sequential write was reasonably balanced among all 5
drives:

![](media/image2.png){width="6.5in" height="4.875in"}

[]{#_Toc430433441 .anchor}Figure 4‑‑: Declustered 2-way mirror over 5
drives

Intel will investigate known declustered parity schemes, e.g. table
based block designs and pseudo-random permutations. Pseudo-random
permutations can be slower than table based methods, due to the
additional computation overhead, but requires little memory overhead.
For the CORAL project where the disk array configuration is known, a
hybrid approach may be adopted: in-memory block-mapping table-based on
pre-calculated pseudo-random permutations may have a smaller memory
overhead than some table-based block designs, yet be faster than
run-time pseudo-random permutations. The HLD will describe the results
of this investigation and subsequent selection of a preferred scheme.

Distributed spare space
-----------------------

In order to provide an interface similar to the hot spare drives, the
distributed spare space should be represented as a logical VDEV. This
gives users and utilities a familiar interface to manage distributed
spare space, for example:

-   When a user runs zpool status command, the output should list the
    spare space as logical VDEVs and display their status properly, e.g.
    available or in use.

-   ZED can issue a command to replace a failed drive with distributed
    spare space by specifying the corresponding logical VDEV.

However, many places in the code assume a hot spare is one physical
drive:

-   User space ZFS library and tools open the hot spare as a device file
    and writes disk labels to it.

-   ZFS kernel module reads/writes überblocks and labels from/to hot
    spare drives.

Such assumptions must be removed from the code in a clean way. In
addition, currently RAIDZ does not support nested RAID - a member drive
in a RAIDZ group must be a physical drive or file. That must be changed
as well so a logical distributed hot spare can act as a member VDEV of a
declustered RAIDZ VDEV.

The ZFS Event Daemon
--------------------

The ZFS event engine consists of two parts:

-   the kernel part that generates event reports, e.g. checksum
    failures;

-   a user space daemon, called ZED, that polls and reads the event
    reports from the kernel, and take actions on them.

The infrastructure is complete, but there are still some corner cases
where it either does not work or it needs improvement. For example, in
some cases, drive failure is not reported by the kernel, so ZED would
not automatically replace the failed drive. Intel will develop thorough
plans to test ZED with various failures and identify/fix cases where it
fails to work and areas where it still needs improvement.

The only addition Intel will need to make to ZED is to make it aware of
distributed spare space, and prefer distributed spare to replace a
failed drive over a dedicated spare drive. ZED should also understand
that distributed spare space is a resource local to its declustered
RAIDZ group - i.e. distributed spare space can only be used to replace a
failed drive in its parent RAIDZ group, and not any drive in other RAIDZ
groups.

The resilver algorithm
----------------------

The ZFS resilver performance is often limited by meta-data traversal,
i.e. small random IO. The following plots during a resilver showed high
seek counts and poor throughput during the initial meta-data scan (from
0 to 19 second):

![](media/image3.png){width="6.5in" height="4.875in"}

[]{#_Toc430433442 .anchor}Figure 4‑: ZFS resilver performance

The plots also demonstrated that large blocks could improve resilver
efficiency: larger blocks (from 20 to 80 second) yielded better
sustained throughput than small blocks (from 80 to 140 second). This is
because:

-   With larger leaf blocks, there is less metadata to scan.

-   2M IO for 16M blocks would increase sustained resilver throughput.

Currently both meta-data traversal and block rebuild happen in
transaction group syncing context. To avoid blocking upcoming
transactions, the amount of work the resilver algorithm can do in a
syncing transaction group is fairly limited. If meta-data traversal
would still end up a bottleneck, Intel has two *potential* options to
address it:

-   Part of meta-data traversal can be effectively moved out of the
    syncing context by intelligent prefetching.

-   If prefetching does not yield sufficient improvement, the meta-data
    traversal can be completely moved to open context, so that in
    syncing context the resilver process can perform more actual block
    rebuild. This will split the resilver process in two parts:

    -   Meta-data traversal is a read-only process and moved out of
        transaction processing completely. Blocks found that need to be
        resilvered are queued for processing in transaction group
        syncing context. Even the reconstruction of lost data/parity can
        be done here.

    -   In syncing context, queued blocks are resilvered by writing
        reconstructed data/parity to stable storage. As most of the work
        has been moved out of transaction processing, more IOs can be
        issued from the syncing context.

The HLD will describe the results of this investigation and subsequent
selection of a preferred method for improving resilver performance.

Acceptance Criteria
===================

-   All requirements listed in [2. Requirements
    Description](#DeclusteredRAIDZSolutionArchitecture-2.) are
    validated.

-   An agreed set of use cases are demonstrated.

-   Test plans described in section [6.
    Testing](#DeclusteredRAIDZSolutionArchitecture-6.) are fully
    executed and all tests pass.

Testing
=======

Example workloads
-----------------

A formal test plan is being developed in parallel with the HLD. The plan
will use workloads generated from the following methods:

-   CI / automated testing with ztest, ZFS regression suite

-   Long running tests with ztest, fsx, and xfstests

-   Resilver / ZED testing using FileSystemAger.py script to age file
    system and then dd and iozone to write new files into the file
    system

-   Data integrity tests using fsx and iozone, td

-   Basic performance testing with IOR and fio

-   the mixed workload performance testing is described in a comment in
    the Lustre Streaming IO

-   Application scale testing will be WRF, HACC IO, and IOR

-   SOAK test currently includes mdtest, IOR, simul, racer, Kcompile,
    blogbench and pct

-   The torture test will likely include various configurations of all
    of these running over time and surviving various faults that will be
    injected.

Unit Tests
----------

As patch sets are developed they should be tested and verified. Where
possible each new major feature will also have a unit test developed
alongside of it that can be run in an autotest environment. Unit tests
should include code verification for the following if code is added or
modified:

-   Declustered RAIDZ mapping scheme code correctness 

-   Recovery from transient disk outages 

-   Throttling tunable for ongoing resilver IO 

-   Spare rebalance with distributed spare

-   ZED detection of failed disk and triggering of resilver using the
    appropriate distributed spare space. 

Integration Tests
-----------------

Integration tests should verify the following functionality:

-   Configured declustered RAIDZ zpools

-   Verify mapping using zdb

-   Simulate transient disk outages and verify the zpool can survive 

-   Inject device type failures into zpool to verify that ZED detects
    failures to include the following methods:

    -   Using zinject –t data to inject VDEV data errors.

    -   Using dmsetup to put error and flakey regions into a disk
        device.

-   Using scsi debug to make one or possibly more devices and make use
    of every n^th^ error injection. Verify that it automatically starts
    resilvering 

-   After a resilver has completed add a new disk and verify a
    successful rebalance 

-   With IO resilver throttle set verify that applications can use
    reserved bandwidth

-   With IO resilver throttle set verify with no application IO the
    resilver uses full bandwidth

-   Verify no regressions to the ZPL or ZVOL with ZFS regression tests

-   Verify data integrity after insert bad data into either the file via
    zinject or the device via dd and verify ZFS records checksum errors
    and takes appropriate action.

System Tests
------------

-   Development of declustered RAIDZ will proceed in parallel with
    development of Lustre and ZFS streaming improvements and expect to
    make use of the ZFS improvements as soon as they are stable. We will
    document performance for a basic performance test (IOR) and a mixed
    workload along with stats from ZFS, ARC, block devices, etc. and
    track these over time to see how different patch sets and
    configurations impact performance. Performance will be tracked for
    each tag that is created.

-   Perform IO sanity tests and benchmarks with Lustre steaming code to
    declustered RAIDZ  

-   Until Cray has a functioning burst buffer stack that they can share,
    Intel will attempt to simulate burst buffer traffic through Lustre
    clients to Lustre servers running Lustre streaming code and onto
    declustered RAIDZ using iozone or IOR to generate traffic we expect
    to see from the burst buffer.

-   Generate large check point files from as large of an MPI job as is
    possible on test hardware. Have CPPR’s data mover send this through
    the function shipper to the IONs to the back end file system (both
    directly to Lustre and also through burst buffer).  

Stability Tests
---------------

-   Long running tests to verify correctness using ztest in various
    configurations to test ZFS components and fsx and xfstests to test
    file system 

-   Using a dirty file system and a long running simulated work load
    inject faults into ZFS VDEVs, block devices and verify that all the
    resilvers happen and system is returned to a fully protected state. 

-   Verify rebalance under heavy load and see how it reacts to new
    injected faults.  A heavily loaded file system is characterized as

    -   file system over 80% utilized

    -   I/O bandwidth at maximum capacity for sustained periods of time

    -   Different loads such as IOR across the whole system followed by
        several different smaller jobs including things like
        applications/kernels, check points being sent to Lustre and
        retrieved, work load simulations like filebench and vdbench.

    -   mix of applications and kernels and IO benchmarks

-   During the test, inject Lustre faults and fail Lustre server nodes
    and clients and verify that normal IO operations continue. There are
    two parts to this:

    -   Using soak testing framework to crash and fail over Lustre
        server nodes

    -   Using OBD\_FAIL\_LOC to inject errors and or delays into LNet
        routers to simulate things like a cable failure, badly
        performing networking causing unexpected behavior to network
        stack, or LNet router failure. The drop rule and drop delay can
        be set via lctl for specific destinations.

-   The stability test is designed to run in weeks, not days, and
    generally keep all available compute clients running jobs


