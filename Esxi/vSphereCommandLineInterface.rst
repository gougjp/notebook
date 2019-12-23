vSphere命令行接口
===========================

在ESXI主机上挂载U盘, 服务器上已配置直通
--------------------------------------------

* 使用putty登录esxi服务器

* 执行命令vim-cmd vmsvc/getallvms查看所有虚拟机

.. code::

    [root@localhost:~] vim-cmd vmsvc/getallvms
    Vmid      Name                         File                       Guest OS      Version   Annotation
    1      ate-tester1   [datastore1] ate-tester1/ate-tester1.vmx   windows7Guest   vmx-11

* 执行命令vim-cmd vmsvc/device.getdevices 1查看虚拟机1上的所有设备

.. code::

    [root@localhost:~] vim-cmd vmsvc/device.getdevices 1
    Devices:

    (vim.vm.VirtualHardware) {
       numCPU = 2,
       numCoresPerSocket = 1,
       memoryMB = 3072,
       virtualICH7MPresent = false,
       virtualSMCPresent = false,
       device = (vim.vm.device.VirtualDevice) [
          (vim.vm.device.VirtualIDEController) {
             key = 200,
             deviceInfo = (vim.Description) {
                label = "IDE 0",
                summary = "IDE 0"
             },
             backing = (vim.vm.device.VirtualDevice.BackingInfo) null,
             connectable = (vim.vm.device.VirtualDevice.ConnectInfo) null,
             slotInfo = (vim.vm.device.VirtualDevice.BusSlotInfo) null,
             controllerKey = <unset>,
             unitNumber = <unset>,
             busNumber = 0,
          },
          (vim.vm.device.VirtualIDEController) {
             key = 201,
             deviceInfo = (vim.Description) {
                label = "IDE 1",
                summary = "IDE 1"
             },
             backing = (vim.vm.device.VirtualDevice.BackingInfo) null,
             connectable = (vim.vm.device.VirtualDevice.ConnectInfo) null,
             slotInfo = (vim.vm.device.VirtualDevice.BusSlotInfo) null,
             controllerKey = <unset>,
             unitNumber = <unset>,
             busNumber = 1,
             device = (int) [
                3002
             ]
          },
          (vim.vm.device.VirtualPS2Controller) {
             key = 300,
             deviceInfo = (vim.Description) {
                label = "PS2 controller 0",
                summary = "PS2 controller 0"
             },
             backing = (vim.vm.device.VirtualDevice.BackingInfo) null,
             connectable = (vim.vm.device.VirtualDevice.ConnectInfo) null,
             slotInfo = (vim.vm.device.VirtualDevice.BusSlotInfo) null,
             controllerKey = <unset>,
             unitNumber = <unset>,
             busNumber = 0,
             device = (int) [
                600,
                700
             ]
          },
          (vim.vm.device.VirtualPCIController) {
             key = 100,
             deviceInfo = (vim.Description) {
                label = "PCI controller 0",
                summary = "PCI controller 0"
             },
             backing = (vim.vm.device.VirtualDevice.BackingInfo) null,
             connectable = (vim.vm.device.VirtualDevice.ConnectInfo) null,
             slotInfo = (vim.vm.device.VirtualDevice.BusSlotInfo) null,
             controllerKey = <unset>,
             unitNumber = <unset>,
             busNumber = 0,
             device = (int) [
                500,
                12000,
                7000,
                1000,
                4000
             ]
          },
          (vim.vm.device.VirtualSIOController) {
             key = 400,
             deviceInfo = (vim.Description) {
                label = "SIO controller 0",
                summary = "SIO controller 0"
             },
             backing = (vim.vm.device.VirtualDevice.BackingInfo) null,
             connectable = (vim.vm.device.VirtualDevice.ConnectInfo) null,
             slotInfo = (vim.vm.device.VirtualDevice.BusSlotInfo) null,
             controllerKey = <unset>,
             unitNumber = <unset>,
             busNumber = 0,
             device = (int) [
                8000
             ]
          },
          (vim.vm.device.VirtualKeyboard) {
             key = 600,
             deviceInfo = (vim.Description) {
                label = "Keyboard ",
                summary = "Keyboard"
             },
             backing = (vim.vm.device.VirtualDevice.BackingInfo) null,
             connectable = (vim.vm.device.VirtualDevice.ConnectInfo) null,
             slotInfo = (vim.vm.device.VirtualDevice.BusSlotInfo) null,
             controllerKey = 300,
             unitNumber = 0
          },
          (vim.vm.device.VirtualPointingDevice) {
             key = 700,
             deviceInfo = (vim.Description) {
                label = "Pointing device",
                summary = "Pointing device; Device"
             },
             backing = (vim.vm.device.VirtualPointingDevice.DeviceBackingInfo) {
                deviceName = "",
                useAutoDetect = false,
                hostPointingDevice = "autodetect"
             },
             connectable = (vim.vm.device.VirtualDevice.ConnectInfo) null,
             slotInfo = (vim.vm.device.VirtualDevice.BusSlotInfo) null,
             controllerKey = 300,
             unitNumber = 1
          },
          (vim.vm.device.VirtualVideoCard) {
             key = 500,
             deviceInfo = (vim.Description) {
                label = "Video card ",
                summary = "Video card"
             },
             backing = (vim.vm.device.VirtualDevice.BackingInfo) null,
             connectable = (vim.vm.device.VirtualDevice.ConnectInfo) null,
             slotInfo = (vim.vm.device.VirtualDevice.BusSlotInfo) null,
             controllerKey = 100,
             unitNumber = 0,
             videoRamSizeInKB = 8192,
             numDisplays = 1,
             useAutoDetect = false,
             enable3DSupport = false,
             enableMPTSupport = <unset>,
             use3dRenderer = "automatic",
             graphicsMemorySizeInKB = 262144
          },
          (vim.vm.device.VirtualVMCIDevice) {
             key = 12000,
             deviceInfo = (vim.Description) {
                label = "VMCI device",
                summary = "Device on the virtual machine PCI bus that provides support for the virtual machine communication interface"
             },
             backing = (vim.vm.device.VirtualDevice.BackingInfo) null,
             connectable = (vim.vm.device.VirtualDevice.ConnectInfo) null,
             slotInfo = (vim.vm.device.VirtualDevice.PciBusSlotInfo) {
                pciSlotNumber = 33
             },
             controllerKey = 100,
             unitNumber = 17,
             id = -1708029162,
             allowUnrestrictedCommunication = false,
             filterEnable = true,
             filterInfo = (vim.vm.device.VirtualVMCIDevice.FilterInfo) null
          },
          (vim.vm.device.VirtualUSBController) {
             key = 7000,
             deviceInfo = (vim.Description) {
                label = "USB controller ",
                summary = "Auto connect Disabled"
             },
             backing = (vim.vm.device.VirtualDevice.BackingInfo) null,
             connectable = (vim.vm.device.VirtualDevice.ConnectInfo) null,
             slotInfo = (vim.vm.device.VirtualUSBController.PciBusSlotInfo) {
                pciSlotNumber = 34,
                ehciPciSlotNumber = 35
             },
             controllerKey = 100,
             unitNumber = 22,
             busNumber = 0,
             device = (int) [
                0
             ],
             autoConnectDevices = false,
             ehciEnabled = true
          },
          (vim.vm.device.VirtualUSB) {
             key = 0,
             deviceInfo = (vim.Description) {
                label = "USB 1",
                summary = "SanDisk Extreme"
             },
             backing = (vim.vm.device.VirtualUSB.USBBackingInfo) {
                deviceName = "path:1/9 version:2",
                useAutoDetect = <unset>
             },
             connectable = (vim.vm.device.VirtualDevice.ConnectInfo) null,
             slotInfo = (vim.vm.device.VirtualDevice.BusSlotInfo) null,
             controllerKey = 7000,
             unitNumber = 0,
             connected = true,
             vendor = 1921,
             product = 21888,
             family = (string) [
                "storage"
             ],
             speed = (string) [
                "high"
             ]
          },
          (vim.vm.device.VirtualLsiLogicSASController) {
             key = 1000,
             deviceInfo = (vim.Description) {
                label = "SCSI controller 0",
                summary = "LSI Logic SAS"
             },
             backing = (vim.vm.device.VirtualDevice.BackingInfo) null,
             connectable = (vim.vm.device.VirtualDevice.ConnectInfo) null,
             slotInfo = (vim.vm.device.VirtualDevice.PciBusSlotInfo) {
                pciSlotNumber = 160
             },
             controllerKey = 100,
             unitNumber = 3,
             busNumber = 0,
             device = (int) [
                2000
             ],
             hotAddRemove = true,
             sharedBus = "noSharing",
             scsiCtlrUnitNumber = 7
          },
          (vim.vm.device.VirtualCdrom) {
             key = 3002,
             deviceInfo = (vim.Description) {
                label = "CD/DVD drive 1",
                summary = "ISO [datastore1] cn_windows_7_professional_with_sp1_vl_build_x64_dvd_u_677816.iso"
             },
             backing = (vim.vm.device.VirtualCdrom.IsoBackingInfo) {
                fileName = "[datastore1] cn_windows_7_professional_with_sp1_vl_build_x64_dvd_u_677816.iso",
                datastore = 'vim.Datastore:5e0026d9-a647150e-d8a8-4cedfb76e9d5',
                backingObjectId = <unset>
             },
             connectable = (vim.vm.device.VirtualDevice.ConnectInfo) {
                startConnected = true,
                allowGuestControl = true,
                connected = true,
                status = "ok"
             },
             slotInfo = (vim.vm.device.VirtualDevice.BusSlotInfo) null,
             controllerKey = 201,
             unitNumber = 0
          },
          (vim.vm.device.VirtualDisk) {
             key = 2000,
             deviceInfo = (vim.Description) {
                label = "Hard disk 1",
                summary = "209,715,200 KB"
             },
             backing = (vim.vm.device.VirtualDisk.FlatVer2BackingInfo) {
                fileName = "[datastore1] ate-tester1/ate-tester1.vmdk",
                datastore = 'vim.Datastore:5e0026d9-a647150e-d8a8-4cedfb76e9d5',
                backingObjectId = "",
                diskMode = "persistent",
                split = false,
                writeThrough = false,
                thinProvisioned = false,
                eagerlyScrub = <unset>,
                uuid = "6000C294-7b3f-53be-cb4b-ee5e3d92722e",
                contentId = "7d0a0813c0306b532b906208c88ac9c7",
                changeId = <unset>,
                parent = (vim.vm.device.VirtualDisk.FlatVer2BackingInfo) null,
                deltaDiskFormat = <unset>,
                digestEnabled = false,
                deltaGrainSize = <unset>,
                deltaDiskFormatVariant = <unset>,
                sharing = "sharingNone"
             },
             connectable = (vim.vm.device.VirtualDevice.ConnectInfo) null,
             slotInfo = (vim.vm.device.VirtualDevice.BusSlotInfo) null,
             controllerKey = 1000,
             unitNumber = 0,
             capacityInKB = 209715200,
             capacityInBytes = 214748364800,
             shares = (vim.SharesInfo) {
                shares = 1000,
                level = "normal"
             },
             storageIOAllocation = (vim.StorageResourceManager.IOAllocationInfo) {
                limit = -1,
                shares = (vim.SharesInfo) {
                   shares = 1000,
                   level = "normal"
                },
                reservation = 0
             },
             diskObjectId = "1-2000",
             vFlashCacheConfigInfo = (vim.vm.device.VirtualDisk.VFlashCacheConfigInfo) null,
          },
          (vim.vm.device.VirtualFloppy) {
             key = 8000,
             deviceInfo = (vim.Description) {
                label = "Floppy drive 1",
                summary = "Remote"
             },
             backing = (vim.vm.device.VirtualFloppy.RemoteDeviceBackingInfo) {
                deviceName = "",
                useAutoDetect = false
             },
             connectable = (vim.vm.device.VirtualDevice.ConnectInfo) {
                startConnected = false,
                allowGuestControl = true,
                connected = false,
                status = "ok"
             },
             slotInfo = (vim.vm.device.VirtualDevice.BusSlotInfo) null,
             controllerKey = 400,
             unitNumber = 0
          },
          (vim.vm.device.VirtualE1000) {
             key = 4000,
             deviceInfo = (vim.Description) {
                label = "Network adapter 1",
                summary = "VM Network"
             },
             backing = (vim.vm.device.VirtualEthernetCard.NetworkBackingInfo) {
                deviceName = "VM Network",
                useAutoDetect = false,
                network = 'vim.Network:HaNetwork-VM Network',
                inPassthroughMode = <unset>
             },
             connectable = (vim.vm.device.VirtualDevice.ConnectInfo) {
                startConnected = true,
                allowGuestControl = true,
                connected = true,
                status = "ok"
             },
             slotInfo = (vim.vm.device.VirtualDevice.PciBusSlotInfo) {
                pciSlotNumber = 32
             },
             controllerKey = 100,
             unitNumber = 7,
             addressType = "generated",
             macAddress = "00:0c:29:31:8b:16",
             wakeOnLanEnabled = true,
             resourceAllocation = (vim.vm.device.VirtualEthernetCard.ResourceAllocation) {
                reservation = 0,
                share = (vim.SharesInfo) {
                   shares = 50,
                   level = "normal"
                },
                limit = -1
             },
             externalId = <unset>,
             uptCompatibilityEnabled = false,
          }
       ]
    }

其中VirtualUSB就是USB设备, 可以看到其名字域: deviceName = "path:1/9 version:2"

* 执行命令vim-cmd vmsvc/device.disconnusbdev 1 "path:1/9 version:2"删除USB设备, 第一个参数1是虚拟机的ID, 第二个参数"path:1/9 version:2"是USB的设备名

* 执行命令vim-cmd vmsvc/device.connusbdev 1 "path:1/9 version:2"添加USB设备, 第一个参数1是虚拟机的ID, 第二个参数"path:1/9 version:2"是USB的设备名

























