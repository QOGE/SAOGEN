root@vmi3403633:~/qogecoin# apt-get install -y build-essential libsodium-dev libboost-all-dev libgmp3-dev python3 python3-pip
npm install -g node-gyp
cd ~
git clone https://github.com/ROZ-MOFUMOFU-ME/zny-nomp
cd zny-nomp
npm install
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
build-essential is already the newest version (12.10ubuntu1).
libboost-all-dev is already the newest version (1.83.0.1ubuntu2).
python3 is already the newest version (3.12.3-0ubuntu2.1).
The following packages were automatically installed and are no longer required:
  libfwupd2 libgusb2
Use 'apt autoremove' to remove them.
Suggested packages:
  gmp-doc libgmp10-doc libmpfr-dev
The following NEW packages will be installed:
  libgmp-dev libgmp3-dev libgmpxx4ldbl libsodium-dev python3-pip python3-wheel
0 upgraded, 6 newly installed, 0 to remove and 10 not upgraded.
Need to get 1910 kB of archives.
After this operation, 9786 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libgmpxx4ldbl amd64 2:6.3.0+dfsg-2ubuntu6.1 [9888 B]
Get:2 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libgmp-dev amd64 2:6.3.0+dfsg-2ubuntu6.1 [340 kB]
Get:3 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libgmp3-dev amd64 2:6.3.0+dfsg-2ubuntu6.1 [2310 B]
Get:4 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libsodium-dev amd64 1.0.18-1ubuntu0.24.04.1 [185 kB]
Get:5 http://archive.ubuntu.com/ubuntu noble/universe amd64 python3-wheel all 0.42.0-2 [53.1 kB]
Get:6 http://archive.ubuntu.com/ubuntu noble-updates/universe amd64 python3-pip all 24.0+dfsg-1ubuntu1.3 [1320 kB]
Fetched 1910 kB in 0s (4465 kB/s)
Selecting previously unselected package libgmpxx4ldbl:amd64.
(Reading database ... 154881 files and directories currently installed.)
Preparing to unpack .../0-libgmpxx4ldbl_2%3a6.3.0+dfsg-2ubuntu6.1_amd64.deb ...
Unpacking libgmpxx4ldbl:amd64 (2:6.3.0+dfsg-2ubuntu6.1) ...
Selecting previously unselected package libgmp-dev:amd64.
Preparing to unpack .../1-libgmp-dev_2%3a6.3.0+dfsg-2ubuntu6.1_amd64.deb ...
Unpacking libgmp-dev:amd64 (2:6.3.0+dfsg-2ubuntu6.1) ...
Selecting previously unselected package libgmp3-dev:amd64.
Preparing to unpack .../2-libgmp3-dev_2%3a6.3.0+dfsg-2ubuntu6.1_amd64.deb ...
Unpacking libgmp3-dev:amd64 (2:6.3.0+dfsg-2ubuntu6.1) ...
Selecting previously unselected package libsodium-dev:amd64.
Preparing to unpack .../3-libsodium-dev_1.0.18-1ubuntu0.24.04.1_amd64.deb ...
Unpacking libsodium-dev:amd64 (1.0.18-1ubuntu0.24.04.1) ...
Selecting previously unselected package python3-wheel.
Preparing to unpack .../4-python3-wheel_0.42.0-2_all.deb ...
Unpacking python3-wheel (0.42.0-2) ...
Selecting previously unselected package python3-pip.
Preparing to unpack .../5-python3-pip_24.0+dfsg-1ubuntu1.3_all.deb ...
Unpacking python3-pip (24.0+dfsg-1ubuntu1.3) ...
Setting up python3-wheel (0.42.0-2) ...
Setting up libgmpxx4ldbl:amd64 (2:6.3.0+dfsg-2ubuntu6.1) ...
Setting up libsodium-dev:amd64 (1.0.18-1ubuntu0.24.04.1) ...
Setting up python3-pip (24.0+dfsg-1ubuntu1.3) ...
Setting up libgmp-dev:amd64 (2:6.3.0+dfsg-2ubuntu6.1) ...
Setting up libgmp3-dev:amd64 (2:6.3.0+dfsg-2ubuntu6.1) ...
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for libc-bin (2.39-0ubuntu8.7) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.

added 20 packages in 3s

2 packages are looking for funding
  run `npm fund` for details
npm notice
npm notice New major version of npm available! 10.8.2 -> 12.0.1
npm notice Changelog: https://github.com/npm/cli/releases/tag/v12.0.1
npm notice To update run: npm install -g npm@12.0.1
npm notice
Cloning into 'zny-nomp'...
remote: Enumerating objects: 6858, done.
remote: Counting objects: 100% (1389/1389), done.
remote: Compressing objects: 100% (147/147), done.
remote: Total 6858 (delta 1298), reused 1242 (delta 1242), pack-reused 5469 (from 3)
Receiving objects: 100% (6858/6858), 2.52 MiB | 9.54 MiB/s, done.
Resolving deltas: 100% (4326/4326), done.

added 234 packages, and audited 235 packages in 1m

50 packages are looking for funding

  run `npm fund` for details

found 0 vulnerabilities
root@vmi3403633:~/zny-nomp# ^C
root@vmi3403633:~/zny-nomp#

# ____________________________________________

root@vmi3403633:~/zny-nomp# cd ~/zny-nomp
find node_modules -iname "*multi-hashing*" -maxdepth 2
grep -ri "yescrypt" node_modules/multi-hashing/*.js node_modules/multi-hashing/**/*.cc 2>/dev/null | head -30
find: warning: you have specified the global option -maxdepth after the argument -iname, but global options are not positional, i.e., -maxdepth affects tests specified before it as well as those specified after it.  Please specify global options before other arguments.
node_modules/multi-hashing
root@vmi3403633:~/zny-nomp# ^C
root@vmi3403633:~/zny-nomp# grep -n "pchMessageStart" ~/qogecoin/src/chainparams.cpp
112:         pchMessageStart[0]  = 0xed;
113:         pchMessageStart[1]  = 0x7d;
114:         pchMessageStart[2]  = 0x20;
115:         pchMessageStart[3]  = 0xe9;
220:        pchMessageStart[0] = 0x39;
221:        pchMessageStart[1] = 0x2d;
222:        pchMessageStart[2] = 0x46;
223:        pchMessageStart[3] = 0x91;
325:        pchMessageStart[0] = 0x0a;
326:        pchMessageStart[1] = 0x03;
327:        pchMessageStart[2] = 0x09;
328:        pchMessageStart[3] = 0x07;
406:        pchMessageStart[0] = 0xfa;
407:        pchMessageStart[1] = 0xbf;
408:        pchMessageStart[2] = 0xb5;
409:        pchMessageStart[3] = 0xda;
root@vmi3403633:~/zny-nomp#

