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
root@vmi3403633:~/zny-nomp# ^C
root@vmi3403633:~/zny-nomp#
