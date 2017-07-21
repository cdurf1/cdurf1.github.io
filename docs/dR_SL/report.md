1.  [[[]{#_Toc353174563 .anchor}]{#_Toc346289181
    .anchor}]{#_Toc346186938
    .anchor}![](media/image1.png){width="1.8298611111111112in"
    height="1.3895833333333334in"}

Lustre Streaming (MS05) and dRAID (MS09) Final Report

Final Report

High Performance Data Division

1.  

INTEL FEDERAL, LLC PROPRIETARY

June 2017

1.  **Generated under Argonne Contract number: B609815**

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

    Copyright © 2015 - 2017, Intel Corporation.

    Technical Lead: (Name, E-mail, Phone) Al Gara, alan.gara@intel.com,
    408-765-0996

    Contract Administrator: (Name, E-mail, Phone) Mary Palmer,
    mary.t.palmer@intel.com, 503-264-0113

    Program Manager: (Name, E-mail, Phone) Jacob Wood,
    jacob.r.wood@intel.com, 503-264-2219

Document Revision History

+-----------------------+-----------------------+-----------------------+
| Revision Number       | Date                  | Comments              |
+=======================+=======================+=======================+
| 1.  1.0               | 1.  June 2017         | 1.  Initial version   |
|                       |                       |     of the Lustre     |
|                       |                       |     Streaming (MS05)  |
|                       |                       |     and dRAID (MS09)  |
|                       |                       |     Final Report      |
|                       |                       |                       |
|                       |                       |     Contributors:\    |
|                       |                       |     Don Brady, Isaac  |
|                       |                       |     Huang, David      |
|                       |                       |     Quigley, John     |
|                       |                       |     Salinas, Sydney   |
|                       |                       |     Vanda             |
+-----------------------+-----------------------+-----------------------+

1.  

Contents

[1 Executive Summary 8](#executive-summary)

[2 Demonstration Environment 9](#demonstration-environment)

[2.1 System Configuration 9](#system-configuration)

[2.1.1 Natasha 9](#natasha)

[2.1.2 Wolf 10](#wolf)

[2.2 Tools 11](#tools)

[2.2.1 ZPOOL 11](#zpool)

[2.2.2 ZDB 11](#zdb)

[3 MS05: Lustre and ZFS Block Allocation Improvements
12](#ms05-lustre-and-zfs-block-allocation-improvements)

[3.1 Introduction 12](#introduction)

[3.1.1 ARC Buffer Data (ABD) Update 12](#arc-buffer-data-abd-update)

[3.2 Metadata Isolation 13](#metadata-isolation)

[3.2.1 Architecture 13](#architecture)

[3.2.2 Implementation 13](#implementation)

[3.2.2.1 Dedicated VDEVs 15](#dedicated-vdevs)

[3.2.2.2 Segregated VDEVs 15](#segregated-vdevs)

[3.2.2.3 VDEV Changes 17](#vdev-changes)

[3.2.3 Changes and Deviations 20](#changes-and-deviations)

[3.2.4 Demonstration 20](#demonstration)

[3.2.4.1 Hybrid Metadata/Smallblock Isolation with dRAID VDEVs
20](#hybrid-metadatasmallblock-isolation-with-draid-vdevs)

[3.2.4.2 Observing Metaslab Regions 22](#observing-metaslab-regions)

[3.2.4.3 Observing Free Space Fragmentation
24](#observing-free-space-fragmentation)

[3.2.4.4 Observing Allocations by Category
26](#observing-allocations-by-category)

[3.3 End-to-End 16 MB File Block I/O
27](#end-to-end-16-mb-file-block-io)

[3.3.1 Architecture 27](#architecture-1)

[3.3.2 Implementation 28](#implementation-1)

[3.3.3 Demonstration 29](#demonstration-1)

[3.3.3.1 Configuring the file system for 16MB I/Os
29](#configuring-the-file-system-for-16mb-ios)

[3.3.3.2 Prepping Lustre Counters 31](#prepping-lustre-counters)

[3.3.3.3 End to End Streaming 34](#end-to-end-streaming)

[3.3.3.4 Fragmentation Improvements 41](#fragmentation-improvements)

[4 MS09: dRAID 46](#ms09-draid)

[4.1 Introduction 46](#introduction-1)

[4.2 Parity Declustered RAIDZ and Rebuild
46](#parity-declustered-raidz-and-rebuild)

[4.2.1 Architecture 46](#architecture-2)

[4.2.2 Implementation 47](#implementation-2)

[4.2.2.1 dRAID 48](#draid)

[4.2.2.2 Sequential Rebuild 49](#sequential-rebuild)

[4.2.2.3 dRAID-aware Spare Space Rebalancing
50](#draid-aware-spare-space-rebalancing)

[4.2.3 Changes and Deviations 50](#changes-and-deviations-1)

[4.2.4 Demonstration 50](#demonstration-2)

[4.2.4.1 Arbitrary Pool Configuration 50](#arbitrary-pool-configuration)

[4.2.4.2 Dynamic Rebuild Throttling 52](#dynamic-rebuild-throttling-1)

[4.2.4.3 Rebuild Stop and Resume 54](#rebuild-stop-and-resume)

[4.2.4.4 Rebalance 56](#rebalance)

[4.3 ZED Fault Handling 58](#zed-fault-handling)

[4.3.1 Architecture 58](#architecture-3)

[4.3.2 Implementation 59](#implementation-3)

[4.3.2.1 Spare Device Matching 60](#spare-device-matching)

[4.3.2.2 Multi-path Support 60](#multi-path-support)

[4.3.2.3 Improve ZED resiliency 60](#improve-zed-resiliency)

[4.3.2.4 Multi-Fault Support 61](#multi-fault-support)

[4.3.3 Changes and Deviations 61](#changes-and-deviations-2)

[4.3.4 Demonstration 61](#demonstration-3)

[4.3.4.1 Multi-Fault Handling 61](#multi-fault-handling)

[5 Validation 71](#validation)

[5.1 ZFS Test Suite 71](#zfs-test-suite)

[5.2 ZTest/zloop Verification Tests 71](#ztestzloop-verification-tests)

[5.3 File System Ager 72](#file-system-ager)

[5.4 dRAID Exerciser 72](#draid-exerciser)

[Appendix A. dRAID Configuration Examples
73](#draid-configuration-examples)

[A.1 ‘zdb –m’ for a dRAID pool without segregation
73](#zdb-m-for-a-draid-pool-without-segregation)

[A.2 ‘zdb –m’ for a dRAID pool with segregation enabled
81](#zdb-m-for-a-draid-pool-with-segregation-enabled)

[A.3 draidcfg output for the 80 drive demonstration (80.nvl)
90](#draidcfg-output-for-the-80-drive-demonstration-80.nvl)

[A.4 Zpool status for the 80 drive JBOD
94](#zpool-status-for-the-80-drive-jbod)

1.  

Tables

[Table 3‑1. Lustre Streaming Performance Design Documents
12](#_Toc488392498)

[Table 3‑2. I/O Size Evaluation Tools 35](#_Toc488392499)

[Table 4‑1. Declustered RAIDZ Design Documents 46](#_Toc488392500)

[Table 5‑1. zTest dRAID Options 72](#_Toc488392501)

1.  

Figures

[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[]{#_Toc487029616
.anchor}]{#_Toc487028773 .anchor}]{#_Toc487027908
.anchor}]{#_Toc487027040 .anchor}]{#_Toc487026198
.anchor}]{#_Toc487025356 .anchor}]{#_Toc487024503
.anchor}]{#_Toc487022064 .anchor}]{#_Toc487019889
.anchor}]{#_Toc487015415 .anchor}]{#_Toc487014649
.anchor}]{#_Toc487013883 .anchor}]{#_Toc487012265
.anchor}]{#_Toc486975576 .anchor}]{#_Toc486974814
.anchor}]{#_Toc486974065 .anchor}]{#_Toc486973299
.anchor}]{#_Toc487029615 .anchor}]{#_Toc487028772
.anchor}]{#_Toc487027907 .anchor}]{#_Toc487027039
.anchor}]{#_Toc487026197 .anchor}]{#_Toc487025355
.anchor}]{#_Toc487024502 .anchor}]{#_Toc487022063
.anchor}]{#_Toc487019888 .anchor}]{#_Toc487015414
.anchor}]{#_Toc487014648 .anchor}]{#_Toc487013882
.anchor}]{#_Toc487012264 .anchor}]{#_Toc486975575
.anchor}]{#_Toc486974813 .anchor}]{#_Toc486974064
.anchor}]{#_Toc486973298 .anchor}]{#_Toc487029614
.anchor}]{#_Toc487028771 .anchor}]{#_Toc487027906
.anchor}]{#_Toc487027038 .anchor}]{#_Toc487026196
.anchor}]{#_Toc487025354 .anchor}]{#_Toc487024501
.anchor}]{#_Toc487022062 .anchor}]{#_Toc487019887
.anchor}]{#_Toc487015413 .anchor}]{#_Toc487014647
.anchor}]{#_Toc487013881 .anchor}]{#_Toc487012263
.anchor}]{#_Toc486975574 .anchor}]{#_Toc486974812
.anchor}]{#_Toc486974063 .anchor}]{#_Toc486973297
.anchor}]{#_Toc487029613 .anchor}]{#_Toc487028770
.anchor}]{#_Toc487027905 .anchor}]{#_Toc487027037
.anchor}]{#_Toc487026195 .anchor}]{#_Toc487025353
.anchor}]{#_Toc487024500 .anchor}]{#_Toc487022061
.anchor}]{#_Toc487019886 .anchor}]{#_Toc487015412
.anchor}]{#_Toc487014646 .anchor}]{#_Toc487013880
.anchor}]{#_Toc487012262 .anchor}]{#_Toc486975573
.anchor}]{#_Toc486974811 .anchor}]{#_Toc486974062
.anchor}]{#_Toc486973296 .anchor}]{#_Toc487029612
.anchor}]{#_Toc487028769 .anchor}]{#_Toc487027904
.anchor}]{#_Toc487027036 .anchor}]{#_Toc487026194
.anchor}]{#_Toc487025352 .anchor}]{#_Toc487024499
.anchor}]{#_Toc487022060 .anchor}]{#_Toc487019885
.anchor}]{#_Toc487015411 .anchor}]{#_Toc487014645
.anchor}]{#_Toc487013879 .anchor}]{#_Toc487012261
.anchor}]{#_Toc486975572 .anchor}]{#_Toc486974810
.anchor}]{#_Toc486974061 .anchor}]{#_Toc486973295
.anchor}]{#_Toc487029611 .anchor}]{#_Toc487028768
.anchor}]{#_Toc487027903 .anchor}]{#_Toc487027035
.anchor}]{#_Toc487026193 .anchor}]{#_Toc487025351
.anchor}]{#_Toc487024498 .anchor}]{#_Toc487022059
.anchor}]{#_Toc487019884 .anchor}]{#_Toc487015410
.anchor}]{#_Toc487014644 .anchor}]{#_Toc487013878
.anchor}]{#_Toc487012260 .anchor}]{#_Toc486975571
.anchor}]{#_Toc486974809 .anchor}]{#_Toc486974060
.anchor}]{#_Toc486973294 .anchor}]{#_Toc487029610
.anchor}]{#_Toc487028767 .anchor}]{#_Toc487027902
.anchor}]{#_Toc487027034 .anchor}]{#_Toc487026192
.anchor}]{#_Toc487025350 .anchor}]{#_Toc487024497
.anchor}]{#_Toc487022058 .anchor}]{#_Toc487019883
.anchor}]{#_Toc487015409 .anchor}]{#_Toc487014643
.anchor}]{#_Toc487013877 .anchor}]{#_Toc487012259
.anchor}]{#_Toc486975570 .anchor}]{#_Toc486974808
.anchor}]{#_Toc486974059 .anchor}]{#_Toc486973293
.anchor}]{#_Toc487029609 .anchor}]{#_Toc487028766
.anchor}]{#_Toc487027901 .anchor}]{#_Toc487027033
.anchor}]{#_Toc487026191 .anchor}]{#_Toc487025349
.anchor}]{#_Toc487024496 .anchor}]{#_Toc487022057
.anchor}]{#_Toc487019882 .anchor}]{#_Toc487015408
.anchor}]{#_Toc487014642 .anchor}]{#_Toc487013876
.anchor}]{#_Toc487012258 .anchor}]{#_Toc486975569
.anchor}]{#_Toc486974807 .anchor}]{#_Toc486974058
.anchor}]{#_Toc486973292 .anchor}]{#_Toc462300734
.anchor}]{#_Toc462301919 .anchor}]{#_Toc462300735
.anchor}]{#_Toc462301920 .anchor}]{#_Toc462300736
.anchor}]{#_Toc462301921 .anchor}]{#_Toc462300737
.anchor}]{#_Toc462301922 .anchor}]{#_Toc462300738
.anchor}]{#_Toc462301923 .anchor}]{#_Toc462300739
.anchor}]{#_Toc462301924 .anchor}]{#_Toc462300740
.anchor}]{#_Toc462301925 .anchor}]{#_Toc462300741
.anchor}]{#_Toc462301926 .anchor}]{#_Toc462300742
.anchor}]{#_Toc462301927 .anchor}]{#_Toc462300743
.anchor}]{#_Toc462301928 .anchor}]{#_Toc462300744
.anchor}]{#_Toc462301929 .anchor}]{#_Toc462300745
.anchor}]{#_Toc462301930 .anchor}]{#_Toc462300746
.anchor}]{#_Toc462301931 .anchor}]{#_Toc462300747
.anchor}]{#_Toc462301932 .anchor}]{#_Toc462300748
.anchor}]{#_Toc462301933 .anchor}]{#_Toc462300749
.anchor}]{#_Toc462301934 .anchor}]{#_Toc462300750
.anchor}]{#_Toc462301935 .anchor}]{#_Toc462300751
.anchor}]{#_Toc462301936 .anchor}]{#_Toc462300752
.anchor}]{#_Toc462301937 .anchor}]{#_Toc462300753
.anchor}]{#_Toc462301938 .anchor}]{#_Toc462300754
.anchor}]{#_Toc462301939 .anchor}]{#_Toc462300755
.anchor}]{#_Toc462301940 .anchor}]{#_Toc462300756
.anchor}]{#_Toc462301941 .anchor}]{#_Toc462300757
.anchor}]{#_Toc462301942 .anchor}]{#_Toc462300758
.anchor}]{#_Toc462301943 .anchor}]{#_Toc462300759
.anchor}]{#_Toc462301944 .anchor}]{#_Toc462300760
.anchor}]{#_Toc462301945 .anchor}]{#_Toc462300761
.anchor}]{#_Toc462301946 .anchor}]{#_Toc462300762
.anchor}]{#_Toc462301947 .anchor}]{#_Toc462300763
.anchor}]{#_Toc462301948 .anchor}]{#_Toc462300764
.anchor}]{#_Toc462301949 .anchor}]{#_Toc462300765
.anchor}]{#_Toc462301950 .anchor}]{#_Toc462300766
.anchor}]{#_Toc462301951 .anchor}]{#_Toc462300767
.anchor}]{#_Toc462301952 .anchor}]{#_Toc462300768
.anchor}]{#_Toc462301953 .anchor}Figure 2‑1. Natasha Pool Configuration
10

[Figure 3‑1. Transition from Unmodified RAIDZ to Hybrid Mirror
Configuration 14](#_Toc488392503)

[Figure 3‑2. End to End Data Flow 28](#_Toc488392504)

[Figure 3‑3. Size Distribution of ZFS Write I/O 37](#_Toc488392505)

[Figure 3‑4. Read/Write Disk Stats for Sequential Workload
38](#_Toc488392506)

[Figure 3‑5. Write/Read Bandwidth for Sequential Workload
39](#_Toc488392507)

[Figure 3‑6. Read/Write Disk Stats for Random Workload
40](#_Toc488392508)

[Figure 3‑7. Write/Read Bandwidth for Random Workload
41](#_Toc488392509)

[Figure 3‑8. Fragmentation Impact on RAIDZ1 Performance
42](#_Toc488392510)

[Figure 3‑9. Steps to Create Fragmented File System 43](#_Toc488392511)

[Figure 3‑10. Fragmentation Comparison of Segregated and Unsegregated
dRAIDs 44](#_Toc488392512)

[Figure 3‑11. Performance Impact of Segregation on Fragmented File
System 45](#_Toc488392513)

[Figure 4‑1. Mapping Failed Drive to dRAID1 Permutation Groups
47](#_Toc488392514)

[Figure 4‑2. dRAID2 State Transitions 49](#_Toc488392515)

[Figure 4‑3. Dynamic Rebuild Throttling 53](#_Toc488392516)

[Figure 4‑4. Rebalance to a Replacement Drive 57](#_Toc488392517)

[Figure 4‑5. ZED Architecture 58](#_Toc488392518)

[Figure 4‑6. ZED FMA Components 59](#_Toc488392519)

Executive Summary
=================

1.  For MS04 and MS08, we demonstrated a prototype of the Lustre
    Streaming and dRAID designs that implemented the core features
    described in each High Level Design (HLD). For MS05 and MS09, we
    presented an integrated Lustre and OpenZFS stack that demonstrated
    the combined Lustre Streaming and dRAID features. This document
    describes these additions to the prototype and the results of the
    final demonstration.

    All code developed during this project are being delivered to the
    upstream repositories for Lustre (http://lustre.org/) and OpenZFS
    (http://zfsonlinux.org/). Where possible, hyperlinks are provided in
    the report to the contributions submitted. ZFS on Linux (ZoL) uses
    GitHub (https://github.com/zfsonlinux/zfs) and the pull request
    number records the submission. For example,
    [PR-5182](https://github.com/zfsonlinux/zfs/pull/5182) is a link to
    Pull Request \#5182 for the Metadata Isolation discussed in section
    3.2.

    Our ultimate goal is to create an integrated, open-source software
    stack capable of meeting the performance requirements of Argonne’s
    Aurora supercomputer. Toward that end, the Intel developers have
    been presenting technology deep dives to Cray in support of their
    MS14 to performance tune the I/O system.

    With the recent release of Lustre 2.10 and ZoL 0.7.0, our target for
    integration and release of the code developed during this program is
    Lustre 2.11 and ZoL 0.8.0. Most of the changes for large block I/O
    are already integrated in Lustre 2.10. We are depending on Lustre
    2.11 to resolve instability seen during our testing (see Section
    3.3.2 for more detail). The ZoL 0.7.0 release already includes ABD
    and ZED/FMA, which we integrated during the prototype. Now that
    dRAID and Metadata Isolation are complete, they are anticipated for
    ZoL 0.8.0. A release candidate will be made available as soon as
    they are merged into the tree to encourage broader testing and
    acceptance of the features before the final 0.8.0 release.

Demonstration Environment 
==========================

System Configuration
--------------------

We used two cluster systems, described below, for feature development
and integrated testing. Natasha was the larger system and was used
extensively for integrated testing. Wolf was a smaller cluster that
allowed individual developers to focus on development and unit testing.
The demonstrations primarily used Natasha. Wolf was used to demonstrate
dynamic dRAID creation because that was a destructive test.

Our intention was to use a single integrated software stack and use
Lustre on ZFS for the demonstrations for both MS04 and MS05.
Unfortunately, due to Lustre errors uncovered during our stability
testing ([LU-9305](https://jira.hpdd.intel.com/browse/LU-9305)), we were
forced to run the Metadata Isolation and dRAID demonstrations without
Lustre. Only the End-to-End 16MB demonstration used the complete Lustre
stack (Section 3.3).

### Natasha

1.  The Natasha configuration in this report is run on a fifteen node
    test cluster. This cluster consists of the following components:

-   Eight Lustre clients with 128GB of memory, 72 cores of Intel(R)
    Xeon(R) CPU E5-2699 v3 @ 2.30GHz and Intel® OPA 1.

-   Four Lustre OSS each with 500 GB of memory, 72 cores of Intel(R)
    Xeon(R) CPU E5-2699 v3 @ 2.30GHz, Intel® OPA 1.

-   Four 90 disk Intel JBOD Storage System JBOD2224S2DP-IDD with 24 2TB
    HDD SAS 7200RPM 128MB Seagate Constellation ST2000NM0034 disks

-   One MDS with 500GB of memory, 72 cores of Intel(R) Xeon(R) CPU
    E5-2699 v3 @ 2.30GHz, Intel® OPA 1 configured as a 4-way mirror with
    1k node set to improve performance.

1.  We created the Lustre servers from available over the counter (COTS)
    components. We built the JBODs with 90 drives to simulate the
    proposed Aurora scalable storage unit (SSU) configuration. However,
    the units we acquired did not have sufficient storage bandwidth
    (SAS) to drive all 90 drives at full bandwidth. Therefore, we only
    used one half of the available drives in our tests.

    Each OSS had 1 OST zpool configured with one dRAID2 VDEV of 43 child
    drives as

-   4 x (8 data + 2 parity) parity groups

-   segregated special allocation class (Section 3.2) for small block
    data and ZFS metadata

-   3 distributed spares.

1.  All OSSs were connected as HA pairs. Though not shown during the
    demonstration, failover between the servers had been done as part of
    the continuous integration testing on the system. The pool
    configuration is shown in below (Figure 2‑1).

[[[[]{#_Toc488392502 .anchor}]{#_Toc486599627 .anchor}]{#_Toc486606479
.anchor}]{#_Ref486509122 .anchor}Figure ‑. Natasha Pool Configuration

  ---------------------------------------------
  \#zpool status
  
  pool: ssu\_1ost1
  
   state: ONLINE
  
    scan: none requested
  
  config:
  
  NAME            STATE     READ WRITE CKSUM
  
  ssu\_1ost1       ONLINE       0     0     0
  
  draid2-0      ONLINE       0     0     0
  
     mpathdw     ONLINE       0     0     0
  
     mpathah     ONLINE       0     0     0
  
     mpatho      ONLINE       0     0     0
  
     mpathbm     ONLINE       0     0     0
  
  ...
  
     mpathfe     ONLINE       0     0     0
  
     mpathar     ONLINE       0     0     0
  
     mpathbw     ONLINE       0     0     0
  
     mpathy      ONLINE       0     0     0
  
  spares
  
  \$draid2-0-s0  AVAIL   
  
  \$draid2-0-s1  AVAIL   
  
  \$draid2-0-s2  AVAIL
  ---------------------------------------------

1.  

### Wolf

1.  The Wolf configuration in this report was run on a single node with
    the following components:

-   2 CPUs: 2 x Xeon DP Haswell (EP E5-2699 v3 LGA2011 2.3GHz 45MB 145W
    18 cores CM8064401739300)

-   64G Memory: 8 x 8GB (2133 Reg ECC 1.2V DDR4)

-   90-drive JBOD: 4U Supermicro (CSE-946ED-R2KJBOD)

-   90 x Seagate SAS3 drives (ST2000NM003401)

1.  Although the node had a 90 drive array, we configured the array with
    one dRAID3 VDEV of 80 drives as

-   7 (8 data + 3 parity) parity groups

-   3 distributed spares

1.  No special allocation class storage was added for this
    demonstration. The pool configuration is shown in Appendix A.4.

Tools
-----

1.  ZFS includes a number of utilities for setting and querying the
    state of its objects. We primarily used
    [zpool(8)](https://www.freebsd.org/cgi/man.cgi?zpool(8)) and
    [zdb(8)](https://www.freebsd.org/cgi/man.cgi?zdb(8)) during the
    demonstration. The options most used are shown below.

### ZPOOL

1.  The ZFS “zpool” command is used throughout to query the status of a
    pool created with a dRAID VDEV.

    zpool status describes the status of the pool and its devices.

    zpool list –o describes the state of the specified options
    associated with the pool

    zpool list –C a new command that describes the allocations made to
    the different categories of storage (generic, small block, metadata)
    within a segregated VDEV

### ZDB

1.  ZDB is a developer tool for debugging the file system. ,Various
    portions of ZDB have been adapted to accommodate allocation class
    information.

    zdb –m describes each metaslab and its assigned allocation class

    zdb -mm provides more information about metaslab usage, including
    the storage category within each assigned class and a fragmentation
    histogram which describes available free space.

    zdb –b generates a block audit that can be used to validate ditto
    block assignments within the VDEV.

MS05: Lustre and ZFS Block Allocation Improvements
==================================================

Introduction
------------

1.  This chapter describes the implementation of the MS05 features and
    their demonstration. The documentation below (Table 3‑1) describes
    the scope, architecture, and high-level design of the Lustre
    Streaming Performance Improvements proposed for this program.

[[[[[[]{#_Toc488392498 .anchor}]{#_Toc486599635 .anchor}]{#_Toc486606485
.anchor}]{#_Toc462224955 .anchor}]{#_Toc462220589
.anchor}]{#_Ref462004737 .anchor}Table ‑. Lustre Streaming Performance
Design Documents

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Document                      Location
  ----------------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  1.  Scope Statement           1.  [MS2\_CORAL\_Lustre\_Streaming\_Performance\_Improvements\_Scope\_Statement\_V1.1\_final.pdf](https://wiki.hpdd.intel.com/download/attachments/36966823/MS2_CORAL_Lustre_Streaming_Performance_Improvements_Scope_Statement_V1.1_final.pdf?version=2&modificationDate=1458856882000&api=v2)
                                

  1.  Solution Architecture     1.  [MS2\_CORAL\_Lustre\_Streaming\_Performance\_Improvements\_Solution\_Architecture\_V1.2\_final.pdf](https://wiki.hpdd.intel.com/download/attachments/36966823/MS2_CORAL_Lustre_Streaming_Performance_Improvements_Solution_Architecture_V1.2_final.pdf?version=2&modificationDate=1458857031000&api=v2)
                                

  1.  High Level Design (HLD)   1.  [MS3\_CORAL\_Lustre\_Streaming\_Improvements\_HLD\_v1.2\_final.pdf](https://wiki.hpdd.intel.com/download/attachments/36966823/MS3_CORAL_Lustre_Streaming_Improvements_HLD_v1.2_final.pdf?version=2&modificationDate=1458857322000&api=v2)
                                

  Prototype Report              1.  [MS4,MS8 Prototype\_Report\_Final.pdf](https://wiki.hpdd.intel.com/download/attachments/36966823/MS4%2CMS8%20Prototype_Report_Final.pdf?version=1&modificationDate=1476907192000&api=v2)
                                
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The final delivery includes the following components:

-   Metadata Isolation

-   ZFS ARC Buffer Data (ABD)

-   End to end 16 MB File Block I/O

    Metadata Isolation and End-to-end 16MB I/O were demonstrated for
    MS05. Details of feature implementation and demonstration results
    follow below.

### ARC Buffer Data (ABD) Update

1.  As we described in MS04, we ported to ZFS on Linux the initial port
    of ABD that Delphix made to their illumos operating system.
    Delphix’s focus on a page-list interface between ZFS and the disk
    I/O system for their compressed ARC project made extending the page
    list from ZFS to Lustre unfeasible. Nonetheless, the new code does
    enable Lustre to send 16MB I/Os to ZFS and for ZFS to efficiently
    pass the resulting blocks to dRAID and disk.

    Our port of ABD was accepted by the ZFS on Linux community after
    MS04 and was incorporated with ZFS on Linux 0.7.0 RC2
    ([PR-5135](https://github.com/zfsonlinux/zfs/pull/5135)). Our work
    also included porting compressed ARC and compressed send/receive,
    which were required by Delphix’s port, and making existing ZFS test
    tools ABD-aware. The project included contributions from the
    community to re-enable vectorized RAIDZ with ABD.

    There have been no further demonstrations of the ARC Buffer Data
    (ABD) because the feature is a fundamental component always enabled
    in our ZFS stack to reduce ARC memory pressure and system memory
    fragmentation. Without the ABD patch, the demonstration of 16MB file
    I/O (section 3.3) would not have been possible because ZFS
    eventually collapses under the pressure of allocating contiguous
    16MB buffers.

Metadata Isolation
------------------

1.  The goal of Metadata Isolation is to improve large block streaming
    performance by ensuring that large allocations for application data
    are not impeded by smaller allocations for ZFS metadata and the
    subsequent data fragmentation that results when smaller and larger
    blocks compete for space in ZFS allocation areas (metaslabs).

### Architecture

1.  ZFS already included the concept of separate storage classes
    associated with a ZFS pool to create write and read caches (SLOG and
    L2ARC, respectively) to improve storage performance. The Metadata
    Isolation feature adds a new allocation class (“special”) to hold
    specific ZFS metadata types and small block application data
    separate from large block application data.

    Allocation classes can be thought of as allocation tiers that are
    dedicated to specific block categories: the special class, which
    captures ZFS metadata and small block application data, will occupy
    a mirrored VDEV and the normal class, which captures all application
    data, will consume the dRAID or RAIDZ VDEV. Metadata Isolation is an
    independent feature from the underlying RAID mechanism used for the
    normal class data.

    Exercising the feature requires that each pool be comprised of at
    least one (special) class. As with the cache or log classes, a
    special allocation class VDEV can be added at the time the storage
    pool is created or it can be added at a later time. If the special
    allocation VDEV is written after the pool is created, only newly
    written metadata will reside in the newly added VDEV. In general,
    the capacity of an allocation class VDEV can be expanded by adding
    additional VDEVs to that class or by replacing existing VDEV devices
    with a larger device (via ZFS auto-expand).

    Each class type represents exclusive allocations but metadata types
    can be combined onto the associated VDEV. The normal class will
    accept all block types not being steered into the special class
    already and serves as the fallback allocator for all classes.

### Implementation

Isolating ZFS metadata and small block I/O to a separate mirrored VDEV
decreases fragmentation within the normal class so that allocations of
large, contiguous data blocks is less constrained and data can be
streamed more efficiently to and from each disk.

Intel demonstrated Metadata Isolation on dedicated mirrored VDEVs in
MS04. The cost of using a dedicated VDEV to isolate the metadata ,
however, is that if the disks in the VDEV are in the same enclosure as
the disks used for dRAID, then those mirrored disks are unavailable to
participate in the dRAID rebuild following a drive failure.

1.  For MS5, Intel developed a new Metadata Isolation to create a hybrid
    mirror VDEV that combined the mirrored ZFS metadata in separate
    regions of the dRAID VDEV. Metadata Isolation can be combined on the
    same disks with dRAID by defining class allocation functionality at
    the metaslab level and mixing metaslabs in the same VDEV
    ([PR-5182](https://github.com/zfsonlinux/zfs/pull/5182)). The Hybrid
    solution makes all drives available for data and ensures their full
    participation in the dRAID rebuild following a drive failure.

    Metadata Isolation enables different allocation policies for each
    ZFS data class. Application data can use RAIDZ VDEVs while ZFS
    metadata and small file system data can use mirrored VDEVs. There
    are two Opt-in variations for allocation classes: Dedicated and
    Segregated. **Error! Reference source not found.**

[]{#_Toc488392503 .anchor}Figure ‑[]{#_Ref487021960 .anchor}1.
Transition from Unmodified RAIDZ to Hybrid Mirror Configuration

  ------------------------------------------------------------------------------------------------------------------------------------
  1.  **(A) Unmodified RAIDZ**                                             1.  **(B) Metadata Isolation - Dedicated**
                                                                           
  ------------------------------------------------------------------------ -----------------------------------------------------------
  1.  ![](media/image2.png){width="3.01in" height="1.93in"}                1.  ![](media/image3.png){width="3.01in" height="1.93in"}
                                                                           

  1.  **(C) Metadata Isolation – Segregated (Hybrid Mirror dRAID only)**
  

  1.  ![](media/image4.png){width="3.01in" height="1.93in"}
  
  ------------------------------------------------------------------------------------------------------------------------------------

1.  The diagram above (1) illustrates the transition from unmodified
    RAIDZ configurations to the Hybrid Mirror configuration:

<!-- -->

A.  Unmodified RAIDz\
    The application data and ZFS metadata are co-allocated within ZFS
    metaslabs of RAIDZ VDEVs.

B.  Metadata Isolation – Dedicated\
    The mirrored VDEV for the ZFS metadata and small block I/O (i.e.,
    the “special” allocation class) is created from a separate set of
    physical VDEVs dedicated for the special class.

C.  Metadata Isolation -- Segregated\
    The mirrored VDEV is created from a set of metaslabs allocated from
    a dRAID VDEV. In this way, the metaslabs for the special class are
    segregated from the metaslabs used for the normal class.

<!-- -->

1.  These variations are mutually exclusive use cases. A VDEV may only
    use one type.

#### Dedicated VDEVs

1.  All metaslabs in the VDEV are dedicated to a specific allocation
    class category. A pool must always have at least one general
    (non-specified) VDEV when using dedicated VDEVs. This configuration
    is selected as an “Opt-in” using a VDEV class designation keyword
    when creating the VDEV. Valid designation keywords are :

  ---------------
  special | log
  ---------------

1.  The dedicated class can currently only be specified with mirrored
    VDEVs.

    Example Syntax:

  --------------------------------------------------------------------
  zpool create demo raidz &lt;disks&gt; special mirror &lt;disks&gt;
  
  zpool add demo special mirror &lt;disks&gt;
  --------------------------------------------------------------------

1.  The first command creates the *demo* pool with ‘special’ class on
    the specified mirror disks. The second command adds additional disks
    to the special mirror created by the 1^st^ command. Adding disks is
    only possible with dedicated VDEVs.

#### Segregated VDEVs

1.  In the segregated use case, a portion of a VDEV's metaslabs are set
    aside for a specific allocation class when the pool is created.
    Opt-in for this feature is global to the pool using a boolean pool
    property. The following properties have been defined:

  -----------------------
  segregate\_log=on
  
  segregate\_special=on
  -----------------------

These “segregate” properties can be combined if multiple segregation
categories are desired (e.g., segregate log class and segregate special
class from normal class).

Example Syntax:

  ----------------------------------------------------------------
  zpool create -o segregate\_special=on demo raidz &lt;disks&gt;
  ----------------------------------------------------------------

1.  This command creates a RAIDz pool named *demo* with segregation
    enabled for the special allocation class.

##### Segregation Percentage

1.  Segregated VDEVs can only be created during zpool creation. It is
    not currently possible to add additional segregated VDEVs to an
    allocation class at a later time, as is possible with dedicated
    VDEVs.

    By default, the following segregation limits are applied:

-   segregate\_log -- sets aside one metaslab per VDEV for the log
    class.

-   segregate\_special -- sets aside 20% of a VDEV's metaslabs for the
    special class.

1.  The segregated metaslabs are dynamically assigned using a first
    available algorithm when the VDEV is opened. Subsequent opens may
    shift which metaslabs are assigned, but once a metaslab is allocated
    from (i.e. it is activated), the preferred bias becomes persistent.

    The percentage allocated to the special class can be tuned to a
    maximum of 50 percent. Although the number of metaslabs representing
    the selected percentages is set at pool creation, the assignment of
    an individual metaslab to the class is deferred until the allocation
    is needed. Nonetheless, because the ditto block policy (Section
    3.2.2.2.2) requires writing ditto copies to three different
    metaslabs, the minimum number of metaslabs initially assigned to the
    “special” allocation class on a segregated metadata VDEV is three.

    ZFS metadata has priority for the special allocations. Since the
    special class includes allocations for both ZFS metadata and small
    block data, 5% of a segregated VDEV’s metaslabs are reserved for ZFS
    metadata. This policy will prevent small block usage of the special
    allocation from competing with ZFS metadata usage of the storage.
    Small block data can take advantage of the segregated VDEV as long
    as space is available, otherwise small block data can easily
    overflow to the normal class.

    Block category allocation accounting can be observed from the CLI
    (see zdb -mm and zpool list -C).

##### Ditto Block Policy

1.  Each ZFS block pointer structure has space for the three data
    virtual address (DVA) pointers. ZFS replicates its metadata and uses
    the DVAs to record the locations of these “ditto blocks.” When there
    is more than one VDEV, the ditto blocks are written to different
    VDEVs in the pool. When there is only one VDEV available and more
    than one DVA is required (ditto copies &gt; 1), the traditional
    ditto placement policy was to place the allocation a distance of
    1/8th of total VDEV allocation space away from the other DVAs. This
    policy put a burden on the allocator to find a metaslab 1/8th above
    or below the current allocation.

    A new, simplified ditto placement policy has been created to
    guarantee that the other DVAs simply land in a different metaslab.
    This policy in turn greatly simplifies ditto DVA placement from a
    segregated VDEV, where a group of metaslabs is not necessarily
    consecutive.

    To validate that the new policy is honored, a zdb(8) block audit
    will report any DVAs that landed in the same metaslab. The expected
    result is that there will be none:

  ------------------------------------------------------------------------
  \#zdb -b ssu\_1ost1
  
  Traversing all blocks to verify nothing leaked ...
  
  loading space map for vdev 0 of 1, metaslab 290 of 291 ...
  
  26.4T completed (357720MB/s) estimated time remaining: 0hr 00min 00sec
  
  No leaks (block sum matches space maps exactly)
  ------------------------------------------------------------------------

1.  If there is a policy failure, it will be manifest as a non-zero
    block audit, as shown in the following zdb audit output in which the
    ditto block allocation was manipulated to force the error:

  -------------------------------------
  Dittoed blocks in same metaslab: 21
  -------------------------------------

#### VDEV Changes

1.  Supporting the new allocation classes for segregated VDEVs required
    changes to the VDEV definition.

##### Feature Flag Encapsulation

1.  The feature@allocation\_classes becomes active when a unique
    allocation class is instantiated by a VDEV. Activating this feature
    makes the pool read-only on ZFS builds that do not support
    allocation classes.

##### VDEV Allocation Bias

1.  The ZFS concept of VDEV allocation bias was extended beyond the
    normal or log classes to include special and segregated classes.
    Instead of defining a number of Boolean flags, the allocation
    classes are now expressed in a runtime VDEV instance as an
    allocation bias:

  --------------------------------------------------------------------------
  typedef enum vdev\_alloc\_bias {
  
  VDEV\_BIAS\_NONE,
  
  VDEV\_BIAS\_LOG, /\* dedicated to ZIL data (SLOG) \*/
  
  VDEV\_BIAS\_DEDUP, /\* dedicated to DDT data \*/
  
  VDEV\_BIAS\_METADATA, /\* dedicated to metadata (DMU and MOS) \*/
  
  VDEV\_BIAS\_SPECIAL, /\* dedicated to small blocks \*/
  
  VDEV\_BIAS\_SEGREGATE, /\* segregated metaslabs into multiple groups \*/
  
  } vdev\_alloc\_bias\_t;
  --------------------------------------------------------------------------

1.  This VDEV allocation class bias information is stored in the
    per-vdev zap object as a string value:

  ---------------------------------------------------
  /\* vdev metaslab allocation bias \*/
  
  \#define VDEV\_ALLOC\_BIAS\_LOG "log"
  
  \#define VDEV\_ALLOC\_BIAS\_SPECIAL "special"
  
  \#define VDEV\_ALLOC\_BIAS\_SEGREGATE "segregate"
  ---------------------------------------------------

1.  The bias is also passed internally in the pool config during a zpool
    create and any internal zpool config query. This information can be
    used by functions in the zpool(8) command.

##### Metaslab Allocation Bias

1.  Class allocation bias occurs at a metaslab granularity. Each
    metaslab has an allocation bias which is assigned when the metaslab
    is initialized, based on the VDEV's allocation bias. The metaslab's
    allocation bias then determines which metaslab group to join.

    There is additional VDEV metadata stored in the
    VDEV\_TOP\_ZAP\_METASLAB\_INFO\_OBJ object. This object is an array
    of ms\_alloc\_phys structures, one per metaslab, which tracks the
    allocation bias assigned to a metaslab and the allocation stats by
    category:

  ---------------------------------------------------------------------------
  /\*
  
  \* Additional per-metaslab allocation info for dedicated/segregated vdevs
  
  \*/
  
  typedef struct ms\_alloc\_phys {
  
  uint64\_t ms\_alloc\_flags; /\* flags: ie segregated bias \*/
  
  uint64\_t ms\_alloc\_metadata; /\* metadata space allocated \*/
  
  uint64\_t ms\_alloc\_smallblks; /\* smallblks space allocated \*/
  
  uint64\_t ms\_alloc\_dedup; /\* dedup space allocated \*/
  
  } ms\_alloc\_phys\_t;
  ---------------------------------------------------------------------------

Metaslabs in dedicated VDEVs inherit the bias of the VDEV. But in
segregated VDEVs, the class allocation bias of a metaslab is assigned
when the metaslab is initialized. The metaslab's allocation bias then
determines which metaslab group to join.

  --------------------------------------------------
  /\*
  
  \* class allocation bias (segregated vdevs only)
  
  \*/
  
  typedef enum {
  
  MS\_ALLOC\_BIAS\_UNASSIGNED = 0x00,
  
  MS\_ALLOC\_BIAS\_LOG = 0x01,
  
  MS\_ALLOC\_BIAS\_SPECIAL = 0x02,
  
  MS\_ALLOC\_BIAS\_NORMAL = 0x03
  
  } ms\_alloc\_bias\_t;
  --------------------------------------------------

1.  []{#_Ref486592294 .anchor}

##### VDEV Allocation Stats

1.  Status of zpool VDEVs is available through the zpool list -v
    command, where the mirrored dedicated VDEVs are shown as distinct
    members of the pool. In the example below, two drives are mirrored
    to create a dedicated VDEV for the special allocation class. The
    pool also includes two dRAID virtual spare drives.

  -------------------------------------------
  \$ zpool list -v ost-d
  
  NAME SIZE ALLOC FREE EXPANDSZ FRAG CAP
  
  ost-d 16.6T 12.9G 16.6T - 0% 0%
  
  draid2 16.2T 11.2G 16.2T - 0% 0.06%
  
  wwn-0x5000c5007adc15a5 - - - - - -
  
  wwn-0x5000c5007adc6d2f - - - - - -
  
  wwn-0x5000c5007adcf3f4 - - - - - -
  
  wwn-0x5000c5007add017e - - - - - -
  
  wwn-0x5000c5007addaf56 - - - - - -
  
  wwn-0x5000c5007adc6d4a - - - - - -
  
  wwn-0x5000c5007b066251 - - - - - -
  
  wwn-0x5000c5007b067415 - - - - - -
  
  wwn-0x5000c5007b065a87 - - - - - -
  
  wwn-0x5000c5007add62b4 - - - - - -
  
  wwn-0x5000c5007addb524 - - - - - -
  
  wwn-0x5000c5007add4c29 - - - - - -
  
  wwn-0x5000c5007add5274 - - - - - -
  
  wwn-0x5000c5007add5c4b - - - - - -
  
  wwn-0x5000c5007adc7092 - - - - - -
  
  wwn-0x5000c5007add591d - - - - - -
  
  wwn-0x5000c5007b34afa6 - - - - - -
  
  wwn-0x5000c5007add870f - - - - - -
  
  wwn-0x5000c5007b06e13e - - - - - -
  
  wwn-0x5000c5007b067081 - - - - - -
  
  special:mirror 372G 1.62G 370G - 0% 0.43%
  
  wwn-0x55cd2e404c033d2e - - - - - -
  
  wwn-0x55cd2e404c033fac - - - - - -
  
  spare - - - - - -
  
  \$draid2-0-s0 - - - - - -
  
  \$draid2-0-s1 - - - - - -
  -------------------------------------------

1.  Segregated VDEVs, however, are essentially a subset of the main RAID
    VDEV and, as a result, status of a segregated VDEV is not available
    through the “zpool list -v” command. The special class allocation
    information was added to the vdev\_stat\_t structure to track the
    number of metaslabs assigned to the special class and the space used
    by the normal and special metaslabs that have been assigned.

  ------------------------------------------------------------------------------
  typedef struct vdev\_stat {
  
  ...
  
       uint64\_t        vs\_normal\_assigned;     /\* ms assigned space    \*/
  
       uint64\_t        vs\_special\_assigned;    /\* ms assigned space    \*/
  
       uint64\_t        vs\_special\_alloc;       /\* special allocated    \*/
  
  } vdev\_stat\_t;
  ------------------------------------------------------------------------------

This information is primarily used by the ‘zpool list –C’ command to
query the allocation info by class category and helps determine if
provisioning was done correctly.

  -------------------------------------------
  \# zpool list -C ssu\_1ost1
  
  NAME SIZE ALLOC FREE CAPACITY
  
  -------------- ----- ----- ----- --------
  
  ssu\_1ost1 72.7T 56.0T 16.8T 77.0%
  
  draid2-0 72.7T 56.0T 16.8T 77.0%
  
  normal 58.2T 55.8T 2.48T 95.8%
  
  special 2.25T 217G 2.04T 9.43%
  
  unassigned 12.2T 0 12.2T -
  -------------------------------------------

This example shows that 2.25TB have been assigned to the special class,
but only 217GB have been used. The normal class has allocated 55.8TB of
the 58.2TB available from the metaslabs assigned to this class. The pool
still has over 20% of its metaslabs (12.2TB) unassigned.

### Changes and Deviations

1.  There were no changes from the HLD or SOW.

### Demonstration

1.  We demonstrated aspects of different allocation class configurations
    using zpool(8), zdb(8) and kstat.

#### Hybrid Metadata/Smallblock Isolation with dRAID VDEVs

1.  This demonstration compared two dRAID zpools:

-   ssu\_1ost1 had VDEV segregation enabled to create a hybrid-mirror
    dRAID. This dRAID VDEV set aside a portion of its allocation areas
    (metaslabs) to host metadata and small blocks. The remaining areas
    were used for generic application data and are intended to stream
    larger 16MB block content.

-   ssu\_2ost0 had dRAID alone. In this configuration of dRAID,
    allocations were not differentiated by category. Each metaslab
    hosted data as it arrived, which could be any mixture of small or
    large data and ZFS metadata.

1.  Both dRAID pools had 43 drives in four 8+2 parity groups and 3
    spares. Note that the zpool status for both pools shows the three
    virtual spares “\$draid-xx”, but otherwise there is nothing to
    indicate that ssu\_1ost1 also had a hybrid mirror for the special
    allocation class.

  --------------------------------------------------------------------------------
  1.  dRAID with VDEV segregation enabled   1.  dRAID with no metadata isolation
                                            
  ----------------------------------------- --------------------------------------
  \# zpool status                           \# zpool status
                                            
    pool: ssu\_1ost1                        pool: ssu\_2ost0
                                            
   state: ONLINE                            state: ONLINE
                                            
    scan: none requested                    scan: none requested
                                            
  config:                                   config:
                                            
  1.  NAME STATE READ WRITE CKSUM           NAME STATE READ WRITE CKSUM
                                            
      ssu\_1ost1 ONLINE 0 0 0               ssu\_2ost0 ONLINE 0 0 0
                                            
      draid2-0 ONLINE 0 0 0                 draid2-0 ONLINE 0 0 0
                                            
      mpathfn ONLINE 0 0 0                  mpathdd ONLINE 0 0 0
                                            
      mpathfa ONLINE 0 0 0                  mpathfa ONLINE 0 0 0
                                            
      mpathcx ONLINE 0 0 0                  mpathcx ONLINE 0 0 0
                                            
      mpathbs ONLINE 0 0 0                  mpathbs ONLINE 0 0 0
                                            
      mpathan ONLINE 0 0 0                  mpathan ONLINE 0 0 0
                                            
      mpathu ONLINE 0 0 0                   mpathu ONLINE 0 0 0
                                            
      mpatheu ONLINE 0 0 0                  mpatheu ONLINE 0 0 0
                                            
      mpathdp ONLINE 0 0 0                  mpathdp ONLINE 0 0 0
                                            
      mpathck ONLINE 0 0 0                  mpathck ONLINE 0 0 0
                                            
      mpathbf ONLINE 0 0 0                  mpathbf ONLINE 0 0 0
                                            
      mpathaa ONLINE 0 0 0                  mpathaa ONLINE 0 0 0
                                            
      mpathh ONLINE 0 0 0                   mpathh ONLINE 0 0 0
                                            
      mpatheh ONLINE 0 0 0                  mpathfm ONLINE 0 0 0
                                            
      mpathdc ONLINE 0 0 0                  mpatheh ONLINE 0 0 0
                                            
      mpathfm ONLINE 0 0 0                  mpathdc ONLINE 0 0 0
                                            
      mpathaz ONLINE 0 0 0                  mpathaz ONLINE 0 0 0
                                            
      mpathcw ONLINE 0 0 0                  mpathcw ONLINE 0 0 0
                                            
      mpathbr ONLINE 0 0 0                  mpathbr ONLINE 0 0 0
                                            
      mpatham ONLINE 0 0 0                  mpatham ONLINE 0 0 0
                                            
      mpatht ONLINE 0 0 0                   mpatht ONLINE 0 0 0
                                            
      mpathdd ONLINE 0 0 0                  mpathet ONLINE 0 0 0
                                            
      mpathdo ONLINE 0 0 0                  mpathdo ONLINE 0 0 0
                                            
      mpathcj ONLINE 0 0 0                  mpathcj ONLINE 0 0 0
                                            
      mpathbe ONLINE 0 0 0                  mpathbe ONLINE 0 0 0
                                            
      mpathg ONLINE 0 0 0                   mpathg ONLINE 0 0 0
                                            
      mpathfl ONLINE 0 0 0                  mpathfl ONLINE 0 0 0
                                            
      mpatheg ONLINE 0 0 0                  mpatheg ONLINE 0 0 0
                                            
      mpathdb ONLINE 0 0 0                  mpathdb ONLINE 0 0 0
                                            
      mpathay ONLINE 0 0 0                  mpathay ONLINE 0 0 0
                                            
      mpathcv ONLINE 0 0 0                  mpathcv ONLINE 0 0 0
                                            
      mpathbq ONLINE 0 0 0                  mpathbq ONLINE 0 0 0
                                            
      mpathal ONLINE 0 0 0                  mpathal ONLINE 0 0 0
                                            
      mpaths ONLINE 0 0 0                   mpaths ONLINE 0 0 0
                                            
      mpathfx ONLINE 0 0 0                  mpathfx ONLINE 0 0 0
                                            
      mpathes ONLINE 0 0 0                  mpathes ONLINE 0 0 0
                                            
      mpathdn ONLINE 0 0 0                  mpathdn ONLINE 0 0 0
                                            
      mpathci ONLINE 0 0 0                  mpathci ONLINE 0 0 0
                                            
      mpathbd ONLINE 0 0 0                  mpathbd ONLINE 0 0 0
                                            
      mpathf ONLINE 0 0 0                   mpathfk ONLINE 0 0 0
                                            
      mpathfk ONLINE 0 0 0                  mpathef ONLINE 0 0 0
                                            
      mpathef ONLINE 0 0 0                  mpathda ONLINE 0 0 0
                                            
      mpathda ONLINE 0 0 0                  mpathax ONLINE 0 0 0
                                            
      mpathax ONLINE 0 0 0                  mpathei ONLINE 0 0 0
                                            
      spares                                spares
                                            
      \$draid2-0-s0 AVAIL                   \$draid2-0-s0 AVAIL
                                            
      \$draid2-0-s1 AVAIL                   \$draid2-0-s1 AVAIL
                                            
      \$draid2-0-s2 AVAIL                   \$draid2-0-s2 AVAIL
                                            
      errors: No known data errors          errors: No known data errors
                                            
  --------------------------------------------------------------------------------

The allocation data for the dRAID with segregation enabled can be seen
with the ‘zpool list –C’ command, as was shown above (section 0).

The Special Class used in these examples comes from enabling
segregation. For dRAID, this is an automatic opt-in as it makes sense to
join the two features. This opt-in can further be observed by examining
the following pool properties using ‘zpool get’. These properties are
automatically set with dRAID and are read-only.

  -----------------------------------------------------------------------------
  \# zpool get feature@allocation\_classes,segregate\_special,smallblkceiling
  
  NAME PROPERTY VALUE SOURCE
  
  ssu\_1ost1 feature@allocation\_classes active local
  
  ssu\_1ost1 segregate\_special on local
  
  ssu\_1ost1 smallblkceiling 32768 local
  -----------------------------------------------------------------------------

1.  Using the ZFS kstat framework, one can track the allocations
    occuring in each of the pool allocation classes while the file
    system is running.

  -------------------------------------------------
  1.  cat /proc/spl/kstat/zfs/alloc\_class\_stats
  
      name type data
  
      normal\_allocated 4 61325031915520
  
      normal\_highest\_allocated 4 61325037486080
  
      special\_allocated 4 233487310848
  
      special\_highest\_allocated 4 233536917504
  
      slog\_allocated 4 0
  
      slog\_highest\_allocated 4 0
  
  -------------------------------------------------

#### Observing Metaslab Regions

1.  Using the zdb(8) tool, one can observe the underlying metaslabs in a
    zpool. With VDEV segregation enabled, ZFS will set aside a portion
    (20% by default) of these regions to service small blocks and
    metadata (section 3.2.2.2).  The remaining regions are used for
    generic application data and large streaming I/O.

    The ‘zdb –m’ command provides copious output. The zpools created for
    this demonstration each had 290 metaslabs. The listing for a pool
    with dRAID alone (ssu\_2ost0) is shown in Appendix A.1. A fragment
    of this listing is below (the size column has been deleted to make
    the data

  ----------------------------------------------------------------------
  1.  \# zdb -m ssu\_2ost0
  
      Metaslabs:
  
      vdev 0
  
      metaslabs 291 offset spacemap free
  
      --------------- ------------------- --------------- ------------
  
      metaslab 0 offset 0 spacemap 114 free 7.36M
  
      metaslab 1 offset 4000006000 spacemap 113 free 1.64G
  
      metaslab 2 offset 8000002000 spacemap 112 free 861M
  
      metaslab 3 offset c000008000 spacemap 123 free 1.04G
  
      metaslab 4 offset 10000004000 spacemap 122 free 1.07G
  
      . . . .
  
      metaslab 286 offset 478000006000 spacemap 407 free 928M
  
      metaslab 287 offset 47c000002000 spacemap 406 free 658M
  
      metaslab 288 offset 480000008000 spacemap 405 free 931M
  
      metaslab 289 offset 484000004000 spacemap 409 free 726M
  
      metaslab 290 offset 488000000000 spacemap 408 free 583M
  
  ----------------------------------------------------------------------

1.  The listing for the dRAID pool segregation enabled is show in
    Appendix A.2. With segregation enabled, the listing now includes an
    additional column for class. There are three entries possible :

    -   ‘special’ means the metaslab is assigned to the special
        allocation class. This metaslab is part of a mirrored VDEV that
        contains ZFS metadata and/or small block data.

    -   ‘normal’ means the metaslab is assigned to the normal class.
        This metaslab is part of the dRAID VDEV and contains large block
        application data.

    -   ‘----‘ means the metaslab is unassigned. It can be assigned to
        the ‘special’ or ‘normal’ class as soon as ZFS needs an
        allocation for that class.

  ---------------------------------------------------------------------------
  \# zdb -m ssu\_lost1
  
  Metaslabs:
  
  vdev 0 segregate
  
  metaslabs 291 offset spacemap free class
  
  --------------- ------------------- --------------- ------------ --------
  
  metaslab 0 offset 0 spacemap 115 free 122G special
  
  metaslab 1 offset 4000000000 spacemap 114 free 208G special
  
  metaslab 2 offset 8000001000 spacemap 113 free 221G special
  
  metaslab 3 offset c000001000 spacemap 4 free 256G special
  
  metaslab 4 offset 10000002000 spacemap 3 free 256G special
  
  metaslab 5 offset 14000000000 spacemap 2 free 256G special
  
  metaslab 6 offset 18000000000 spacemap 7 free 256G special
  
  metaslab 7 offset 1c000001000 spacemap 6 free 256G special
  
  metaslab 8 offset 20000001000 spacemap 5 free 256G special
  
  metaslab 9 offset 24000002000 spacemap 0 free 256G ----
  
  metaslab 10 offset 28000000000 spacemap 0 free 256G ----
  
  metaslab 11 offset 2c000000000 spacemap 0 free 256G ----
  
  metaslab 12 offset 30000001000 spacemap 0 free 256G ----
  
  metaslab 13 offset 34000001000 spacemap 0 free 256G ----
  
  . . . .
  
  metaslab 58 offset e8000008000 spacemap 123 free 7.70G normal
  
  metaslab 59 offset ec000004000 spacemap 125 free 1.43G normal
  
  metaslab 60 offset f0000000000 spacemap 124 free 1.58G normal
  
  metaslab 61 offset f4000006000 spacemap 126 free 1.27G normal
  
  metaslab 62 offset f8000002000 spacemap 127 free 1.66G normal
  
  metaslab 63 offset fc000008000 spacemap 128 free 2.05G normal
  
  metaslab 64 offset 100000004000 spacemap 129 free 2.23G normal
  
  metaslab 65 offset 104000000000 spacemap 130 free 2.01G normal
  
  metaslab 66 offset 108000006000 spacemap 131 free 1.59G normal
  
  . . . .
  
  metaslab 284 offset 470000004000 spacemap 349 free 12.7G normal
  
  metaslab 285 offset 474000000000 spacemap 350 free 21.9G normal
  
  metaslab 286 offset 478000006000 spacemap 351 free 19.0G normal
  
  metaslab 287 offset 47c000002000 spacemap 352 free 1.16G normal
  
  metaslab 288 offset 480000008000 spacemap 353 free 912M normal
  
  metaslab 289 offset 484000004000 spacemap 354 free 902M normal
  
  metaslab 290 offset 488000000000 spacemap 355 free 1.35G normal
  ---------------------------------------------------------------------------

#### Observing Free Space Fragmentation

We can observe the free space fragmentation details of each metaslab by
running ‘zdb –mm’ to dump histograms of data allocations in each
metaslab. The free space fragmentation affects the new data block
allocations and the resulting I/O performance of new files.

In the samples below, the fragmentation histograms are the free segments
for a power-of-two size. So 2\^13 represents 8KB free chunks and 2\^24
represents 16MB free chunks. After an aging run, there typically are no
free regions in the non-segregated pool large enough to satisfy a 16MB
block on the pool with segregation disabled. In that case, ZFS would
have to stitch together a set of blocks to satisfy a 16MB block request.

As expected, the pool with large and small block isolation provided by
segregation has different fragmentation characteristics. For a metaslab
that is servicing small blocks and metadata, it is acceptable to have
lots of smaller blocks that are free since later small allocations can
fill in those holes. For a metaslab that is servicing larger blocks, it
would ideally have plenty of larger contiguous ares from which to draw
from. In the segregated pool, there are still 106+2\*9+4\*1=128 16MB
chunks free in the normal class.

  --------------------------------------------------------------------------------------------
  metaslab 34 offset 88000004000 size 3fffffc000 spacemap 153 free 21.1G
  
  On-disk histogram: fragmentation 21
  
  15: 31492 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
  
  16: 9869 \*\*\*\*\*\*\*\*\*\*\*\*\*
  
  17: 2356 \*\*\*
  
  18: 1665 \*\*\*
  
  19: 2275 \*\*\*
  
  20: 3543 \*\*\*\*\*
  
  21: 2593 \*\*\*\*
  
  22: 1097 \*\*
  --------------------------------------------------------------------------------------------

As expected, the pool with large and small block isolation provided by
segregation has different fragmentation characteristics. For a metaslab
that is servicing small blocks and metadata (special class), it is
acceptable to have lots of smaller blocks that are free since later
small allocations can fill in those holes.

  ---------------------------------------------------------------------------------------------
  metaslab 2 offset 8000001000 size 3ffffff000 spacemap 113 free 221G special
  
  On-disk histogram: fragmentation 8
  
  13: 448909 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
  
  14: 161708 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
  
  15: 91420 \*\*\*\*\*\*\*\*\*
  
  16: 29406 \*\*\*
  
  17: 13000 \*\*
  
  18: 6912 \*
  
  19: 5800 \*
  
  20: 2823 \*
  
  21: 2205 \*
  
  22: 1317 \*
  
  23: 819 \*
  
  24: 431 \*
  
  25: 211 \*
  
  26: 163 \*
  
  27: 135 \*
  
  28: 2 \*
  
  29: 0
  
  30: 0
  
  31: 0
  
  32: 0
  
  33: 0
  
  34: 0
  
  35: 0
  
  36: 1 \*
  ---------------------------------------------------------------------------------------------

For a metaslab that is servicing larger blocks (normal class), it would
ideally have plenty of larger contiguous ares from which to draw from.
In the segregated pool, there are still 106+2\*9+4\*1=128 16MB chunks
free in the normal class.

  -------------------------------------------------------------------------------------------
  metaslab 77 offset 134000002000 size 3fffffe000 spacemap 142 free 19.9G normal
  
  On-disk histogram: fragmentation 13
  
  16: 285 \*\*\*
  
  17: 484 \*\*\*\*\*
  
  18: 810 \*\*\*\*\*\*\*\*
  
  19: 1449 \*\*\*\*\*\*\*\*\*\*\*\*\*\*
  
  20: 4302 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
  
  21: 1997 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
  
  22: 839 \*\*\*\*\*\*\*\*
  
  23: 107 \*
  
  24: 106 \*
  
  25: 9 \*
  
  26: 1 \*
  -------------------------------------------------------------------------------------------

#### Observing Allocations by Category

1.  The ‘zdb –mm’ command also includes an Allocation Summary section
    that shows what allocations were made by category. This can be used
    to confirm that the metaslab regions are allocating from the
    expected class. Both examples below are from a dump of a zpool with
    segregation enabled.

    For a normal class metaslab, the Allocation Summary shows that all
    of the blocks allocated to the metaslab are in the generic category.

  ----------------------------------------------------------------------------------
  metaslab 75 offset 12c000000000 size 4000000000 spacemap 141 free 24.1G normal
  
  Allocation Summary: 232G allocated
  
  metadata: 0.0%
  
  smallblks: 0.0%
  
  dedup: 0.0%
  
  generic: 100.0% \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
  ----------------------------------------------------------------------------------

For a special class metaslab, the blocks allocated belong to the
metadata and small block categories.

  ------------------------------------------------------------------------------
  metaslab  0  offset    0   size 4000000000  spacemap 115  free 122G  special
  
  Allocation Summary:          134G allocated
  
             metadata:  62.1%  \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
  
            smallblks:  37.9%  \*\*\*\*\*\*\*\*\*\*\*\*
  
                dedup:   0.0%
  
              generic:   0.0%
  ------------------------------------------------------------------------------

A normal class allocation may include metadata and small block
categories as well as generic. But a special class allocation will only
hold metadata and small blocks. The special class cannot hold a generic
category allocation.

End-to-End 16 MB File Block I/O 
--------------------------------

### Architecture

1.  The best performance from a file system results when the underlying
    physical storage achieves peak performance. As discussed in the
    Solution Architecture, Intel’s analysis of random I/O workloads
    indicated that exceeding 100 MB/s per hard disk drive would be
    possible with 2 MB block I/Os from ZFS to disk. Since Aurora will
    use 8+2 double parity RAID, providing each drive with 2MB I/Os, this
    will require the Lustre servers to handle 16 MB file I/Os from
    Lustre clients and to pass these 16MB I/Os to ZFS.

    The demonstration configuration (Figure 3‑2) will show that the new
    software stack is able to

    -   send 16MB I/Os from the Lustre clients across the network to the
        Lustre OSS,

    -   pass a 16MB block through the Lustre OSD layer into ZFS,

    -   spread the 16MB I/O through ZFS across the 8 data drives in a
        parity group, and

    -   send 2MB IOs through the Linux block layer to each disk.

    The demonstration will also show that the metadata segregation tier
    in the zpool will receive the metadata and small IOs to keep them
    from being mixed into the Lustre Streaming large IOs.

[[[[]{#_Toc488392504 .anchor}]{#_Toc486599629 .anchor}]{#_Toc486606481
.anchor}]{#_Ref486515772 .anchor}Figure ‑. End to End Data Flow

1.  ![](media/image5.png){width="3.59375in"
    height="4.008413167104112in"}

    [[[[]{#_Toc462301971 .anchor}]{#_Toc462300786
    .anchor}]{#_Toc462267085 .anchor}]{#_Toc462216581 .anchor}

### Implementation

Our previous demonstration of 16MB I/Os did not require new
implementation, but it did combine work from different sources to ensure
that both Lustre and ZFS had the support necessary to handle 16MB file
blocks.

Lustre 2.9, which we had used for MS04, was no longer receiving
additional maintenance updates and we were forced for MS05 to move to
use Lustre 2.9.59, a pre-release version of Lustre 2.10. Though still
undergoing qualification testing, this version includes several critical
patches for issues we had encountered during our project development.
For example:

  [LU-9659](https://jira.hpdd.intel.com/browse/LU-9659)   lnet assert after timeout on reconnect
  ------------------------------------------------------- ---------------------------------------------------------------------
  [LU-9504](https://jira.hpdd.intel.com/browse/LU-9504)   LBUG ptlrpc\_handle\_rs()) ASSERTION( lock != ((void \*)0) ) failed
  [LU-9305](https://jira.hpdd.intel.com/browse/LU-9305)   osd-zfs: arc\_buf could be non-pagesize aligned

When released, Lustre 2.10 will include a number of new feature
additions from the Community, for example:

  LU-7734   Lnet Multi-rail Project
  --------- -------------------------------
  LU-8900   Simplified Userspace snapshot
  LU-8998   Progressive File Layout (PFL)
  LU-6283   NRS Delay Policy (Cray)
  LU-4017   Project Quotas (DDN)

ZFS on Linux (ZoL) is the source repository for OpenZFS for Lustre. The
ZoL community is working to release 0.7.0 this summer. The current
stable pre-release version is tagged as
[v0.7.0-rc4](https://github.com/zfsonlinux/zfs/releases/tag/zfs-0.7.0-rc4).

Our final single code repository combined Lustre 2.9.59 with ZFS on
Linux 0.7.0 RC4, with the inclusion of our patches for dRAID and
Metadata Isolation. As we began more extensive testing with this stack,
we encountered a series of bugs around large page allocation and memory
alignment with Lustre. We have added test cases into the Lustre
continuous integration environment to be run on every patch to Lustre to
catch these issues. As a result, we are now finding duplicate bugs and
our original bug report,
[LU-9305](https://jira.hpdd.intel.com/browse/LU-9305), has become an
umbrella for these related issues.

Though the Lustre team fixed some of the bugs we encountered, remaining
issues prevented us from using 16MB blocks end-to-end with the
integrated stack for performance tuning and stability testing. This will
be the work Intel continues to do as part of our efforts to get the code
accepted into the upstream repository and put into a release.
[[]{#_Toc462224975 .anchor}]{#_Toc462220606 .anchor}In preparation for
the demonstration, however, we were able to mitigate these issues by
using 8MB blocks. Using 8MB blocks enabled us to get more testing
completed before encountering the Lustre error. The Lustre team is
targeting resolution of LU-9305 in the 2.11 release.

### Demonstration

The demonstration consisted of two parts: End-to-End Streaming, to show
the transfer of 16MB from Lustre clients to the ZFS, and Fragmentation
Improvements, to show the improved performance with Metadata Isolation
(Section 3.2). For both demos, we used the Natasha cluster (Figure 3‑2),
which had 8 Lustre clients and 4 Lustre OSSs. Each OSS had a single 43
drive dRAID2 OST (Section 2.1.1).

#### Configuring the file system for 16MB I/Os

Each file system component along the I/O path must be configured to
enable 16MB I/O. Starting from the Lustre server, 16MB I/Os are set
first at ZFS, then Linux, then Lustre OSS, and then, finally, the Lustre
client.

##### ZFS on the Lustre OSS

On each Lustre OSS, set ZFS to accept and use 16MB I/Os with the
following steps:

1.  Set “zfs\_max\_recordsize” to 16MB (16777216).

  ---------------------------------------------------------------------------------
  \# echo "**16777216**" &gt; /sys/module/zfs/parameters/**zfs\_max\_recordsize**
  ---------------------------------------------------------------------------------

1.  

<!-- -->

1.  Create the zpool while specifying a 16MB record size using the
    “recordsize” option.

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  \# zpool create -o ashift=12 -o segregate\_special=on -o cachefile=none **-O recordsize=16MB** ssu\_1ost1 draid2 cfg=test\_2\_8\_3\_43\_draidcfg.nvl mpathfn mpathfa mpathcx mpathbs mpathan mpathu mpatheu mpathdp mpathck mpathbf mpathaa mpathh mpatheh mpathdc mpathfm mpathaz mpathcw mpathbr mpatham mpatht mpathdd mpathdo mpathcj mpathbe mpathg mpathfl mpatheg mpathdb mpathay mpathcv mpathbq mpathal mpaths mpathfx mpathes mpathdn mpathci mpathbd mpathf mpathfk mpathef mpathda mpathax
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

1.  The “-o ashift=12” option is only necessary to force 4KB sectors on
    hard drives that pretend to have 512-byte sectors for backward
    compatibility.

<!-- -->

1.  Enable the ZFS feature@large\_blocks flag for the zpool. Verify the
    feature with the zpool get all command.

  -------------------------------------------------------
  \# zpool **feature@large\_blocks=enabled** ssu\_1ost1
  
  \# **zpool get all** ssu\_1ost1 |grep large\_blocks
  
  ssu\_1ost1 feature@large\_blocks active local
  -------------------------------------------------------

##### Linux on the Lustre OSS

The Linux block I/O layer for each disk drive on the Lustre OSS must be
configured to handle 2MB I/Os. This is done by setting the
max\_sectors\_kb parameter to 4096 (512B/sector \* 4096 sectors = 2MB)
and the scheduler to noop. The following script was run before the
demonstration started:

  -------------------------------------------------------------------------------
  for i in \$(find /sys/devices -print |grep **max\_sectors\_kb** |grep -v ata)
  
  do
  
  echo 4096 &gt; \$i
  
  done
  
  for x in \$(find /sys/devices -print |grep **scheduler** |grep -v ata)
  
  do
  
  echo noop &gt; \$x
  
  done
  -------------------------------------------------------------------------------

##### Lustre OSS

The Lustre OSS itself is configured to use 16MB by using the Lustre
control interface, lctl, to set the obdfilter read and write size
(brw\_size) to 16:

  ------------------------------------------------------
  \# **lctl** set\_param **obdfilter.\*.brw\_size**=16
  
  obdfilter.nlsdraid-OST0001.brw\_size=16
  ------------------------------------------------------

##### Lustre Client 

The RPC size used by the Lustre client is controlled by the
max\_pages\_per\_rpc parameter. Each page is 4096 bytes. By default,
Lustre sets max\_pages\_per\_rpc to 256 to use 1MB RPCs
(256\*4096=1048576). Starting in Lustre 2.9, it is possible to raise the
parameter to 4096 to use 16MB RPCs (4096\*4096 = 16MB). To make this
change, we use pdsh to run lctl on each compute node to set the RPC size
on the Lustre client for each OSS connection.

  -----------------------------------------------------------------------------------------
  \# pdsh -w node0\[1-8\] "/usr/sbin/lctl set\_param **osc.\*.max\_pages\_per\_rpc=16M**"
  
  node01: osc.nlsdraid-OST0000-osc-ffff8820228e3000.max\_pages\_per\_rpc=4096
  
  node01: osc.nlsdraid-OST0001-osc-ffff8820228e3000.max\_pages\_per\_rpc=4096
  
  node01: osc.nlsdraid-OST0002-osc-ffff8820228e3000.max\_pages\_per\_rpc=4096
  
  node01: osc.nlsdraid-OST0003-osc-ffff8820228e3000.max\_pages\_per\_rpc=4096
  
  ...
  
  node04: osc.nlsdraid-OST0000-osc-ffff8816686ea000.max\_pages\_per\_rpc=4096
  
  node04: osc.nlsdraid-OST0001-osc-ffff8816686ea000.max\_pages\_per\_rpc=4096
  
  node04: osc.nlsdraid-OST0002-osc-ffff8816686ea000.max\_pages\_per\_rpc=4096
  
  node04: osc.nlsdraid-OST0003-osc-ffff8816686ea000.max\_pages\_per\_rpc=4096
  -----------------------------------------------------------------------------------------

#### Prepping Lustre Counters

1.  Lustre maintains a number of useful counters on the client and
    server to help evaluate the performance of different components of
    the file system. For the End-to-End 16MB demonstration, we used the
    “rpc\_stats” structures on the client and “brw\_stats” structure on
    the server. The scripts used during the demonstration are described
    below.

##### RPC Stats

1.  RPC stats are kept on the Lustre client to show the distribution of
    RPCs issued by the client to the Lustre server. The rpc\_stats
    variable on each client can be cleared before the test and the read
    after the test completes.

###### clear\_rpc.sh

1.  The clear\_rpc.sh script cleared the rpc\_stats counter structure on
    all 8 clients on the Natasha cluster. This script is run before each
    performance test.

  ---------------------------------------------------------------------
  \#!/bin/bash
  
  for host in node01 node02 node03 node04 node05 node06 node07 node08
  
  do
  
  echo "clear on \$host"
  
  ssh \$host "/usr/sbin/lctl set\_param osc.\*.rpc\_stats=0"
  
  echo
  
  done
  ---------------------------------------------------------------------

When complete, the script shows that the counters have been zeroed:

  --------------------------------------------------------
  \# ./clear\_rpc.sh
  
  clear on node01
  
  osc.nlsdraid-OST0000-osc-ffff8820228e3000.rpc\_stats=0
  
  osc.nlsdraid-OST0001-osc-ffff8820228e3000.rpc\_stats=0
  
  osc.nlsdraid-OST0002-osc-ffff8820228e3000.rpc\_stats=0
  
  osc.nlsdraid-OST0003-osc-ffff8820228e3000.rpc\_stats=0
  
  ...
  
  clear on node08
  
  osc.nlsdraid-OST0000-osc-ffff880eea3f7000.rpc\_stats=0
  
  osc.nlsdraid-OST0001-osc-ffff880eea3f7000.rpc\_stats=0
  
  osc.nlsdraid-OST0002-osc-ffff880eea3f7000.rpc\_stats=0
  
  osc.nlsdraid-OST0003-osc-ffff880eea3f7000.rpc\_stats=0
  --------------------------------------------------------

###### show\_rpc.sh

1.  The show\_rpc script displays the rpc\_stats structure from each
    Lustre client.

  ---------------------------------------------------------------------------------------------------------------------------------------------------------
  \#!/bin/bash
  
  for host in node01 node02 node03 node04 node05 node06 node07 node08
  
  do
  
  ssh \$host "/bin/hostname; cat /proc/fs/lustre/osc/nlsdraid-OST000\[1-3\]\*/rpc\_stats | grep -A14 'pages per' " |egrep --color=always '.\*4096:.\*|\$'
  
  echo
  
  done
  ---------------------------------------------------------------------------------------------------------------------------------------------------------

The output from this script is used to evaluate how the Lustre clients
sent RPCs to the servers, as shown in Section 3.3.3.3.1.

##### BRW Stats

The Lustre server maintains counters for the block I/O requests that it
sends to the underlying Linux file system. The counters are cleared
before a test, and then read afterwards.

###### clear\_brw.sh

1.  The Natasha cluster had 4 Lustre OSS. This script cleared the
    brw\_stats structure on each server.

  --------------------------------------------------------------------------------------------------------------------------------
  \#!/bin/bash
  
  for host in lustre1 lustre2 lustre3 lustre4
  
  do
  
  \#ssh \$host "/bin/hostname; cat /proc/fs/lustre/osd-zfs/\*/brw\_stats|grep -A36 'size' " |egrep --color=always '.\*16M.\*|\$'
  
  ssh \$host "lctl set\_param obdfilter.\*.brw\_stats=0"
  
  echo
  
  done
  --------------------------------------------------------------------------------------------------------------------------------

1.  When complete, the script shows that the servers have been zeroed:

  -----------------------------------------
  \# ./clear\_brw.sh
  
  obdfilter.nlsdraid-OST0000.brw\_stats=0
  
  obdfilter.nlsdraid-OST0001.brw\_stats=0
  
  obdfilter.nlsdraid-OST0002.brw\_stats=0
  
  obdfilter.nlsdraid-OST0003.brw\_stats=0
  -----------------------------------------

###### show\_brw.sh

1.  The show\_brw script shows the block distribution sent to the
    underlying storage.

  ------------------------------------------------------------------------------------------------------------------------------
  \#!/bin/bash
  
  for host in lustre1 lustre2 lustre3 lustre4
  
  do
  
  ssh \$host "/bin/hostname; cat /proc/fs/lustre/osd-zfs/\*/brw\_stats|grep -A36 'size' " |egrep --color=always '.\*16M.\*|\$'
  
  echo
  
  done
  ------------------------------------------------------------------------------------------------------------------------------

The output from this script is shown in Section 3.3.3.3.2.

#### End to End Streaming

1.  We used IOR to generate 16MB I/O from the clients using file per
    process with a sequential workload.

  -----------------------------------------------------------------------------------------------------------
  \# mpirun -wdir /mnt/lustre -np 8 -machinefile hosts /root/natasha-bin/ior -F -i 1 -s 20480 -b 16m -t 16m
  
  ...
  
  Command line used: /root/natasha-bin/ior -F -i 1 -s 20480 -b 16m -t 16m
  
  Machine: Linux node01
  
  Test 0 started: Tue Jun 20 14:37:07 2017
  
  Summary:
  
  api = POSIX
  
  test filename = testFile
  
  access = file-per-process
  
  ordering in a file = sequential offsets
  
  ordering inter file= no tasks offsets
  
  clients = 8 (1 per node)
  
  repetitions = 1
  
  xfersize = 16 MiB
  
  blocksize = 16 MiB
  
  aggregate filesize = 2560 GiB
  -----------------------------------------------------------------------------------------------------------

1.  We configured the IOR test so that each of the four OSS received I/O
    from two different clients:

  ------------------------------------
  node01 → testFile.00000000 on ost0
  
  node02 → testFile.00000001 on ost1
  
  node03 → testFile.00000002 on ost2
  
  node04 → testFile.00000003 on ost3
  
  node05 → testFile.00000004 on ost0
  
  node05 → testFile.00000005 on ost1
  
  node07 → testFile.00000006 on ost2
  
  node08 → testFile.00000007 on ost3
  ------------------------------------

1.  While the workload tests ran, we used a number of Linux tools to
    expose the I/O sizes at each step of the I/O flow, from the Lustre
    clients to the ZFS disk devices. These tools are summarized in Table
    3‑2 and show at what point each tool is used in the I/O flow.

[[[[]{#_Toc488392499 .anchor}]{#_Toc486599636 .anchor}]{#_Toc486606486
.anchor}]{#_Ref486598358 .anchor}Table ‑. I/O Size Evaluation Tools

  -----------------------------------------------------------------------------------------------------------------------------------------------------------
          Metric            Tool                                      Run on                      Display
  ------- ----------------- ----------------------------------------- --------------------------- -----------------------------------------------------------
  1.  1   1.  RPC Stats     Lctl get\_param osc.\*.rpc\_stats         1.  Lustre Client           1.  Histogram of the RPC sizes and number from the client
                                                                                                  

  1.  2   1.  BRW Stats     Lctl get\_param obdfilter.\*.brw\_stats   1.  Lustre Server (OST)     1.  Histograms of RPC sizes received on each OST
                                                                                                  

  1.  3   1.  ZFS iostats   Zpool iostat –r ost0                      1.  Lustre Server (ZFS)     1.  Tables of request sizes in each zpool issued to disk
                                                                                                  

  1.  4   1.  Disk Stats    visualized with netdump                   1.  Lustre Server (Linux)   1.  graphic display of
                                                                                                  

  -----------------------------------------------------------------------------------------------------------------------------------------------------------

##### Lustre Client RPC stats

Before running the IOR test, the clear\_rpc.sh script (section
3.3.3.2.1.1) cleared the rcp\_stats structure on each Lustre client and
the clear\_brw.sh script (section 3.3.3.2.2.1) cleared the brw\_stats
structure on each Lustre OSS.

The show\_rpc.sh script (section 3.3.3.2.1.2) displayed the rpc\_stats
structure from each client. With 4KB sized pages, 4096 pages per RPC
represent 16MB per RPC. All clients showed a result similar to the one
below in which all RPCs sent during the IOR run work 16MB in size.

  -------------------------------------------
  node01 ost0
  
  pages per rpc rpcs % cum % | rpcs % cum %
  
  1: 0 0 0 | 0 0 0
  
  2: 0 0 0 | 0 0 0
  
  4: 0 0 0 | 0 0 0
  
  8: 0 0 0 | 0 0 0
  
  16: 0 0 0 | 0 0 0
  
  32: 0 0 0 | 0 0 0
  
  64: 0 0 0 | 0 0 0
  
  128: 0 0 0 | 0 0 0
  
  256: 0 0 0 | 0 0 0
  
  512: 0 0 0 | 0 0 0
  
  1024: 0 0 0 | 0 0 0
  
  208: 0 0 0 | 0 0 0
  
  4096: 0 0 0 | **3500 100 100**
  -------------------------------------------

##### Lustre Server BRW stats

We then verified that 16MB IO blocks were sent through the Lustre server
with the show\_brw.sh script (section 3.3.3.2.2.2):

  -----------------------------------------
  \# ./show\_brw.sh
  
  ssu1\_oss1
  
  disk I/O size ios % cum % | ios % cum %
  
  16M: 0 0 0 | 2048 100 100
  
  ssu1\_oss2
  
  disk I/O size ios % cum % | ios % cum %
  
  16M: 0 0 0 | 2048 100 100
  
  ssu2\_oss1
  
  disk I/O size ios % cum % | ios % cum %
  
  16M: 0 0 0 | 314290 100 100
  
  ssu2\_oss2
  
  disk I/O size ios % cum % | ios % cum %
  
  16M: 0 0 0 | 306408 100 100
  -----------------------------------------

##### ZFS I/O Sizes 

Using the ‘zpool iostat’ command, it is possible to see how the 16MB
I/Os from Lustre are converted to disk I/Os:

  ----------------------------------------------------------------------------------------------
  \$ zpool iostat -r
  
  ssu\_2ost0     sync\_read     sync\_write    async\_read    async\_write   scrub  
  
  req\_size      ind    agg    ind    agg    ind    agg    ind    agg    ind    agg
  
  ----------  -----  -----  -----  -----  -----  -----  -----  -----  -----  -----
  
  512             0      0      0      0      0      0      0      0      0      0
  
  1K              0      0      0      0      0      0      0      0      0      0
  
  2K              0      0      0      0      0      0      0      0      0      0
  
  **4K**              0      0      0      0      0      0    **171**      0      0      0
  
  **8K**              0      0      0      0      0      0      **3**     **42**      0      0
  
  16K             0      0      0      0      0      0      0      0      0      0
  
  32K             0      0      0      0      0      0      0      0      0      0
  
  64K             0      0      0      0      0      0      0      0      0      0
  
  128K            0      0      0      0      0      0      0      0      0      0
  
  256K            0      0      0      0      0      0      0      0      0      0
  
  512K            0      0      0      0      0      0      0      0      0      0
  
  1M              0      0      0      0      0      0      0      0      0      0
  
  **2M**              0      0      0      0      0      0  **2.05K**      0      0      0
  
  4M              0      0      0      0      0      0      0      0      0      0
  
  8M              0      0      0      0      0      0      0      0      0      0
  
  16M             0      0      0      0      0      0      0      0      0      0
  ----------------------------------------------------------------------------------------------

A plot of the output (Figure 3‑3) clearly shows that the IOR test
generated mostly 2MB write I/Os to disk.

[[]{#_Toc488392505 .anchor}]{#_Ref486835808 .anchor}Figure ‑. Size
Distribution of ZFS Write I/O

![](media/image7.png){width="5.5472222222222225in"
height="3.013888888888889in"}

##### Linux Disk stats and Bandwidths

We used the Linux netdump utility to display graphically the I/O sizes
and bandwidths for each disk during the IOR run. The plots show reads on
top and writes on the bottom. Each line represents the data from a
single drive. Time is scrolling right to left along the horizontal axis
so that the oldest data are on the left of the screen. The time interval
shown is at the transition from when IOR completes the writing phase of
the test and begins reading the files just written.

The following figure (Figure 3‑4. ) shows the I/O size plot. The average
write size from the 43 disks varies from 1000 KB to 1700 KB in size.
Although the results above (section 3.3.3.3.3) show that ZFS is sending
2MB I/Os to Linux , netdump reports the average Linux I/O size. Since
ZFS writes a lot of metadata during commits, as the block pointer tree
is updated at the end of each write transaction group, the average write
size is expected to be less than 2MB. The read plot, however,
consistently shows that Linux is reading 2000 KB from all disks.

[[[[]{#_Toc488392506 .anchor}]{#_Toc486599630 .anchor}]{#_Ref486598773
.anchor}]{#_Ref486537781 .anchor}Figure ‑. Read/Write Disk Stats for
Sequential Workload

1.  ![](media/image8.png){width="6.500694444444444in"
    height="3.7534722222222223in"}

    The next figure (Figure 3‑5) shows the write and read bandwidth
    during the same interval. The data show that all disks are writing
    60-80 MB/s and reading 30-50 MB/s. Though we are using an untuned
    configuration, the discrepancy between the read and write bandwidths
    is an area for future investigation.

[[[]{#_Toc488392507 .anchor}]{#_Toc486599631 .anchor}]{#_Ref486598849
.anchor}Figure ‑. Write/Read Bandwidth for Sequential Workload

![](media/image9.png){width="6.5in" height="2.895138888888889in"}

##### Linux disk stats for a random workload

1.  We repeated the IOR test using random file offsets for the 16MB I/Os
    to generate a random workload.

  ---------------------------------------------------------------------------------------------------------------
  \# mpirun -wdir /mnt/lustre -np 12 -machinefile hosts /root/natasha-bin/ior -z -F -i 1 -s 10240 -b 16m -t 16m
  
  IOR-3.0.1: MPI Coordinated Test of Parallel I/O
  
  Began: Wed Jun 21 08:43:14 2017
  
  Command line used: /root/natasha-bin/ior -z -F -i 1 -s 10240 -b 16m -t 16m
  
  Machine: Linux node01
  
  Test 0 started: Wed Jun 21 08:43:14 2017
  
  Summary:
  
  api = POSIX
  
  test filename = testFile
  
  access = file-per-process
  
  ordering in a file = random offsets
  
  ordering inter file= no tasks offsets
  
  clients = 12 (2 per node)
  
  repetitions = 1
  
  xfersize = 16 MiB
  
  blocksize = 16 MiB
  ---------------------------------------------------------------------------------------------------------------

The impact of the unaligned I/O will be seen most clearly, when the data
are written to disk. The following diagram (Figure 3‑6) shows the I/O
size plots during IOR’s transition from writes to reads. As above, read
I/Os are consistently 2000 KB in size, as expected. Write I/Os, however,
are more variable, showing a smaller size range (200-1600 KB) and
greater differences between drives.

[[]{#_Toc488392508 .anchor}]{#_Ref486796353 .anchor}Figure ‑. Read/Write
Disk Stats for Random Workload

![](media/image10.png){width="4.933333333333334in"
height="3.5493055555555557in"}

The bandwidth plot for the random workload (Figure 3‑7) shows that, as
above, the drives are writing and reading at fairly consistent rates,
with no clear outliers. Nonetheless, performance per drive of 40 MB/s
for writes and 20 MB/s for reads is lower than with the sequential
workload above.

[[]{#_Toc488392509 .anchor}]{#_Ref486780391 .anchor}Figure ‑. Write/Read
Bandwidth for Random Workload

![](media/image11.png){width="4.925in" height="2.8930555555555557in"}

[]{#_Ref486416630 .anchor}

#### Fragmentation Improvements

Our initial testing of large 16MB I/O had shown the impact of file
system fragmentation, which occurred naturally as the file system aged,
on performance. The metadata isolation project grew out of these early
experiments. To demonstrate the benefit of metadata isolation we
compared pools with and without segregation and then showed how
segregation improves performance as the file system becomes fragmented.

##### File System Fragmention

When ZFS on Linux enabled support 16MB blocks, our testing found that as
on-disk fragmentation increased, performance on I/O benchmarks
decreased. Our early testing used a python file-ager. An iozone
benchmark would be run on a clean file system, then we would run a
python-based file ager to fragment the file system. The initial tool
would write a concurrent combination of large files with large blocks,
many smaller files 1/16th of the large block size, and random sized
files. The initial tests varied the I/O request size against the ZFS
block size. Running the file-ager with 16MB I/Os uncover LU-9305
(section 3.3.2). Nonetheless, the results clearly show the lower
performance for the dirty file systems (Figure 3‑8).

[[]{#_Toc488392510 .anchor}]{#_Ref486798638 .anchor}Figure ‑.
Fragmentation Impact on RAIDZ1 Performance

![](media/image12.png){width="5.947222222222222in"
height="3.4069444444444446in"}

To get around the problems described in LU-9305, we developed an
alternate mechanism for fragmenting the file system (Figure 3‑9).

A.  Fill the file system with 16MB files.

B.  Randomly replace a large file with multiple small files.

C.  Generate allocation holes by randomly deleting small or large files.

[[]{#_Toc488392511 .anchor}]{#_Ref486800563 .anchor}Figure ‑. Steps to
Create Fragmented File System

  A. Fill with Large Blocks                                                                B. Replace Big with Small                                                            C. Randomly Create Holes
  ---------------------------------------------------------------------------------------- ------------------------------------------------------------------------------------ ---------------------------------------------------------------------------------------
  ![Nice and ordered.png](media/image13.png){width="2.0in" height="2.138632983377078in"}   ![UglyFilling.png](media/image14.png){width="2.0in" height="2.1324814085739283in"}   ![Removethenfill.png](media/image15.png){width="2.0in" height="2.1324814085739283in"}

#####  Performance Improvements with Segregated Metadata

1.  We created two pools, one without segregation (ssu\_2ost0) and one
    with segregation enabled (ssu\_1ost1). Both pools were fragmented
    using the procedure described in the previous section, which left
    each pool with over 90% of the storage space allocated. An immediate
    difference could be seen from the fragmentation metric that ZFS
    maintains for the pool: the zpool without fragmentation had a
    significantly higher fragmentation score than the zpool with
    segregation:

-   ssu\_2ost0, dRAID without segregation enabled:

  ------------------------------------------------------------------------------------
  \#zpool list
  
  NAME        SIZE  ALLOC   FREE  EXPANDSZ   **FRAG**    CAP  DEDUP  HEALTH  ALTROOT
  
  ssu\_2ost0  72.7T  69.6T  3.15T         -    **24%**    95%  1.00x  ONLINE  -
  ------------------------------------------------------------------------------------

1.  

-   ssu\_1ost1, dRAID with segregation of special allocation class
    enabled:

  ------------------------------------------------------------------------------------
  \#zpool list
  
  NAME        SIZE  ALLOC   FREE  EXPANDSZ   **FRAG**    CAP  DEDUP  HEALTH  ALTROOT
  
  ssu\_1ost1  72.7T  62.9T   9.9T         -     **3%**    92%  1.00x  ONLINE  -
  ------------------------------------------------------------------------------------

1.  Segregating the ZFS metadata and small block data to a separate
    group of metaslabs within the zpool keeps the rest of the dRAID
    available for efficient large block allocations. The fragmentation
    metric for each metaslab is available through the ‘zdb –mm’ command
    (Section 3.2.4.3). The plot of the ZFS fragmentation metric (Figure
    3‑10**Error! Reference source not found.**) shows that without
    segregation, the metaslabs for the plain dRAID are more fragmented
    than the metaslabs on a dRAID with segregation enabled.

[[]{#_Toc488392512 .anchor}]{#_Ref487020669 .anchor}Figure ‑.
Fragmentation Comparison of Segregated and Unsegregated dRAIDs

![](media/image16.png){width="6.5in" height="3.564244313210849in"}

The average ZFS fragmentation score is over 17 with many metaslabs
peaking over 30. On the other hand, segregation of the metadata keeps
the metslab fragmentation score below 10 (average &lt;3). On the dRAID
with the segregated VDEV, the first 20% of the metaslabs are assigned to
the special class. This means that the metaslabs numbered 58 and higher
(Figure 3‑10**Error! Reference source not found.**) are all normal class
and contain generic data larger than 32KB in size. Segregation
simplifies block allocation to these metaslabs, reduces fragmentation,
and improves performance.

1.  We ran the following iozone test on Natasha (section 2.1.1) using
    all 8 Lustre clients.

  --------------------------------------------
  1.  \#iozone -t 8 -r 16m -i 0 -i 1 -s 256g
  
  --------------------------------------------

1.  The test results provide aggregate throughput for all 8 nodes after
    writing, rewriting, reading and rereading to the Lustre servers. The
    results (Figure 3‑11) show that without the special class enabled,
    performance on a fragmented file system is worse than on a file
    system with segregation enabled.

[[]{#_Toc488392513 .anchor}]{#_Ref486802616 .anchor}Figure ‑.
Performance Impact of Segregation on Fragmented File System

1.  ![](media/image17.png){width="6.0in" height="3.7753029308836394in"}

MS09: dRAID
===========

Introduction
------------

1.  This chapter describes the implementation of the MS09 of dRAID:
    Declustered RAID for ZFS. The documentation below (Table 4‑1)
    describes the scope, architecture, and high-level design developed
    for this program.

[[[[[[]{#_Toc488392500 .anchor}]{#_Toc486599637 .anchor}]{#_Toc486606487
.anchor}]{#_Toc462224957 .anchor}]{#_Toc462220591
.anchor}]{#_Ref462004715 .anchor}Table ‑. Declustered RAIDZ Design
Documents

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Document                      Location
  ----------------------------- -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  1.  Scope Statement           1.  [MS6\_CORAL\_Declustered\_RAIDZ\_Scope\_Statement\_V1.2\_final.pdf](https://wiki.hpdd.intel.com/display/PUB/CORAL+NRE+Documents)
                                

  1.  Solution Architecture     1.  [MS6\_CORAL\_Declustered\_RAIDZ\_Solution\_Architecture\_V1.2\_final.pdf](https://wiki.hpdd.intel.com/download/attachments/36966823/MS6_CORAL_Declustered_RAIDZ_Solution_Architecture_V1.2_final.pdf?version=2&modificationDate=1458856267000&api=v2)
                                

  1.  High Level Design (HLD)   1.  [MS7\_CORAL\_Declustered\_RAIDZ\_HLD\_v1.4\_final.pdf](https://wiki.hpdd.intel.com/download/attachments/36966823/MS7_CORAL_Declustered_RAIDZ_HLD_v1.4_final.pdf?version=2&modificationDate=1458856713000&api=v2)
                                

  1.  Prototype Report          1.  [MS4,MS8 Prototype\_Report\_Final.pdf](https://wiki.hpdd.intel.com/download/attachments/36966823/MS4%2CMS8%20Prototype_Report_Final.pdf?version=1&modificationDate=1476907192000&api=v2)
                                
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

1.  Since the MS08 prototype, the dRAID project has implemented the
    following additional contributions to OpenZFS, as defined in the
    HLD:

    -   Support for Generic Array Configurations

    -   Support double and triple parity

    -   Hybrid mirror support

    -   Dynamic rebuild throttling

    -   Persistent rebuild progress

    -   dRAID aware spare space rebalancing

        The implementation of each of these features will be discussed
        in the sections below.

Parity Declustered RAIDZ and Rebuild
------------------------------------

### Architecture

1.  Parity declustered RAID for ZFS, or dRAID, distributes data, parity,
    and spare capacity across all drives in a ZFS pool so that all
    drives participate in the rebuild process equally. Data blocks are
    written to the zpool using the permutation development algorithm
    described in the high level design (Table 4‑1) to ensure that all
    available drives in the storage enclosure are utilized to return the
    file system to full redundancy after a drive failure. Drive failures
    only affect those slices of the redundancy groups that span the
    failed device (Figure 4‑1).

    dRAID is a new ZFS VDEV driver written from scratch. It creates two
    new VDEVs: the dRAID VDEV itself and a virtual spare VDEV created
    from the distributed spare blocks in the dRAID VDEV. The VDEV uses
    existing RAIDZ code already in ZFS to generate and validate parity.
    Minor changes were made in the ZFS common code to integrate dRAID
    VDEV. The distributed spare VDEV is created at the same time the
    dRAID VDEV is created and is assigned to a dRAID VDEV. The
    distributed spare VDEV replaces the traditional global hot spare
    drives typically used to restore redundancy during ZFS resilvering.

[[]{#_Toc488392514 .anchor}]{#_Ref487028765 .anchor}Figure ‑. Mapping
Failed Drive to dRAID1 Permutation Groups

![](media/image18.png){width="6.0in" height="3.435980971128609in"}

1.  Sequential Rebuild reduces the time needed to restore a dRAID pool
    to full redundancy by replacing the traversal of the block pointer
    tree to find data blocks on the failed drive with a sequential scan
    of the Zpool spacemap array. The new sequential rebuild algorithm
    decouples the reconstruction of lost data blocks and the validation
    of the block pointer tree referencing those data blocks. Sequential
    Rebuild reconstructs the lost data blocks and then an explicit
    scrubbing of the tree using existing ZFS tools validates the block
    pointer tree.

    The rebuild process is able to adjust to changes in application IO
    and pool redundancy state. This is done automatically. If a pool is
    idle, the rebuild process is able to use all the available
    bandwidth. When the application is active, the rebuild process will
    slow, making room for the application process in the I/O stream.

### Implementation

1.  The development effort after the prototype demonstration in MS08
    focused on enhancing the dRAID device driver and the Sequential
    Rebuild solution.[]{#_Ref487016234 .anchor}

#### dRAID

##### Arbitray Pool Configurations

1.  dRAID has been extended to accommodate generic arrray configurations
    using any data-to-parity ratio for all RAIDz parity levels (1,2,3).
    Mirroring is also supported by specifying 1 data and 1-P parity. A
    new utility, draidcfg, generates optimal permutations based on the
    requested data-to-parity ratio and number of distributed spares. The
    number of parity groups will vary depending on the total number of
    drives available in the zpool.

    The command line to support this function is as follows:

  --------------------------------------------------
  \# draidcfg -n N -d D -p P -s S &lt;file&gt;.nvl
  --------------------------------------------------

1.  where

    -n number of drives in the pool (N)

    -d number of data drives in a parity group (D)

    -p number of parity drives in a parity group (P)

    -s number of distributed spares in the pool (S)

    > &lt; file&gt; is the name of the file holding the base permutation
    > table that will be used to create the dRAID with ‘zpool create’

    The number of distributed spares is usually set to at least the
    redundancy level of the dRAID (P), but can range from zero to any
    number that fits within the specified valid array configuration.
    Using S&lt;P is not recommended. However, since it avoids the value
    of the distributed spares to improve rebuild speed.

    We had previously looked at generating and storing the single seed.
    This was rejected by the community and drove the development of the
    automatic base permutations. The permutations are now stored
    redundantly in the ZFS disk labels on each disk in the dRAID layout.

##### Double and Triple Parity dRAID

1.  The prototype shown in MS08 only used single parity RAIDZ1. The
    implementation being pushed upstream to the community supports
    double- and triple-parity RAID by by calling into ZFS’s RAIDZ2 and
    RAIDZ3 code to generate the parity blocks.

    dRAID uses skip sectors to pad the data blocks in order to span the
    drives in a parity group. The minimum data size equals the number of
    data drives in the parity group multiplied by the drive sector size.
    For example, the pad for a 4KB block over 8 data drives would have a
    multiple of 32K with 8. Modifications to the dRAID code were
    required to include the dRAID skip sectors into the parity
    calculation, even if they contain no data.

    Note that the skip sectors are not included in RAIDZ2/3 parity
    computations, but are included in the dRAID code.

##### Hybrid Mirrors 

1.  The metadata isolation demonstration shows that a zpool can have
    mirrored VDEVs for ZFS metadata and dRAID VDEVs for large block I/Os
    (Section 3.2.4). The goal is to make both classes (normal and
    special) available with metaslab granularity within the same pool of
    disks.

    Hybrid mirror supports the same redundancy as dRAID by using just
    one data driver (1+P). This provides the same level of redundancy
    for the mirror as is available with the RAIDZ and enables the
    filesystem to tolerate the same number of drive failures in both the
    generic class and special class allocations.

    Mirroring is done on a drive level basis and left over space is
    currently not able to allocate the unused space. This restriction is
    being addressed and should be remedied in a future
    release.[[[[[[[[[[]{#_Toc487024643 .anchor}]{#_Toc487024642
    .anchor}]{#_Toc487024641 .anchor}]{#_Toc487024640
    .anchor}]{#_Toc487024639 .anchor}]{#_Toc487024638
    .anchor}]{#_Toc487024637 .anchor}]{#_Toc487024636
    .anchor}]{#_Toc487024635 .anchor}]{#_Toc487024634 .anchor}

#### Sequential Rebuild

##### Dynamic Rebuild Throttling 

The prototype demonstrated static throttling, where the rebuild speed
could be changed directly with a tunable. The production code allows
administrators to set targets for application I/O during rebuild,
depending on the status of the zpool (degraded, critical, etc), but
changes the level of throttling automatically.

As depicted diagram below (Figure 4‑2), with a single drive failure the
dRAID2 is in the degraded state and application I/O has priority over
rebuild I/O. If there is a second drive failure and the dRAID has lost
redundancy, then the dRAID is in the critical state and rebuild I/O has
priority over application I/O.

[[]{#_Toc488392515 .anchor}]{#_Ref487027899 .anchor}Figure ‑. dRAID2
State Transitions

![](media/image19.png){width="6.0in" height="2.4969641294838145in"}

1.  Dynamic rebuild throttling delays making rebuild I/O requests while
    the array is busy with application I/Os. The amount of delay is
    tunable. The delay time is ignored, however, if the file system
    enters a critical state, at which point the rebuild will resume at
    full throttle (thus, competing with the Application I/O). Rebuild
    throttling will reset once the dRAID is no longer in a critical
    state.

##### Persistent Rebuild Progress

1.  The prototype did not maintain a progress state during a rebuild.
    The production solution maintains the persistent state of a rebuild
    progress so that an interrupted rebuild, such as might occur during
    a power loss, can resume from the last committed metaslab.

    The dRAID code now maintains the progress of the rebuild in system
    memory and periodically persists the progress to disk. If a rebuild
    is interrupted, then the last progress update to disk will be read
    when the pool is imported after power is restored and incomplete
    rebuild is found. Then the last persisted progress update will be
    where the rebuild will restart.

#### dRAID-aware Spare Space Rebalancing

1.  After rebuild completes, a distributed spare will stay in the
    *INUSE* state, until the failed drive is replaced by another drive.
    This process is called rebalance.

    The current implementation uses existing ZFS resilvering code to
    traverse the block pointer tree and move data from the spare blocks
    to the new drive. The spare blocks are then recovered when the
    rebalance completes. Rebalancing is only possible when the dRAID is
    in the normal state (Figure 4‑2).

### Changes and Deviations

1.  The HLD proposed calculating base permutations using only a random
    seed stored when the dRAID VDEV was created. Based on feedback from
    the ZFS community, we changed this plan to generate random base
    permutations only once, during VDEV creation, and storing the
    resulting table in the ZFS disk label structure. Multiple copies are
    stored on each drive in the array to ensure redundancy in case a
    drive is corrupted or fails.

### Demonstration 

1.  Arbitrary pool configuration used a server on the Wolf cluster with
    a large 90 drive JBOD. Creating the dRAID was destructive and would
    have interfered with the other milestone demonstrations. The rest of
    the dRAID demos ran on one of Natasha’s storage servers, which had
    been configured as a 43-drive dRAID2 with four (8+2) parity groups
    and 3 distributed spares (Figure 2‑1).

#### Arbitrary Pool Configuration

1.  dRAID supports all RAIDz parity levels. The new “draidcfg” tool is
    currently required to find the best set of random base permutations
    for the specified array configuration (Section 0).

    The dRAID pool configuration starts with a new command, “draidcfg,”
    to find the best set of random permutations for the specified input
    parameters. The output from the draidcfg tool is a configuration
    file that ‘zpool create’ uses when creating the dRAID. The list of
    base permutations in the configuration file will be stored in the
    ZFS disk labels during zpool creation.

In this demonstration, we created a triple-parity dRAID from 80 drives
with seven 8+3 redundancy groups and three distributed spares.

  -----------------------------------------
  \# draidcfg -n 80 -d 8 -p 3 -s 3 80.nvl
  
  Worst ( 7 x 11 + 3) x 5120: 0.998
  
  Seed chosen: b79e65a91d440fc
  -----------------------------------------

1.  The output from the draidcfg tool indicates the resulting
    configuration and the random seed that gave the best distribution of
    the parity groups and distributed spares. The output file holds the
    list of base permutations.

    The following shows the first three base permutations contained in
    the 80.nvl file created above. The complete file listing is
    available in Appendix A.3.

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  \# draidcfg -r 80.nvl
  
  dRAID3 vdev of 80 child drives: 7 x (8 data + 3 parity) and 3 distributed spare
  
  Using 64 base permutations
  
  23,54,38,76,61,14,34,48, 9,31,52,10, 3,41,46,70, 1, 6,59,47,28,32,29,49,30,22,27,11,44,20,56, 5,74, 8,50,15,62,66,33,67,16,65,36,71,75,18,68,21,69,26,64,60,55,42,43,63,35,37,24, 7,17,45, 0, 2,58,78,57,13,12,72,73, 4,19,25,51,79,39,53,77,40,
  
  41,54,75,48, 2,57,36, 8,76,44, 5, 3,22,30,61,69,47,28,13, 0, 6,71,34,55,33,46,70,79,66,45,27,74,18,25,60,72,11,50,68, 1,53,32,19,64,40,51, 4,31,17,62,42,39,26,56, 7,16,24,12,38,15,78,35,37,67, 9,23,20,49,10,43,14,59,77,29,63,73,58,52,21,65,
  
  14,65,43, 9,16,53,46,69,17,40,20, 3,47,70,28,39,54, 5,12,24,78, 2,49,61,11,51,75,79,41,50,73,34,18,21,25,52,44,22,32,77, 8,59,15, 7,74,66, 0,71,45,56, 4,36,58,23,68, 6,67,42,29,64,26,33,72,10,37,13, 1,76,60,38,48,31,63,27,35,62,55,19,30,57,
  
  . . . .
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

1.  The command line syntax for creating the dRAID pool is similar to
    that for creating a RAIDz pool, with the addition of the draidcfg
    filename before listing the drives to be used in the pool.

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  \# zpool create -f -o ashift=12 -o cachefile=none -o segregate\_special=on -O recordsize=16MB MS09 **draid3** **cfg=/root/80.nvl** sdb sdd sde sdg sdh sdi sdk sdl sdm sdo sdp sdq sds sdt sdu sdw sdx sdy sdz sdab sdac sdad sdae sdc sdf sdj sdn sdr sdv sdaa sdaf sdag sdah sdai sdaj sdak sdal sdam sdan sdao sdap sdaq sdar sdas sdat sdau sdav sdaw sdax sday sdaz sdba sdbb sdbc sdbd sdbe sdbf sdbg sdbh sdbi sdbj sdbk sdbl sdbm sdbn sdbo sdbp sdbq sdbr sdbs sdbt sdbu sdbv sdbw sdbx sdby sdbz sdca sdcb sdcd
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

1.  Zpool status for this pool lists all 80 drives contained in the
    dRAID and the three distributed spare VDEVs. The beginning and end
    of the listing are shown in the following table. The complete
    listing is available in Appendix A.4.

  ------------------------------
  \# zpool status
  
  pool: MS09
  
  state: ONLINE
  
  scan: none requested
  
  config:
  
  NAME STATE READ WRITE CKSUM
  
  MS09 ONLINE 0 0 0
  
  draid3-0 ONLINE 0 0 0
  
  sdb ONLINE 0 0 0
  
  sdd ONLINE 0 0 0
  
  sde ONLINE 0 0 0
  
  > . . . .
  
  sdca ONLINE 0 0 0
  
  sdcb ONLINE 0 0 0
  
  sdcd ONLINE 0 0 0
  
  spares
  
  \$draid3-0-s0 AVAIL
  
  \$draid3-0-s1 AVAIL
  
  \$draid3-0-s2 AVAIL
  
  errors: No known data errors
  ------------------------------

1.  

####  Dynamic Rebuild Throttling

1.  The rebuild process observes and responds to changes in application
    I/O and pool redundancy level, then throttles itself accordingly.
    The demonstration showed that the rebuild process :

    -   slowed down to give more I/O resources to the application

    -   sped up when the pool lost all redundancy critical mode (2
        drives failed on dRAID2) and reached a critical state.

    For this test we used a 43 drive Lustre OST from Natasha. The
    results of the test were displayed with netdump (Figure 4‑3). The
    upper graph shows read throughput in MB/s of all 43 individual
    drives. The lower graph shows the write throughput. The Y axis is
    throughput in MB/s, and the X axis is wallclock time, moving from
    right to left (older times are on the left).

[[]{#_Toc488392516 .anchor}]{#_Ref487017457 .anchor}Figure ‑. Dynamic
Rebuild Throttling

1.  To initiate the test, we took one drive offline and then started the
    rebuild by manually replacing the offline drive with a distributed
    spare.

  -------------------------------------------------
  \# zpool offline ssu\_1ost0 sde
  
  \# zpool replace ssu\_1ost0 sde '\$draid2-0-s0'
  -------------------------------------------------

1.  With no application I/O, the rebuild proceeded at full speed. We
    then ran the md5sum application to generate I/O by reading a list of
    files from the file system.

  ------------------------------
  \# md5sum /ssu\_1ost0/\*.iso
  ------------------------------

As seen on the netdump plot, the rebuild process throttled itself as
soon as the application i/O started. Both reads and writes dropped as
the drives were busy switching between the two i/O streams from the
application and the rebuild. Since this application is not I/O intensive
and only did read I/Os, the write I/Os indicate the rebuild is
proceeding slowly in the background. The application IO kept the file
system busy enough to demonstrate the rebuild throttling mechanism

When another drive is taken offline before the first rebuild completes,
the pool reaches a critical state since redundancy of the dRAID2 array
has been lost and the rebuild throughput resumes to full speed without
delay.

  ---------------------------------
  \# zpool offline ssu\_1ost0 sdk
  ---------------------------------

When the rebuild completes, the zpool status showed that drive *sde* had
been replaced by *\$draid2-0-s0*, which is now *INUSE. *

  --------------------------------------
  \# zpool status
  
  NAME STATE READ WRITE CKSUM
  
  ssu\_1ost0 DEGRADED 0 0 0
  
  draid2-0 DEGRADED 0 0 0
  
  sdb ONLINE 0 0 0
  
  sdd ONLINE 0 0 0
  
  spare-2 DEGRADED 0 0 0
  
  sde OFFLINE 0 0 0
  
  \$draid2-0-s0 ONLINE 0 0 0
  
  sdg ONLINE 0 0 0
  
  sdh ONLINE 0 0 0
  
  sdi ONLINE 0 0 0
  
  sdk OFFLINE 0 0 0
  
  ......
  
  spares
  
  \$draid2-0-s0 INUSE currently in use
  
  \$draid2-0-s1 AVAIL
  
  \$draid2-0-s2 AVAIL
  --------------------------------------

#### Rebuild Stop and Resume

1.  Rebuild progress is periodically persisted so that if the rebuild
    process t is interrupted, the rebuild will be able to resume again
    from where progress was last saved, rather than restarting from the
    beginning.

    In the previous demonstration, we had taken two drives offline
    (*sde* and *sdk*) and rebuilt *sde* onto *\$draid2-0-s0*. For this
    demonstration, we began the rebuild of *sdk* onto *\$draid2-0-s1*
    and then interrupted the ongoing rebuild by exporting the pool.

  -------------------------------------------------
  \# zpool replace ssu\_1ost0 sdk '\$draid2-0-s1'
  
  \# zpool export ssu\_1ost0
  -------------------------------------------------

We used the Linux dmesg log to view the debug messages from the rebuild
process showing that rebuild was interrupted at metaslab 31.

  -----------------------------------------------------------------------
  \[265158.886650\] Scanning 4 segments for MS 30
  
  \[265158.892179\] MS (30 at 8053063680K) segment: 720K + 80K
  
  \[265158.898894\] MS (30 at 8053063680K) segment: 1920K + 80K
  
  \[265158.905767\] MS (30 at 8053063680K) segment: 2080K + 80K
  
  \[265158.912586\] MS (30 at 8053063680K) segment: 2880K + 80K
  
  \[265158.919333\] Completed rebuilding metaslab 30
  
  \[265158.924989\] All metaslabs \[0, 29) fully rebuilt.
  
  \[265158.931215\] Scanning 4 segments for MS 31
  
  \[265158.936432\] MS (31 at 8321499160K) segment: 160K + 81920K
  
  \[265158.943346\] MS (31 at 8321499160K) segment: 98280K + 101785600K
  
  \[265160.549318\] Completed rebuilding metaslab 29
  
  \[265160.555119\] All metaslabs \[0, 31) fully rebuilt.
  
  \[265163.080670\] Aborted rebuilding metaslab 31
  -----------------------------------------------------------------------

1.  Note that the rebuild progress shown in the debug message log
    represents the saved status in-memory. The on-disk persisted
    progress usually lags behind the saved in-memory state by a number
    of metaslabs. As a result, the rebuild was expected to resume
    metaslab 31 or a bit earlier.

    When the pool is imported, rebuild will resume from the progress
    persisted to disk. The following debug messages showed that the
    rebuild started from metaslab 29.

  ---------------------------------------------------------------------
  \[265196.221793\] Restarting rebuild at metaslab 29
  
  \[265197.190622\] Scanning 36 segments for MS 29
  
  \[265197.195950\] MS (29 at 7784628240K) segment: 0K + 229120K
  
  \[265197.208742\] MS (29 at 7784628240K) segment: 229360K + 491480K
  
  \[265197.220629\] MS (29 at 7784628240K) segment: 720880K + 264320K
  
  \[265197.227520\] MS (29 at 7784628240K) segment: 985360K + 106320K
  
  \[265197.234404\] MS (29 at 7784628240K) segment: 1091760K + 49960K
  
  \[265197.241346\] MS (29 at 7784628240K) segment: 1141800K + 37960K
  
  \[265197.248362\] MS (29 at 7784628240K) segment: 1179840K + 37200K
  ---------------------------------------------------------------------

1.  After the rebuild is completed, both distributed spares are now
    *INUSE*:

  --------------------------------------
  \# zpool status
  
  NAME STATE READ WRITE CKSUM
  
  ssu\_1ost0 DEGRADED 0 0 0
  
  draid2-0 DEGRADED 0 0 0
  
  sdb ONLINE 0 0 0
  
  sdd ONLINE 0 0 0
  
  spare-2 DEGRADED 0 0 0
  
  sde OFFLINE 0 0 0
  
  \$draid2-0-s0 ONLINE 0 0 0
  
  sdg ONLINE 0 0 0
  
  sdh ONLINE 0 0 0
  
  sdi ONLINE 0 0 0
  
  spare-6 DEGRADED 0 0 0
  
  sdk OFFLINE 0 0 0
  
  \$draid2-0-s1 ONLINE 0 0 0
  
  sdl ONLINE 0 0 0
  
  sdm ONLINE 0 0 0
  
  ......
  
  spares
  
  \$draid2-0-s0 INUSE currently in use
  
  \$draid2-0-s1 INUSE currently in use
  
  \$draid2-0-s2 AVAIL
  --------------------------------------

1.  

#### Rebalance

1.  In this demonstration, we replaced the failed sde drive with the
    sdas drive to make the *\$draid2-0-s0* spare available again:

  -----------------------------------------
  \# zpool replace -f ssu\_1ost0 sde sdas
  -----------------------------------------

1.  The rebalance process uses the traditional ZFS resilver mechanism.
    Although it is essentially reconstructing the same lost redundancy
    as the previous rebuild, rebalance is much slower as it has to
    traverse the block pointer tree and write to a single spare drive.
    As shown below (Figure 4‑4), only the replacement drive (sdas) does
    write IO, while during a sequential rebuild all surviving drives
    share the write workload (Figure 4‑3).

[[]{#_Toc488392517 .anchor}]{#_Ref487019833 .anchor}Figure ‑. Rebalance
to a Replacement Drive

![](media/image21.png){width="6.5in" height="3.6326388888888888in"}

1.  After rebalance completed, zpool status reports the corresponding
    distributed spare (*\$draid2-0-s0*) as being available (AVAIL).

  --------------------------------------
  \# zpool status
  
  NAME STATE READ WRITE CKSUM
  
  ssu\_1ost0 DEGRADED 0 0 0
  
  draid2-0 DEGRADED 0 0 0
  
  sdb ONLINE 0 0 0
  
  sdd ONLINE 0 0 0
  
  sdas ONLINE 0 0 0
  
  sdg ONLINE 0 0 0
  
  sdh ONLINE 0 0 0
  
  sdi ONLINE 0 0 0
  
  spare-6 DEGRADED 0 0 0
  
  sdk OFFLINE 0 0 0
  
  \$draid2-0-s1 ONLINE 0 0 0
  
  sdl ONLINE 0 0 0
  
  .......
  
  spares
  
  \$draid2-0-s0 AVAIL
  
  \$draid2-0-s1 INUSE currently in use
  
  \$draid2-0-s2 AVAIL
  --------------------------------------

1.  

ZED Fault Handling
------------------

1.  In the previous milestone, we reported our efforts to migrate the
    Fault Management Architecture (FMA) from OpenZFS to the Linux ZFS
    Event Daemon (ZED). Before this integration, ZED received events
    from the ZFS kernel module and called scripts, called zedlets, to
    respond to specific events. The addition of FMA allowed ZED to
    refine event processing so zedlets would only be called for specific
    faults (Figure 4‑5). FMA provides critical fault logic to ZED and
    for MS09, we built on this platform to enable automatic rebuild and
    rebalance for dRAID and RAIDZ VDEVs.

[[]{#_Toc488392518 .anchor}]{#_Ref486876630 .anchor}Figure ‑. ZED
Architecture

1.  ![](media/image22.png){width="4.4in" height="3.8463057742782154in"}

### Architecture

1.  The Fault Management Architecture (Figure 4‑6**Error! Reference
    source not found.**) consists of four components-- the Diagnosis
    Engine, the Retire Agent, the SLM (syseventd loadable module) and
    the Disk Event Monitor -- that evaluate and act upon storage events
    and faults. The Diagnosis Engine receives events from ZFS and
    evaluates faults for the VDEVs in the system. The Retire Agent
    responds to diagnosed faults and, if necessary, initiates automatic
    rebuilds. The Agent will notify the ZFS kernel of the change in the
    VDEV state (degraded or faulted). When the Disk Monitor encounters a
    drive replacement, the event will be received by the Retire Agent
    which will initiate rebalancing data from the surviving drives to
    replace the new drive for the failed one.

[[]{#_Toc488392519 .anchor}]{#_Ref487020668 .anchor}Figure ‑. ZED FMA
Components

![](media/image23.PNG){width="6.0in" height="3.0871839457567805in"}

### Implementation

1.  We have followed a phased development approach, with each phase
    building on previous work. The initial phase I work, reported in
    MS08, was pushed upstream and has been accepted to the zfsonlinx
    github repository.

    **Phase 1 – Completed in MS 4/8:**

-   [PR-4416](https://github.com/zfsonlinux/zfs/pull/4416) new code to
    add vdev tags required for matching. Built on libudev.

-   [PR-4673](https://github.com/zfsonlinux/zfs/pull/4673) provides the
    foundation for FMA feature parity. This work also provides
    auto-online, auto-replace and auto-expand capabilities.

1.  **Phase 2 – Completed in MS 5/9:**

-   [PR-5343](https://github.com/zfsonlinux/zfs/pull/5343) adds the
    Diagnosis and Retire components and incorporated full multi-fault
    support in
    [collaboration](https://github.com/zfsonlinux/zfs/commit/6078881aa18a45ea065a887e2a8606279cdc0329)
    with the ZoL community.

-   [PR-5383](https://github.com/zfsonlinux/zfs/pull/5383) add zedlet to
    respond to state change events.

-   [PR-6276](https://github.com/zfsonlinux/zfs/pull/6276) allows ZED to
    respond to multiple faults.

    We had proposed to the community a possible third phase of
    development, but since Lawrence Livermore National Laboratory (LLNL)
    began deploying the current ZFS/FMA code in production on their
    Lustre/ZFS file systems, we realized that the additional work was
    not required at this time.

1.  

#### Spare Device Matching

1.  With the addition of Allocation Classes and dRAID, the nature of
    spare devices has changed. Before these two features were added, all
    spares were essentially equal and any spare could be used to
    effectively replace any drive in the pool. With the introduction of
    dRAID, however, spare drives are no longer physicaldevices. With the
    introduction of the special allocation classes, additional
    characteristics, such as size and type of drive, are important in
    selecting a spare.

    A spare drive in a dRAID is a virtual spare composed of blocks
    randomly scattered across all of the physical devices in the pool.
    The nature of a dRAID spare means that a zpool with multiple dRAID
    VDEVs can only replace a failed drive with a spare that shares the
    same dRAID parent VDEV. A normal physical drive can also be used as
    a spare for a dRAID VDEV, but doing so will trigger a resilver of
    the pool. Since resilvering is a significantly slower operation than
    a dRAID sequential rebuild, using a normal drive defeats the purpose
    of using dRAID and should only be done if a dRAID distributed spare
    is unavailable. To address these concerns the Retire Agent in ZED
    has been modified to only attempt to spare-in a drive to a dRAID
    VDEV if the spare VDEV is a distributed spare (\$draid) that has the
    same parent identifier as the dRAID VDEV it is being spared into.

    Metadata Isolation (Section 3.2) uses a special allocation class to
    save ZFS metadata and small block data to metaslabs segregated from
    the RAID VDEV or to physical disks in a dedicated VDEV. When a
    dedicated tier is used, a different type of physical disk may be
    used to back this tier. For example, for a metadata heavy workload,
    a dedicated pair of SSDs may be used and a spare for this tier also
    be an SSD of similar size. The not just match the type, but the size
    of dedicated device being replaced. To accommodate these concerns,
    ZED will check if the drive being replaced has an allocation bias
    and then take into account these characteristics in selecting an
    appropriate spare.

    Since a segregated VDEV is allocated from the parent RAIDz or dRAID
    VDEV, sparing is done in the context of the parent. In other words,
    sparing the parent will also spare the segregated VDEV.

#### Multi-path Support

1.  Lustre servers are deployed in high-availability (HA) pairs in which
    paired servers have access to each other’s storage pools. On
    failure, the surviving server depends on Linux multi-pathing to
    mount the other server’s storage. As a result, the FMA Diagnosis
    Engine and Retire Agent
    ([PR-5343](https://github.com/zfsonlinux/zfs/pull/5343)) must be
    able to support the Linux architecture. ZFS multi-path support had
    been started by Lawrence Livermore National Laboratory (LLNL). Our
    team collaborated with their engineer to ensure their code worked
    with our feature (see this
    [commit](https://github.com/zfsonlinux/zfs/commit/6078881aa18a45ea065a887e2a8606279cdc0329)
    for details).

#### Improve ZED resiliency

1.  In the prototype, a hung zedlet could hang ZED all together. The
    production code includes a watchdog timer (10s) to keep zedlets from
    hanging. This addition was made with the phase 2 code
    ([PR-5343](https://github.com/zfsonlinux/zfs/pull/5343)).

#### Multi-Fault Support

A RAIDZ VDEV can handle multiple drive failures in parallel. The
structure of the block pointer tree traversal effectively enables
queueing of subsequent failures. Reconstruction of a second drive can
proceed after the repair of the first driver completes ahead of it in
the tree. Because scanning metaslabs during the dRAID sequential rebuild
is a serial process, dRAID cannot repair a second driver until the first
failure is completely rebuilt. As a result, while rebuilding one failed
drive, dRAID does not have the ability to queue subsequent failures.

Due to the way ZED interacts with the ZFS kernel module when it attempts
to attach a spare drive to a pool, if a rebuild or resilver is in
progress it will be told that the pool is busy and the attach cannot
happen. Before making adding multi-fault support, ZED would mark the
spare attempt as resolved and move on to processing other events, thus
losing the event and need to rebuild the dRAID. To fix this error, the
Retire Agent in ZED was modified to save off the spare request to be
replayed later.

The Retire Agent was also modified to receive resilver\_finished and
rebuild\_finished events. When either of these events arrive, the Retire
Agent will check for a saved spare request and, if it finds one, will
replay the request to begin rebuilding the failed drive. This has been
implemented so that any number of drives can be faulted and spared-in as
long as there are spares available.

### Changes and Deviations

1.  There were no changes from the HLD or SOW.

### Demonstration 

#### Multi-Fault Handling

1.  The demonstration focuses on the interaction of the Diagnosis Engine
    and Retire Agent with ZFS in the presence of multiple drive
    failures, which exercises the features described above in Section
    4.3.2. The live demonstration included windows that showed the zpool
    status in one window and the ZED log for the ZFS events in another.

    The demo showed that while dRAID rebuild was in progress, the
    arrival of the second or third failure in the array would be saved
    within the Retire Agent until the current rebuild completed, upon
    which the Agent would retry the pending request.

    To introduce the drive faults for the demonstration, we injected IO
    read errors using zinject to force ZED to fail the drives. The
    Diagnosis Engine receives these error events and once the number of
    faults received exceeds a failure rate threshold, the Engine will
    initiate a fault message. The Retire Agent then automatically begins
    to use the first of the dRAID spares to rebuid the failed drive.

##### dRAID Configuration

1.  The triple parity dRAID was created on the Wolf cluster with a
    single parity group (7+3) and 3 distributed spares. We started with
    a clean, populated pool with all cache cleared:

  ---------------------------------------------------------------------------
  pool: mfault
  
  state: ONLINE
  
  scan: resilvered 368K in 0h0m1s with 0 errors on Tue Jun 27 20:11:10 2017
  
  config:
  
  NAME STATE READ WRITE CKSUM
  
  mfault ONLINE 0 0 0
  
  draid3-0 ONLINE 0 0 0
  
  sdb ONLINE 0 0 0
  
  sdc ONLINE 0 0 0
  
  sdd ONLINE 0 0 0
  
  sde ONLINE 0 0 0
  
  sdf ONLINE 0 0 0
  
  sdg ONLINE 0 0 0
  
  sdh ONLINE 0 0 0
  
  sdi ONLINE 0 0 0
  
  sdj ONLINE 0 0 0
  
  sdk ONLINE 0 0 0
  
  sdl ONLINE 0 0 0
  
  sdm ONLINE 0 0 0
  
  spares
  
  \$draid3-0-s0 AVAIL
  
  \$draid3-0-s1 AVAIL
  
  \$draid3-0-s2 AVAIL
  
  errors: No known data errors
  ---------------------------------------------------------------------------

1.  

##### First Failure

1.  We used zinject() to send I/O read errors to drives sdb, sdg and
    sdm. When the Diagnosis Engine began receiving errors, the zed log
    showed that the Diagnosis Engine opened a failure case for each
    VDEV:

  -----------------------------------------------------------------------------------------
  Diagnosis Engine: case opened (cc7e90a9-f96d-4937-ace2-54502acfc9ec)
  
  Diagnosis Engine: opening case for vdev 13059866864003676862 due to 'ereport.fs.zfs.io'
  
  . . .
  
  Diagnosis Engine: case opened (602adf2c-c4dc-4613-88de-de3442182e6d)
  
  Diagnosis Engine: opening case for vdev 5296963598540981156 due to 'ereport.fs.zfs.io'
  
  . . .
  
  Diagnosis Engine: case opened (ae95fc42-ccd1-4a03-924c-649ecda66de8)
  
  Diagnosis Engine: opening case for vdev 4853321467358484743 due to 'ereport.fs.zfs.io'
  -----------------------------------------------------------------------------------------

The Diagnosis Engine used each opened case to track the errors received
for each VDEV. Eventually the first drive (sdb) accumulated enough
errors that the Diagnosis Engine generated a fault event for the drive.
The Retire Agent received the fault, which caused it to report the
failed drive to ZFS, and then initiated the dRAID rebuild of the first
distributed spare (\$draid3-0-s0).

  -------------------------------------------------------------------------
  Diagnosis Engine: solving fault 'fault.fs.zfs.vdev.io'
  
  zed\_fault\_event:
  
  uuid: cc7e90a9-f96d-4937-ace2-54502acfc9ec
  
  class: fault.fs.zfs.vdev.io
  
  code: ZFS-8000-FD
  
  certainty: 100
  
  scheme: zfs
  
  pool: 1320611588736634121
  
  vdev: 13059866864003676862
  
  Diagnosis Engine: case solved (cc7e90a9-f96d-4937-ace2-54502acfc9ec)
  
  Diagnosis Engine: removing timer (0x7f12d000d720)
  
  Retire Agent: zfs\_retire\_recv: 'list.suspect'
  
  Retire Agent: matched vdev 13059866864003676862
  
  Retire Agent: zpool\_vdev\_fault: vdev 13059866864003676862 on 'mfault'
  
  Retire Agent: zpool\_vdev\_replace 'sdb' with spare '\$draid3-0-s0'
  
  Diagnosis Engine: resource event 'resource.fs.zfs.statechange'
  
  Retire Agent: zfs\_retire\_recv: 'resource.fs.zfs.statechange'
  -------------------------------------------------------------------------

The last log messages show that ZFS had faulted the drive and sent the
event through the Diagnosis Engine to the Retire Agent.

Zpool status confirms that the first drive had been faulted and that the
rebuild was in progress.

  -----------------------------------------------------------------------------
  pool: mfault
  
  state: DEGRADED
  
  status: One or more devices are faulted in response to persistent errors.
  
  Sufficient replicas exist for the pool to continue functioning in a
  
  degraded state.
  
  action: Replace the faulted device, or use 'zpool clear' to mark the device
  
  repaired.
  
  scan: rebuild in progress since Tue Jun 27 20:03:41 2017
  
  2.70M scanned out of 128G at 923K/s, 40h25m to go
  
  1.40M rebuilt, 0.00% done
  
  config:
  
  NAME STATE READ WRITE CKSUM
  
  mfault DEGRADED 0 0 0
  
  draid3-0 DEGRADED 0 0 0
  
  spare-0 DEGRADED 0 0 0
  
  sdb FAULTED 66 0 0 too many errors
  
  \$draid3-0-s0 ONLINE 0 0 0 (repairing)
  
  sdc ONLINE 0 0 0
  
  sdd ONLINE 0 0 0
  
  sde ONLINE 0 0 0
  
  sdf ONLINE 0 0 0
  
  sdg ONLINE 65 0 0 (repairing)
  
  sdh ONLINE 0 0 0 (repairing)
  
  sdi ONLINE 0 0 0
  
  sdj ONLINE 0 0 0
  
  sdk ONLINE 0 0 0
  
  sdl ONLINE 0 0 0
  
  sdm ONLINE 38 0 0 (repairing)
  
  spares
  
  \$draid3-0-s0 INUSE currently in use
  
  \$draid3-0-s1 AVAIL
  
  \$draid3-0-s2 AVAIL
  
  errors: No known data errors
  -----------------------------------------------------------------------------

##### Second Failure

1.  While the first rebuild was proceeding, the zinject() continued to
    send I/O read failures to the other two drives (sdg, sdm).
    Eventually, the Diagnosis Engine faulted the second drive (sdg). The
    Retire Agent received the fault and forwarded the fault event to
    ZFS. The Agent then attempted to use the second distributed spare
    but detected that the zpool was busy with the first rebuild and
    saved the spare-in request. ZFS received the fault event from the
    Retire Agent, then sent a state change event, which both the
    Diagnosis Engine and Retire Agent received.

  --------------------------------------------------------------------------------------------
  Diagnosis Engine: solving fault 'fault.fs.zfs.vdev.io'
  
  zed\_fault\_event:
  
  uuid: 602adf2c-c4dc-4613-88de-de3442182e6d
  
  class: fault.fs.zfs.vdev.io
  
  code: ZFS-8000-FD
  
  certainty: 100
  
  scheme: zfs
  
  pool: 1320611588736634121
  
  vdev: 5296963598540981156
  
  Diagnosis Engine: case solved (602adf2c-c4dc-4613-88de-de3442182e6d)
  
  Diagnosis Engine: removing timer (0x7f12d0045880)
  
  Retire Agent: zfs\_retire\_recv: 'list.suspect'
  
  Retire Agent: matched vdev 5296963598540981156
  
  Retire Agent: zpool\_vdev\_fault: vdev 5296963598540981156 on 'mfault'
  
  Retire Agent: zpool\_vdev\_replace 'sdg' with spare '\$draid3-0-s1'
  
  Retire Agent: zpool\_vdev\_attach 'sdg' busy. Saving request.'
  
  Retire Agent: Saved request pool\_guid 1320611588736634121 vdev\_guid 5296963598540981156.
  
  Diagnosis Engine: resource event 'resource.fs.zfs.statechange'
  
  Retire Agent: zfs\_retire\_recv: 'resource.fs.zfs.statechange'
  --------------------------------------------------------------------------------------------

1.  After the first rebuild completed, the Retire Agent received the
    rebuild\_finish event, then replayed the retained spare request for
    the second distributed spare drive (\$draid3-0-s1).

  ------------------------------------------------------------------------------------------------------
  Retire Agent: zfs\_retire\_recv: 'sysevent.fs.zfs.rebuild\_finish'
  
  Retire Agent: Replaying spare request pool\_guid 1320611588736634121 vdev\_guid 5296963598540981156.
  
  Retire Agent: matched vdev 5296963598540981156
  
  Retire Agent: zpool\_vdev\_replace 'sdg' with spare '\$draid3-0-s1'
  ------------------------------------------------------------------------------------------------------

Initiation of this second dRAID rebuild could be seen in the zpool
status.

  -----------------------------------------------------------------------------
  pool: mfault
  
  state: DEGRADED
  
  status: One or more devices are faulted in response to persistent errors.
  
  Sufficient replicas exist for the pool to continue functioning in a
  
  degraded state.
  
  action: Replace the faulted device, or use 'zpool clear' to mark the device
  
  repaired.
  
  scan: rebuild in progress since Tue Jun 27 20:05:43 2017
  
  11.6G scanned out of 128G at 1.29G/s, 0h1m to go
  
  762M rebuilt, 9.07% done
  
  config:
  
  NAME STATE READ WRITE CKSUM
  
  mfault DEGRADED 0 0 0
  
  draid3-0 DEGRADED 0 0 0
  
  spare-0 DEGRADED 0 0 0
  
  sdb FAULTED 66 0 0 too many errors
  
  \$draid3-0-s0 ONLINE 0 0 0 (repairing)
  
  sdc ONLINE 0 0 0 (repairing)
  
  sdd ONLINE 0 0 0 (repairing)
  
  sde ONLINE 0 0 0 (repairing)
  
  sdf ONLINE 0 0 0 (repairing)
  
  spare-5 DEGRADED 0 0 0
  
  sdg FAULTED 65 0 0 too many errors
  
  \$draid3-0-s1 ONLINE 0 0 0 (repairing)
  
  sdh ONLINE 0 0 0 (repairing)
  
  sdi ONLINE 0 0 0 (repairing)
  
  sdj ONLINE 0 0 0 (repairing)
  
  sdk ONLINE 0 0 0 (repairing)
  
  sdl ONLINE 0 0 0 (repairing)
  
  sdm FAULTED 495 0 0 too many errors
  
  spares
  
  \$draid3-0-s0 INUSE currently in use
  
  \$draid3-0-s1 INUSE currently in use
  
  \$draid3-0-s2 AVAIL
  
  errors: No known data errors
  -----------------------------------------------------------------------------

##### Third Failure

1.  The third fault occurred while the first rebuild was in progress.
    The Retire Agent received the fault from the Diagnosis Engine and
    sent a fault event for the drive to ZFS. The Agent attempted to swap
    in the next available spare (still the 2^nd^ distributed spare
    drive) only to find that the device was busy because a rebuild was
    in progress. The Retire Agent saved the request to be replayed later
    after the rebuild completes.

  --------------------------------------------------------------------------------------------
  Diagnosis Engine: solving fault 'fault.fs.zfs.vdev.io'
  
  zed\_fault\_event:
  
  uuid: ae95fc42-ccd1-4a03-924c-649ecda66de8
  
  class: fault.fs.zfs.vdev.io
  
  code: ZFS-8000-FD
  
  certainty: 100
  
  scheme: zfs
  
  pool: 1320611588736634121
  
  vdev: 4853321467358484743
  
  Diagnosis Engine: case solved (ae95fc42-ccd1-4a03-924c-649ecda66de8)
  
  Diagnosis Engine: removing timer (0x7f12d0045460)
  
  Retire Agent: zfs\_retire\_recv: 'list.suspect'
  
  Retire Agent: matched vdev 4853321467358484743
  
  Retire Agent: zpool\_vdev\_fault: vdev 4853321467358484743 on 'mfault'
  
  Retire Agent: zpool\_vdev\_replace 'sdm' with spare '\$draid3-0-s1'
  
  Retire Agent: zpool\_vdev\_attach 'sdm' busy. Saving request.'
  
  Retire Agent: Saved request pool\_guid 1320611588736634121 vdev\_guid 4853321467358484743.
  
  Diagnosis Engine: resource event 'resource.fs.zfs.statechange'
  
  Retire Agent: zfs\_retire\_recv: 'resource.fs.zfs.statechange'
  --------------------------------------------------------------------------------------------

Because the fault of sdm is the second spare request saved, the drive
will not be replaced until after the dRAID recovers from the fault of
the second failed drive (sdg). The replacement of sdm to the third
distributed spare (\$draid3-0-s2) starts after the Retire Agent received
the rebuild\_finish for sdg.

  ------------------------------------------------------------------------------------------------------
  Retire Agent: zfs\_retire\_recv: 'sysevent.fs.zfs.rebuild\_finish'
  
  Retire Agent: Replaying spare request pool\_guid 1320611588736634121 vdev\_guid 4853321467358484743.
  
  Retire Agent: matched vdev 4853321467358484743
  
  Retire Agent: zpool\_vdev\_replace 'sdm' with spare '\$draid3-0-s2'
  ------------------------------------------------------------------------------------------------------

Zpool status shows that rebuild of the third failed drive was in
progress.

  -----------------------------------------------------------------------------
  pool: mfault
  
  state: DEGRADED
  
  status: One or more devices are faulted in response to persistent errors.
  
  Sufficient replicas exist for the pool to continue functioning in a
  
  degraded state.
  
  action: Replace the faulted device, or use 'zpool clear' to mark the device
  
  repaired.
  
  scan: rebuild in progress since Tue Jun 27 20:07:44 2017
  
  1.96G scanned out of 128G at 1002M/s, 0h2m to go
  
  103M rebuilt, 1.53% done
  
  config:
  
  NAME STATE READ WRITE CKSUM
  
  mfault DEGRADED 0 0 0
  
  draid3-0 DEGRADED 0 0 0
  
  spare-0 DEGRADED 0 0 0
  
  sdb FAULTED 66 0 0 too many errors
  
  \$draid3-0-s0 ONLINE 0 0 0 (repairing)
  
  sdc ONLINE 0 0 0 (repairing)
  
  sdd ONLINE 0 0 0 (repairing)
  
  sde ONLINE 0 0 0 (repairing)
  
  sdf ONLINE 0 0 0 (repairing)
  
  spare-5 DEGRADED 0 0 0
  
  sdg FAULTED 65 0 0 too many errors
  
  \$draid3-0-s1 ONLINE 0 0 0 (repairing)
  
  sdh ONLINE 0 0 0 (repairing)
  
  sdi ONLINE 0 0 0 (repairing)
  
  sdj ONLINE 0 0 0 (repairing)
  
  sdk ONLINE 0 0 0 (repairing)
  
  sdl ONLINE 0 0 0 (repairing)
  
  spare-11 DEGRADED 0 0 0
  
  sdm FAULTED 495 0 0 too many errors
  
  \$draid3-0-s2 ONLINE 0 0 0 (repairing)
  
  spares
  
  \$draid3-0-s0 INUSE currently in use
  
  \$draid3-0-s1 INUSE currently in use
  
  \$draid3-0-s2 INUSE currently in use
  
  errors: No known data errors
  -----------------------------------------------------------------------------

##### Rebuild Complete

ZFS will send a state change event to the Diagnosis Engine after the
rebuild of each drive completes to indicate that the repair is complete
and the replaced drive is healthy.

  --------------------------------------------------------------------------------------
  Diagnosis Engine: resource event 'resource.fs.zfs.statechange'
  
  Diagnosis Engine: closing case after a device statechange to healthy
  
  Diagnosis Engine: case closed (cc7e90a9-f96d-4937-ace2-54502acfc9ec)
  
  Diagnosis Engine: serd\_destroy zfs\_1253c13e38e94d09\_b53df7b7fad57abe\_io
  
  Retire Agent: zfs\_retire\_recv: 'resource.fs.zfs.statechange'
  
  Retire Agent: marking repaired vdev 13059866864003676862 on pool 1320611588736634121
  
  Diagnosis Engine: resource event 'resource.fs.zfs.statechange'
  
  Diagnosis Engine: closing case after a device statechange to healthy
  
  Diagnosis Engine: case closed (602adf2c-c4dc-4613-88de-de3442182e6d)
  
  Diagnosis Engine: serd\_destroy zfs\_1253c13e38e94d09\_498298540f35c3a4\_io
  
  Retire Agent: zfs\_retire\_recv: 'resource.fs.zfs.statechange'
  
  Retire Agent: marking repaired vdev 5296963598540981156 on pool 1320611588736634121
  
  Diagnosis Engine: resource event 'resource.fs.zfs.statechange'
  
  Diagnosis Engine: closing case after a device statechange to healthy
  
  Diagnosis Engine: case closed (ae95fc42-ccd1-4a03-924c-649ecda66de8)
  
  Diagnosis Engine: serd\_destroy zfs\_1253c13e38e94d09\_435a76291aae1907\_io
  
  Retire Agent: zfs\_retire\_recv: 'resource.fs.zfs.statechange'
  
  Retire Agent: marking repaired vdev 4853321467358484743 on pool 1320611588736634121
  --------------------------------------------------------------------------------------

Upon completion of the last rebuild, zpool status showed that all three
spares were in use and the pool was restored to full redundancy. Note,
however, that ZFS considers use of a spare device to be a fault and the
zpool status will continue to report the array to be in a degraded state
until the failed drives are physically replaced, the recovered blocks
are rebalanced to the replacement drives, and the distributed spare
drives are restored and available for the next failure.

  -----------------------------------------------------------------------------
  pool: mfault
  
  state: DEGRADED
  
  status: One or more devices are faulted in response to persistent errors.
  
  Sufficient replicas exist for the pool to continue functioning in a
  
  degraded state.
  
  action: Replace the faulted device, or use 'zpool clear' to mark the device
  
  repaired.
  
  scan: rebuilt 12.8G in 0h2m25s with 0 errors on Tue Jun 27 20:10:09 2017
  
  config:
  
  NAME STATE READ WRITE CKSUM
  
  mfault DEGRADED 0 0 0
  
  draid3-0 DEGRADED 0 0 0
  
  spare-0 DEGRADED 0 0 0
  
  sdb FAULTED 66 0 0 too many errors
  
  \$draid3-0-s0 ONLINE 0 0 0
  
  sdc ONLINE 0 0 0
  
  sdd ONLINE 0 0 0
  
  sde ONLINE 0 0 0
  
  sdf ONLINE 0 0 0
  
  spare-5 DEGRADED 0 0 0
  
  sdg FAULTED 65 0 0 too many errors
  
  \$draid3-0-s1 ONLINE 0 0 0
  
  sdh ONLINE 0 0 0
  
  sdi ONLINE 0 0 0
  
  sdj ONLINE 0 0 0
  
  sdk ONLINE 0 0 0
  
  sdl ONLINE 0 0 0
  
  spare-11 DEGRADED 0 0 0
  
  sdm FAULTED 495 0 0 too many errors
  
  \$draid3-0-s2 ONLINE 0 0 0
  
  spares
  
  \$draid3-0-s0 INUSE currently in use
  
  \$draid3-0-s1 INUSE currently in use
  
  \$draid3-0-s2 INUSE currently in use
  
  errors: No known data errors
  -----------------------------------------------------------------------------

Validation 
===========

ZFS Test Suite 
---------------

1.  For the prior milestone, we reported our efforts to improve the ZFS
    Test Suite (ZTS) for ZFS on Linux. ZTS is code from OpenZFS that
    provides a set of regression tests to ensure that new feature
    development does not negatively impact existing ZFS Features. Before
    new components are merged into our ZFS source tree, they must pass
    the ZTS.

    For MS05/09 we added the following tests :

-   **Auto on line**
    ([PR-5350](https://github.com/zfsonlinux/zfs/pull/5350)) –
    Automatically places a drive online if it was previously included in
    the pool and is recognized as the same device that had been removed.

-   **Auto replace**
    ([PR-5869](https://github.com/zfsonlinux/zfs/pull/5869)) – Allows a
    new drive to be added and populated to the pool, resilvering the
    data to the new drive.

-   **Auto spare**
    ([PR-6280](https://github.com/zfsonlinux/zfs/pull/6280)) –
    Identifies when a drive has had too many read/write faults and
    transfers the data to an existing hot spare in the pool.

-   **Allocation Classes** – Tests both segregated and dedicated biases
    and ensures that spillover into the normal class works properly in
    the segregated case.

1.  In addition, we modified zinject to enable fault injection as a
    percentage rather than as an all or none event
    ([PR-6227](https://github.com/zfsonlinux/zfs/pull/6227)).

ZTest/zloop Verification Tests
------------------------------

1.  Ztest and zloop have been modified to test new functionality related
    to ABD, Allocation Classes and dRAID. When creating configurations,
    zloop and ztest will randomly opt for creating dRAID pools and opt
    to turn allocation classes on for those pools. In addition randomly
    through the tests it will flip linear vs scatter gather allocation
    on and off for ABD. The Allocation classes and dRAID functionality
    can be specified through a new set of command line options to both
    Ztest and Zloop.

    Ztest is one of the most valuable tools we have for the verification
    of the ZFS file system. Several developers worked to implement ztest
    to be able to test dRAID. Once this work was done, it was necessary
    to improve the script that ran as a test harness driving Ztest to
    test the product. Two improved versions of zloop came out of this
    effort.

    The first improved the random test coverage of zloop for both raidz
    and dRAID. This was done by writing a new section, which chose
    between a basic configuration, a raidz mix and a dRAID mix. The goal
    was to see around 25% coverage of basic, 25% coverage of raidz, and
    50% of dRAID mix. This was intentionally slanted to dRAID, as it was
    the major feature we were developing. The size of the vdevs for all
    configurations was raised to at least 256 and if file system space
    allowed, it could grow all the way up to 512-360m depending on the
    configuration.

    The second zloop test was created to overcome a shortcoming of the
    first to rely on randomly selecting configurations to test, which
    did not programmatically ensure every configuration was tested. A
    zloop was made that would go through dRAID parity, number of data
    drives, groups of data drives, and spares systematically to ensure
    coverage. This test also allowed us to find the boundaries of what
    configuration ztest would allow — 159 VDEVs in any configuration is
    the limit before ztest will fail. These tests were set to run for
    hours at a time and bugs were added to our Jira tracker. This proved
    to be one of the best ways to get a quick read on a patch or set of
    patches when checked into a branch.

[[[]{#_Toc488392501 .anchor}]{#_Toc486599638 .anchor}]{#_Toc486606488
.anchor}Table ‑. zTest dRAID Options

  -------------------------------------------------------------------------------------------------------
  Parameter                                           Default      Description
  --------------------------------------------------- ------------ --------------------------------------
  1.  -K&lt;kind&gt;                                  1.  random   1.  raidz| draid| random
                                                                   

  1.  -D &lt;number&gt;                               1.  4        1.  Data drives per redundancy group
                                                                   

  1.  -G &lt;number&gt;                               1.  2        1.  Number of redundancy groups
                                                                   

  1.  -S &lt;number&gt;                               1.  1        1.  Distributedspare drives
                                                                   

  1.  -R&lt;number&gt;                                1.  1        1.  RAID group parity
                                                                   

  1.  -s &lt;number&gt;                               1.  128M     1.  Size of each leaf disk
                                                                   

  1.  **Example:**
  
      ztest-VVV -K draid-D 4 -G 3 -S 1 -R 1 -s 256m
  
  -------------------------------------------------------------------------------------------------------

1.  

File System Ager
----------------

1.  As reported in the previous milestone report, we developed a file
    system ager in Python so that testing was not always on clean file
    systems. Unfortunately, instead of extending the test coverage for
    the new ZFS code, the tool uncovered LU-9305 in its many forms
    (Section 3.3.2). Once the Lustre team resolves this bug, the tool
    will be available for continuous integration testing as Intel
    continues to support the ZFS features within the Lustre and OpenZFS
    communities.

dRAID Exerciser
---------------

1.  Another tool developed for feature validation is the dRAID
    exerciser. The tool currently uses the ‘draidcfg’ command to
    generate dRAID configurations and zinject to force random drive
    failures. Further development is needed to move the dRAID exerciser
    upstream to the community.

dRAID Configuration Examples {#draid-configuration-examples .Appendix1}
============================

1.  The dRAID demonstration used a storage array with 80 drives. As a
    result the configuration file and ‘zpool status’ output were too
    large to include in the main body of the report.

‘zdb –m’ for a dRAID pool without segregation {#zdb-m-for-a-draid-pool-without-segregation .Appendix2}
=============================================

The following listing shows the status of the simple 43-drive dRAID
described in section 3.2.4.2. This dRAID was not configured to use
metadata isolation. As a result, all metaslabs will be used for all
categories of ZFS storage (generic, metadata, small block).

  ----------------------------------------------------------------------------------
  \[root@ssu2\_oss1\]\# zdb -m ssu\_2ost0
  
  Metaslabs:
  
  vdev 0
  
  metaslabs 291 offset size spacemap free
  
  --------------- ------------------- --------------- --------------- ------------
  
  metaslab 0 offset 0 size 4000000000 spacemap 114 free 7.36M
  
  metaslab 1 offset 4000006000 size 3fffffa000 spacemap 113 free 1.64G
  
  metaslab 2 offset 8000002000 size 3fffffe000 spacemap 112 free 861M
  
  metaslab 3 offset c000008000 size 3fffff8000 spacemap 123 free 1.04G
  
  metaslab 4 offset 10000004000 size 3fffffc000 spacemap 122 free 1.07G
  
  metaslab 5 offset 14000000000 size 4000000000 spacemap 124 free 993M
  
  metaslab 6 offset 18000006000 size 3fffffa000 spacemap 125 free 794M
  
  metaslab 7 offset 1c000002000 size 3fffffe000 spacemap 126 free 1.03G
  
  metaslab 8 offset 20000008000 size 3fffff8000 spacemap 128 free 973M
  
  metaslab 9 offset 24000004000 size 3fffffc000 spacemap 127 free 1.19G
  
  metaslab 10 offset 28000000000 size 4000000000 spacemap 130 free 1.86G
  
  metaslab 11 offset 2c000006000 size 3fffffa000 spacemap 129 free 1.15G
  
  metaslab 12 offset 30000002000 size 3fffffe000 spacemap 132 free 1.47G
  
  metaslab 13 offset 34000008000 size 3fffff8000 spacemap 131 free 746M
  
  metaslab 14 offset 38000004000 size 3fffffc000 spacemap 134 free 1.34G
  
  metaslab 15 offset 3c000000000 size 4000000000 spacemap 133 free 1.25G
  
  metaslab 16 offset 40000006000 size 3fffffa000 spacemap 136 free 1001M
  
  metaslab 17 offset 44000002000 size 3fffffe000 spacemap 135 free 1.31G
  
  metaslab 18 offset 48000008000 size 3fffff8000 spacemap 138 free 1.03G
  
  metaslab 19 offset 4c000004000 size 3fffffc000 spacemap 137 free 1.01G
  
  metaslab 20 offset 50000000000 size 4000000000 spacemap 140 free 1.17G
  
  metaslab 21 offset 54000006000 size 3fffffa000 spacemap 139 free 1.13G
  
  metaslab 22 offset 58000002000 size 3fffffe000 spacemap 142 free 992M
  
  metaslab 23 offset 5c000008000 size 3fffff8000 spacemap 141 free 863M
  
  metaslab 24 offset 60000004000 size 3fffffc000 spacemap 144 free 7.61G
  
  metaslab 25 offset 64000000000 size 4000000000 spacemap 143 free 4.92G
  
  metaslab 26 offset 68000006000 size 3fffffa000 spacemap 145 free 29.4G
  
  metaslab 27 offset 6c000002000 size 3fffffe000 spacemap 147 free 17.7G
  
  metaslab 28 offset 70000008000 size 3fffff8000 spacemap 146 free 1.28G
  
  metaslab 29 offset 74000004000 size 3fffffc000 spacemap 148 free 1.05G
  
  metaslab 30 offset 78000000000 size 4000000000 spacemap 151 free 13.4G
  
  metaslab 31 offset 7c000006000 size 3fffffa000 spacemap 150 free 1.86G
  
  metaslab 32 offset 80000002000 size 3fffffe000 spacemap 149 free 1.42G
  
  metaslab 33 offset 84000008000 size 3fffff8000 spacemap 154 free 7.43G
  
  metaslab 34 offset 88000004000 size 3fffffc000 spacemap 153 free 21.1G
  
  metaslab 35 offset 8c000000000 size 4000000000 spacemap 152 free 1009M
  
  metaslab 36 offset 90000006000 size 3fffffa000 spacemap 156 free 15.9G
  
  metaslab 37 offset 94000002000 size 3fffffe000 spacemap 155 free 2.11G
  
  metaslab 38 offset 98000008000 size 3fffff8000 spacemap 157 free 1.59G
  
  metaslab 39 offset 9c000004000 size 3fffffc000 spacemap 160 free 43.9G
  
  metaslab 40 offset a0000000000 size 4000000000 spacemap 159 free 3.93G
  
  metaslab 41 offset a4000006000 size 3fffffa000 spacemap 158 free 1.10G
  
  metaslab 42 offset a8000002000 size 3fffffe000 spacemap 163 free 27.7G
  
  metaslab 43 offset ac000008000 size 3fffff8000 spacemap 162 free 2.59G
  
  metaslab 44 offset b0000004000 size 3fffffc000 spacemap 161 free 2.38G
  
  metaslab 45 offset b4000000000 size 4000000000 spacemap 165 free 33.6G
  
  metaslab 46 offset b8000006000 size 3fffffa000 spacemap 164 free 25.1G
  
  metaslab 47 offset bc000002000 size 3fffffe000 spacemap 166 free 1.25G
  
  metaslab 48 offset c0000008000 size 3fffff8000 spacemap 169 free 27.1G
  
  metaslab 49 offset c4000004000 size 3fffffc000 spacemap 168 free 2.46G
  
  metaslab 50 offset c8000000000 size 4000000000 spacemap 167 free 40.4G
  
  metaslab 51 offset cc000006000 size 3fffffa000 spacemap 171 free 49.5G
  
  metaslab 52 offset d0000002000 size 3fffffe000 spacemap 170 free 2.11G
  
  metaslab 53 offset d4000008000 size 3fffff8000 spacemap 172 free 1.05G
  
  metaslab 54 offset d8000004000 size 3fffffc000 spacemap 174 free 28.1G
  
  metaslab 55 offset dc000000000 size 4000000000 spacemap 173 free 1.34G
  
  metaslab 56 offset e0000006000 size 3fffffa000 spacemap 175 free 1.44G
  
  metaslab 57 offset e4000002000 size 3fffffe000 spacemap 178 free 27.5G
  
  metaslab 58 offset e8000008000 size 3fffff8000 spacemap 177 free 2.59G
  
  metaslab 59 offset ec000004000 size 3fffffc000 spacemap 176 free 708M
  
  metaslab 60 offset f0000000000 size 4000000000 spacemap 181 free 22.6G
  
  metaslab 61 offset f4000006000 size 3fffffa000 spacemap 180 free 2.70G
  
  metaslab 62 offset f8000002000 size 3fffffe000 spacemap 179 free 1003M
  
  metaslab 63 offset fc000008000 size 3fffff8000 spacemap 184 free 25.2G
  
  metaslab 64 offset 100000004000 size 3fffffc000 spacemap 183 free 2.73G
  
  metaslab 65 offset 104000000000 size 4000000000 spacemap 182 free 647M
  
  metaslab 66 offset 108000006000 size 3fffffa000 spacemap 187 free 29.2G
  
  metaslab 67 offset 10c000002000 size 3fffffe000 spacemap 186 free 2.49G
  
  metaslab 68 offset 110000008000 size 3fffff8000 spacemap 185 free 45.8G
  
  metaslab 69 offset 114000004000 size 3fffffc000 spacemap 189 free 26.1G
  
  metaslab 70 offset 118000000000 size 4000000000 spacemap 188 free 1.97G
  
  metaslab 71 offset 11c000006000 size 3fffffa000 spacemap 190 free 1.07G
  
  metaslab 72 offset 120000002000 size 3fffffe000 spacemap 193 free 27.5G
  
  metaslab 73 offset 124000008000 size 3fffff8000 spacemap 192 free 2.74G
  
  metaslab 74 offset 128000004000 size 3fffffc000 spacemap 191 free 1.77G
  
  metaslab 75 offset 12c000000000 size 4000000000 spacemap 196 free 28.1G
  
  metaslab 76 offset 130000006000 size 3fffffa000 spacemap 195 free 40.2G
  
  metaslab 77 offset 134000002000 size 3fffffe000 spacemap 194 free 777M
  
  metaslab 78 offset 138000008000 size 3fffff8000 spacemap 198 free 26.7G
  
  metaslab 79 offset 13c000004000 size 3fffffc000 spacemap 197 free 1.78G
  
  metaslab 80 offset 140000000000 size 4000000000 spacemap 199 free 35.1G
  
  metaslab 81 offset 144000006000 size 3fffffa000 spacemap 201 free 27.9G
  
  metaslab 82 offset 148000002000 size 3fffffe000 spacemap 200 free 41.0G
  
  metaslab 83 offset 14c000008000 size 3fffff8000 spacemap 202 free 38.2G
  
  metaslab 84 offset 150000004000 size 3fffffc000 spacemap 203 free 17.9G
  
  metaslab 85 offset 154000000000 size 4000000000 spacemap 204 free 1.70G
  
  metaslab 86 offset 158000006000 size 3fffffa000 spacemap 205 free 1.84G
  
  metaslab 87 offset 15c000002000 size 3fffffe000 spacemap 208 free 27.3G
  
  metaslab 88 offset 160000008000 size 3fffff8000 spacemap 207 free 3.12G
  
  metaslab 89 offset 164000004000 size 3fffffc000 spacemap 206 free 1.51G
  
  metaslab 90 offset 168000000000 size 4000000000 spacemap 211 free 25.8G
  
  metaslab 91 offset 16c000006000 size 3fffffa000 spacemap 210 free 3.29G
  
  metaslab 92 offset 170000002000 size 3fffffe000 spacemap 209 free 2.14G
  
  metaslab 93 offset 174000008000 size 3fffff8000 spacemap 214 free 22.3G
  
  metaslab 94 offset 178000004000 size 3fffffc000 spacemap 213 free 2.64G
  
  metaslab 95 offset 17c000000000 size 4000000000 spacemap 212 free 1.52G
  
  metaslab 96 offset 180000006000 size 3fffffa000 spacemap 217 free 22.4G
  
  metaslab 97 offset 184000002000 size 3fffffe000 spacemap 216 free 2.64G
  
  metaslab 98 offset 188000008000 size 3fffff8000 spacemap 215 free 1.79G
  
  metaslab 99 offset 18c000004000 size 3fffffc000 spacemap 220 free 26.5G
  
  metaslab 100 offset 190000000000 size 4000000000 spacemap 219 free 2.58G
  
  metaslab 101 offset 194000006000 size 3fffffa000 spacemap 218 free 1.72G
  
  metaslab 102 offset 198000002000 size 3fffffe000 spacemap 223 free 27.0G
  
  metaslab 103 offset 19c000008000 size 3fffff8000 spacemap 222 free 2.66G
  
  metaslab 104 offset 1a0000004000 size 3fffffc000 spacemap 221 free 1.24G
  
  metaslab 105 offset 1a4000000000 size 4000000000 spacemap 226 free 26.1G
  
  metaslab 106 offset 1a8000006000 size 3fffffa000 spacemap 225 free 2.84G
  
  metaslab 107 offset 1ac000002000 size 3fffffe000 spacemap 224 free 1.55G
  
  metaslab 108 offset 1b0000008000 size 3fffff8000 spacemap 229 free 44.5G
  
  metaslab 109 offset 1b4000004000 size 3fffffc000 spacemap 228 free 2.96G
  
  metaslab 110 offset 1b8000000000 size 4000000000 spacemap 227 free 1.53G
  
  metaslab 111 offset 1bc000006000 size 3fffffa000 spacemap 231 free 28.0G
  
  metaslab 112 offset 1c0000002000 size 3fffffe000 spacemap 230 free 2.73G
  
  metaslab 113 offset 1c4000008000 size 3fffff8000 spacemap 232 free 1.62G
  
  metaslab 114 offset 1c8000004000 size 3fffffc000 spacemap 235 free 46.8G
  
  metaslab 115 offset 1cc000000000 size 4000000000 spacemap 234 free 4.07G
  
  metaslab 116 offset 1d0000006000 size 3fffffa000 spacemap 233 free 1.59G
  
  metaslab 117 offset 1d4000002000 size 3fffffe000 spacemap 238 free 25.9G
  
  metaslab 118 offset 1d8000008000 size 3fffff8000 spacemap 237 free 2.92G
  
  metaslab 119 offset 1dc000004000 size 3fffffc000 spacemap 236 free 1.54G
  
  metaslab 120 offset 1e0000000000 size 4000000000 spacemap 241 free 28.7G
  
  metaslab 121 offset 1e4000006000 size 3fffffa000 spacemap 240 free 2.55G
  
  metaslab 122 offset 1e8000002000 size 3fffffe000 spacemap 239 free 1.38G
  
  metaslab 123 offset 1ec000008000 size 3fffff8000 spacemap 244 free 28.7G
  
  metaslab 124 offset 1f0000004000 size 3fffffc000 spacemap 243 free 4.02G
  
  metaslab 125 offset 1f4000000000 size 4000000000 spacemap 242 free 1.64G
  
  metaslab 126 offset 1f8000006000 size 3fffffa000 spacemap 247 free 28.9G
  
  metaslab 127 offset 1fc000002000 size 3fffffe000 spacemap 246 free 3.00G
  
  metaslab 128 offset 200000008000 size 3fffff8000 spacemap 245 free 2.16G
  
  metaslab 129 offset 204000004000 size 3fffffc000 spacemap 250 free 26.7G
  
  metaslab 130 offset 208000000000 size 4000000000 spacemap 249 free 2.63G
  
  metaslab 131 offset 20c000006000 size 3fffffa000 spacemap 248 free 38.5G
  
  metaslab 132 offset 210000002000 size 3fffffe000 spacemap 252 free 17.1G
  
  metaslab 133 offset 214000008000 size 3fffff8000 spacemap 251 free 2.03G
  
  metaslab 134 offset 218000004000 size 3fffffc000 spacemap 253 free 2.19G
  
  metaslab 135 offset 21c000000000 size 4000000000 spacemap 255 free 43.2G
  
  metaslab 136 offset 220000006000 size 3fffffa000 spacemap 254 free 2.22G
  
  metaslab 137 offset 224000002000 size 3fffffe000 spacemap 256 free 1.58G
  
  metaslab 138 offset 228000008000 size 3fffff8000 spacemap 259 free 25.6G
  
  metaslab 139 offset 22c000004000 size 3fffffc000 spacemap 258 free 2.58G
  
  metaslab 140 offset 230000000000 size 4000000000 spacemap 257 free 35.2G
  
  metaslab 141 offset 234000006000 size 3fffffa000 spacemap 261 free 26.5G
  
  metaslab 142 offset 238000002000 size 3fffffe000 spacemap 260 free 1.74G
  
  metaslab 143 offset 23c000008000 size 3fffff8000 spacemap 262 free 1.92G
  
  metaslab 144 offset 240000004000 size 3fffffc000 spacemap 265 free 25.5G
  
  metaslab 145 offset 244000000000 size 4000000000 spacemap 264 free 3.25G
  
  metaslab 146 offset 248000006000 size 3fffffa000 spacemap 263 free 2.17G
  
  metaslab 147 offset 24c000002000 size 3fffffe000 spacemap 268 free 44.3G
  
  metaslab 148 offset 250000008000 size 3fffff8000 spacemap 267 free 3.64G
  
  metaslab 149 offset 254000004000 size 3fffffc000 spacemap 266 free 1.56G
  
  metaslab 150 offset 258000000000 size 4000000000 spacemap 271 free 27.5G
  
  metaslab 151 offset 25c000006000 size 3fffffa000 spacemap 270 free 2.11G
  
  metaslab 152 offset 260000002000 size 3fffffe000 spacemap 269 free 1.91G
  
  metaslab 153 offset 264000008000 size 3fffff8000 spacemap 274 free 26.9G
  
  metaslab 154 offset 268000004000 size 3fffffc000 spacemap 273 free 3.66G
  
  metaslab 155 offset 26c000000000 size 4000000000 spacemap 272 free 1.43G
  
  metaslab 156 offset 270000006000 size 3fffffa000 spacemap 277 free 25.6G
  
  metaslab 157 offset 274000002000 size 3fffffe000 spacemap 276 free 2.62G
  
  metaslab 158 offset 278000008000 size 3fffff8000 spacemap 275 free 1.89G
  
  metaslab 159 offset 27c000004000 size 3fffffc000 spacemap 280 free 27.4G
  
  metaslab 160 offset 280000000000 size 4000000000 spacemap 279 free 3.38G
  
  metaslab 161 offset 284000006000 size 3fffffa000 spacemap 278 free 1.39G
  
  metaslab 162 offset 288000002000 size 3fffffe000 spacemap 283 free 26.4G
  
  metaslab 163 offset 28c000008000 size 3fffff8000 spacemap 282 free 2.76G
  
  metaslab 164 offset 290000004000 size 3fffffc000 spacemap 281 free 1.72G
  
  metaslab 165 offset 294000000000 size 4000000000 spacemap 286 free 23.1G
  
  metaslab 166 offset 298000006000 size 3fffffa000 spacemap 285 free 3.01G
  
  metaslab 167 offset 29c000002000 size 3fffffe000 spacemap 284 free 48.2G
  
  metaslab 168 offset 2a0000008000 size 3fffff8000 spacemap 288 free 26.0G
  
  metaslab 169 offset 2a4000004000 size 3fffffc000 spacemap 287 free 2.25G
  
  metaslab 170 offset 2a8000000000 size 4000000000 spacemap 289 free 38.5G
  
  metaslab 171 offset 2ac000006000 size 3fffffa000 spacemap 291 free 27.0G
  
  metaslab 172 offset 2b0000002000 size 3fffffe000 spacemap 290 free 40.5G
  
  metaslab 173 offset 2b4000008000 size 3fffff8000 spacemap 292 free 993M
  
  metaslab 174 offset 2b8000004000 size 3fffffc000 spacemap 294 free 18.7G
  
  metaslab 175 offset 2bc000000000 size 4000000000 spacemap 293 free 1.82G
  
  metaslab 176 offset 2c0000006000 size 3fffffa000 spacemap 295 free 1.64G
  
  metaslab 177 offset 2c4000002000 size 3fffffe000 spacemap 298 free 3.25G
  
  metaslab 178 offset 2c8000008000 size 3fffff8000 spacemap 297 free 2.57G
  
  metaslab 179 offset 2cc000004000 size 3fffffc000 spacemap 296 free 1.36G
  
  metaslab 180 offset 2d0000000000 size 4000000000 spacemap 301 free 6.21G
  
  metaslab 181 offset 2d4000006000 size 3fffffa000 spacemap 300 free 2.58G
  
  metaslab 182 offset 2d8000002000 size 3fffffe000 spacemap 299 free 1.73G
  
  metaslab 183 offset 2dc000008000 size 3fffff8000 spacemap 304 free 27.6G
  
  metaslab 184 offset 2e0000004000 size 3fffffc000 spacemap 303 free 2.64G
  
  metaslab 185 offset 2e4000000000 size 4000000000 spacemap 302 free 1.11G
  
  metaslab 186 offset 2e8000006000 size 3fffffa000 spacemap 307 free 6.57G
  
  metaslab 187 offset 2ec000002000 size 3fffffe000 spacemap 306 free 2.35G
  
  metaslab 188 offset 2f0000008000 size 3fffff8000 spacemap 305 free 1.52G
  
  metaslab 189 offset 2f4000004000 size 3fffffc000 spacemap 310 free 6.39G
  
  metaslab 190 offset 2f8000000000 size 4000000000 spacemap 309 free 2.17G
  
  metaslab 191 offset 2fc000006000 size 3fffffa000 spacemap 308 free 4.71G
  
  metaslab 192 offset 300000002000 size 3fffffe000 spacemap 313 free 4.68G
  
  metaslab 193 offset 304000008000 size 3fffff8000 spacemap 312 free 4.54G
  
  metaslab 194 offset 308000004000 size 3fffffc000 spacemap 311 free 1.39G
  
  metaslab 195 offset 30c000000000 size 4000000000 spacemap 316 free 3.55G
  
  metaslab 196 offset 310000006000 size 3fffffa000 spacemap 315 free 2.23G
  
  metaslab 197 offset 314000002000 size 3fffffe000 spacemap 314 free 1.38G
  
  metaslab 198 offset 318000008000 size 3fffff8000 spacemap 319 free 25.4G
  
  metaslab 199 offset 31c000004000 size 3fffffc000 spacemap 318 free 3.14G
  
  metaslab 200 offset 320000000000 size 4000000000 spacemap 317 free 39.3G
  
  metaslab 201 offset 324000006000 size 3fffffa000 spacemap 321 free 26.6G
  
  metaslab 202 offset 328000002000 size 3fffffe000 spacemap 320 free 1.99G
  
  metaslab 203 offset 32c000008000 size 3fffff8000 spacemap 322 free 1.68G
  
  metaslab 204 offset 330000004000 size 3fffffc000 spacemap 325 free 28.8G
  
  metaslab 205 offset 334000000000 size 4000000000 spacemap 324 free 2.95G
  
  metaslab 206 offset 338000006000 size 3fffffa000 spacemap 323 free 1.78G
  
  metaslab 207 offset 33c000002000 size 3fffffe000 spacemap 328 free 24.4G
  
  metaslab 208 offset 340000008000 size 3fffff8000 spacemap 327 free 29.8G
  
  metaslab 209 offset 344000004000 size 3fffffc000 spacemap 326 free 8.55G
  
  metaslab 210 offset 348000000000 size 4000000000 spacemap 330 free 26.9G
  
  metaslab 211 offset 34c000006000 size 3fffffa000 spacemap 329 free 1.99G
  
  metaslab 212 offset 350000002000 size 3fffffe000 spacemap 331 free 1.83G
  
  metaslab 213 offset 354000008000 size 3fffff8000 spacemap 334 free 26.7G
  
  metaslab 214 offset 358000004000 size 3fffffc000 spacemap 333 free 2.35G
  
  metaslab 215 offset 35c000000000 size 4000000000 spacemap 332 free 2.82G
  
  metaslab 216 offset 360000006000 size 3fffffa000 spacemap 337 free 26.5G
  
  metaslab 217 offset 364000002000 size 3fffffe000 spacemap 336 free 8.60G
  
  metaslab 218 offset 368000008000 size 3fffff8000 spacemap 335 free 2.16G
  
  metaslab 219 offset 36c000004000 size 3fffffc000 spacemap 340 free 21.6G
  
  metaslab 220 offset 370000000000 size 4000000000 spacemap 339 free 7.50G
  
  metaslab 221 offset 374000006000 size 3fffffa000 spacemap 338 free 1.52G
  
  metaslab 222 offset 378000002000 size 3fffffe000 spacemap 343 free 42.7G
  
  metaslab 223 offset 37c000008000 size 3fffff8000 spacemap 342 free 3.92G
  
  metaslab 224 offset 380000004000 size 3fffffc000 spacemap 341 free 2.40G
  
  metaslab 225 offset 384000000000 size 4000000000 spacemap 345 free 26.4G
  
  metaslab 226 offset 388000006000 size 3fffffa000 spacemap 344 free 2.50G
  
  metaslab 227 offset 38c000002000 size 3fffffe000 spacemap 346 free 32.9G
  
  metaslab 228 offset 390000008000 size 3fffff8000 spacemap 348 free 44.8G
  
  metaslab 229 offset 394000004000 size 3fffffc000 spacemap 347 free 1.67G
  
  metaslab 230 offset 398000000000 size 4000000000 spacemap 349 free 2.25G
  
  metaslab 231 offset 39c000006000 size 3fffffa000 spacemap 352 free 27.8G
  
  metaslab 232 offset 3a0000002000 size 3fffffe000 spacemap 351 free 3.14G
  
  metaslab 233 offset 3a4000008000 size 3fffff8000 spacemap 350 free 2.73G
  
  metaslab 234 offset 3a8000004000 size 3fffffc000 spacemap 355 free 24.6G
  
  metaslab 235 offset 3ac000000000 size 4000000000 spacemap 354 free 4.52G
  
  metaslab 236 offset 3b0000006000 size 3fffffa000 spacemap 353 free 1.96G
  
  metaslab 237 offset 3b4000002000 size 3fffffe000 spacemap 358 free 34.8G
  
  metaslab 238 offset 3b8000008000 size 3fffff8000 spacemap 357 free 2.18G
  
  metaslab 239 offset 3bc000004000 size 3fffffc000 spacemap 356 free 26.0G
  
  metaslab 240 offset 3c0000000000 size 4000000000 spacemap 359 free 1.15G
  
  metaslab 241 offset 3c4000006000 size 3fffffa000 spacemap 362 free 20.4G
  
  metaslab 242 offset 3c8000002000 size 3fffffe000 spacemap 361 free 2.05G
  
  metaslab 243 offset 3cc000008000 size 3fffff8000 spacemap 360 free 642M
  
  metaslab 244 offset 3d0000004000 size 3fffffc000 spacemap 365 free 15.7G
  
  metaslab 245 offset 3d4000000000 size 4000000000 spacemap 364 free 1.97G
  
  metaslab 246 offset 3d8000006000 size 3fffffa000 spacemap 363 free 1.09G
  
  metaslab 247 offset 3dc000002000 size 3fffffe000 spacemap 368 free 2.12G
  
  metaslab 248 offset 3e0000008000 size 3fffff8000 spacemap 367 free 1.25G
  
  metaslab 249 offset 3e4000004000 size 3fffffc000 spacemap 366 free 1.27G
  
  metaslab 250 offset 3e8000000000 size 4000000000 spacemap 371 free 737M
  
  metaslab 251 offset 3ec000006000 size 3fffffa000 spacemap 370 free 1.31G
  
  metaslab 252 offset 3f0000002000 size 3fffffe000 spacemap 369 free 1.43G
  
  metaslab 253 offset 3f4000008000 size 3fffff8000 spacemap 374 free 1.10G
  
  metaslab 254 offset 3f8000004000 size 3fffffc000 spacemap 373 free 1.12G
  
  metaslab 255 offset 3fc000000000 size 4000000000 spacemap 372 free 1.06G
  
  metaslab 256 offset 400000006000 size 3fffffa000 spacemap 377 free 829M
  
  metaslab 257 offset 404000002000 size 3fffffe000 spacemap 376 free 820M
  
  metaslab 258 offset 408000008000 size 3fffff8000 spacemap 375 free 774M
  
  metaslab 259 offset 40c000004000 size 3fffffc000 spacemap 380 free 1014M
  
  metaslab 260 offset 410000000000 size 4000000000 spacemap 379 free 648M
  
  metaslab 261 offset 414000006000 size 3fffffa000 spacemap 378 free 1.15G
  
  metaslab 262 offset 418000002000 size 3fffffe000 spacemap 383 free 1.04G
  
  metaslab 263 offset 41c000008000 size 3fffff8000 spacemap 382 free 1002M
  
  metaslab 264 offset 420000004000 size 3fffffc000 spacemap 381 free 955M
  
  metaslab 265 offset 424000000000 size 4000000000 spacemap 386 free 896M
  
  metaslab 266 offset 428000006000 size 3fffffa000 spacemap 385 free 986M
  
  metaslab 267 offset 42c000002000 size 3fffffe000 spacemap 384 free 1.15G
  
  metaslab 268 offset 430000008000 size 3fffff8000 spacemap 389 free 706M
  
  metaslab 269 offset 434000004000 size 3fffffc000 spacemap 388 free 1.10G
  
  metaslab 270 offset 438000000000 size 4000000000 spacemap 387 free 831M
  
  metaslab 271 offset 43c000006000 size 3fffffa000 spacemap 392 free 903M
  
  metaslab 272 offset 440000002000 size 3fffffe000 spacemap 391 free 777M
  
  metaslab 273 offset 444000008000 size 3fffff8000 spacemap 390 free 1.04G
  
  metaslab 274 offset 448000004000 size 3fffffc000 spacemap 393 free 794M
  
  metaslab 275 offset 44c000000000 size 4000000000 spacemap 394 free 695M
  
  metaslab 276 offset 450000006000 size 3fffffa000 spacemap 395 free 1.01G
  
  metaslab 277 offset 454000002000 size 3fffffe000 spacemap 398 free 899M
  
  metaslab 278 offset 458000008000 size 3fffff8000 spacemap 397 free 927M
  
  metaslab 279 offset 45c000004000 size 3fffffc000 spacemap 396 free 823M
  
  metaslab 280 offset 460000000000 size 4000000000 spacemap 401 free 821M
  
  metaslab 281 offset 464000006000 size 3fffffa000 spacemap 400 free 505M
  
  metaslab 282 offset 468000002000 size 3fffffe000 spacemap 399 free 926M
  
  metaslab 283 offset 46c000008000 size 3fffff8000 spacemap 404 free 960M
  
  metaslab 284 offset 470000004000 size 3fffffc000 spacemap 403 free 867M
  
  metaslab 285 offset 474000000000 size 4000000000 spacemap 402 free 509M
  
  metaslab 286 offset 478000006000 size 3fffffa000 spacemap 407 free 928M
  
  metaslab 287 offset 47c000002000 size 3fffffe000 spacemap 406 free 658M
  
  metaslab 288 offset 480000008000 size 3fffff8000 spacemap 405 free 931M
  
  metaslab 289 offset 484000004000 size 3fffffc000 spacemap 409 free 726M
  
  metaslab 290 offset 488000000000 size 4000000000 spacemap 408 free 583M
  ----------------------------------------------------------------------------------

 ‘zdb –m’ for a dRAID pool with segregation enabled {#zdb-m-for-a-draid-pool-with-segregation-enabled .Appendix2}
===================================================

The following listing shows the status of the 43-drive hybrid dRAID
described in section 3.2.4.2. This dRAID was configured to use metadata
isolation with segregation enabled. The listing includes an extra column
of that describes the class assignment for each metaslab. The first 20%
are reserved for the special class and will contain both metadata and
small block categories. The normal class will be used first for data
larger than 32KB in size. When the special class metaslabs are consumed,
small block I/O will spill over into the normal class.

  -------------------------------------------------------------------------------------------
  \[root@ssu1\_oss2\]\# zdb -m ssu\_lost1
  
  Metaslabs:
  
  vdev 0 segregate
  
  metaslabs 291 offset size spacemap free class
  
  --------------- ------------------- --------------- --------------- ------------ --------
  
  metaslab 0 offset 0 size 4000000000 spacemap 115 free 122G special
  
  metaslab 1 offset 4000000000 size 4000000000 spacemap 114 free 208G special
  
  metaslab 2 offset 8000001000 size 3ffffff000 spacemap 113 free 221G special
  
  metaslab 3 offset c000001000 size 3ffffff000 spacemap 4 free 256G special
  
  metaslab 4 offset 10000002000 size 3fffffe000 spacemap 3 free 256G special
  
  metaslab 5 offset 14000000000 size 4000000000 spacemap 2 free 256G special
  
  metaslab 6 offset 18000000000 size 4000000000 spacemap 7 free 256G special
  
  metaslab 7 offset 1c000001000 size 3ffffff000 spacemap 6 free 256G special
  
  metaslab 8 offset 20000001000 size 3ffffff000 spacemap 5 free 256G special
  
  metaslab 9 offset 24000002000 size 3fffffe000 spacemap 0 free 256G ----
  
  metaslab 10 offset 28000000000 size 4000000000 spacemap 0 free 256G ----
  
  metaslab 11 offset 2c000000000 size 4000000000 spacemap 0 free 256G ----
  
  metaslab 12 offset 30000001000 size 3ffffff000 spacemap 0 free 256G ----
  
  metaslab 13 offset 34000001000 size 3ffffff000 spacemap 0 free 256G ----
  
  metaslab 14 offset 38000002000 size 3fffffe000 spacemap 0 free 256G ----
  
  metaslab 15 offset 3c000000000 size 4000000000 spacemap 0 free 256G ----
  
  metaslab 16 offset 40000000000 size 4000000000 spacemap 0 free 256G ----
  
  metaslab 17 offset 44000001000 size 3ffffff000 spacemap 0 free 256G ----
  
  metaslab 18 offset 48000001000 size 3ffffff000 spacemap 0 free 256G ----
  
  metaslab 19 offset 4c000002000 size 3fffffe000 spacemap 0 free 256G ----
  
  metaslab 20 offset 50000000000 size 4000000000 spacemap 0 free 256G ----
  
  metaslab 21 offset 54000000000 size 4000000000 spacemap 0 free 256G ----
  
  metaslab 22 offset 58000001000 size 3ffffff000 spacemap 0 free 256G ----
  
  metaslab 23 offset 5c000001000 size 3ffffff000 spacemap 0 free 256G ----
  
  metaslab 24 offset 60000002000 size 3fffffe000 spacemap 0 free 256G ----
  
  metaslab 25 offset 64000000000 size 4000000000 spacemap 0 free 256G ----
  
  metaslab 26 offset 68000000000 size 4000000000 spacemap 0 free 256G ----
  
  metaslab 27 offset 6c000001000 size 3ffffff000 spacemap 0 free 256G ----
  
  metaslab 28 offset 70000001000 size 3ffffff000 spacemap 0 free 256G ----
  
  metaslab 29 offset 74000002000 size 3fffffe000 spacemap 0 free 256G ----
  
  metaslab 30 offset 78000000000 size 4000000000 spacemap 0 free 256G ----
  
  metaslab 31 offset 7c000000000 size 4000000000 spacemap 0 free 256G ----
  
  metaslab 32 offset 80000001000 size 3ffffff000 spacemap 0 free 256G ----
  
  metaslab 33 offset 84000001000 size 3ffffff000 spacemap 0 free 256G ----
  
  metaslab 34 offset 88000002000 size 3fffffe000 spacemap 0 free 256G ----
  
  metaslab 35 offset 8c000000000 size 4000000000 spacemap 0 free 256G ----
  
  metaslab 36 offset 90000000000 size 4000000000 spacemap 0 free 256G ----
  
  metaslab 37 offset 94000001000 size 3ffffff000 spacemap 0 free 256G ----
  
  metaslab 38 offset 98000001000 size 3ffffff000 spacemap 0 free 256G ----
  
  metaslab 39 offset 9c000002000 size 3fffffe000 spacemap 0 free 256G ----
  
  metaslab 40 offset a0000000000 size 4000000000 spacemap 0 free 256G ----
  
  metaslab 41 offset a4000000000 size 4000000000 spacemap 0 free 256G ----
  
  metaslab 42 offset a8000001000 size 3ffffff000 spacemap 0 free 256G ----
  
  metaslab 43 offset ac000001000 size 3ffffff000 spacemap 0 free 256G ----
  
  metaslab 44 offset b0000002000 size 3fffffe000 spacemap 0 free 256G ----
  
  metaslab 45 offset b4000000000 size 4000000000 spacemap 0 free 256G ----
  
  metaslab 46 offset b8000000000 size 4000000000 spacemap 0 free 256G ----
  
  metaslab 47 offset bc000001000 size 3ffffff000 spacemap 0 free 256G ----
  
  metaslab 48 offset c0000001000 size 3ffffff000 spacemap 0 free 256G ----
  
  metaslab 49 offset c4000002000 size 3fffffe000 spacemap 0 free 256G ----
  
  metaslab 50 offset c8000000000 size 4000000000 spacemap 0 free 256G ----
  
  metaslab 51 offset cc000000000 size 4000000000 spacemap 0 free 256G ----
  
  metaslab 52 offset d0000001000 size 3ffffff000 spacemap 0 free 256G ----
  
  metaslab 53 offset d4000001000 size 3ffffff000 spacemap 0 free 256G ----
  
  metaslab 54 offset d8000002000 size 3fffffe000 spacemap 0 free 256G ----
  
  metaslab 55 offset dc000000000 size 4000000000 spacemap 0 free 256G ----
  
  metaslab 56 offset e0000000000 size 4000000000 spacemap 0 free 256G ----
  
  metaslab 57 offset e4000001000 size 3ffffff000 spacemap 0 free 256G ----
  
  metaslab 58 offset e8000008000 size 3fffff8000 spacemap 123 free 7.70G normal
  
  metaslab 59 offset ec000004000 size 3fffffc000 spacemap 125 free 1.43G normal
  
  metaslab 60 offset f0000000000 size 4000000000 spacemap 124 free 1.58G normal
  
  metaslab 61 offset f4000006000 size 3fffffa000 spacemap 126 free 1.27G normal
  
  metaslab 62 offset f8000002000 size 3fffffe000 spacemap 127 free 1.66G normal
  
  metaslab 63 offset fc000008000 size 3fffff8000 spacemap 128 free 2.05G normal
  
  metaslab 64 offset 100000004000 size 3fffffc000 spacemap 129 free 2.23G normal
  
  metaslab 65 offset 104000000000 size 4000000000 spacemap 130 free 2.01G normal
  
  metaslab 66 offset 108000006000 size 3fffffa000 spacemap 131 free 1.59G normal
  
  metaslab 67 offset 10c000002000 size 3fffffe000 spacemap 132 free 1.08G normal
  
  metaslab 68 offset 110000008000 size 3fffff8000 spacemap 133 free 1.29G normal
  
  metaslab 69 offset 114000004000 size 3fffffc000 spacemap 134 free 1.59G normal
  
  metaslab 70 offset 118000000000 size 4000000000 spacemap 135 free 1.52G normal
  
  metaslab 71 offset 11c000006000 size 3fffffa000 spacemap 136 free 949M normal
  
  metaslab 72 offset 120000002000 size 3fffffe000 spacemap 137 free 1.59G normal
  
  metaslab 73 offset 124000008000 size 3fffff8000 spacemap 138 free 14.5G normal
  
  metaslab 74 offset 128000004000 size 3fffffc000 spacemap 139 free 1.85G normal
  
  metaslab 75 offset 12c000000000 size 4000000000 spacemap 141 free 24.1G normal
  
  metaslab 76 offset 130000006000 size 3fffffa000 spacemap 140 free 1.07G normal
  
  metaslab 77 offset 134000002000 size 3fffffe000 spacemap 142 free 19.9G normal
  
  metaslab 78 offset 138000008000 size 3fffff8000 spacemap 144 free 14.9G normal
  
  metaslab 79 offset 13c000004000 size 3fffffc000 spacemap 143 free 1.49G normal
  
  metaslab 80 offset 140000000000 size 4000000000 spacemap 145 free 22.2G normal
  
  metaslab 81 offset 144000006000 size 3fffffa000 spacemap 146 free 1.11G normal
  
  metaslab 82 offset 148000002000 size 3fffffe000 spacemap 147 free 1.34G normal
  
  metaslab 83 offset 14c000008000 size 3fffff8000 spacemap 148 free 36.2G normal
  
  metaslab 84 offset 150000004000 size 3fffffc000 spacemap 150 free 23.0G normal
  
  metaslab 85 offset 154000000000 size 4000000000 spacemap 149 free 946M normal
  
  metaslab 86 offset 158000006000 size 3fffffa000 spacemap 151 free 38.8G normal
  
  metaslab 87 offset 15c000002000 size 3fffffe000 spacemap 152 free 1.04G normal
  
  metaslab 88 offset 160000008000 size 3fffff8000 spacemap 153 free 25.3G normal
  
  metaslab 89 offset 164000004000 size 3fffffc000 spacemap 154 free 1.66G normal
  
  metaslab 90 offset 168000000000 size 4000000000 spacemap 155 free 34.4G normal
  
  metaslab 91 offset 16c000006000 size 3fffffa000 spacemap 156 free 1.51G normal
  
  metaslab 92 offset 170000002000 size 3fffffe000 spacemap 157 free 27.5G normal
  
  metaslab 93 offset 174000008000 size 3fffff8000 spacemap 159 free 23.8G normal
  
  metaslab 94 offset 178000004000 size 3fffffc000 spacemap 158 free 1.08G normal
  
  metaslab 95 offset 17c000000000 size 4000000000 spacemap 160 free 1.54G normal
  
  metaslab 96 offset 180000006000 size 3fffffa000 spacemap 161 free 1.46G normal
  
  metaslab 97 offset 184000002000 size 3fffffe000 spacemap 162 free 24.6G normal
  
  metaslab 98 offset 188000008000 size 3fffff8000 spacemap 163 free 1.07G normal
  
  metaslab 99 offset 18c000004000 size 3fffffc000 spacemap 165 free 43.3G normal
  
  metaslab 100 offset 190000000000 size 4000000000 spacemap 164 free 633M normal
  
  metaslab 101 offset 194000006000 size 3fffffa000 spacemap 166 free 1.27G normal
  
  metaslab 102 offset 198000002000 size 3fffffe000 spacemap 167 free 20.8G normal
  
  metaslab 103 offset 19c000008000 size 3fffff8000 spacemap 169 free 24.3G normal
  
  metaslab 104 offset 1a0000004000 size 3fffffc000 spacemap 168 free 11.4G normal
  
  metaslab 105 offset 1a4000000000 size 4000000000 spacemap 170 free 35.3G normal
  
  metaslab 106 offset 1a8000006000 size 3fffffa000 spacemap 171 free 1.45G normal
  
  metaslab 107 offset 1ac000002000 size 3fffffe000 spacemap 172 free 24.3G normal
  
  metaslab 108 offset 1b0000008000 size 3fffff8000 spacemap 174 free 36.1G normal
  
  metaslab 109 offset 1b4000004000 size 3fffffc000 spacemap 173 free 1.54G normal
  
  metaslab 110 offset 1b8000000000 size 4000000000 spacemap 175 free 28.0G normal
  
  metaslab 111 offset 1bc000006000 size 3fffffa000 spacemap 176 free 1.43G normal
  
  metaslab 112 offset 1c0000002000 size 3fffffe000 spacemap 177 free 35.6G normal
  
  metaslab 113 offset 1c4000008000 size 3fffff8000 spacemap 178 free 1.09G normal
  
  metaslab 114 offset 1c8000004000 size 3fffffc000 spacemap 179 free 1.43G normal
  
  metaslab 115 offset 1cc000000000 size 4000000000 spacemap 181 free 28.4G normal
  
  metaslab 116 offset 1d0000006000 size 3fffffa000 spacemap 180 free 1.21G normal
  
  metaslab 117 offset 1d4000002000 size 3fffffe000 spacemap 182 free 22.7G normal
  
  metaslab 118 offset 1d8000008000 size 3fffff8000 spacemap 184 free 45.0G normal
  
  metaslab 119 offset 1dc000004000 size 3fffffc000 spacemap 183 free 1.63G normal
  
  metaslab 120 offset 1e0000000000 size 4000000000 spacemap 185 free 19.0G normal
  
  metaslab 121 offset 1e4000006000 size 3fffffa000 spacemap 186 free 22.6G normal
  
  metaslab 122 offset 1e8000002000 size 3fffffe000 spacemap 188 free 29.7G normal
  
  metaslab 123 offset 1ec000008000 size 3fffff8000 spacemap 187 free 1.62G normal
  
  metaslab 124 offset 1f0000004000 size 3fffffc000 spacemap 189 free 26.4G normal
  
  metaslab 125 offset 1f4000000000 size 4000000000 spacemap 190 free 1.45G normal
  
  metaslab 126 offset 1f8000006000 size 3fffffa000 spacemap 191 free 25.4G normal
  
  metaslab 127 offset 1fc000002000 size 3fffffe000 spacemap 193 free 32.9G normal
  
  metaslab 128 offset 200000008000 size 3fffff8000 spacemap 192 free 1.02G normal
  
  metaslab 129 offset 204000004000 size 3fffffc000 spacemap 194 free 21.2G normal
  
  metaslab 130 offset 208000000000 size 4000000000 spacemap 195 free 17.8G normal
  
  metaslab 131 offset 20c000006000 size 3fffffa000 spacemap 196 free 22.8G normal
  
  metaslab 132 offset 210000002000 size 3fffffe000 spacemap 197 free 1.47G normal
  
  metaslab 133 offset 214000008000 size 3fffff8000 spacemap 198 free 1.22G normal
  
  metaslab 134 offset 218000004000 size 3fffffc000 spacemap 199 free 31.4G normal
  
  metaslab 135 offset 21c000000000 size 4000000000 spacemap 200 free 1.20G normal
  
  metaslab 136 offset 220000006000 size 3fffffa000 spacemap 201 free 24.0G normal
  
  metaslab 137 offset 224000002000 size 3fffffe000 spacemap 202 free 1.38G normal
  
  metaslab 138 offset 228000008000 size 3fffff8000 spacemap 203 free 22.8G normal
  
  metaslab 139 offset 22c000004000 size 3fffffc000 spacemap 204 free 1.40G normal
  
  metaslab 140 offset 230000000000 size 4000000000 spacemap 206 free 29.0G normal
  
  metaslab 141 offset 234000006000 size 3fffffa000 spacemap 205 free 1.60G normal
  
  metaslab 142 offset 238000002000 size 3fffffe000 spacemap 207 free 22.2G normal
  
  metaslab 143 offset 23c000008000 size 3fffff8000 spacemap 209 free 20.7G normal
  
  metaslab 144 offset 240000004000 size 3fffffc000 spacemap 208 free 1.22G normal
  
  metaslab 145 offset 244000000000 size 4000000000 spacemap 210 free 30.2G normal
  
  metaslab 146 offset 248000006000 size 3fffffa000 spacemap 211 free 1.32G normal
  
  metaslab 147 offset 24c000002000 size 3fffffe000 spacemap 212 free 20.4G normal
  
  metaslab 148 offset 250000008000 size 3fffff8000 spacemap 213 free 1.32G normal
  
  metaslab 149 offset 254000004000 size 3fffffc000 spacemap 214 free 15.6G normal
  
  metaslab 150 offset 258000000000 size 4000000000 spacemap 215 free 1.01G normal
  
  metaslab 151 offset 25c000006000 size 3fffffa000 spacemap 216 free 34.1G normal
  
  metaslab 152 offset 260000002000 size 3fffffe000 spacemap 218 free 18.7G normal
  
  metaslab 153 offset 264000008000 size 3fffff8000 spacemap 217 free 1.21G normal
  
  metaslab 154 offset 268000004000 size 3fffffc000 spacemap 219 free 36.5G normal
  
  metaslab 155 offset 26c000000000 size 4000000000 spacemap 220 free 1.44G normal
  
  metaslab 156 offset 270000006000 size 3fffffa000 spacemap 221 free 33.3G normal
  
  metaslab 157 offset 274000002000 size 3fffffe000 spacemap 222 free 1.17G normal
  
  metaslab 158 offset 278000008000 size 3fffff8000 spacemap 223 free 1.58G normal
  
  metaslab 159 offset 27c000004000 size 3fffffc000 spacemap 224 free 1.33G normal
  
  metaslab 160 offset 280000000000 size 4000000000 spacemap 225 free 21.4G normal
  
  metaslab 161 offset 284000006000 size 3fffffa000 spacemap 227 free 24.5G normal
  
  metaslab 162 offset 288000002000 size 3fffffe000 spacemap 226 free 1.50G normal
  
  metaslab 163 offset 28c000008000 size 3fffff8000 spacemap 228 free 19.1G normal
  
  metaslab 164 offset 290000004000 size 3fffffc000 spacemap 229 free 1.33G normal
  
  metaslab 165 offset 294000000000 size 4000000000 spacemap 230 free 32.4G normal
  
  metaslab 166 offset 298000006000 size 3fffffa000 spacemap 231 free 907M normal
  
  metaslab 167 offset 29c000002000 size 3fffffe000 spacemap 232 free 17.1G normal
  
  metaslab 168 offset 2a0000008000 size 3fffff8000 spacemap 234 free 29.6G normal
  
  metaslab 169 offset 2a4000004000 size 3fffffc000 spacemap 233 free 1.46G normal
  
  metaslab 170 offset 2a8000000000 size 4000000000 spacemap 235 free 16.2G normal
  
  metaslab 171 offset 2ac000006000 size 3fffffa000 spacemap 236 free 1.36G normal
  
  metaslab 172 offset 2b0000002000 size 3fffffe000 spacemap 237 free 1.07G normal
  
  metaslab 173 offset 2b4000008000 size 3fffff8000 spacemap 238 free 895M normal
  
  metaslab 174 offset 2b8000004000 size 3fffffc000 spacemap 239 free 15.6G normal
  
  metaslab 175 offset 2bc000000000 size 4000000000 spacemap 241 free 17.4G normal
  
  metaslab 176 offset 2c0000006000 size 3fffffa000 spacemap 240 free 1.03G normal
  
  metaslab 177 offset 2c4000002000 size 3fffffe000 spacemap 242 free 17.7G normal
  
  metaslab 178 offset 2c8000008000 size 3fffff8000 spacemap 243 free 1.29G normal
  
  metaslab 179 offset 2cc000004000 size 3fffffc000 spacemap 244 free 22.8G normal
  
  metaslab 180 offset 2d0000000000 size 4000000000 spacemap 246 free 18.4G normal
  
  metaslab 181 offset 2d4000006000 size 3fffffa000 spacemap 245 free 950M normal
  
  metaslab 182 offset 2d8000002000 size 3fffffe000 spacemap 247 free 932M normal
  
  metaslab 183 offset 2dc000008000 size 3fffff8000 spacemap 248 free 987M normal
  
  metaslab 184 offset 2e0000004000 size 3fffffc000 spacemap 249 free 14.2G normal
  
  metaslab 185 offset 2e4000000000 size 4000000000 spacemap 250 free 1.09G normal
  
  metaslab 186 offset 2e8000006000 size 3fffffa000 spacemap 251 free 582M normal
  
  metaslab 187 offset 2ec000002000 size 3fffffe000 spacemap 252 free 16.0G normal
  
  metaslab 188 offset 2f0000008000 size 3fffff8000 spacemap 254 free 13.0G normal
  
  metaslab 189 offset 2f4000004000 size 3fffffc000 spacemap 253 free 1.40G normal
  
  metaslab 190 offset 2f8000000000 size 4000000000 spacemap 255 free 15.6G normal
  
  metaslab 191 offset 2fc000006000 size 3fffffa000 spacemap 256 free 10.9G normal
  
  metaslab 192 offset 300000002000 size 3fffffe000 spacemap 257 free 32.9G normal
  
  metaslab 193 offset 304000008000 size 3fffff8000 spacemap 258 free 1.12G normal
  
  metaslab 194 offset 308000004000 size 3fffffc000 spacemap 259 free 16.6G normal
  
  metaslab 195 offset 30c000000000 size 4000000000 spacemap 260 free 1.34G normal
  
  metaslab 196 offset 310000006000 size 3fffffa000 spacemap 261 free 28.0G normal
  
  metaslab 197 offset 314000002000 size 3fffffe000 spacemap 262 free 892M normal
  
  metaslab 198 offset 318000008000 size 3fffff8000 spacemap 263 free 17.8G normal
  
  metaslab 199 offset 31c000004000 size 3fffffc000 spacemap 265 free 16.7G normal
  
  metaslab 200 offset 320000000000 size 4000000000 spacemap 264 free 1.25G normal
  
  metaslab 201 offset 324000006000 size 3fffffa000 spacemap 266 free 30.3G normal
  
  metaslab 202 offset 328000002000 size 3fffffe000 spacemap 267 free 1.26G normal
  
  metaslab 203 offset 32c000008000 size 3fffff8000 spacemap 268 free 29.4G normal
  
  metaslab 204 offset 330000004000 size 3fffffc000 spacemap 269 free 1.26G normal
  
  metaslab 205 offset 334000000000 size 4000000000 spacemap 270 free 757M normal
  
  metaslab 206 offset 338000006000 size 3fffffa000 spacemap 271 free 1.59G normal
  
  metaslab 207 offset 33c000002000 size 3fffffe000 spacemap 272 free 571M normal
  
  metaslab 208 offset 340000008000 size 3fffff8000 spacemap 273 free 1.37G normal
  
  metaslab 209 offset 344000004000 size 3fffffc000 spacemap 274 free 27.6G normal
  
  metaslab 210 offset 348000000000 size 4000000000 spacemap 275 free 717M normal
  
  metaslab 211 offset 34c000006000 size 3fffffa000 spacemap 276 free 24.7G normal
  
  metaslab 212 offset 350000002000 size 3fffffe000 spacemap 278 free 33.1G normal
  
  metaslab 213 offset 354000008000 size 3fffff8000 spacemap 277 free 1.32G normal
  
  metaslab 214 offset 358000004000 size 3fffffc000 spacemap 279 free 25.6G normal
  
  metaslab 215 offset 35c000000000 size 4000000000 spacemap 280 free 1.29G normal
  
  metaslab 216 offset 360000006000 size 3fffffa000 spacemap 281 free 1.38G normal
  
  metaslab 217 offset 364000002000 size 3fffffe000 spacemap 282 free 1.23G normal
  
  metaslab 218 offset 368000008000 size 3fffff8000 spacemap 283 free 22.7G normal
  
  metaslab 219 offset 36c000004000 size 3fffffc000 spacemap 284 free 1.33G normal
  
  metaslab 220 offset 370000000000 size 4000000000 spacemap 285 free 670M normal
  
  metaslab 221 offset 374000006000 size 3fffffa000 spacemap 287 free 37.0G normal
  
  metaslab 222 offset 378000002000 size 3fffffe000 spacemap 286 free 22.8G normal
  
  metaslab 223 offset 37c000008000 size 3fffff8000 spacemap 288 free 15.6G normal
  
  metaslab 224 offset 380000004000 size 3fffffc000 spacemap 289 free 22.5G normal
  
  metaslab 225 offset 384000000000 size 4000000000 spacemap 290 free 22.9G normal
  
  metaslab 226 offset 388000006000 size 3fffffa000 spacemap 291 free 15.2G normal
  
  metaslab 227 offset 38c000002000 size 3fffffe000 spacemap 292 free 22.3G normal
  
  metaslab 228 offset 390000008000 size 3fffff8000 spacemap 293 free 1.07G normal
  
  metaslab 229 offset 394000004000 size 3fffffc000 spacemap 294 free 948M normal
  
  metaslab 230 offset 398000000000 size 4000000000 spacemap 295 free 901M normal
  
  metaslab 231 offset 39c000006000 size 3fffffa000 spacemap 296 free 815M normal
  
  metaslab 232 offset 3a0000002000 size 3fffffe000 spacemap 297 free 1.33G normal
  
  metaslab 233 offset 3a4000008000 size 3fffff8000 spacemap 298 free 1.15G normal
  
  metaslab 234 offset 3a8000004000 size 3fffffc000 spacemap 299 free 1.08G normal
  
  metaslab 235 offset 3ac000000000 size 4000000000 spacemap 300 free 442M normal
  
  metaslab 236 offset 3b0000006000 size 3fffffa000 spacemap 301 free 712M normal
  
  metaslab 237 offset 3b4000002000 size 3fffffe000 spacemap 302 free 994M normal
  
  metaslab 238 offset 3b8000008000 size 3fffff8000 spacemap 303 free 1.17G normal
  
  metaslab 239 offset 3bc000004000 size 3fffffc000 spacemap 304 free 1.14G normal
  
  metaslab 240 offset 3c0000000000 size 4000000000 spacemap 305 free 285M normal
  
  metaslab 241 offset 3c4000006000 size 3fffffa000 spacemap 306 free 947M normal
  
  metaslab 242 offset 3c8000002000 size 3fffffe000 spacemap 307 free 1.34G normal
  
  metaslab 243 offset 3cc000008000 size 3fffff8000 spacemap 308 free 490M normal
  
  metaslab 244 offset 3d0000004000 size 3fffffc000 spacemap 309 free 23.0G normal
  
  metaslab 245 offset 3d4000000000 size 4000000000 spacemap 310 free 1.47G normal
  
  metaslab 246 offset 3d8000006000 size 3fffffa000 spacemap 311 free 21.5G normal
  
  metaslab 247 offset 3dc000002000 size 3fffffe000 spacemap 312 free 726M normal
  
  metaslab 248 offset 3e0000008000 size 3fffff8000 spacemap 313 free 1.24G normal
  
  metaslab 249 offset 3e4000004000 size 3fffffc000 spacemap 314 free 20.9G normal
  
  metaslab 250 offset 3e8000000000 size 4000000000 spacemap 316 free 946M normal
  
  metaslab 251 offset 3ec000006000 size 3fffffa000 spacemap 315 free 26.0G normal
  
  metaslab 252 offset 3f0000002000 size 3fffffe000 spacemap 317 free 677M normal
  
  metaslab 253 offset 3f4000008000 size 3fffff8000 spacemap 318 free 1.37G normal
  
  metaslab 254 offset 3f8000004000 size 3fffffc000 spacemap 319 free 1.28G normal
  
  metaslab 255 offset 3fc000000000 size 4000000000 spacemap 320 free 14.0G normal
  
  metaslab 256 offset 400000006000 size 3fffffa000 spacemap 322 free 874M normal
  
  metaslab 257 offset 404000002000 size 3fffffe000 spacemap 321 free 22.3G normal
  
  metaslab 258 offset 408000008000 size 3fffff8000 spacemap 323 free 1.06G normal
  
  metaslab 259 offset 40c000004000 size 3fffffc000 spacemap 324 free 1.00G normal
  
  metaslab 260 offset 410000000000 size 4000000000 spacemap 326 free 23.0G normal
  
  metaslab 261 offset 414000006000 size 3fffffa000 spacemap 325 free 670M normal
  
  metaslab 262 offset 418000002000 size 3fffffe000 spacemap 327 free 506M normal
  
  metaslab 263 offset 41c000008000 size 3fffff8000 spacemap 328 free 1.25G normal
  
  metaslab 264 offset 420000004000 size 3fffffc000 spacemap 329 free 948M normal
  
  metaslab 265 offset 424000000000 size 4000000000 spacemap 330 free 17.1G normal
  
  metaslab 266 offset 428000006000 size 3fffffa000 spacemap 331 free 887M normal
  
  metaslab 267 offset 42c000002000 size 3fffffe000 spacemap 332 free 846M normal
  
  metaslab 268 offset 430000008000 size 3fffff8000 spacemap 333 free 1.06G normal
  
  metaslab 269 offset 434000004000 size 3fffffc000 spacemap 334 free 1.28G normal
  
  metaslab 270 offset 438000000000 size 4000000000 spacemap 335 free 793M normal
  
  metaslab 271 offset 43c000006000 size 3fffffa000 spacemap 336 free 23.5G normal
  
  metaslab 272 offset 440000002000 size 3fffffe000 spacemap 337 free 20.3G normal
  
  metaslab 273 offset 444000008000 size 3fffff8000 spacemap 338 free 1.06G normal
  
  metaslab 274 offset 448000004000 size 3fffffc000 spacemap 339 free 840M normal
  
  metaslab 275 offset 44c000000000 size 4000000000 spacemap 340 free 20.4G normal
  
  metaslab 276 offset 450000006000 size 3fffffa000 spacemap 341 free 1000M normal
  
  metaslab 277 offset 454000002000 size 3fffffe000 spacemap 342 free 1.04G normal
  
  metaslab 278 offset 458000008000 size 3fffff8000 spacemap 343 free 17.7G normal
  
  metaslab 279 offset 45c000004000 size 3fffffc000 spacemap 344 free 1.23G normal
  
  metaslab 280 offset 460000000000 size 4000000000 spacemap 345 free 905M normal
  
  metaslab 281 offset 464000006000 size 3fffffa000 spacemap 346 free 22.1G normal
  
  metaslab 282 offset 468000002000 size 3fffffe000 spacemap 347 free 19.7G normal
  
  metaslab 283 offset 46c000008000 size 3fffff8000 spacemap 348 free 1.15G normal
  
  metaslab 284 offset 470000004000 size 3fffffc000 spacemap 349 free 12.7G normal
  
  metaslab 285 offset 474000000000 size 4000000000 spacemap 350 free 21.9G normal
  
  metaslab 286 offset 478000006000 size 3fffffa000 spacemap 351 free 19.0G normal
  
  metaslab 287 offset 47c000002000 size 3fffffe000 spacemap 352 free 1.16G normal
  
  metaslab 288 offset 480000008000 size 3fffff8000 spacemap 353 free 912M normal
  
  metaslab 289 offset 484000004000 size 3fffffc000 spacemap 354 free 902M normal
  
  metaslab 290 offset 488000000000 size 4000000000 spacemap 355 free 1.35G normal
  -------------------------------------------------------------------------------------------

draidcfg output for the 80 drive demonstration (80.nvl) {#draidcfg-output-for-the-80-drive-demonstration-80.nvl .Appendix2}
=======================================================

The following is the complete listing of the base permuation table
created for the dRAID configuration shown in the demonstration of
arbitrary pool configuration (section 4.2.4.1). Each line represents the
random ordering for the permutation of the 80 drives in the array.

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  \# draidcfg -r 80.nvl
  
  dRAID3 vdev of 80 child drives: 7 x (8 data + 3 parity) and 3 distributed spare
  
  Using 64 base permutations
  
  23,54,38,76,61,14,34,48, 9,31,52,10, 3,41,46,70, 1, 6,59,47,28,32,29,49,30,22,27,11,44,20,56, 5,74, 8,50,15,62,66,33,67,16,65,36,71,75,18,68,21,69,26,64,60,55,42,43,63,35,37,24, 7,17,45, 0, 2,58,78,57,13,12,72,73, 4,19,25,51,79,39,53,77,40,
  
  41,54,75,48, 2,57,36, 8,76,44, 5, 3,22,30,61,69,47,28,13, 0, 6,71,34,55,33,46,70,79,66,45,27,74,18,25,60,72,11,50,68, 1,53,32,19,64,40,51, 4,31,17,62,42,39,26,56, 7,16,24,12,38,15,78,35,37,67, 9,23,20,49,10,43,14,59,77,29,63,73,58,52,21,65,
  
  14,65,43, 9,16,53,46,69,17,40,20, 3,47,70,28,39,54, 5,12,24,78, 2,49,61,11,51,75,79,41,50,73,34,18,21,25,52,44,22,32,77, 8,59,15, 7,74,66, 0,71,45,56, 4,36,58,23,68, 6,67,42,29,64,26,33,72,10,37,13, 1,76,60,38,48,31,63,27,35,62,55,19,30,57,
  
  2,56,48,51,68,15,75,41,58,35,50,14,36,16,63,77,30,69,11,10,26, 7,62,19,24,44,28,37,31,43,64,25,49,32,54,53, 9,76,39,57,33,74, 8,34,27,23, 3,40,72,59,67,55,65,47,66, 1,71,61,46,18,17,29,79,38,12,70,22,45,78,60, 5, 6,21,73,13, 0,42,20, 4,52,
  
  64,76,20, 7,34,21,63,13, 0,47,51,41,59,57,74, 6,25,71,54,33,35,46,19,15,43, 8,23,18,24,61,10,39,72,27,26, 9,62,17,53,78, 2,58,29,60,77,44,36,66,70,22,67,75,65,69,30, 4,40,14,42,45,38,49,32,11,31,16,28,79, 3,68,56,73, 1,48, 5,52,12,55,50,37,
  
  62,33,67,58,38,57,61,24, 3,47, 0,37,53,72,40,39,35,10,20,60,43,41,69,55,23,21,59,25,13,28, 9,12,51,19,52,27,63,45, 2,31,46,15, 5, 7,14,68, 1,76,78,50,29,26, 6,42,22,56,11,64,16, 4,49,79,73,74,54,36, 8,77,65,17,75,66,44,71,70,32,34,18,48,30,
  
  10,57,30,46,42,55,34,16,52,49,44,36,53,18,79,21,38,77,60,39,45, 8,19,24,68,17,73,63,66,70,65, 7, 4,37,61, 6,64,12, 9,26,28,14,78,31,41,27,11,33,51, 3,29,35,74,32,23,58,76,13, 0,22,15,69,47, 1,56, 5,72,67,54,50,43,48,59,25, 2,75,62,40,71,20,
  
  59,53,27,26,72,23,33,56,66,10,73,52,51, 8,24,18,11,68, 6,77,45,19,37, 2, 4,76,47,17,34,62,49, 0,50,28,74,22,21,15,78, 7,25,20,40,32,35,38,31,71,57,65,12,16,13,48,43, 1,54,58,36, 9,63,64,79, 5,42,44,75,14,39,55,60,29,30,46,61,70,41,69,67, 3,
  
  40,14,16,31, 5,63,69,53,43,37,73,50,77,20,29,61,41,48,45,67,15,55,47,79,60,25,76,54,12,57,46,56,35, 1, 7,65,22,11,34,26,13, 70, 27,72, 2,74,19, 8,28,23,66,62,71,33,64,18,17, 3, 0,49, 6,36,75,59,78,30,32,58,52,24, 4,21,10,68,39,51, 9,38,44,42,
  
  53,55,59,77,64,39,40,62,76,16,74, 5,26,23,66,47,21, 0, 6,60,69,27,11,58,72,34,73,45,38, 3,43,49,15, 9,19, 1,79,35,67,31,44,75, 46,68,50,71, 2, 8,48,65,54,14,78,63,41,13,10,29,12,32,20,57,30,25,24,51,36,18,56,33, 4,70,37,61,17,28,52,42, 7,22,
  
  47,54,51, 3,49,45,32,71,68,26,31,65,30, 8,74, 9,76,78,46,25,38,53,60,19,11,50, 6,33,58,37,39, 7,20, 2, 0, 5,75,28, 63,48,44, 40,34,41,17,15,67,16,66,13,22,72,73,21,35,36,77,12,70, 1, 4,10,69,23,52,43,79,56,59,27,62,57,55, 14,64,24,18,42,29,61,
  
  33,65,36,19,47,41,53, 9,26,17,66, 1, 5,77,69,59, 6,27, 7,14,71,68,12,25,28,10,13,18,61,51,22,54,34,31,48,57,32,67,49,62, 8,23,40,58,39,79,50,74,45,44,73,64,30, 0, 2,56,75,21,29,43,63,76,72,60,46, 4,42,20,70,16, 3,11,52,55,38,78,37,24,15,35,
  
  75,79,64,78,22,61,28, 2,50,51,21, 5,46,72,16,15,70, 3,49,32,36,26,60,27, 0,24,52,56, 7,43,12,73,42,30,45,17,48,10,33,74,37,35,44, 71,29,68,53,23,13,59,38,34,19,18, 8,62,65,66,39,76,20,55,77,58,25,40,41,11,31,47,57, 9,14, 4,67, 1,54,69,63, 6,
  
  12,76,64,36,37,28,44,57,49,13,39,73,58, 5,27,52,33,15,26,56,51,66,14,63,23,71,43,62,65,11,35,74,16,18, 0, 7, 1, 6,34,60,47,46,29,20,79, 3,10,53,42,78,68,31,21,50,54,70, 4,61,55,38,30,59,69, 8,32,77, 9,22,41,24,48,75,19,17,25,67, 2,40,45,72,
  
  19, 2,64, 8,37,58,25,70,33,23,28,68,52,69,34,30,27,50, 7,56,62,47,51,57,45,61,65,67,31,75,79,43,42,13,29,53,66,72,77,35,26, 5,48,15,38,10, 1,59,14,16,11,40, 3,20,76,63,49,74,41,39, 0,55,46,54,78,12, 4,22,71, 9,21,36,44,60,18,73,17,32,24, 6,
  
  73,55,35,71,46,40,26,32,29,23,13,15,30, 0,72,68,22,64,25, 1,56,76,43,36,44, 8,27,39,18,63,41, 9,19,11,58,28,50,24, 2,77, 7,53,34,66,49,65,78,67,59,69,57,48,70,79,33, 5,61,60,51,62, 3,14,75,16,17, 4,52,74,10,20,45,12,47,37,54,31, 6,38,21,42,
  
  0,43,38,78,21,73,18,16, 3,79,70,24,47,74,12,67,63,30,41, 6,34,27,76,72,25,65, 9,37,55,39,31, 4,71, 8,10,58,44,11,35,36,23,46, 48,53,52,69,32,20,59,66,26,14,62,61,57,50,15,13,17,29,60,56,68,33, 5,75,54,51,28,42,45, 2,40,22,49, 7,19, 1,77,64,
  
  74,19,37,28,34,69,59,31,78,13,61,65,54,40,75,73,70,38,22, 6,63,10,27,48, 4,46,14,21,41, 3,15,20,76,26,77,62,60, 8,66,45,23,55,29,50,67,11,52,25, 5, 2,36, 0,30,51,33,49,43,44,53,39,32, 1,79,71,72,12,47,58, 9,24,42,16,57,35,56, 7,68,17,18,64,
  
  65,10, 5,51,26, 4,68,22, 7,23,55, 1,21,34,61,75,20,76, 9,74,62,48,54,73,35,13,58, 8,44,53,33,38,42,60,64,17,36,15,32,29,67,66, 41,52,63,31,47,79, 3, 0,24,49,78,72,18,12,14,46,37,11,77,39,56,30,59,69, 2,25, 6,16,71,28,50,45,57,19,27,40,43,70,
  
  14,79,16,51, 9,50,41,15, 2, 8,32,75,21,43,11,26,65,36,27,47,38,17,67, 3,71,45,60,42, 1,54,22, 4,20, 6,40,76,74,56,12,61,28, 0,25,78,55,23,18,44, 5,73,52,29,77,57,34,24,58,19, 7,46,37,69,35,62,66,13,53,59,70,64,39,63,48,33,10,31,68,49,72,30,
  
  30,52, 4,58,14,78,62, 5,76,29,36,28,63,64,74,56, 3,32,39,33,18,48,65,10,68,50,66, 0,16,57,26, 9,60,37,23,17,34,25,67,69, 8,79,77,53,15,73,44,71,12,59, 1,51, 2,11,35, 6,47,22,46,75,38,55,49, 7,61,54,19,41,13,40,27,31,42,70,72,20,21,43,45,24,
  
  20,21,44, 0,30,28,46, 6, 7,40,13,76,72,37,53, 8,61,57,18,35,12,78,31,17,29,79,70, 2,26,77,50,25,41,23,47, 1,34,16,69,68,10, 3,74,59,14,55,54,60,27,49, 9,39,73,65,42,67,15,33,56,58,11,64,22,24,43, 4,36,66,38,62,51,48, 5,32,63,52,45,75,71,19,
  
  11,31,79,26,14, 9,27,62, 1,39,29,54, 6, 5,41,28,22,65, 7,34,57,77,59,73,42,32,46,25,38,63,74,47,17,60,72,67,33,64,53,40, 66,12,48,15,71,56,23, 3,76,44,30, 4,16,49,37,24,55,52,58,61,51,70,35,43,18,69,13,75, 2, 8,36,45,19,50, 0,10,68,78,20,21,
  
  66,31,41,34,44,77,79,75,33,18,22,16,27,19,40,17,57, 8, 0,26,50,28,15,37,29,49,24, 6, 9,13,53,60,73,71,25,52,78,10,58,65,11, 3,62, 1, 4,38,70,43, 5,12,20,61,63,47,23,55, 7,74,35,76,48,68,67,59,30,42,72,46,39,54,69,51,36,56,64,45,32,21,14, 2,
  
  15,41, 7,18,34,77,36, 0,45,66,22,59,56,42,48, 8,31,61,43,30,70,13,52,60,49,75,46,53,14,64,28,16, 3,58, 2,35,26,67,72,44,79, 29,51,74, 1, 5,55, 4,17,71,73,65,27,54,38,78,23,76,68,47,32,19,40,69,21,24,39,33,37,62,10,11, 9,20,63,25,50,12,57, 6,
  
  74,76,51, 8,19,78,32,46, 9,63,67,49,75,26,16, 6,66,25,30,53,56,37,77,22, 1,17,45,65,59,12,68,31,55,27,23,34,44,14,11,48,54,18,36, 64, 0,70, 3,21,39, 4,62,79,20,40,57,15,35,60,38,61,10,69,41,43,24,50,73,47,33, 7,71,13,72,42,58,28, 2,52, 5,29,
  
  4,39,24,66,41,72,29,25, 2,55, 6,43,44,52,45,75,19, 7,22,11,20,16,63,49,59,60,26,35,54,70,64, 3,56,42,58,71,48,57,51,77,18,76,61, 9,62,36,53,38,79, 1,50,74,47,15,14,34, 8,78,40,10,73,13,12,30,28, 5,33,17,27,37, 0,65,67,23,46,31,21,69,32,68,
  
  27,46,61,37,41, 9,74, 7,79,73,67, 3,25,45,33,10,59,49,52,77,78,43,53, 4, 0,17,35,21,14,60,44,54,32,24,63,42,72,39,62,36, 40,64,66,34,75, 6,30,69,38, 5,51,47,28,22,18,57,31,71,15,23,68,20,12, 1,56,76, 8,26,55, 2,70,29,13,65,48,58,11,16,50,19,
  
  18,13, 9,72,41,35,20,11,10,70,43,37,67,73,56,33,52,46,29,30,45,61, 1, 4, 7,16,55,23,40,66,36,48,71,63,34,28,64,12,79, 8,14,75,50,26,60,74,44,69,59,15, 3, 2,21,65,27,22,76,77,53,51,32,31,19,62, 0,58,25, 6,42,49,17,54, 5,24,78,68,57,38,39,47,
  
  33,63,57,10,18,21, 7,41,34,71,51,68,70,52, 0, 4,13,62,69,30,15,14,67,35,29, 1,78,54, 5,24,64, 3,60,65,20,48,50,45,31,17,79,39,38, 28,11,36,49,12,72,66,77,44, 6,40,59,61,53,22,37,16,43,19, 9,55,46,74,73,26, 8,23,75, 2,32,27,58,47,56,25,42,76,
  
  72,69,52, 4,59,22,14, 3,70,26,61,36,63,29,53,79,49,15,62,33, 9,66, 0,16,44,45,58,41,43,24,71,27,23,67,21,25,32,46,39,55, 64,50,35,57,60,13,19,11, 1,12,37, 5,56,10,20,75,30,68, 6,28,47,65,34,40, 8,76,78,48,77, 7, 2,74,51,73,38,17,42,31,54,18,
  
  15,20,40, 9,61,45,32,63,28,64,37,34, 4, 3,65,27,25,66,30, 5,24,29,70,26,59,22,77,54,78,11,19,60,33, 8,55,10,31,46,13,16,69, 51,73,44,39,53, 1,17,72,48,57,74,56,43,36, 2,76,68,38,14,79,52, 0,21,12,49,18,50,41, 6,42, 7,62,71,75,47,58,35,67,23,
  
  43,40, 9,64,62,17, 3,72,22,52,20,63,29,37, 1,74,79,76,57,59,77, 2,33,25,58,12,75,47,31,26, 8,38,27,45,55, 6,19,67,69,48,61,23,32, 5,15,66,14,30,36, 7,24,34,71,42,73,49, 0,39,51,70,78, 4,41,56,28,50,21,18,35,10,68,60,44,53,46,11,16,13,65,54,
  
  57,34,72,75,79, 8,53,60,43,64,41,35,25,16,24,70, 5, 4,76,46,44,74,45,32, 1,49,39,37,19,21,65,68,78,27,47, 2,22,11, 0,20,13,12,18, 3,66,42,36,62,17,54,29,71,26,56, 6,40,38,50,61,73,77,59,67,63,23,10,15,14,28,33,69,55, 9,48,58,30,51,31,52, 7,
  
  48,30,19,59,52,26, 2,37,22,13,40,42, 4,72,63,28,71,56,21,73,79,15,50,20,64,58,70,47,53,65,27, 7,25,76, 5,61,45,67, 9, 0,18,60,31,29,14, 6,17, 3,57,68,24,43,34,66,49,33,36,55,35,69,46,62,16,39,78, 1,32,75,51,11,12,10,77,23,54, 8,74,44,38,41,
  
  29, 3,54,68,16, 1,55,32,69,67,19,75,37,46,71,33,62,30,57,27,58,53,42,52,38,28,56,73,63,24,25,39, 8,79,18, 2,34,36,10,12, 15,11,50,49,51,22,14,70,66,23,31,44,61,74,17,76, 7,77,48,43, 5, 4,65,35,78, 9,13, 6,45,26,40,41,64,47,20,72,21,59, 0,60,
  
  19,44,49,56, 5,14,40,63,66, 0,11,51,25,13,78, 3,74,20, 6,70,64,65, 9,23,37,67,36,38, 8,59,60,18,21, 2,42,35,53,43,75,62, 39,33,12,79,73,68,32,30,16, 7,57,31,41,26,71,22,17,24,47,10,15,34,61,55,46,72,54,77,50, 1,29,27,76,45,52,69,28,58,48, 4,
  
  16,13, 0,55,40,42,15, 1,23,32,47,63,68,73,52,74,10,78,21, 8,43,66,39,11,29,37,45,53,27,36,41,49,54,12,69,76, 2,64,19,60, 3,44,34,79,22, 6,20,58,14,56,71, 4,72,70,25,48,57,67,46,65,31, 5,75,33,38,18,30,35, 9,59,62,17, 7,28,61,50,77,51,24,26,
  
  39,18,56,32,11,54,73,72,21,24,41,14,76, 7, 9,70,71,12,58,38,57,50,67,51,28,26,79,77,33,53,35,63, 8,37,68,64,75,20,62,16,55, 2,10, 4,42,69,22,13,43,48,52,46, 0,23,60, 6,15,40,17,36,29, 3,31,19,25,27,44,49,59, 5,65,66,45,30, 1,47,34,74,61,78,
  
  7,23,74,78,56,53, 0,54,20, 9,50,73, 6,35,45,33,42,52,18, 1,65,72,55,79,77,58,17,34,22,30,36,70,59,26,57,21,71,44,43,28,75,10,13, 8,63,64, 4, 5,27,39,31,62,25,60,48, 3,12,11,14,66,16,37,40,38, 2,46,67,24,51,49,15,69,68,76,41,47,19,61,32,29,
  
  44,42,17,12,66,11, 0,69,49,67,53,65,61, 9,72,10,18,68,25,58,22,75,51,16,64,19,79,55,27,15,77,23,54,47, 7,50,60,41,57,73,34,56, 2,21,62, 6,26, 5,13,40, 1, 4,29,38,33,31,78,39,46,37,24,63,20,74,59,36,45, 3,35,70,30,43,71,32,14, 8,28,48,76,52,
  
  48,77,61,65, 8,23,55,37,21,28,36,18,26,24, 1,43,57, 9, 4,53,78,67,79,63,50,72,29,58,59,20,60,46, 0,44,70,30,15, 5,49,73,54,40,71,64,10,69, 3,56,17,41,76,42,34,32,19,27,47,62, 7,25,51,68,31,74,13,14,38,35,75, 2,22,45, 6,39,11,16,33,66,12,52,
  
  38,77,71,48,20,52,25,36,27,79,22,16, 2,18,32,46,39,59,73, 1,75,78,57,44,42, 4,10, 9,58,60,61,17,15,50,51,31,35,54,63,62, 72,65,19,53,14,56,34,26, 6,67,24,70,66,43,29,23,76,49,37, 7,74,28,12,13, 8,40,47,30,11, 5,69,45, 3,68, 0,64,21,41,33,55,
  
  73,57,21,48,53,12,36,17,58,78,75, 7,50,64, 8, 0,29,77,22,55,54,61,38,59,70,24,68,13, 6,11,35,41,44,45,52,76,23,60,39, 9,67,18,43,66, 2,10,72,28,47,15,62,42,56,51,33,65,74, 5,69,40,25,20, 1,34,16,27, 3,37,19,14,26,32,79, 4,30,46,49,31,71,63,
  
  58,29,10,41,51,59, 0,13, 5,63, 4,37, 8, 3,61,54,79,47,67,23,48,77,52,71, 9, 6,11,68,25,60,15,62,65,32,21,70,73,55,46,22,56,45, 40,38,64,33,75,18,57,69,53,76,16,42,35,19, 7,34,74,78,44,72,14, 2,17,24,12,27,26,50,66,36,20,43,49,31, 1,30,39,28,
  
  73,36,69,25,66,45,11,29,27,42,23,10,22, 5, 3, 6,38,50,75, 7,55,43,79,16,47,63,48,68,72,58, 9,67,60,40,37,35,14,15, 4,46,52,65, 0,53,18,32,19,64,17,44,77, 8,57,74,21,70,20,28,62,54,39,12,61,13,26,30,49,41,59, 1,78,76,34,71,24,33,56,31, 2,51,
  
  22,52,71,57,31, 1,36,50,28,63,30,21, 3,13,16,10,58, 5,35,23,29,60,20,73,24,79,75, 8,51,66,26,62,43,45,78,27,49,25,41, 0,11,38,67,14, 2,61,55,46,53,64,42,47,59, 6,65,40,39, 9, 4,54,70,69,68,19,72,17,34,15, 7,18,37,74,76,32,12,44,33,56,77,48,
  
  29,44,76, 9,30,25,60,35,45, 5,22,15,40,66,34,59,18,13,58, 3,33,65,14,36,75,72,39,20,69,55,38,79,26,17,50,48,54,78, 52,31,43,27,32,68, 2,67,42, 1,71,46, 4,41,53,37,74,12, 6,57,28,16,56,23,64,19, 7,10,47,21,63,70,62,11,24, 0,49,77, 8,61,51,73,
  
  25,38,27,56,65,49,42,28,10,76,37,34,74,57,59,67, 8,47,26,15,33,23,40,71,50, 2,46,21,19, 6, 4, 5, 9,43,18,63,45,73,58,53,14,64, 7, 0,48,20,54,68, 1,52,60,77,22,72,62,51,30,78,11,61,39,36,66,31,32,17,35,79, 3,24,69,13,55,16,44,75,70,41,29,12,
  
  45,71,66,11,54,56, 3,51,35, 0,53,19,21,55,10, 2,73,27,59, 1,31,60,20,62,68,22,38,74,61,50,58,23,24,49,48,28,12,13, 6,44,43,37,25,26,29,75,36,46,32,70,16,76,39,17,30,41,47,67,40,15,78,14, 9, 8,69,63,79,72,57,33, 4,42,34,52,65,77, 7,18, 5,64,
  
  16,55,63,42,48,27,38,28,57,72,62,19,79,33,15,77,51, 1,36, 7,78,52,29,25, 2,59,18,12,39,64,46, 3,24,43,60,34,61,40,54,14,44,74, 5,47,75, 9,49,23,68,58,65,26,56,22,37,20,66,21, 6,71, 0,17,73,50,41,10,35, 8, 4,70,30,67,69,76,53,32,13,45,31,11,
  
  49,60, 4,13,30,20,78, 3,37, 5,50,43,73,75, 6,40,23,29,11,53,27,31,34,74,64,41,36,17,44,48,77,21,16,18,58,79,57,10, 7,22,54,52,35,25,26,33,28,71,12,69,63,65,24,59,47,76,15,56,72, 8,46,39,42,45,55,38,66,14,67,62, 2,51,68, 1,70,32,19, 0, 9,61,
  
  19, 3,75,59,10, 8,14,11,12,39,67,41,28,74,76,73, 7,33,35,55,65,15,77,49,24,37,13,44,30,45,47, 4,70,36,50,69, 5,62,34,22, 0,61, 1,71,42,54,20,60,40,53,57,26,72,46,31,63,52,18,17,29,21, 9,58,66,51, 6,38,23,16,64,32,27,79, 2,68,48,78,56,43,25,
  
  27,71,73,66,12,47,44,63,33,11,61,72,46,69,31,48, 3,16,65,24,40,49,77, 1,58,70,14,52,57,21, 8,64,13,59,10,55,23, 0,17,53,41,54,68,78,67,38,39, 9,51,76,45, 7,62,60, 4,22,15,37,29,25,34,26,28,79, 5,20,74,18,56,32,42,43, 6,19,75,50,30,35,36, 2,
  
  27,20,10,54, 8,57,40, 0,22,12,47,36,75, 7,35,45,19,34,72,58,74,23,16,33,64,14,78,39,59,24,11,26, 6,28,32,43,73,38,67,25,70,71, 42,66, 3,46, 9,60,15, 2,51,21,79,53,30,65,41,68,13,56,76,77, 4, 5,37,44,49,52,63,17,29,31,50,61,69,62,55,48, 1,18,
  
  56,59,62,67,26,44,10,18,57,55,70,63,78,73, 0,40,34,29,30,42,11,68,72,64,12,35,50, 9,39,17,27,61,24, 2,31,47,41,53, 5,33,49,54, 1,45,15,25,77, 8,20,75,60,16,32,36,28,52,38,43,74,71,37,21,13,66,79,65, 7,19,48,23,22, 3,46,58,76,14,69,51, 4, 6,
  
  8,56,60,48,53, 2,40,66,29,10,61,59,11,67,35,51,14,36,63,76,54,44,69, 0, 3,15,32,79,73,58,72,23,12,71,57,50,19,52, 7,75,77,45,70,33, 1,55,68,22,42,49,38, 4,47,24,30,17,37,27,43,39,78,34,65,28,13,64, 9,25, 6,20,46,74,62,31, 5,41,16,21,26,18,
  
  40,73,77,48,59,37, 7,56,60,58,79,13, 6,21,30,23,47, 9,42,36,74,72,66, 3,38,25,45, 0,39,67,34,31,61,29,33,75,27, 1,68,15,41, 5,50,78,69,63,76,71,51,44,52,12,54,19,32, 8,11,64, 4,28,65,26,49, 2,55,46,43,62,35,17,70,22,20,57,14,24,10,16,18,53,
  
  34,16,44,30,25,78,29, 0,71,54,49,47,14,26, 2,20,22,61,53,36,59,27,38,42,43,52,74,39,24, 4,70,63,45,46,64,56, 1,66,35, 5,68,51,65,72,28,19, 7,48,18,13,33,21,60,50,40,76,37,55,32,79,77,69, 3,23,75,58, 6,41,62,10,67,15, 8,11,73,12,57, 9,17,31,
  
  35,34,68,54,11,13,41,33,52,16,63,72,60,46,48,69,28,40,12,73, 2, 0,29,53,37, 9, 5, 4,14,78,39,50,47,70,45,64, 7,25,79,18,21, 1,36,67,55,42,20, 6,76,26,59,51,19, 8,24,65,43,44,49,57,56,17,38,61,27,77,32,66,30,74,23,10,15,75,31,58,62,71,22, 3,
  
  35,16,41,21,28,79, 3,32,27, 2,51,48,31,44,18,39,13,74, 6,59,76,58, 8, 9,14,68,63,62,24, 0,15,22,49,23,47,65,33,78,70,56,66, 1,42,10,17,38,40,67,64,69,12,19,71,36,11,52,50,72,77, 7,53, 4,26,46,43,37,34,73,20,45,57,54,55,30, 5,25,60,75,61,29,
  
  1,30,52,57,60,34,67,10,72, 5,19,27,21,44, 4,74,61,47,64,39,40,28,45,26, 6,42,53, 8, 3,55,37,36,73,24,15, 0,56,13,77,14,16,25, 62,51,70,79,22,18,43,12,17, 9,29,65,59,68,63,31,48,32,46,33,58,35,38,41,11,66, 7,69,50,23,75,71, 2,54,49,76,78,20,
  
  18,34, 1,31,14,40,79,74,78,63,70,19,17,12,62,69, 9,48,24,77,33,76,20,21,55,64,66, 8,51,26,25, 2,73,60,49,75,43,29,54, 28,53,23,38,45,50,52,35, 5, 4,22,57,61,68,27,11, 3,32,72,56, 7,41,39,30,36,42,65,58,59,37,47, 6,15, 0,13,16,46,44,10,67,71,
  
  68,69,58,72,26,42,32,38,31,70,67,64,55,13,29,59,33,78,14,66,11,28,48,44,36,75,35,76, 5,54,77, 8,30, 0,37,62,73,21,63,25,34,12, 6,23,60,27,53,40,52,56,46, 7, 1,39, 3,41,24,10,20,19,18,57,71,15,51,16,47,43,17,74, 2,65, 9,45,61,22,79,50, 4,49,
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Zpool status for the 80 drive JBOD {#zpool-status-for-the-80-drive-jbod .Appendix2}
==================================

1.  The following is the complete ‘zpool status’ listing for the 80
    drive dRAID created in Section 4.2.4.1. The array has 3 distributed
    spare drives and 7 (8+3) parity groups.

  ------------------------------
  \# zpool status
  
  pool: MS09
  
  state: ONLINE
  
  scan: none requested
  
  config:
  
  NAME STATE READ WRITE CKSUM
  
  MS09 ONLINE 0 0 0
  
  draid3-0 ONLINE 0 0 0
  
  > sdb ONLINE 0 0 0
  >
  > sdd ONLINE 0 0 0
  >
  > sde ONLINE 0 0 0
  >
  > sdg ONLINE 0 0 0
  >
  > sdh ONLINE 0 0 0
  >
  > sdi ONLINE 0 0 0
  >
  > sdk ONLINE 0 0 0
  >
  > sdl ONLINE 0 0 0
  >
  > sdm ONLINE 0 0 0
  >
  > sdo ONLINE 0 0 0
  >
  > sdp ONLINE 0 0 0
  >
  > sdq ONLINE 0 0 0
  >
  > sds ONLINE 0 0 0
  >
  > sdt ONLINE 0 0 0
  >
  > sdu ONLINE 0 0 0
  >
  > sdw ONLINE 0 0 0
  >
  > sdx ONLINE 0 0 0
  >
  > sdy ONLINE 0 0 0
  >
  > sdz ONLINE 0 0 0
  >
  > sdab ONLINE 0 0 0
  >
  > sdac ONLINE 0 0 0
  >
  > sdad ONLINE 0 0 0
  >
  > sdae ONLINE 0 0 0
  >
  > sdc ONLINE 0 0 0
  >
  > sdf ONLINE 0 0 0
  >
  > sdj ONLINE 0 0 0
  >
  > sdn ONLINE 0 0 0
  >
  > sdr ONLINE 0 0 0
  >
  > sdv ONLINE 0 0 0
  >
  > sdaa ONLINE 0 0 0
  >
  > sdaf ONLINE 0 0 0
  >
  > sdag ONLINE 0 0 0
  >
  > sdah ONLINE 0 0 0
  >
  > sdai ONLINE 0 0 0
  >
  > sdaj ONLINE 0 0 0
  >
  > sdak ONLINE 0 0 0
  >
  > sdal ONLINE 0 0 0
  >
  > sdam ONLINE 0 0 0
  >
  > sdan ONLINE 0 0 0
  >
  > sdao ONLINE 0 0 0
  >
  > sdap ONLINE 0 0 0
  >
  > sdaq ONLINE 0 0 0
  >
  > sdar ONLINE 0 0 0
  >
  > sdas ONLINE 0 0 0
  >
  > sdat ONLINE 0 0 0
  >
  > sdau ONLINE 0 0 0
  >
  > sdav ONLINE 0 0 0
  >
  > sdaw ONLINE 0 0 0
  >
  > sdax ONLINE 0 0 0
  >
  > sday ONLINE 0 0 0
  >
  > sdaz ONLINE 0 0 0
  >
  > sdba ONLINE 0 0 0
  >
  > sdbb ONLINE 0 0 0
  >
  > sdbc ONLINE 0 0 0
  >
  > sdbd ONLINE 0 0 0
  >
  > sdbe ONLINE 0 0 0
  >
  > sdbf ONLINE 0 0 0
  >
  > sdbg ONLINE 0 0 0
  >
  > sdbh ONLINE 0 0 0
  >
  > sdbi ONLINE 0 0 0
  >
  > sdbj ONLINE 0 0 0
  >
  > sdbk ONLINE 0 0 0
  >
  > sdbl ONLINE 0 0 0
  >
  > sdbm ONLINE 0 0 0
  >
  > sdbn ONLINE 0 0 0
  >
  > sdbo ONLINE 0 0 0
  >
  > sdbp ONLINE 0 0 0
  >
  > sdbq ONLINE 0 0 0
  >
  > sdbr ONLINE 0 0 0
  >
  > sdo ONLINE 0 0 0
  >
  > sdp ONLINE 0 0 0
  >
  > sdq ONLINE 0 0 0
  >
  > sds ONLINE 0 0 0
  >
  > sdt ONLINE 0 0 0
  >
  > sdu ONLINE 0 0 0
  >
  > sdw ONLINE 0 0 0
  >
  > sdx ONLINE 0 0 0
  >
  > sdy ONLINE 0 0 0
  >
  > sdz ONLINE 0 0 0
  >
  > sdab ONLINE 0 0 0
  >
  > sdac ONLINE 0 0 0
  >
  > sdad ONLINE 0 0 0
  >
  > sdae ONLINE 0 0 0
  >
  > sdc ONLINE 0 0 0
  >
  > sdf ONLINE 0 0 0
  >
  > sdj ONLINE 0 0 0
  >
  > sdn ONLINE 0 0 0
  >
  > sdr ONLINE 0 0 0
  >
  > sdv ONLINE 0 0 0
  >
  > sdaa ONLINE 0 0 0
  >
  > sdaf ONLINE 0 0 0
  >
  > sdag ONLINE 0 0 0
  >
  > sdah ONLINE 0 0 0
  >
  > sdai ONLINE 0 0 0
  >
  > sdaj ONLINE 0 0 0
  >
  > sdak ONLINE 0 0 0
  >
  > sdal ONLINE 0 0 0
  >
  > sdam ONLINE 0 0 0
  >
  > sdan ONLINE 0 0 0
  >
  > sdao ONLINE 0 0 0
  >
  > sdap ONLINE 0 0 0
  >
  > sdaq ONLINE 0 0 0
  >
  > sdar ONLINE 0 0 0
  >
  > sdas ONLINE 0 0 0
  >
  > sdat ONLINE 0 0 0
  >
  > sdau ONLINE 0 0 0
  >
  > sdav ONLINE 0 0 0
  >
  > sdaw ONLINE 0 0 0
  >
  > sdax ONLINE 0 0 0
  >
  > sday ONLINE 0 0 0
  >
  > sdaz ONLINE 0 0 0
  >
  > sdba ONLINE 0 0 0
  >
  > sdbb ONLINE 0 0 0
  >
  > sdbc ONLINE 0 0 0
  >
  > sdbd ONLINE 0 0 0
  >
  > sdbe ONLINE 0 0 0
  >
  > sdbf ONLINE 0 0 0
  >
  > sdbg ONLINE 0 0 0
  >
  > sdbh ONLINE 0 0 0
  >
  > sdbi ONLINE 0 0 0
  >
  > sdbj ONLINE 0 0 0
  >
  > sdbk ONLINE 0 0 0
  >
  > sdbl ONLINE 0 0 0
  >
  > sdbm ONLINE 0 0 0
  >
  > sdbn ONLINE 0 0 0
  >
  > sdbo ONLINE 0 0 0
  >
  > sdbp ONLINE 0 0 0
  >
  > sdbq ONLINE 0 0 0
  >
  > sdbr ONLINE 0 0 0
  >
  > sdbs ONLINE 0 0 0
  >
  > sdbt ONLINE 0 0 0
  >
  > sdbu ONLINE 0 0 0
  >
  > sdbv ONLINE 0 0 0
  >
  > sdbw ONLINE 0 0 0
  >
  > sdbx ONLINE 0 0 0
  >
  > sdby ONLINE 0 0 0
  >
  > sdbz ONLINE 0 0 0
  >
  > sdca ONLINE 0 0 0
  >
  > sdcb ONLINE 0 0 0
  >
  > sdcd ONLINE 0 0 0
  
  spares
  
  > \$draid3-0-s0 AVAIL
  >
  > \$draid3-0-s1 AVAIL
  >
  > \$draid3-0-s2 AVAIL
  
  errors: No known data errors
  ------------------------------


