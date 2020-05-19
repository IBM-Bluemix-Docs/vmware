---

copyright:
  years: 1994, 2020
lastupdated: "2020-05-19"

keywords: iSCSI, VMWare ESXi, mount iscsi, mount esxi, 

subcollection: vmware

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:table: .aria-labeledby="caption"}

# Mounting iSCSI VMWare ESXi
{: #mount-iscsi-esxi}

Mounting iSCSI into VMWare ESXi can be accomplished with a few steps and only the details of the server and storage node.
{:shortdesc}

1. Log into the vSphere by using the Primary Private IP and user **root** and root's password.
* From the Welcome Page, click the **Configuration** tab.
* Click **Storage Adapters** > **Add…**.
* Click **OK** to add the Software iSCSI Adapter.
* Confirm by clicking **OK**.
* After the refresh, the new iSCSI Adapter is listed.
* Click **Properties**.
* From the Properties Window, click **Configure** and set the **Name** to the IQN for the server(found on the storage device page under authorized hosts).
* Click the **Dynamic Discovery Tab** then click the **Add...**.
* The iSCSI server will be the target IP of the storage device then click **CHAP..**.
* Select **Use CHAP** and clear **Inherit from Parent**. Input the **Username** (found on the storage device page under authorized hosts) and its password.
* Select **Do not use CHAP** under the Mutual CHAP section then click **OK**.
* You will now see the device in the Dynamic Discovery Window, click **Close**.
* Confirm the Rescan of the storage devices.
* You will then see the device turn grey and 'unmounted.'
* Right click the device name and select **attach**.
* Click the **Datastore** Menu on the left side column then click **add storage** and choose **Disc/LUN**.
* Click the device with the 'iqn.'
* Choose the File system Version that you want and click **Next** to continue through the wizard.
* You can now use the iSCSI as needed by the host and the VMs you create.



<!--Attaching a Data Transfer Service iSCSI device is the same process, with the exception that you need to get the IQN from the server. Complete the following steps from the ESXi console:-->

<!--First, you need to get the device:-->

<!-- `esxcfg-scsidevs -a | grep iSCSI`-->

<!-- Then, you need to get the IQN (in this case, vmhba33 is the iSCSI device): -->

<!-- `vmkiscsi-tool -I -l vmhba33` -->
