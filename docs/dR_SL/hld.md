[]{#_Toc342408269 .anchor}

dRAID: Declustered RAID for ZFS

High Level Design

Revision 1.4

January 29, 2016

[]{#_Toc46119879 .anchor}Contents

[1 Introduction 1](#introduction)

[2 Definitions 2](#definitions)

[3 Changes from Solution Architecture
4](#changes-from-solution-architecture)

[4 Specification 5](#specification)

[4.1 Parity Declustering and Distributed Sparing
5](#parity-declustering-and-distributed-sparing)

[4.2 Declustered Layouts 7](#declustered-layouts)

[4.2.1 Layout Criteria 8](#layout-criteria)

[4.2.2 Imbalance Ratio 8](#imbalance-ratio)

[4.2.3 Known Parity Declustering Layouts
10](#known-parity-declustering-layouts)

[4.2.4 Random Permutations Layout 11](#random-permutations-layout)

[4.2.5 Extended Permutation Development Data Layout
13](#extended-permutation-development-data-layout)

[4.3 Declustering with ZFS 17](#declustering-with-zfs)

[4.3.1 dRAID Limitations 17](#draid-limitations)

[4.3.2 File Block Size Restrictions 17](#file-block-size-restrictions)

[4.3.3 Hybrid Mirror/RAIDZ on dRAID VDEV
19](#hybrid-mirrorraidz-on-draid-vdev)

[4.3.4 Drive Slice Size 20](#drive-slice-size)

[4.4 Block Allocation 22](#block-allocation)

[4.4.1 Permutation Redundancy Group Boundary
25](#permutation-redundancy-group-boundary)

[4.4.2 Degraded Permutation Redundancy Groups
25](#degraded-permutation-redundancy-groups)

[4.5 Distributed Spare Space 26](#distributed-spare-space)

[4.6 Fast Sequential Resilver 27](#fast-sequential-resilver)

[4.6.1 Sequential Resilver Algorithm Outline
27](#sequential-resilver-algorithm-outline)

[4.6.2 Concurrency Between Applications and Sequential Resilver
29](#concurrency-between-applications-and-sequential-resilver)

[4.7 Sequential Resilver Prototype 30](#sequential-resilver-prototype)

[4.7.1 Reducing Runtime Memory Overhead
32](#reducing-runtime-memory-overhead)

[4.7.2 Smart Drive Skipping 33](#smart-drive-skipping)

[4.7.3 Throttling Resilver IO 35](#throttling-resilver-io)

[4.8 Persistent Storage of dRAID Configuration Parameters
37](#persistent-storage-of-draid-configuration-parameters)

[4.8.1 PRNG Shuffling of Base Permutations
37](#prng-shuffling-of-base-permutations)

[4.8.2 dRAID Configuration Parameter Storage
39](#draid-configuration-parameter-storage)

[4.8.3 Compatibility with Future ZFS Releases
40](#compatibility-with-future-zfs-releases)

[4.9 ZFS Event Daemon 41](#zfs-event-daemon)

[5 API and Protocol 42](#api-and-protocol)

[6 Risks and Unknowns 43](#risks-and-unknowns)

[7 References 44](#_Toc441814251)

Figures

[Figure 1. Two RAID5 (4+1) Redundancy Groups 5](#_Toc441814188)

[Figure 2. A Degraded Group in a Traditional RAID Layout
6](#_Toc441814189)

[Figure 3. Parity Declustered RAID Layout with a Degraded Group
7](#_Toc441814190)

[Figure 4. Relation of Permutation to Drive Layout 9](#_Toc441814191)

[Figure 5. Rebuild IO Distribution for Drive 0 Failure
9](#_Toc441814192)

[Figure 6. Random-Permutation Layout 11](#_Toc441814193)

[Figure 7. Imbalance Ratio of Random Permutation Layouts
12](#_Toc441814194)

[Figure 8. Permutation Development of Random Base Permutations
14](#_Toc441814195)

[Figure 9. Relation of Permutation Development to Drive Layout
15](#_Toc441814196)

[Figure 10. Random Permutations vs Extended PDDL 16](#_Toc441814197)

[Figure 11. ZFS RAIDZ Layout 18](#_Toc441814198)

[Figure 12. Hybrid Declustered Mirror/RAIDZ 19](#_Toc441814199)

[Figure 13. IO Size and Resilver Performance 21](#_Toc441814200)

[Figure 14. Logical Block Address 22](#_Toc441814201)

[Figure 15. ZFS Block Allocation 24](#_Toc441814202)

[Figure 16. Distributed Spare Logical VDEV 26](#_Toc441814203)

[Figure 17. Resilver Prototype: Throughput and Memory Overhead
31](#_Toc441814204)

[Figure 18. Improved Resilver Prototype: Throughput and Memory Overhead
33](#_Toc441814205)

[Figure 19. Resilver Prototype: Making Best Use of Available Bandwidth
34](#_Toc441814206)

[Figure 20. Resilver Comparison: Default vs. Increased Resilver Queue
36](#_Toc441814207)

[Figure 21. Resilver Throttling Algorithm 36](#_Toc441814208)

[Figure 22. Permutation Generation Algorithm 38](#_Toc441814209)

[Figure 23. ZFS VDEV Label 40](#_Toc441814210)

Revision History

  ---------- ---------------------------------------------------------------- ------------ --------------------------------------
  Revision   Description                                                      Date         Author
  0.1        First draft                                                      2015-09-02   Isaac Huang
  1.0        Revision following 9/23 F2F review                               2015-09-30   Isaac Huang, John Carrier
  1.1        Revision following Argonne review of rev. 1.0                    2015-11-24   Isaac Huang, Don Brady, John Carrier
  1.2        Revision following Argonne and Livermore review                  2015-12-22   Isaac Huang, John Carrier
  1.3        Revision following Argonne and Livermore review of version 1.2   2016-01-25   Isaac Huang
  1.4        Revision following Argonne and Livermore review of version 1.3   2016-01-29   Isaac Huang
  ---------- ---------------------------------------------------------------- ------------ --------------------------------------

[[[[]{#_Ref46905290 .anchor}]{#_Ref46905265 .anchor}]{#_Toc46134343
.anchor}]{#_Toc82841952 .anchor}Author: Isaac Huang

Contributors: Eric Barton, Don Brady, Andreas Dilger, John Carrier

Introduction
============

In large-scale storage configurations needed to meet the IO requirements
of future HPC systems, disk failures are inevitable and must be viewed
as a normal incident rather than an exceptional event.

When disk failures happen, it is important that RAID parity
reconstruction completes as quickly as possible. Shorter rebuild times
significantly reduce exposure to multiple concurrent disk failures,
which could lead to data loss. It is also important that the RAID
rebuilding process minimally impacts the application IO. A drop in IO
performance would cause applications to run longer or even fail to
complete in their allotted time window.

The process of rebuilding a “traditional” RAID array, when replacing a
failing/failed drive by a new one, consists of reading all the data,
block-by-block, on all the surviving disks in the array, reconstructing
the original blocks of the failed drive, and then writing the
reconstructed data to the replacement drive. After this process is
complete, the array is restored to its original full redundancy.

In ZFS, there is a similar and equivalent process, called resilvering,
which is implemented differently from traditional RAID reconstruction as
volume management is a built-in part of ZFS. This process starts by
traversing the ZFS block pointer tree to discover all the blocks of the
ZFS pool that were affected by the failed drive. Upon reaching one of
these blocks, the block is read, or reconstructed if necessary from the
redundant/parity information, its checksum is verified, and the missing
data or parity from the failed drive is written to free blocks on a new
drive.

In both cases, the speed of rebuilding or resilvering is bounded by the
write throughput of a single replacement drive. As a result, the total
resilver time will grow at least linearly (often much worse) with drive
capacity. As drive capacity continues to grow with little increase in
drive throughput, rebuild time can increase significantly. For example,
it would take about 27 hours to rebuild a 10TB drive at 100MB/s. Since
idle time is rare, a drive failure and subsequent rebuild process can
significantly affect system performance.

Parity declustered RAID for ZFS will distribute data, parity, and spare
capacity across all drives in a pool so that they all participate in the
rebuild process equally. Since the pool is many times larger than the
redundancy group size, aggregate read performance during reconstruction
is correspondingly increased. And since reconstructed data is written to
spare space distributed across all drives, the bottleneck of having to
write a single replacement drive to restore redundancy is eliminated.

Though declustered RAID for ZFS will use the existing RAIDZ code to
calculate parity and reconstruct lost blocks, we call the solution dRAID
to make clear the distinction between the layout of data, parity, and
spare space in this design and the layout of data and parity with
existing RAIDZ. The dRAID solution makes possible a mechanism to further
speed up recovery after a drive failure by decoupling the recreation of
redundancy from the verification of the recreated blocks to ensure that
no drive in the storage array is idle during the rebuild.

[[]{#_Toc438071187 .anchor}]{#_Ref437852280 .anchor}

Definitions
===========

[]{#_Toc423427246 .anchor}The following definitions are used in the rest
of this document:

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  ZFS                A combined file system and logical volume manager.
  ------------------ -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  RAIDZ              A generic term to refer to ZFS RAIDZ1, RAIDZ2, RAIDZ3, and mirror, when there is no need to distinguish between them. Otherwise the more specific terms are used. The generic term RAID is also used when there is no need to distinguish between traditional RAID and RAIDZ.

  VDEV               A "virtual device" describes a single device or a collection of devices organized according to certain performance and fault characteristics. ZFS currently supports the following VDEV types: disk, file, mirror, RAIDZ, spare, log, and cache.

  ZFS Pool           Unlike traditional file systems which reside on single devices and thus require a volume manager to use more than one device, ZFS file systems are built on top of virtual storage pools called ZFS pools. A ZFS Pool is a constructed of a set of VDEVs.

  dRAID              A modification of RAIDZ, as defined in this document, to implement declustered RAID for ZFS using fixed stripe-width redundancy groups to improve RAIDZ resilver speed. This is a generic term that can refer to ZFS dRAID1 (single parity), ZFS dRAID2 (double parity), ZFS dRAID3 (triple parity), and ZFS dRAIDM (mirror).
                     
                     Though dRAID will use the existing RAIDZ code to calculate parity and reconstruct lost blocks, we call the solution dRAID to make clear the distinction between the layout of data, parity, and spare space in this design and the layout of data and parity with existing RAIDZ.

  Drive Slice        All drives in a dRAID VDEV are divided into equal sized units called slices. A slice is the basic unit of parity declustering. Slice size must be a multiple of hardware sector size of the drive.

  Metaslab           An allocation region in a VDEV. ZFS divides a top-level VDEV into equal-sized regions called metaslabs. A ZFS block cannot cross metaslab boundary.

  Permutation        Blah blah: base permutation and developed permutation derived from the base permutation.

  Redundancy Group   The redundancy group is composed of data and parity units that RAIDZ generates from the file block it receives from ZFS. Reconstruction of the group is possible if one or more (depending on the RAIDZ type) of its units are unreadable.

  Resilver           The process of reconstructing data/parity on a failed drive in a RAIDZ group to a replacement drive, or failed drive in a dRAID group to spare space.

  Scrub              The process of examining all ZFS blocks in a pool to verify block checksums. For replicated VDEVs (mirror, RAIDZ, or dRAID), ZFS automatically repairs any damage discovered.

  Spacemap           Persistent on-disk data structure that keeps track of allocated space in a metaslab. There is one spacemap for each metaslab.

  Spare Rebalance    The process of copying reconstructed data/parity from previous spare blocks to a replacement drive so that distributed spare blocks become available again.

  Spare Space        This is spare capacity distributed over all drives in a dRAID VDEV, reserved for recovery. For the sake of simplicity hereafter in this document, N spare drives is used as a shorthand for distributed spare space with sufficient capacity to rebuild data on N failed drives.

  Uberblock          A VDEV label contains an array of uberblocks. The uberblock is the portion of the label containing information necessary to access the contents of the pool. Only one uberblock in the pool is active at any point in time. The uberblock with the highest transaction group number and valid SHA-256 checksum is the active uberblock.

  Unit               A unit is a portion of a redundancy group written to a drive slice. A redundancy group is composed of data and parity units.

  VDEV Label         Each physical VDEV within a ZFS pool contains four copies of a 256KB structure called a VDEV label, two at the beginning of the VDEV and two at the end. The VDEV label contains information describing this particular physical VDEV and all other VDEVs which share a common top-level VDEV as an ancestor.

  DVA                The Data Virtual Address is the ZFS notion of block address. It consists of two parts: VDEV, and offset. It determines the physical location of a ZFS block on a top-level VDEV.

  ZED                ZFS Event Daemon monitors events generated by the ZFS kernel module. When a zevent (ZFS Event) is posted, ZED will run any ZEDLETs (ZFS Event Daemon Linkage for Executable Tasks) that have been enabled for the corresponding zevent class.
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Changes from Solution Architecture
==================================

There is no change from the solution architecture.

Specification
=============

[[[[[[]{#_Toc438071323 .anchor}]{#_Toc431395026 .anchor}]{#_Ref431334572
.anchor}]{#_Ref430978403 .anchor}]{#_Ref429986334
.anchor}]{#_Toc428958183 .anchor}Sections 4.1 and 4.2 provide an
overview and justification for the design. Sections 4.3 and following
describe the details proposed for the implementation of the declustered
parity design for ZFS.

Parity Declustering and Distributed Sparing
-------------------------------------------

For the purposes of this discussion, we will use a storage array of 12
drives. In a traditional RAID5 configuration, this array is configured
with two 4+1 redundancy groups and two global hot spares (Figure 1).
Each column in the figure represents a single drive (the number in each
cell is the drive number). Within each redundancy group, data and parity
blocks are written as a stripe across slices of each drive in the
redundancy group. RAID5 provides single parity protection, which means
the redundancy group can lose one drive and still provide consistent
data. If a data block is missing, it can be recalculated from the
remaining data and parity drives.

![](media/image2.png){width="3.0in" height="3.75in"}

[[[[]{#_Toc441814188 .anchor}]{#_Toc441792668 .anchor}]{#_Toc441193357
.anchor}]{#_Ref438504189 .anchor}Figure 1. Two RAID5 (4+1) Redundancy
Groups

When a drive fails, the redundancy group is in a degraded state. As
shown in the following figure (Figure 2), if drive 0 fails, then the
remaining drives in its 4+1 group are degraded, as indicated by the red
color in the figure.

![](media/image3.png){width="4.6in" height="2.51in"}

[[[[]{#_Toc441814189 .anchor}]{#_Toc441792669 .anchor}]{#_Toc441193358
.anchor}]{#_Ref438502793 .anchor}Figure 2. A Degraded Group in a
Traditional RAID Layout

For this single-parity RAID configuration, during the rebuild process:

1.  Only drives 1 to 4 will be read from.

2.  Only the 1^st^ hot spare (drive 10) will be written to.

3.  Drives 5 to 9 and drive 11 will not participate in the rebuild.

In other words, although there are 11 remaining drives in the array,
reconstruction will be gated by the speed of reading from 4 drives and
writing to just one.

The goal of parity declustering and distributed sparing is to engage all
drives in the array in the rebuild process in order to restore full data
redundancy as quickly as possible. Writing to all drives is enabled by
reserving spare space on each drive so that spare capacity is
distributed across all the drives in the array rather than being
restricted to a few drives reserved as global hot spares. Redistributing
redundancy groups, slice-by-slice, across all drives, ensures that each
drive contains some part of any degraded redundancy group so that all
drives will be read during reconstruction.

Parity declustered RAID distributes data, parity, and spare space across
all drives so that all drives participate in the rebuild process. If
blocks of each row in the traditional RAID layout are randomly shuffled,
without moving the positions of the data/parity/spare columns, the
result is a simple parity declustered layout, as shown in the graph
below (Figure 3).

![](media/image4.png){width="4.6in" height="2.51in"}

[[[[]{#_Toc441814190 .anchor}]{#_Toc441792670 .anchor}]{#_Toc441193359
.anchor}]{#_Ref438506736 .anchor}Figure 3. Parity Declustered RAID
Layout with a Degraded Group

Compared to the traditional RAID layout in Figure 2, each row in the
declustered layout is a random permutation of all drives, and each
column is no longer a single drive. Data, parity, and spare blocks
(*i.e.*, the columns) for both redundancy groups in the figure are
effectively spread out to all the drives.

Both the blocks to be read from during rebuild (red blocks) and the
blocks to be written to during rebuild (green blocks) are mapped onto
all twelve drives. Since the pool is many times larger than the
redundancy group size (in this example, 12/(4+1)=2.4 times), aggregate
read performance during reconstruction is correspondingly increased. And
since reconstructed data is written across all drives, the bottleneck of
having to write a single replacement drive to restore redundancy is
eliminated. As a result, all surviving drives will participate in the
rebuild process, and none will stay idle.

Parity declustering and distributed spare space are technically separate
concepts, but in practice they are used together to parallelize both
read and write IOs during rebuild. The rest of this document will simply
refer to both as parity declustering.

Declustered Layouts
-------------------

A declustered layout is a scheme that maps redundancy groups, including
spare blocks, across all drives in the declustered VDEV or, as defined
here, the ZFS dRAID VDEV. Drives in the pool are partitioned into
logical slices of equal capacity. Data, parity and spare units are then
assigned to the slices of the child drives by a declustered layout. In
other words, a drive slice is the portion of a drive allocated to each
layout. The slice may contain data, parity, or spare space, depending on
the layout definition.

Distributing spare blocks among all drives in the array greatly improves
resilver speed because all remaining drives have spare space that can be
used to store the reconstructed redundancy. Once the failed drive is
replaced with a new drive, data on the spare blocks are copied back to
the new drive and spare blocks will be available again on the remaining
drives. As described in the scope statement and the solution
architecture , this process is called rebalance to distinguish it from
the rebuild, or resilver process.

The assignment of each drive slice within a layout is called a
permutation. The list of all permutations in the ZFS dRAID VDEV is a
layout map that points to the drive slice for each redundancy group
unit. Since each drive is mapped to multiple different permutations, it
will contain a combination of data, parity and spare space units
aggregated from the layout mapped to the drive.

The challenge is to find declustered layouts that optimize utilization
of the drives in the array following drive failure(s).

### Layout Criteria

A usable layout should fulfill the following three criteria:

1.  Unit Distribution\
    Each drive contains approximately the same number of data,
    redundancy, and spare slices.

2.  Redundancy Correctness\
    No two units in a block redundancy group reside on the same drive,
    otherwise a single drive failure could cause data loss. This
    criterion should hold true even after some failed data and parity
    units have been replaced with spare units.

3.  Resilver Load Balance\
    The resilver workload to restore redundancy after a drive failure is
    evenly distributed among all surviving drives.

Criteria \#1 and \#2 are easy to quantify or verify. To verify criterion
\#3, we use the notion of imbalance ratio, introduced below in Section
4.2.2.

### Imbalance Ratio

In Figure 4 below, a layout map using random permutations of drives is
shown on the left side, while the effective declustered mapping of data,
parity, and spare blocks per drive is shown on the right side.

![](media/image5.png){width="6.0in" height="4.06in"}

[[[[]{#_Toc441814191 .anchor}]{#_Toc441792671 .anchor}]{#_Toc441193360
.anchor}]{#_Ref438508192 .anchor}Figure 4. Relation of Permutation to
Drive Layout

Given the small number of random permutations shown, the distribution of
parity and spare space is not equivalent between drives. For example,
drive 0 has three parity blocks and one spare block, while drive 2 has
three spare blocks and one parity block. Therefore the rebuild IO will
not be distributed equally to all twelve drives. The following table
(Figure 5) shows the discrepancy between the I/O loads for each drive
during the rebuild following the failure of drive 0. This information is
tabulated by looking at the drives in the redundancy groups in each
permutation that contain drive 0. If drive 0 is a data or parity drive,
then the remaining 4 drives (data and/or parity) will be read. The first
virtual spare drive in each permutation (first green column) will be the
target for writing the reconstructed data or parity.

  Drive \#   1   2   3   4   5   6   7   8   9   10   11
  ---------- --- --- --- --- --- --- --- --- --- ---- ----
  Read       5   2   3   2   4   2   2   5   5   2    4
  Write      0   2   1   1   2   1   1   0   0   2    0
  Combined   5   4   4   3   6   3   3   5   5   4    4

[[[[]{#_Toc441814192 .anchor}]{#_Toc441792672 .anchor}]{#_Toc441193361
.anchor}]{#_Ref438509314 .anchor}Figure 5. Rebuild IO Distribution for
Drive 0 Failure

In an ideal declustered layout, the distribution of the redundancy
groups and spare space will be such that all remaining drives in the
array will share rebuild workload evenly. The imbalance ratio is the
measure of the deviation from this ideal. For any set of failed drives
recoverable from redundancy, the imbalance ratio is defined as follows:

1.  For each surviving drive, calculate the number of times it is read
    from and written to during rebuild. This number represents the
    amount of rebuild workload assigned to the drive (*i.e.*, the
    numbers in the “Combined” row in Figure 5).

<!-- -->

1.  The imbalance ratio is the maximum of these numbers divided by the
    minimum.

In the example above, drive 5 has the greatest workload with 6 reads and
writes. Drives 4, 6, and 7 have the lowest workload with 3 reads and
writes. The imbalance ratio when drive 0 has failed is 2 (6 divided by
3).

The imbalance ratio for the layout must consider all drive failures. The
ratio for any particular layout will be the maximum ratio for any drive
failure (*i.e.*, the worst possible ratio for the layout).

When the imbalance ratio is 1, the layout is said to be perfectly
balanced: the surviving drives would share the rebuild workload equally.
The closer the ratio gets to 1, the more balanced the layout is.

### Known Parity Declustering Layouts

There has been a lot of research on parity declustering layouts. It has
been shown that ideal layouts (*i.e.*, imbalance ratio == 1) do not
exist in general . Known ideal layouts all come with certain
restrictions on disk array configurations.

Holland and Gibson have compiled a list of Balanced Incomplete Block
Designs (BIBD) for parity declustering layouts based on block designs ,
which are known to exist for only some array configurations. For
example, in their list there is no block design for an array of 90
drives. In addition, BIBD based designs may require the storage of large
tables in memory and on disk, which we want to avoid.

Alvarez *et al.* provided formal proof that ideal layouts are possible
only for very limited array configurations. They introduced the PRIME
layout, which requires the total number of drives to be a prime number,
and the RELPR layout, which removed the prime number restriction but
complicated integration of distributed spare space into the layout.

In the Permutation Development Data Layout (PDDL) , a larger set of
permutations are developed algorithmically from a smaller set of base
permutations. Their base permutation was constructed from the well-known
Bose algorithm, for which the total number of drives is restricted to a
subset of prime numbers and the number of spare drives is restricted to
one. For example, for a storage enclosure with 90 drives, the largest
base permutation possible with our implementation of the Bose algorithm
was 71 drives[^1]. And this base permutation would develop to an ideal
layout only if there was one drive capacity of spare space.

Our goal is to find a flexible and general solution. We want to support
multiple-parity RAID for a range of commercially available storage array
configurations. Spare space capacity cannot be limited to the capacity
of one and, instead, must handle simultaneous drive failures. In
addition, we prefer a layout that can be stored economically on disk
that does not consume considerable resources when expanded in system
memory. Therefore, instead of searching for ideal declustering layouts,
we chose to focus on nearly ideal layouts, in which the imbalance ratio
is reasonably close to 1, with support for flexible spare space and
minimal run-time overhead.

We also surveyed the more general research on random data placement
(*e.g.*,,, , ). This research has focused on two main goals: to maximize
load balance during normal application IO and to minimize data migration
when adding or removing drives. We found that the research results in
this area were not directly applicable to our design goal of maximizing
load balance during rebuild IO.

### Random Permutations Layout

As described above, previously published declustering schemes placed too
many restrictions on the number of drives in the declustered layout. As
a result, we pursued layouts based on random permutations of parity
groups and spare space. This method of generating layouts does not pose
restrictions on the total number of drives or the total amount of
distributed spare space because the algorithm simply shuffles the drives
into random permutations.

The following graph (Figure 6) illustrates such a layout based on random
permutations for a 12-drive 4+1 dRAID1 with 2 spare drives.

![](media/image6.png){width="6.159374453193351in"
height="3.356858048993876in"}

[[[[]{#_Toc441814193 .anchor}]{#_Toc441792673 .anchor}]{#_Toc441193362
.anchor}]{#_Ref438519329 .anchor}Figure 6. Random-Permutation Layout

o block redundancy group crosses a permutation boundary, which is a row
boundary in the figure. It follows that any permutation-based layout
fulfills the Redundancy Guarantee requirement (Section 4.2.1, criterion
\#2), because no two drives in a permutation are the same.

Note that this is a simplified illustration that does not show the
allocation of ZFS blocks within the drive slices. For an illustration of
detailed block placement, please refer to Figure 15. ZFS Block
Allocation in Section 4.4.

With only 10 permutations, this layout has an imbalance ratio of 2 (see
Section 4.2.2). With more permutations, the imbalance introduced by
individual permutations would cancel out each other and the imbalance
ratio would approach 1.

To find out the number of permutations needed to fulfill the Resilver
Load Balance requirement (Section 4.2.1, criterion \#3), we studied
layouts of increasing number of total permutations for 82-drive dRAID2
(10 x (8+2) with 2 spares). The permutations are generated using the
algorithm described in section 4.8.1. The results are shown in the
figure below (Figure 7).

![](media/image7.png){width="6.223369422572178in"
height="4.667527340332459in"}

[[[[]{#_Toc441814194 .anchor}]{#_Toc441792674 .anchor}]{#_Toc441193363
.anchor}]{#_Ref438511894 .anchor}Figure 7. Imbalance Ratio of Random
Permutation Layouts

As expected for a randomly generated sequence, the imbalance ratio
approaches perfect balance (*i.e.*, equals 1) as the total number of
permutations grows. But note that the plot is a curve and that the X
axis is a log scale. As a result, although the plot approaches an
imbalance ratio of 1 with increasing number of permutations, the net
change is less and less as the number of permutations increases and the
benefit of increasing permutations diminishes quickly.

At 10496 total permutations, the ratio reaches 1.027. In other words,
during the resilver process with this layout, the busiest drive does
2.7% more work than the idlest one. But this low imbalance ratio comes
at a cost since all permutations must be stored in memory to be
available to the dRAID driver. The 10496 permutations for this 82 drive
array would require about 1.64MB if a short integer is used to store the
drive index number.

The random permutation layout is flexible in terms of array
configuration, it has no restriction on the total number of drives in
the array or the amount of spare space allocated. Nonetheless we pursued
layout optimizations that would retain this flexibility, but reduce the
amount of memory needed to hold the base permutation map.

### Extended Permutation Development Data Layout

Despite the limitations on array configuration, the permutation
development method generates layouts with a very desirable property:
data, redundancy, and spare units are equally distributed to all drives.
In other words, all PDDL layouts satisfy the Unit Distribution
requirement (Section 4.2.1, criterion \#1).

The random permutations layouts have no such limitations on array
configuration, so we applied the permutation development method to
random base permutations to create layouts that meet Unit Distribution
requirement (layout criterion \#1) without the configuration limitations
in the original PDDL. We call this layout, built from a random base
permutation with permutation development, extended PDDL.

The following graph (Figure 8) illustrates such a layout of a 12-drive
4+1 dRAID1 with 2 spare drives. With 12 drives in the array, each
permutation is developed to create an additional 11 permutations. The
permutations are now divided into groups:

-   Each permutation group consists of 12 permutations (equal to the
    number of drives in the array).

-   The first permutation in a group is called the base permutation,
    which is generated randomly. Only base permutations are stored in
    memory.

-   The Nth (N starts from 0) permutation in a group is computed from
    the base permutation by adding N to each number in the base
    permutation and then mod 12. Since this computation can be done on
    the fly, these derived permutations are not stored in memory.

![](media/image8.png){width="3.504116360454943in"
height="8.63437445319335in"}

[[[[]{#_Toc441814195 .anchor}]{#_Toc441792675 .anchor}]{#_Toc441193364
.anchor}]{#_Ref438512784 .anchor}Figure 8. Permutation Development of
Random Base Permutations

Due to the nature of permutation development, each column in a
permutation group is also a permutation of all drives. It follows that
each drive shows up in every column exactly once. Therefore, each drive
in the permutation group has exactly the same number of
data/redundancy/spare units, which meets the Unit Distribution
requirement (layout criterion \#1) .

The impact of permutation development within one permutation group
(*i.e.*, one base permutation and its developed permutations) on the
drive layout is shown below in Figure 9. The result is to distribute
data parity and spare units equally across the drives in the pool.

![](media/image9.png){width="6.0in" height="4.0in"}

[[[[]{#_Toc441814196 .anchor}]{#_Toc441792676 .anchor}]{#_Toc441193365
.anchor}]{#_Ref438513089 .anchor}Figure 9. Relation of Permutation
Development to Drive Layout

Moreover, extended PDDL layout meets the Redundancy Guarantee
requirement (Section 4.2.1, criterion \#2) since extended PDDL layout is
still completely based on permutations and, as discussed in the previous
section, our permutation-based layouts meet this requirement.

Next we studied imbalance ratio of extended PDDL layouts and compared it
against random permutations layout. The base permutations are generated
using the algorithm described in Section 4.8.1. Results are shown in the
figure below (Figure 10).

![](media/image10.png){width="6.160416666666666in"
height="4.620138888888889in"}

[[[[[]{#_Toc441814197 .anchor}]{#_Toc441792677 .anchor}]{#_Toc441193366
.anchor}]{#_Ref438516258 .anchor}]{#_Ref438513321 .anchor}Figure 10.
[]{#_Ref438516404 .anchor}Random Permutations vs Extended PDDL

Again, the X-axis uses a log scale. It can be seen from the lower,
dotted line in Figure 10 that extended PDDL approaches imbalance ratio
of 1 about twice faster than the randomly generated base.

The extended PDDL gets a better ratio with less total permutations due
to an effect of the column shift during permutation development. In each
permutation group, every drive shows up in every column exactly once. It
follows that regardless of the failed drive, the imbalance ratio would
be the same. But for random permutations layout, the imbalance ratio
varies a lot depending on which drive has failed. The definition of
imbalance ratio picks the worst ratio for all possible drive failures,
see Section 4.2.2.

In addition to a lower imbalance ratio, only the base permutations need
to be stored in memory. The rest can be computed on the fly when needed
with simple arithmetic. If there are N drives in the array and M total
permutations, then instead of storing M permutations, extended PDDL only
requires storing M/N base permutations. For example, the imbalance ratio
of 1.03 in Figure 10 is reached at 5248 total permutations. With N=82 in
this example, 5248 total permutations were developed from only 64 base
permutations. Only these base permutations need be stored in memory.

In summary, any extended PDDL layout fulfills the Unit Distribution and
Redundancy requirements (layout criteria \#1 and \#2, Section 4.2.1),
and one with sufficient number of base permutations also reasonably
fulfills the Resilver Load Balance requirement (criterion \#3), while
being flexible with array configuration. The run-time memory and
computation overhead is also minimal. The extended PDDL layout fulfills
our goal to find a flexible and general solution with minimal run-time
overhead.

Declustering with ZFS
---------------------

### dRAID Limitations

In addition to the criteria defined in Section 4.2.1, declustered RAID
on ZFS has these additional limitations:

-   Usable capacity of any drive in a ZFS dRAID VDEV equals that of the
    smallest drive.

-   Each drive is divided into equal sized units called slices; all
    drives in a ZFS dRAID VDEV must have the same slice size. Slice size
    cannot change once the dRAID VDEV has been created.

### File Block Size Restrictions

To this point, the discussion assumes that file blocks are written as a
stripe across all the drives in the redundancy group. ZFS, however,
implements RAIDZ to enable variable-length, full-stripe writes that
avoid read modify writes and, as a result, avoid the dreaded
“write-hole” that threatens traditional RAID solutions.

The cost of this flexibility is that reconstructing lost data/parity
after a disk failure requires ZFS to traverse its block-pointer tree in
order to identify the affected blocks.

The following diagram (Figure 11) shows a typical layout of data and
parity units on a 4+1 RAIDZ1 VDEV.

![](media/image11.png){width="3.09in" height="4.0in"}

[[[[]{#_Toc441814198 .anchor}]{#_Toc441792678 .anchor}]{#_Toc441193367
.anchor}]{#_Ref438514396 .anchor}Figure 11. ZFS RAIDZ Layout

In contrast to traditional RAID systems, in a ZFS RAIDZ:

-   Redundancy group size is not fixed. For example, the size is 4+1 at
    LBA 0 (A to E), 3+1 at LBA 2 (A to D), 2+1 at LBA 7 (A to C), and
    1+1 at LBA 3 (D to E).

-   Location of data/parity units is not known until the block pointer
    tree is traversed.

Therefore, existing research results cannot be applied to ZFS RAIDZ
directly.

In addition, the ZFS block pointer tree must be traversed during
resilver, and this tree traversal often generates small and random IOs.
In some cases, the tree traversal can become a bottleneck of the
resilver process, offsetting the parallelism enabled by parity
declustering.

To overcome these difficulties in the declustering layout we propose, it
is required that all ZFS blocks stored on a dRAID VDEV must meet the
following size restriction:

*If redundancy group has D data units and drives have sector size of S,
then all ZFS block sizes on the dRAID VDEV must be a multiple of D\*S.*

For example, on an 8+2 dRAID of 4KB sectors, all ZFS block sizes must be
a multiple of 32KB. Implications of this requirement are discussed below
(see Section 4.3.3).

This size restriction enables analysis of imbalance ratio because
redundancy group size is now fixed. In addition, since data and parity
units are now at known locations, it is possible to perform resilver
without traversing the block pointer tree (see section 4.6).

Moreover, there will be no need for RAIDZ style skip sectors for dRAID.
ZFS pads blocks on RAIDZ with skip sectors to make sure the total number
of sectors is a multiple of 1 plus the number of parity units (*e.g.*, 3
for RAIDZ2). For example, sectors marked with “**X**” in Figure 11 are
skip sectors. The elimination of skip sectors will save space and
improve IO merging.

### Hybrid Mirror/RAIDZ on dRAID VDEV

ZFS metadata blocks often compress down to a 4KB block. It is not space
efficient to store these blocks on a RAIDZ VDEV. For example, a 4KB
block on an 8+2 RAIDZ2 would require 1 4KB data block and 2 4KB parity
blocks; for the same level of redundancy, it would be more efficient to
store the block on a 3-way mirror, which eliminates the parity
computation at the same space overhead. It would be even more
inefficient to round the ZFS metadata block up to 32KB in order to store
it on an 8+2 dRAID2 VDEV. Note that in this section we use the 32KB
alignment requirement as an example – the exact alignment will be
determined by the dRAID VDEV configuration (see section 4.3.2).

To efficiently store metadata blocks and data blocks under 32KB,
declustered RAIDZ and declustered mirror redundancy groups can be
interleaved on a dRAID VDEV, as show in the graph below (Figure 12).

![](media/image12.png){width="6.346729002624672in"
height="3.34328302712161in"}

[[[[]{#_Toc441814199 .anchor}]{#_Toc441792679 .anchor}]{#_Toc441193368
.anchor}]{#_Ref438515201 .anchor}Figure 12. Hybrid Declustered
Mirror/RAIDZ

Metaslabs on a dRAID VDEV can be either a declustered RAIDZ metaslab or
a declustered mirror metaslab. ZFS metadata blocks and data blocks under
32KB are stored on declustered mirror metaslabs, without the 32KB size
alignment requirement. All ZFS data blocks of 32KB or larger are rounded
up to 32KB multiples, and stored in declustered RAIDZ metaslabs. When
data compression is not on, no rounding up is needed because ZFS logical
block sizes grow by doubling and already align on 32K boundary. When
data compression is on, physical block sizes can be arbitrary and must
be rounded up to the next 32KB boundary. To some degree, the space lost
by rounding up can be made up for by the elimination of skip sectors.

MS3 Lustre Streaming Improvements High Level Design introduced the
concept of metadata allocation classes as a means of isolating ZFS
metadata from large block data allocations. dRAID extends this concept
so that the metadata allocation class can store both the small ZFS
metadata and small file data (&lt;32KB in the example above). For this
hybrid redundancy dRAID, a metadata allocation class will be backed by a
mirrored metaslab in the dRAID VDEV.

The mirror variant metaslab will be differentiated from normal RAID
metaslabs using two different data structures. The first method uses the
block pointer, which is required in order to read the block. Since all
metadata blocks need to be read without any reference to other metadata
objects, capturing the mirror information in the block pointer is an
obvious choice. In this case, the existing grid bits in the block
pointer will be used. A non-zero grid value will indicate the mirrored
redundancy for the block. Note that Oracle ZFS uses the grid bits in a
similar manner for its hybrid raidz/mirror VDEVs.

The second method uses the spacemap object for the metaslab. This object
is used when instantiating a metaslab and setting up the allocation
classes. It is also used when the sequential resilver algorithm chooses
a reconstruction algorithm to resilver the metaslab (see section 4.6.1).
A variable similar to the grid bits in the block pointer will be
introduced to the spacemap object to indicate that the metaslab is using
mirrored redundancy.

The number of metaslabs dedicated to mirror use can vary, but an initial
allotment will be 2.5%, with the maximum allowed is 10%. Additional
mirror-metaslabs can be activated as needed up to this maximum. As an
example, if we have 200 metaslabs in a VDEV, 5 (2.5%) would initially be
set aside for mirroring and up to 20 (10%) could be dedicated to mirror
use. If the mirror-metaslab limit is reached, then small allocations are
allocated from normal RAID-metaslabs. This avoids the extreme case of
activating too many mirror-metaslabs and not reserving enough for larger
file blocks.

More details about the metadata allocation class for isolating small
blocks can be found in MS3 Lustre Streaming Improvements High Level
Design, section 4.1 ZFS Metadata Isolation.

### Drive Slice Size

To study the impact of IO request size on resilver performance, we ran
four resilver benchmarks with different ZFS block sizes (128KB, 1MB,
4MB, and 16MB) on an 8+2 RAIDZ2 of 10 1T drives. Disk IO sizes on RAIDZ
child VDEVs grow linearly with ZFS block size (disk block size = file
block size / 8). The pool was filled with mostly large files to mimic
Lustre OSS workload. One drive was taken offline and replaced, and write
IO requests issued to the replacement drive were plotted and shown in
the graph below (Figure 13):

-   The X axis shows resilver time in seconds, and Y sizes of IO
    requests in bytes.

-   Four plots differing only by ZFS block sizes (a.k.a. record size
    property) are shown in different colors (128KB = red, 1MB = green,
    4MB = blue, 16MB = magenta).

![](media/image13.png){width="6.5in" height="4.875in"}

[[[[[[[]{#_Toc441814200 .anchor}]{#_Toc441792680
.anchor}]{#_Toc441193369 .anchor}]{#_Toc438071196
.anchor}]{#_Toc431395012 .anchor}]{#_Toc428957984
.anchor}]{#_Ref431391500 .anchor}Figure 13. IO Size and Resilver
Performance

Clearly the larger the ZFS block sizes (hence larger resilver IO sizes),
the shorter the total resilver time and the higher the throughput. In
addition, although the graph shows only write IOs on the replacement
drive, read IOs on the surviving drives should be very similar – to
write to the replacement drive for S bytes, all surviving drives must be
read for the same S bytes.

In a ZFS dRAID VDEV, the size of each redundancy group must be at least
16MB to ensure that a 16MB ZFS block will not be split into multiple
groups. For example, on a ZFS dRAID2 with 1MB slice size, a 16MB block
will fit into 2 redundancy groups (*i.e.*, 16 data drives), so each
drive will get a 1MB IO when accessing the block.

Therefore, in order to maximize the benefits of large blocks, drive
slice size must be sufficiently large so that a redundancy group will be
able to accommodate the largest ZFS block (16MB). For example, for 8+2
dRAID2 the drive slice size should be at least 2MB to guarantee IO sizes
for 16MB blocks are 2MB on each child VDEV.

For more details on block assignment in a ZFS dRAID VDEV, please refer
to Section 4.4, Block Allocation.

Block Allocation
----------------

Before we look at block allocation in permutations of drive slices,
Logical Block Address (LBA) ordering of ZFS dRAID VDEV must be clearly
defined. For ease of illustration, the examples in this section all use
a 12-drive ZFS dRAID1 VDEV using two (4+1) redundancy groups with 2
spares. The sector size of each drive is 4KB and the drive slice size is
16KB. Each sector has an LBA. The graph below (Figure 14) shows LBAs in
two permutations of drive slices:

-   Each cell in the LBAs view (the upper part of the graph) is a
    sector. The number in each sector is its LBA.

-   Each cell in the permutation group view (the lower part of the
    graph) is a drive slice. The number in each slice is its drive
    number.

![](media/image14.png){width="6.160416666666666in"
height="4.936111111111111in"}

[[[[[[]{#_Toc441814201 .anchor}]{#_Toc441792681 .anchor}]{#_Toc441193370
.anchor}]{#_Toc438071199 .anchor}]{#_Toc431395014
.anchor}]{#_Ref431391157 .anchor}Figure 14. Logical Block Address

As shown above in Figure 14:

-   Sectors of a redundancy group are numbered contiguously. For
    example, sectors 0 to 19 form a redundancy group.

-   The four sectors of a slice are not numbered contiguously. For
    example, sectors 0, 5, 10, and 15 form the slice of drive 10 in the
    first permutation.

-   A redundancy group is filled up before the LBA continues to the next
    redundancy group.

-   A permutation is filled up before the LBA continues to the next
    permutation.

The following graph (Figure 15) shows allocated ZFS blocks in two
permutations:

-   Each cell in the detailed permutation view (upper part of the graph)
    is a sector.

-   Each cell in the permutation group view (lower part of the graph) is
    a slice.

-   The number in each cell is the number of the drive to which the
    sector/slice belongs

![](media/image15.png){width="4.565693350831146in"
height="7.484744094488189in"}

[[[[[[[[[[[]{#_Toc441814202 .anchor}]{#_Toc441792682
.anchor}]{#_Toc441193371 .anchor}]{#_Toc438071200
.anchor}]{#_Toc431395015 .anchor}]{#_Ref431072031
.anchor}]{#_Ref429988214 .anchor}]{#_Ref429987989
.anchor}]{#_Ref429987033 .anchor}]{#_Ref429738129
.anchor}]{#_Ref431334068 .anchor}Figure 15. ZFS Block Allocation

For the discussion of block allocation, the notion of redundancy group
needs to be refined:

-   Permutation redundancy group: the set of contiguous drive slices
    that form a group of data and parity slices. For example, in the
    first permutation, the first five slices on drives 10, 9, 11, 4, and
    7 are a permutation redundancy group. For other parts of the
    document redundancy group is used as a shorthand for permutation
    redundancy group.

-   Block redundancy group: each ZFS block consists of several block
    redundancy groups. For example, in the detailed view of permutation
    0 in the above graph (Figure 15), the 32KB block has two block
    redundancy groups, and the 64KB block has four. Each cell in a block
    redundancy group is a hardware sector on the corresponding drive.

Declustered RAID for ZFS, though providing the same level of redundancy,
differs from conventional RAIDZ in many ways that must be made visible
to the ZFS block allocator:

-   A metaslab must begin at a permutation boundary. This can be easily
    achieved by rounding up metaslab size to the next multiple of
    permutation size, *i.e.*, the drive slice size multiplied by the
    total number of drives.

-   Block allocations that cross permutation redundancy group boundary
    should be avoided.

-   Block redundancy groups cannot cross permutation boundaries,
    otherwise the Redundancy Guarantee requirement (Section 4.2.1,
    criterion \#2) cannot be met. Due to the limitation on block size
    (see Section 4.3.2) and metaslab alignment, this condition is
    automatically met and no change to the ZFS block allocator is
    needed. Note that it is even acceptable for a data block to cross a
    permutation boundary, *e.g.*, block 2 in the graph above (Figure
    15).

-   New block allocations should avoid degraded permutation redundancy
    groups when there is no more spare space available.

### Permutation Redundancy Group Boundary

In the detailed view of permutation 0 in Figure 15. ZFS Block
Allocation, the 64KB data block crosses a permutation redundancy group
boundary. This is not a problem for correctness, but is a problem for
performance: each drive gets an IO size of 8KB, instead of 16KB if the
block had not crossed permutation redundancy group boundary.

As discussed in Drive Slice Size (Section 4.3.4), the slice size will be
at least 2MB instead of 16KB as shown in the example (Figure 15. ZFS
Block Allocation). As a result, the redundancy group size will be 128
times larger and thus the chance of crossing permutation redundancy
group will be 128 times smaller. In addition, for large HPC Lustre file
systems, the majority of file blocks will be 16MB. By using dedicated
metaslabs for 16MB blocks and slice size of 2MB multiples, no 16MB block
will ever cross permutation redundancy group, which ensures 2MB IO for
16MB large blocks.

### Degraded Permutation Redundancy Groups

In Figure 15. ZFS Block Allocation, if drive 0 fails, only the first
redundancy group of the second permutation is degraded. All other
permutation redundancy groups are in healthy condition. Even with
multiple drive failures, some permutation redundancy groups can still
stay in healthy condition. Moreover, when there are enough spare blocks
available, even a degraded group can be used for new ZFS blocks by
writing to the corresponding spare blocks the new blocks still have full
redundancy.

Current ZFS block allocation algorithm will avoid using a degraded RAIDZ
VDEV when allocating the only copy of a block (*e.g.*, a data block when
data ditto is disabled). But for reasons mentioned in the above
paragraph, block allocation within a degraded ZFS dRAID VDEV should be
allowed.

The block allocator should instead avoid using degraded redundancy
groups in a ZFS dRAID VDEV where there are not enough spare blocks.
However, care must be taken not to add to fragmentation in the free
space. ZFS keeps a per-metaslab stat on free space fragmentation, which
can be used to disable the avoidance algorithm once it reaches a certain
threshold.

Distributed Spare Space
-----------------------

Distributed spare space is represented as special child VDEV(s) of the
parent ZFS dRAID VDEV, as shown by the following *zpool status* outputs
running our prototype declustered 2-way mirror code (Figure 16).

![](media/image16.png){width="5.0in" height="2.94in"}

[[[[[[[]{#_Toc441814203 .anchor}]{#_Toc441792683
.anchor}]{#_Toc441193372 .anchor}]{#_Toc438071198
.anchor}]{#_Toc431395013 .anchor}]{#_Toc428957985
.anchor}]{#_Ref431392210 .anchor}Figure 16. Distributed Spare Logical
VDEV

Although Figure 16 shows only one distributed spare VDEV, there will be
one for each spare column configured in the extended PDDL layout.

Unlike traditional dedicated hot spare drive, distributed spare VDEV is
a resource local to the parent ZFS dRAID VDEV and can only be used to
replace a sibling VDEV under the same parent VDEV.

Distributed spare VDEV will be implemented as a new VDEV driver, because
user space programs (*e.g.*, the ZFS Event Daemon) may need to
distinguish it from traditional dedicated hot spare drive. The complete
solution will include a dRAID VDEV for the declustered array as a whole,
and distributed spare VDEV or VDEVs to represent the spare space in the
dRAID VDEV.

Fast Sequential Resilver
------------------------

In traditional RAIDZ, ZFS blocks can start at any VDEV offset and can be
of any size, as shown in the Figure 11, which depicts a typical RAIDZ
block layout. There is no way to tell where the data and parity blocks
are without scanning the block pointer tree of the whole pool. The
resilver process repairs blocks as they are encountered during the block
pointer tree traversal. On aged pools, this often results in random
resilver IO pattern and is usually bottlenecked by the IOPs intensive
metadata tree scan. In a ZFS dRAID VDEV with a much larger number of
child drives, it will be difficult for this process to saturate
available throughput of all drives.

With the block size restrictions on a ZFS dRAID VDEV (see Section
4.3.2), the offsets of data and parity blocks are known without the help
of any metadata. Thus much like traditional RAID, a ZFS dRAID VDEV can
be resilvered sequentially without an expensive metadata scan.

In addition, by reading minimal space accounting metadata (*i.e.*,
spacemaps of the metaslabs), unused space can be skipped by the resilver
process. Essentially a ZFS dRAID becomes a hybrid of traditional RAID
and object-based ZFS RAIDZ: it can be sequentially rebuilt without
expensive metadata scanning like a traditional RAID, yet unused space
can be skipped like a ZFS RAIDZ.

If resilver is done following block pointer tree traversal, it may have
to search arbitrarily far ahead in the tree to find blocks which will
keep nearly idle drives loaded. But with fixed geometry this search can
be done with a simple enumeration of the permutations.

Since block pointers, which contain the checksums of the blocks, are not
scanned, data integrity cannot be verified during the fast resilver. As
a result, silent data errors encountered during the fast resilver will
go unnoticed. However, the lack of data integrity check during fast
resilver does not affect ZFS’s ability to detect data corruption, it
just delays detection and correction until ZFS reads the corrupted data
blocks either for an application or by scrubbing. Therefore, after a
fast resilver completes, the VDEV should be scrubbed to find and correct
any potential corruption. Since the failed drive and redundancy have
been restored by the fast resilver process, however, the scrub IO can be
given a much lower priority.

To be clear, this is a brand new resilver algorithm, rather than a
modification of the current ZFS resilver algorithm. The outlines of the
sequential resilver algorithm will be given in the next section, where
“resilver” is used as shorthand for “sequential resilver”.

### Sequential Resilver Algorithm Outline

Sequential resilver can be started by a command from user space to
replace a dRAID member drive with distributed spare space. The command
may come automatically from the ZED or manually from the administrator.
The drive to be replaced can be a failed drive or a good one, *e.g.*,
the administrator may opt to proactively replace a drive that has shown
some signs of pending failure. Sequential resilver can be resumed by the
kernel dRAID driver as well upon pool import, after a previous resilver
is interrupted. Once sequential resilver is started or resumed, the
kernel dRAID driver initiates the process outlined below:

1.  Initiate resilver state, which includes the transaction group number
    at which the resilver process starts, the total number of metaslabs
    that have been resilvered, and, optionally, a reference to the root
    of the block pointer tree so that blocks don’t get freed and reused
    during the resilver.\
    \
    Note that:

    -   Resilver state is both kept in memory and saved in a persistent
        on-disk structure at the end of each transaction group sync.

    -   Transaction group number is the current syncing transaction
        group number for a new resilver, or read from disk in case of
        restarting a previous resilver interrupted by power loss.

    -   Total number of resilvered metaslabs is initialized to 0 for a
        new resilver, or read from disk in case of restarting a previous
        resilver interrupted by power loss.

2.  Sequentially resilver metaslabs from the N^th^ to the last, one by
    one, where N equals the total number of resilvered metaslabs
    initialized in step 1.

    -   Load metaslab into memory, *e.g.*, reading the spacemap and
        creating the range trees that keep track of free space.

    -   Resilver used space in the metaslab.

        -   First determine if the metaslab is a declustered RAIDZ or
            mirror, and choose a reconstruction algorithm accordingly.

        -   Used spare will not be freed and reused as a result of the
            block pointer tree reference in step 1, otherwise data
            corruption may happen because the resilver process may
            overwrite newer data with old data. One possible
            optimization here is to not reference the block pointer tree
            and thus allow used space to be freed during resilver. The
            resilver process will then have to look at the per
            transaction group range trees of freed and delayed free
            space to avoid rebuilding freed space.

        -   If the drive to be replaced has failed, the units are
            rebuilt from redundancy. Otherwise, data/redundancy unit is
            directly copied from the original drive. In the latter case,
            all read IO is handled by a single drive, but there is still
            the advantage of not having to traverse the block pointer
            tree.

    -   Increase the total number of resilvered metatslabs by 1, both in
        memory and on disk.

3.  If the server loses power in the middle of a sequential resilver, it
    can be restarted from step 1 by reading the resilver state from
    disk. Resilver will resume from the metaslab that was being rebuilt
    when resilver was interrupted. Since only the metaslab number is
    saved persistently, in the worst case, a whole metaslab worth of
    work is lost and will be redone.

    -   Currently ZFS divides a top-level VDEV into 200 metaslabs by
        default, so at most 0.5% of total resilver work can be lost.
        However, a dRAID VDEV is much larger than traditional ZFS
        top-level VDEV, because it allows a larger number of child
        VDEVs. Therefore the total number of metaslabs for a dRAID VDEV
        will be increased from the default, which will reduce the worst
        case percentage of lost work.

4.  When the last metaslab has been resilvered, remove persistent
    resilver state from disk and free its in-memory copy.

### Concurrency Between Applications and Sequential Resilver

The sequential resilver process must coexist with application IO, so the
algorithm must be able to handle concurrency between resilver IO and
application read and write. Because dRAID allows a much larger number of
child drives than a traditional RAIDz VDEV does, some blocks will remain
unaffected by a drive failure. The sequential resilver will not touch
these blocks, so there is no race between applications and resilver
process. The discussion hereafter in this section is only for the rest
of the blocks affected by drive failure.

When an application writes a ZFS block, due to the copy-on-write nature
of ZFS, it is always written into free space. Since the sequential
resilver algorithm will not reconstruct free space, there is no
concurrency between resilver writes and application writes to a same
location. New blocks are always written to spare space with full
redundancy.

-   When the sequential resilver process works on used space in a
    metaslab, it does not have any knowledge of the ZFS blocks there
    because the block pointer tree has not been traversed. Therefore it
    is possible that a new block written with full redundancy after the
    resilver has started is resilvered again. This is correct, but is
    unnecessary and wasteful. As a mitigation, the block allocator
    should prefer resilvered metaslabs for block allocation during
    resilver, so that new blocks are written to resilvered metaslabs.

-   If used space can be freed and allocated to new blocks while
    sequential resilver is working on it, corruption may happen as both
    application and resilver process may write to a same location. This
    race is avoided by taking a reference on the root of the block
    pointer tree at the 1^st^ step of the resilver process.

When an application reads a ZFS block, if the block cannot be found in
the ARC cache, the dRAID VDEV driver will be called to handle the read.
The metaslab to which the block belongs can be determined from its DVA
(Data Virtual Address), and by comparing it against the current metaslab
being resilvered:

1.  If the block’s metaslab has already being resilvered, then there is
    no race between application and the resilver process. The block has
    restored full redundancy and can be read as normal without
    reconstruction.

2.  The block’s metaslab is either being resilvered or to be resilvered:

    a.  If the block to be read is born after the resilver begins
        (*i.e.*, its birth transaction group is no earlier than the
        transaction group when the resilver begins), the block has been
        written with full redundancy, and can be read as normal without
        reconstruction.

    b.  The block is born before the resilver begins. The block is
        always rebuilt from redundancy even though there is a chance
        that it has just been resilvered and restored full redundancy.
        This way, application read will not use the spare block that the
        resilver process may be writing to. Therefore the race is
        essentially eliminated.

During implementation, this area must be investigated thoroughly and
treaded carefully, as the outcome of any mishandled race can lead to
corruption.

Sequential Resilver Prototype
-----------------------------

As shown in Figure 10, Random Permutations vs Extended PDDL, in order to
reach an even workload distribution among all drives, the resilver
algorithm must generate enough IO to cover sufficient permutation
groups. For example, to reach a 2.3% imbalance, the resilver IO needs to
cover 128 permutation groups – the total IO buffers needed for this many
permutation groups will be around 160GB, which is not acceptable.

However, the run time behavior will be different since, as the
sequential resilver process generates more IO, the drives also complete
pending IOs whose buffers can be freed up. It would be hard to get
accurate results from theoretical analysis alone because hard drives are
very complicated mechanical devices and the run time dynamics of kernel
IO scheduling and VM is also very complex. As a result, to guide our
analysis, we developed a prototype sequential resilver code and ran it
over real hardware:

-   The prototype runs in the kernel, and uses the Linux bio API to read
    and write the drives. Since this is the same API used by ZFS to
    access drives, we kept the prototype as close to ZFS as possible.

-   The prototype was run on a 43-drive ZFS dRAID VDEV (the largest test
    hardware available at the time of the test), using 32 permutation
    groups with imbalance ratio at about 4% and 2MB drive slice size.
    The total IO buffer to cover all 32 groups is about 24,768MB.

-   The prototype sequentially resilvered 32768 16MB blocks, assuming
    the 1^st^ drive failed. Note that the imbalance ratio for extended
    PDDL layouts was the same regardless of which drive had failed, so
    there was no need to simulate other drive failures.

The following graph (Figure 17) shows plots of the combined read and
write throughput of all surviving drives and the total memory overhead
over time during drive resilvering using the prototype code. The source
of the memory overhead is the IO buffers, which cannot be freed until an
IO is complete. (Read IO buffers cannot be freed until the corresponding
lost data or parity has been rebuilt and a certain amount of pending IO
has to be maintained to keep the drives
busy.)![](media/image17.png){width="6.5in" height="4.875in"}

[[[[[[[[[[]{#_Toc441814204 .anchor}]{#_Toc441792684
.anchor}]{#_Toc441193373 .anchor}]{#_Toc438071201
.anchor}]{#_Toc431395016 .anchor}]{#_Toc428957986
.anchor}]{#_Ref428957512 .anchor}]{#_Ref428957360
.anchor}]{#_Ref428956321 .anchor}]{#_Ref430730097 .anchor}Figure 17.
Resilver Prototype: Throughput and Memory Overhead

In this figure, the X axis shows the time in seconds, the Y axis on the
left shows combined read and write throughput in MB/s, and the Y axis on
the right shows total memory overhead in bytes.

-   The green shaded curve shows total memory overhead (Y axis on the
    right).

-   Initially total memory increased to close to 2GB, then quickly
    dropped below 1GB as the drives began to complete pending IO and IO
    buffers were freed.

-   The second half of the resilver never went over 500MB, which is much
    smaller than theoretical upper limit at 24768MB.

-   The colored solid curves show the combined throughput of individual
    drives (Y axis on the left). Variance in throughput of the drives
    began at about 4% and became smaller and smaller over time. The
    following section will explain the steady drop in variance.

    [[[[]{#_Toc441193346 .anchor}]{#_Toc441189748
    .anchor}]{#_Toc438071336 .anchor}]{#_Toc431395055 .anchor}The
    initial peak in memory overhead, albeit only for a brief period of
    time, will add to system memory pressure; and there have been known
    issues with ZFS ARC under memory pressure. Moreover, the initial
    peak will grow proportionally with the total number of drives, and
    will last longer with faster CPUs and/or slower drives. Therefore it
    is desirable to reduce or remove the initial peak in memory
    overhead. The next section will discuss an optimization in order to
    eliminate the initial peak without adversely affecting resilver
    performance.

### Reducing Runtime Memory Overhead

The runtime memory overhead is proportional to the number of inflight
blocks, because all IO buffers are freed once a block is rebuilt.
Therefore, if the total number of inflight blocks is bounded to an upper
limit, for example, a limit that corresponded to the 500MB memory
overhead, then:

-   The initial peak in memory usage will be eliminated. The resilver
    process will not issue more IO until the drives begin the catch up.

-   Drive throughput will not be affected since the throughput did not
    drop during the second half of the resilver, when the number of
    concurrent active blocks was much lower.

-   The total memory overhead at any time during the resilver will not
    exceed the amount of memory needed for the maximum number of
    concurrent blocks.

We modified the resilver prototype to limit maximum concurrent ZFS
blocks being resilvered, ran it on the same hardware with the same
parameters. The results (Figure 18) matched our expectations:

-   The initial peak in memory usage was gone.

-   The overall memory overhead was reduced.

-   The throughput stayed the same with reduced variance among the
    drives.

The maximum concurrent ZFS blocks was chosen empirically. Larger arrays
would require a larger maximum value. Nonetheless, the key point is that
we can find a maximum value to lower/limit memory overhead without
affecting the aggregate resilver throughput and variance in individual
drive throughput.

![](media/image18.png){width="6.5in" height="4.875in"}

[[[[[[[]{#_Toc441814205 .anchor}]{#_Toc441792685
.anchor}]{#_Toc441193374 .anchor}]{#_Toc438071202
.anchor}]{#_Ref435702774 .anchor}]{#_Toc431395017
.anchor}]{#_Ref430731151 .anchor}Figure 18. Improved Resilver Prototype:
Throughput and Memory Overhead[[]{#_Toc431395056
.anchor}]{#_Toc428958192 .anchor}

### Smart Drive Skipping

In a degraded ZFS dRAID VDEV of D data units and P parity units and F
failed drives (*e.g.*, D=8 and P=2 in a 8+2 dRAID2), when F is less than
P, the surviving (P - F) drives do not need to be all read during
resilver, because only D drives need to be read in order to repair any
degraded redundancy group.

This gives the resilver algorithm some room to counteract the imbalance
introduced by the permutations. A simple way is to skip the (P – F)
drives that have been assigned to most IO since the resilver started.
Figure 17. Resilver Prototype: Throughput and Memory Overhead shows the
results of such an algorithm. Initially the throughput variance among
drives was close to 4%, the imbalance inherent to the permutations
chosen, because at the beginning of the resilver the historically
accumulated IO could not reflect the permutation imbalance yet. But as
the resilver continued, the variance became smaller and smaller (less
than the 4%).

In many cases, this is the desired outcome: workload gets evenly
distributed among all the drives. But sometimes not all the drives have
equal available throughput, for example not all drives have an equal
share of the PCI bus due to hardware configuration or more likely the
concurrent application IO is not making even use of all drives (making
the throughput available to resilver uneven among the drives). In such
scenarios, it is better to schedule more IO to the drives with more
available throughput, which will result in larger imbalance in resilver
workload distribution but a shorter overall resilver time.

We developed a variant of the algorithm above, which, instead of
skipping (P - F) drives with the most historically accumulated IO,
skipped the drives with the longest current pending IO queue. The more
capable drives will drain their pending queue faster and thus have a
shorter pending queue.

For example, in a 43-drive JBOD available for this test, the first half
of the drives have about 20% less throughput than the second half due to
the way those drives are connected to the host. The following graph
(Figure 19) shows the results of running the variant under the same
parameters as in the previous two figures:

-   The drives with more throughput received about 20% more IO, which
    matched the imbalance due to hardware configuration, rather than the
    4% imbalance inherent in the permutations.

-   All drives completed at the same time. The resilver using the
    modified prototype completed about 10 seconds earlier (about 6%
    faster) than did the original prototype (Figure 17).

![](media/image19.png){width="5.586666666666667in" height="4.19in"}

[[[[[[[[]{#_Toc441814206 .anchor}]{#_Toc441792686
.anchor}]{#_Toc441193375 .anchor}]{#_Toc438071203
.anchor}]{#_Toc431395018 .anchor}]{#_Ref431138444
.anchor}]{#_Toc428957987 .anchor}]{#_Ref430731065 .anchor}Figure 19.
Resilver Prototype: Making Best Use of Available
Bandwidth[[[[[[[[[[[[[[[[[[[[[[[[]{#_Toc438071339
.anchor}]{#_Toc431395060 .anchor}]{#_Toc428958194
.anchor}]{#_Toc431395019 .anchor}]{#_Ref429992442
.anchor}]{#_Ref430731048 .anchor}]{#_Toc431391684
.anchor}]{#_Toc431395058 .anchor}]{#_Toc431391470
.anchor}]{#_Toc431391422 .anchor}]{#_Toc431390632
.anchor}]{#_Toc431388992 .anchor}]{#_Toc431387439
.anchor}]{#_Toc431336436 .anchor}]{#_Toc430977247
.anchor}]{#_Toc431391683 .anchor}]{#_Toc431395057
.anchor}]{#_Toc431391469 .anchor}]{#_Toc431391421
.anchor}]{#_Toc431390631 .anchor}]{#_Toc431388991
.anchor}]{#_Toc431387438 .anchor}]{#_Toc431336435
.anchor}]{#_Toc430977246 .anchor}

### Throttling Resilver IO

The ZFS IO scheduler puts resilver IO requests in a separate queue, and
provides tunables to adjust the queue size. To study the impact of queue
size on resilver performance, we ran two resilver benchmarks on an 8+2
dRAID2 with 10 1TB drives and 16MB block size. The pool was filled with
mostly large files to mimic Lustre OSS workload. One drive was taken
offline and replaced, and write IO requests issued to the replacement
drive were plotted. The following graph (Figure 20) shows that resilver
performance improves significantly by increasing the resilver queue size
from the default:

-   The X axis shows time in seconds, and the Y axis shows IO request
    sizes in bytes.

-   The red dots correspond to resilver with queue default settings.

-   The green dots corresponds to resilver with increased queue size:

-   Total resilver time dropped from 189 seconds to 99 seconds.

-   Average throughput (not shown in the graph) nearly doubled from
    68.3MB/s to 130.3MB/s.

-   IO sizes were also larger (*e.g.*, at 3MB and 4MB sizes) which
    indicated more merging due to more concurrent IOs.

![](media/image20.png){width="6.5in" height="4.875in"}

[[[[[[[]{#_Toc441814207 .anchor}]{#_Toc441792687
.anchor}]{#_Toc441193376 .anchor}]{#_Toc438071204
.anchor}]{#_Toc431395020 .anchor}]{#_Toc428957988
.anchor}]{#_Ref430731018 .anchor}Figure 20. Resilver Comparison: Default
vs. Increased Resilver Queue

Therefore, by dynamically changing the resilver IO queue size at run
time, there is sufficient room to achieve any level of desired resilver
throttling. The following graph (Figure 21) illustrates an algorithm to
throttle resilver IO based on resilver severity and application IO:

-   When there is no application IO, resilver queue size is set to the
    max.

-   When application IO increases to a certain point, the resilver IO
    queue is reduced to the minimum, and stays at the minimum as
    application IO grows further.

-   Between the above two points, as application IO grows, resilver
    queue size is decreased linearly.

-   By tuning the minimum resilver queue size and the application IO
    rate where that size is reached, the administrator will be able to
    determine when different levels of throttling can be achieved per
    VDEV. (Mechanisms will exist to manage these tunables at scale.) :

-   Normal resilver: the minimum is lower, and reached faster

-   Severe resilver: the minimum is larger, and reached slower

![](media/image21.png){width="6.2in" height="4.65in"}

[[[[[[[]{#_Toc441814208 .anchor}]{#_Toc441792688
.anchor}]{#_Toc441193377 .anchor}]{#_Toc438071205
.anchor}]{#_Toc431395021 .anchor}]{#_Toc428957989
.anchor}]{#_Ref430730998 .anchor}Figure 21. Resilver Throttling
Algorithm

Application IO rate is the input to the throttling algorithm.
Application writes are already tracked by ZFS as the amount of per-pool
dirty buffer. We may need to add new counters to keep track of
application reads, which should include prefetching reads as well. The
policy will be set by the administrator. For example, responsiveness can
be tuned by setting the point where the resilver queue reaches the
minimum, which changes the steepness (*i.e.*, responsiveness) of the
line between the max and min resilver queue in Figure 21.

Persistent Storage of dRAID Configuration Parameters
----------------------------------------------------

[[[[[[[[[]{#_Toc441193351 .anchor}]{#_Toc441189753
.anchor}]{#_Ref441187177 .anchor}]{#_Ref441181586
.anchor}]{#_Ref441181576 .anchor}]{#_Ref441181340
.anchor}]{#_Ref441481717 .anchor}]{#_Ref441481701
.anchor}]{#_Toc438545834 .anchor}The focus of this section is the
discussion of the parameters that the dRAID VDEV driver needs to
implement declustered parity with distributed sparing on ZFS. Our
implementation will use extended PDDL to generate the permutation
layouts. Therefore, we start by discussing the algorithm we use for
generating the base permutations using a random seed and a pseudo-random
number generator (PRNG).

### PRNG Shuffling of Base Permutations

Our experiments have shown that extended PDDL layout is flexible to a
variety of storage configurations and has lower imbalance ratios than
purely random layouts. The kernel dRAID VDEV driver will store a base
permutation layout table in system memory to access the correct slice on
each drive in the pool. Since permutation development will be computed
on-the-fly, only the base permutation table will be stored in memory.

The issue then is how best to generate the base table. There may be
other methods found later that better generate balanced layouts, but for
now we define the “PRNG Shuffle” algorithm for generating the base
permutation table from which the dRAID VDEV driver will populate the
extended PDDL layout for each permutation group.

The method has two steps:

1.  Initialize the pseudo random number generator (PRNG) with a *seed*.

> The *seed* comes from a random source, *e.g.*, /dev/random on Linux.

1.  Generate a set of random base permutations of all drives (see
    Figure 22. Permutation Generation Algorithm):

    a.  Assign a random number to each drive.

    b.  Sort the drives by the random numbers assigned in the previous
        step.

> Repeat steps a. and b. to create a new base permutation from the
> initial random seed. The total number of base permutations generated
> will depend on the storage array configuration (*e.g.*, number of
> drives and redundancy group size). The beauty of this algorithm is
> that, unlike the well-known Knuth shuffle , it does not require random
> number of a certain range, which in practice can lead to biased
> results.

Given a certain array configuration, the results of running the
algorithm is determined by the seed. Therefore only the seed needs to be
saved in order to reproduce the same base permutations (see Section
4.8.2 for information on seed storage).

The base permutations are generated at the following times:

1.  When a dRAID VDEV is created, the *zpool* command generates the base
    permutations and passes both the base permutations and the seed of
    the shuffling algorithm to the kernel to complete the VDEV creation.
    The kernel only needs to save the seed persistently.

2.  When a pool is being imported, the *zpool* command reads the seed
    from disk labels, generate the base permutations, and passes the
    base permutations back to the kernel to complete the import.

Note that base permutations generation is done completely in the user
space tool. The kernel dRAID VDEV driver just needs the base
permutations but does not care how they are generated. This allows some
flexibility to upgrade or change base permutations generation algorithm
without changing kernel dRAID VDEV driver code or on-disk format.

![](media/image22.png){width="4.944520997375328in"
height="4.2332042869641295in"}

[[[[]{#_Toc441814209 .anchor}]{#_Toc441792689 .anchor}]{#_Toc438545862
.anchor}]{#_Ref438518387 .anchor}Figure 22. Permutation Generation
Algorithm

Base permutations can be compared by calculating the Imbalance Ratio of
the resulting layouts (Section 4.2.2). Different seeds may be necessary
for different storage array configurations. Results from our resilver
prototype (section 4.7) show that an imbalance ratio of 1.04 achieved a
nearly balanced distribution of resilver workload. This suggests that an
imbalance ratio lower than 1.04 should be considered sufficiently good.
Once a seed is found for a particular configuration, it can be reused
for all storage arrays built from that configuration.

For small storage array configurations, prescribed static layouts may
result in better balance. Such “verbatim” layouts are discussed briefly
in Section 4.8.2.

### dRAID Configuration Parameter Storage

The dRAID configuration parameters consist of two parts:

1.  Redundancy group size, spare capacity, and slice size:

    a.  For ZFS RAIDZ, redundancy group size is implicit. For example, a
        RAIDZ2 VDEV of 10 child drives has an 8+2 redundancy group.
        Since dRAID allows more child drives than the number of units in
        a redundancy group, the group size must be specified explicitly,
        in the form of two integers (\# data units, and \# parity
        units). Similarly redundancy level of the hybrid mirror part is
        given by one integer, *i.e.*, the number of copies.

    b.  Spare capacity is given as an integer S, meaning spare space of
        the capacity of S drives.

    c.  Slice size is given as the number of bytes in a drive slice.

2.  Extended PDDL layout metadata:

    a.  Base permutation generation algorithm: string name of the
        algorithm used. Currently only two are defined: “PRNG shuffle”,
        and “verbatim”. Other algorithms may be defined in the future.

    b.  Version number of the generation algorithm. This allows
        improvements to be made to an existing algorithm while staying
        compatible with dRAID VDEVs created by previous implementations
        of the algorithm.

    c.  Parameters to the generation algorithm: this is determined by
        the algorithm.

        i.  For “PRNG shuffle” algorithm, an 8-byte PRNG seed: this is
            used as input to the base permutation shuffling algorithm to
            generate the base permutations.

        ii. For the “verbatim” algorithm, the base permutations array:
            this is meant for smaller dRAID VDEVs where the total number
            of base permutations is small, therefore the array can
            easily fit into the disk labels. The output of the algorithm
            is the same as the input array.

    d.  Total number of base permutations, and a checksum of the base
        permutations array, which is used to verify the integrity of the
        generated base permutations array.

All dRAID configuration parameters must be known when the pool is being
imported. Since no ZFS block can be read without knowing these
parameters, they cannot be stored in any block which is beneath the ZFS
uberblock in the block tree. Currently ZFS stores pool configurations in
VDEV disk labels which contain the uberblock arrays. There is a 112KB
space per label reserved for configuration parameters in the form of
serialized name-value pairs, as show below:

![](media/image23.png){width="6.160416666666666in"
height="1.7916666666666667in"}

[[[[[]{#_Toc441814210 .anchor}]{#_Toc441792690 .anchor}]{#_Toc441193378
.anchor}]{#_Toc438071197 .anchor}]{#_Ref438065285 .anchor}Figure 23. ZFS
VDEV Label

The dRAID parameters will be stored in this 112KB space (from 16K to
128K offset in the label), since there is sufficient room here. ZFS code
has routines to serialize name-value pairs and other binary data into
XDR format for persistent storage. ZFS pool configuration parameters are
stored in the disk labels using these routines. The dRAID configuration
parameters will be persistently stored by using the same routines and
binary format. The *zdb* command will be modified to support dRAID
configuration parameters.

ZFS already saves four redundant copies of disk labels on each disk, two
at the beginning of each disk and two at the end, as shown in Figure 23
(L0, L1, L2, and L3). However, there have been known bugs that lead to
corrupted disk labels and consequently unimportable pools. It is,
therefore, advisable that the administrator should back up dRAID
configuration parameters to a location external to the ZFS pool.

ZFS has a mechanism to cache pool configuration to an external location,
by default at */etc/zfs/zpool.cache*, and allows a pool to be imported
using cached pool configuration. However, there have been some
discussions among ZFS maintainers to obsolete this feature or make it
optional. Therefore, in the long term, it may not be a good candidate
for a stable interface to back up and restore dRAID configuration.

The exact mechanism to back up and restore dRAID configuration depends
on upstream ZFS development and is not decided at this point. In the
event that there is no stable mechanism in stock ZFS code, the *zdb*
command will be used to export dRAID configuration and the *zpool*
command will be modified so that such configuration parameters can be
specified on the command line to override corresponding values stored in
corrupted disk labels. However, it is best to make use of existing
mechanism rather than creating our own.

### Compatibility with Future ZFS Releases

When a dRAID VDEV is created, the configuration parameters are given to
the *zpool* command by the administrator. They are then saved in the
labels of all member drives. ZFS already supports generic name-value
pairs in the labels, so it will not introduce any on-disk format change
by saving the dRAID parameters there. Therefore a dRAID VDEV will
continue to work with future ZFS code releases since it is created.

However, once a dRAID VDEV has been created, none of its configuration
parameters can be changed. If a new ZFS release comes with an improved
base permutation generation algorithm which is able to create more
balanced base permutations, the existing dRAID VDEV will have to
continue to use its original base permutations. There is no way to
upgrade to newer base permutations without recreating the dRAID VDEV,
which will wipe out all data on it.

ZFS Event Daemon
----------------

The ZFS event engine consists of two parts: the kernel part that
generates event reports, (*e.g.*, checksum failures) and a user space
daemon, called ZED, that polls and reads the event reports from the
kernel, and take actions on them. The ZED will be modified so that:

-   ZED is aware of the new distributed spare VDEV, and prefer the
    distributed spare to replace failed drive over a dedicated spare
    drive.

-   ZED understands that a distributed spare VDEV is a resource local to
    its ZFS dRAID VDEV so that the distributed spare VDEV can be used to
    replace only a failed drive in its parent dRAID VDEV and not a drive
    in some other dRAID VDEV.

-   ZED connects to Linux device events to detect drive
    insertion/removals.

-   ZED incorporates heuristics available from the Illumos (formerly
    OpenSolaris) version of OpenZFS for diagnosing ZFS reports before
    responding.

API and Protocol
================

The changes to ZFS will be mostly localized to ZFS dRAID VDEV driver,
resilver algorithm, and ZED. There will not be any API changes visible
to other system components. However, there will be some changes to the
administration interfaces to support the ZFS dRAID VDEV:

-   Changes to the zpool utility to create and manage ZFS dRAID VDEVs

-   New module options and tunables to configure resilver throttling.

Such changes will be documented in the ZFS manual pages.

Risks and Unknowns
==================

The ZFS on Linux implementation depends on ZED to detect drive failure
and automatically start resilvering. While ZED polls events generated by
ZFS, it currently does not read events from other parts of the kernel,
*e.g.*, drive online/offline notifications from the block device
subsystem. In such cases, ZED may to fail to start resilvering
automatically.

**No license (express or implied, by estoppel or otherwise) to any
intellectual property rights is granted by this document.**

**Intel disclaims all express and implied warranties, including without
limitation, the implied warranties of merchantability, fitness for a
particular purpose, and non-infringement, as well as any warranty
arising from course of performance, course of dealing, or usage in
trade.**

**This document contains information on products, services and/or
processes in development. All information provided here is subject to
change without notice. Contact your Intel representative to obtain the
latest forecast, schedule, specifications and roadmaps.**

**The products and services described may contain defects or errors
known as errata which may cause deviations from published
specifications. Current characterized errata are available on request.**

**Copies of documents which have an order number and are referenced in
this document may be obtained by calling 1-800-548-4725 or by visiting
[www.intel.com/design/literature.htm](http://www.intel.com/design/literature.htm).**

**Intel, the Intel logo, {List the Intel trademarks in your document}
are trademarks of Intel Corporation in the U.S. and/or other
countries.**

**\*Other names and brands may be claimed as the property of others**

**© 2015 Intel Corporation.**

[^1]: The base permutation generated for 71 drives via the Bose
    algorithm is the following sequence:\
    1, 14, 54, 46, 5, 70, 57, 17, 25, 66, 7, 27, 23, 38, 35, 64, 44, 48,
    33, 36, 49, 47, 19, 53, 32, 22, 24, 52, 18, 39, 59, 45, 62, 16, 11,
    12, 26, 9, 55, 60, 58, 31, 8, 41, 6, 13, 40, 63, 30, 65, 51, 4, 56,
    3, 42, 20, 67, 15, 68, 29, 2, 28, 37, 21, 10, 69, 43, 34, 50, 61, 0
