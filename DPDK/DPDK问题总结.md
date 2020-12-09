# DPDK问题总结

## 问题一

在编译成功helloworld程序后，可能出现下图这种情况

这个问题主要是因为IOMMU的机制都导致，必须要使网卡所在iommu_group的所有设备全部都绑定VFIO驱动或者将其他