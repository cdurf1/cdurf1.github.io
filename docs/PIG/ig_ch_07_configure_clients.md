# Configuring Clients 
====================

A client (compute node) accessing a storage appliance must be running
Intel® EE for Lustre\* 3.1.0.3 client software. The Lustre file system
must first be created or discovered at the Intel® Manager for Lustre\*
dashboard (see the Intel® Manager for Lustre\* Online Help to do this).
The Lustre client software must be installed on the client, and then the
Lustre file system can be mounted on the client as described on the
Online Help.

Client Requirements 
--------------------

Each file system client must be running Red Hat Enterprise Linux (RHEL)
or CentOS Linux, version 6.8 or 7.3.

**Notes**:

-   Before using the Red Hat or RHEL software referenced herein, please
    refer to Red Hat’s website for more information, including without
    limitation, information regarding the mitigation of potential
    security vulnerabilities in the Red Hat software.

-   Client support for el6 is distributed in the
    ee-contrib-3.1.0.3/el6/lustre-client-&lt;lu-version&gt;-bundle.tar.gz
    tarball.

Intel® EE for Lustre\* software may be installed on file system
*servers and clients* running SUSE Linux Enterprise version 11 with SP4,
and on clients-only running SLES12 with SP1. However, for SLES
installations, Intel® Manager for Lustre\* software is *not supported or
installed.* As a result, automatic configuration and/or monitoring of
high availability is not supported by Intel® Manager for Lustre\*
software on installations running SUSE.

For information about clients running SUSE, see [Installing Lustre on
SUSE Linux Enterprise Server](#_Installing_Lustre_on).

LNET provides the client network infrastructure required by the Lustre
file system and LNET must be configured for each client. See [LNET
Configuration](#_LNET_Configuration).

Installing Intel® EE for Lustre\* software on Clients Running RHEL or CentOS
----------------------------------------------------------------------------

The following instructions detail how to install and configure client
software.

The following Lustre packages are installed on clients:

|Package|Description|
|---|---|
|lustre-client-modules-<ver>|Lustre module RPM for clients.|
|lustre-client-<ver>|Lustre utilities for clients.|

**To configure a Lustre client running RHEL or CentOS version 6.8,
perform these steps:**

1.  Download the Lustre client RPMs for your platform from the [Lustre\*
    Releases](https://wiki.hpdd.intel.com/display/PUB/Lustre+Releases)
    repository. See [Table 8.3, “Packages Installed on Lustre
    Clients”](http://doc.lustre.org/lustre_manual.xhtml#table_j3r_ym3_gk)
    for a list of required packages. (You may need to paste the URL in a
    browser.)

2.  Install the Lustre client packages on all Lustre clients.

    **Note**: The version of the kernel running on a Lustre client must be
the same as the version of the lustre-client-modules-ver package being
installed. If not, a compatible kernel must be installed on the client
before the Lustre client packages are installed.

    a.  Log onto a Lustre client as the root user.

    b.  Use the yum command to install the packages:


    ```
# yum --nogpgcheck install pkg1.rpm pkg2.rpm ...
```


    c.  Verify the packages were installed correctly:


    ```
# rpm -qa|egrep "lustre|kernel"|sort
```


    d.  Reboot the client.

    e.  Repeat these steps on each Lustre client.


1.  Configure LNET on the client.

2.  Launch Intel® Manager for Lustre\* software and login as
    administrator. Go to the manager GUI to obtain mount point
    information:


    a.  Go to **Configuration** &gt; **File Systems.**

    b.  In the table listing available file systems, click the name of the
    file system to be accessed by the client. A page showing file system
    details will be displayed.

    c.  Click **View** **Client Mount Information**. The mount command to be
    used to mount the file system will be displayed as shown in this
    *example*:


    ```
mount -t lustre 10.214.13.245@tcp0:/test /mnt/test
```


1.  On the client, enter the mount command provided.

**To configure a Lustre client running RHEL or CentOS version 7.3,
perform these steps:**

1.  For clients running RHEL or CentOS version 7.3, add a client
    repository with the following command.

    ```
    # yum-config-manager --add-repo=
    https://<command_center_server>/client/7
```


2.  This will create a file in /etc/yum.repos.d named ```<\server.fqdn>_client.repo```(e.g. foo.bar.baz_client.repo)

3.  Edit the generated file &lt;server.fqdn&gt;\_client.repo and add the
    following lines at the end of the file:

    ```
    sslverify = 0
    gpgcheck = 0
```

    Then save and close.

4.  Install the required Lustre packages on each client:


    a.  Enter (on one line):


    ```
# yum install lustre-client-modules-<ver>.<arch>.rpm
    ```


    b.  Update the bootloader (grub.conf or lilo.conf) configuration file as
    needed.

    **Note**: Verify that the bootloader configuration file has been updated with an entry for the new kernel. Before you can boot to a  kernel, an entry for it must be included in the bootloader configuration file. Often it is added automatically when the kernel RPM is installed.

1.  Launch Intel® Manager for Lustre\* software and login as
    administrator. Go to the manager GUI to obtain mount point
    information:


    a.  Go to **Configuration > File Systems.**

    b.  In the table listing available file systems, click the name of the
    file system to be accessed by the client. A page showing file system
    details will be displayed.

    c.  Click **View Client Mount Information**. The mount command to be
    used to mount the file system will be displayed as shown in this
    *example*:


    ```
mount -t lustre 10.214.13.245@tcp0:/test /mnt/test
```


1.  On the client, enter the mount command provided.[[[[]{#_Toc325121001
    .anchor}]{#_Ref327448701 .anchor}]{#_Toc325385416
    .anchor}]{#InstallingChromaManager-InstallingAdditi .anchor}