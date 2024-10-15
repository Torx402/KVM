There are a few things which I would like to touch on when it comes to specific libvirt XML tags that I have used in my **Windows** guest. The first one is the `<smbios mode="host"/>` tag, which passes on the host's hardware information to the guest. This can be useful for software that have implemented mechanisms to stop VMs from running the software. This tag is placed as part of the OS tag as follows:

```XML
<os firmware='efi'>
.
.
.
  <smbios mode='host'/>
</os>
```

The next topic of interest is the CPU topology, placement, and tuning. There is a difference in how Linux and Windows map physical cores to threads. To illustrate, consider a 2 core 2 thread CPU, the mapping for my CPU (checked using `lscpu`) was as follows:

| Thread | Core |
| ------ | ---- |
| 0      | 0    |
| 1      | 1    |
| 2      | 0    |
| 3      | 1    |
Where as in Windows, this looked like:

| Thread | Core |
| ------ | ---- |
| 0      | 0    |
| 1      | 0    |
| 2      | 1    |
| 3      | 1    |
This difference in mapping can be detrimental for the performance of guests. For this reason, it is always important to check how your OS of choice does this mapping (or just disable hyperthreading/SMT in the BIOS). To this end, you may use the `lscpu -e` command on Linux, or [Coreinfo](https://learn.microsoft.com/en-us/sysinternals/downloads/coreinfo) utility on Windows. Currently, the CPU pinning for my Windows guest looks like:

```XML
  <vcpu placement="static" cpuset="6-11">6</vcpu>
  .
  .
  .
  <cputune>
    <vcpupin vcpu="0" cpuset="6"/>
    <vcpupin vcpu="1" cpuset="7"/>
    <vcpupin vcpu="2" cpuset="8"/>
    <vcpupin vcpu="3" cpuset="9"/>
    <vcpupin vcpu="4" cpuset="10"/>
    <vcpupin vcpu="5" cpuset="11"/>
  </cputune>
  .
  .
  .
  <cpu mode="host-passthrough" check="partial" migratable="off">
    <topology sockets="1" dies="1" clusters="1" cores="6" threads="1"/>
    <cache mode="passthrough"/>
    <feature policy="disable" name="hypervisor"/>
    <feature policy="require" name="topoext"/>
    <feature policy="require" name="invtsc"/>
  </cpu>
```

If I had hyperthreading/SMT on, this would mean that I would allocated 6 cores and 12 threads, in that case the pinning would be:

```XML
  <vcpu placement="static" cpuset="6-11, 18-23">12</vcpu>
  .
  .
  .
  <cputune>
    <vcpupin vcpu="0" cpuset="6"/>
    <vcpupin vcpu="1" cpuset="18"/>
    <vcpupin vcpu="2" cpuset="7"/>
    <vcpupin vcpu="3" cpuset="19"/>
    <vcpupin vcpu="4" cpuset="8"/>
    <vcpupin vcpu="5" cpuset="20"/>
    <vcpupin vcpu="6" cpuset="9"/>
    <vcpupin vcpu="7" cpuset="21"/>
    <vcpupin vcpu="8" cpuset="10"/>
    <vcpupin vcpu="9" cpuset="22"/>
    <vcpupin vcpu="10" cpuset="11"/>
    <vcpupin vcpu="11" cpuset="23"/>
  </cputune>
  .
  .
  .
  <cpu mode="host-passthrough" check="partial" migratable="off">
    <topology sockets="1" dies="1" clusters="1" cores="6" threads="2"/>
    <cache mode="passthrough"/>
    <feature policy="disable" name="hypervisor"/>
    <feature policy="require" name="topoext"/>
    <feature policy="require" name="invtsc"/>
  </cpu>
```

Another tuning tip which can be used to achieve better performance comes from [this comment](https://www.reddit.com/r/VFIO/comments/gm580m/comment/fr8x2gh/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button) on Reddit, by exposing the architecture of the CPU rather manually, it allows the guest OS to better understand the cache hierarchy and core access to them. For a 6 core setup with no hyperthreading/SMT, this looks like:

```XML
  <vcpu placement="static" cpuset="6-11" current="6">12</vcpu>
  <vcpus>
    <vcpu id="0" enabled="yes" hotpluggable="no"/>
    <vcpu id="1" enabled="yes" hotpluggable="yes"/>
    <vcpu id="2" enabled="yes" hotpluggable="yes"/>
    <vcpu id="3" enabled="yes" hotpluggable="yes"/>
    <vcpu id="4" enabled="yes" hotpluggable="yes"/>
    <vcpu id="5" enabled="yes" hotpluggable="yes"/>
    <vcpu id="6" enabled="no" hotpluggable="yes"/>
    <vcpu id="7" enabled="no" hotpluggable="yes"/>
    <vcpu id="8" enabled="no" hotpluggable="yes"/>
    <vcpu id="9" enabled="no" hotpluggable="yes"/>
    <vcpu id="10" enabled="no" hotpluggable="yes"/>
    <vcpu id="11" enabled="no" hotpluggable="yes"/>
  </vcpus>
  <cputune>
    <vcpupin vcpu="0" cpuset="6"/>
    <vcpupin vcpu="1" cpuset="7"/>
    <vcpupin vcpu="2" cpuset="8"/>
    <vcpupin vcpu="3" cpuset="9"/>
    <vcpupin vcpu="4" cpuset="10"/>
    <vcpupin vcpu="5" cpuset="11"/>
  </cputune>
  .
  .
  .
  <cpu mode="host-passthrough" check="partial" migratable="off">
    <topology sockets="1" dies="1" clusters="1" cores="12" threads="1"/>
    <cache mode="passthrough"/>
    <feature policy="disable" name="hypervisor"/>
    <feature policy="require" name="topoext"/>
    <feature policy="require" name="invtsc"/>
  </cpu>
```

