# Intel-GPU-Passthrough-to-Proxmox-Container
## Steps taken to passthrough Intel iGPU to an Ubuntu Container on Proxmox 7

After a recent in place upgrade of Proxmox 6 to Proxmox 7, my hardware based transcoding within Plex no longer functioned. I took the opprotunity to lifecycle my old container into a new one, and following these steps I was able to get hardware transcoding. 

1. One the proxmox hosts BIOS, enable intel integrated graphics and disable IOMMU
2. In the command line of the host, run `ls -l /dev/dri` which should have something resembling the following output
```
drw-rw---- 2 root root         80 Aug 17 11:03 by-path
crw-rw---- 1 root video  226,   0 Aug 17 11:03 card0
crw-rw---- 1 root render 226, 128 Aug 17 11:03 renderD128
```
3. Create a privledged container using the ubuntu-20.04-standard_20.04-1_amd64 template with all the necessary hardware. Start, then shutdown the container
4. On the Proxmox host edit the container config file "/etc/pve/lxc/XXX.conf", replacing XXX with your container number. Add the following lines to the bottom of the file, note that you may need to replace the numbers with the results you got above with `ls -l /dev/dri`
```
lxc.cgroup2.devices.allow: c 226:0 rwm
lxc.cgroup2.devices.allow: c 226:128 rwm
lxc.cgroup2.devices.allow: c 29:0 rwm
lxc.mount.entry: /dev/dri dev/dri none bind,optional,create=dir
lxc.mount.entry: /dev/fb0 dev/fb0 none bind,optional,create=file
```
4. Power machine back up and update `apt update && apt dist-upgrade -y`
5. Install the intel drivers into the container
```
mkdir ~/neo
wget https://github.com/intel/compute-runtime/releases/download/21.49.21786/intel-gmmlib_21.3.3_amd64.deb -P ~/neo
wget https://github.com/intel/intel-graphics-compiler/releases/download/igc-1.0.9441/intel-igc-core_1.0.9441_amd64.deb -P ~/neo
wget https://github.com/intel/intel-graphics-compiler/releases/download/igc-1.0.9441/intel-igc-opencl_1.0.9441_amd64.deb -P ~/neo
wget https://github.com/intel/compute-runtime/releases/download/21.49.21786/intel-opencl-icd_21.49.21786_amd64.deb -P ~/neo
dpkg -i ~/neo/intel-gmmlib_21.3.3_amd64.deb
dpkg -i ~/neo/intel-igc-core_1.0.9441_amd64.deb
dpkg -i ~/neo/intel-igc-opencl_1.0.9441_amd64.deb
dpkg -i ~/neo/intel-opencl-icd_21.49.21786_amd64.deb
```
