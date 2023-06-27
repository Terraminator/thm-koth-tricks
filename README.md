# thm-koth-tricks

Note:
First of all I suggest trying to find tricks for yourself before reading this as you will learn the most by inventing things for yourself. You wont find any flags or writeups for koth machines here, as the machines are not rotating and I want to keep the game playable.
I also want to thank some people i had fun with during playing koth: MatheuzSec(https://github.com/MatheuZSecurity), F11Snipe(https://github.com/f11snipe), H00dy(https://github.com/hoodietramp)

# Introduction:
King of the Hill (KoTH) is a competitive hacking game, where you play against 10 other hackers to compromise a machine and then patch its vulnerabilities to stop other players from also gaining access. The longer your name stays in king.txt, the more points you get.
The real challenge in this game is to defend the king.txt file.

# Ways to protect king.txt:

## good old immutable bit:

you can use chattr to set the immutable bit on the king file:
<pre>echo USERNAME > king.txt</pre>
<pre>chattr +i king.txt</pre>
according to the docs(https://man7.org/linux/man-pages/man1/chattr.1.html):
i      A file with the 'i' attribute cannot be modified: it
       cannot be deleted or renamed, no link can be created to
       this file, most of the file's metadata can not be
       modified, and the file can not be opened in write mode.
       Only the superuser or a process possessing the
       CAP_LINUX_IMMUTABLE capability can set or clear this
       attribute.
    
Setting 'a' and 'i' attributes will not
       affect the ability to write to already existing file descriptors.

Additionally the immutable bit can easily be removed by just using:

<pre>chattr -i king.txt</pre>

To view the file attributes on a file you can use lsattr(https://man7.org/linux/man-pages/man1/lsattr.1.html):

<pre>lsattr king.txt</pre>

kingmaker:
you now might think of a loop which constantly resets the immutable bit on king.txt and i would like to add a reference to Aquinas github as he wrote a good kingmaker script:
https://github.com/ChrisPritchard/ctf-writeups/blob/master/tryhackme-koth/tools/kingmaker.c
this tool uses the ioctl syscall to set the immutable bit very fast.
Disadvantages:
Script can easily be found via lsof and killed. On some older machines(f.e. space jam) it also leads to an empty king file due to race conditions.

There are also some more tricks for example using noclobber in bashrc but I want to focus on the main tricks
## Advanced techniques:

## The mount trick:
<pre>
sudo lessecho USERNAME > /root/king.txt
sudo dd if=/dev/zero of=/dev/shm/root_f bs=1000 count=100
sudo mkfs.ext3 /dev/shm/root_f
sudo mkdir /dev/shm/sqashfs
sudo mount -o loop /dev/shm/root_f /dev/shm/sqashfs/
sudo chmod -R 777 /dev/shm/sqashfs/
sudo lessecho USERNAME > /dev/shm/sqashfs/king.txt
sudo mount -o ro,remount /dev/shm/sqashfs
sudo mount -o bind /dev/shm/sqashfs/king.txt /root/king.txt
sudo rm -rf /dev/shm/root_f
</pre>

This trick creates a second file system and links a folder to it. After that this file system gets mounted read only and mounted over the original king file. In the end the unnecessary file system file system gets deleted.

Disadvantages:
Can break the machine if used by noobs who dont understand what they are doing. The king file can be unmounted using umount -l king.txt.
The file system can be mounted to be writable again using: mount -o rw,remount /dev/shm/sqashfs

## Rootkits:

I want to say that i dont like users who spam their rootkits in public games, as they are completely gamebreaking. Also they are not really practical in real world.

##Userspace rootkits:
see https://github.com/Terraminator/kirito

##Kernelmode rootkits:
here are some ressources i used to build my own ones:
https://github.com/m0nad/Diamorphine
https://xcellerator.github.io/posts/linux_rootkits_01/

If you want to see one of my rootkits even though it is not specific for king of the hill you can look at the rootkit locutus i developed for DemonizedShell:
https://github.com/MatheuZSecurity/D3m0n1z3dShell/tree/main/locutus
you can find the installer here:
https://raw.githubusercontent.com/MatheuZSecurity/D3m0n1z3dShell/main/install_locutus.sh

Disadvantages:
Rootkits are really invasive and can breaks machines. Kernelmodules in general need to be recompiled for every single target architecture. If they do not have any hiding capabilities you can find them by running lsmod(https://man7.org/linux/man-pages/man8/lsmod.8.html).
If you find a rootkit by using lsmod you can usually remove it by running rmmod module_name or modprobe -r module_name. Sometimes they also leave traces in /var/log/kern.log and dmesg. You can clear dmesg by running dmesg --clear. They are also not persistent and will be unloaded after a reboot.
If a rootkit was coded by people who know what they are doing there is litterally no way to find it then scanning your filesystem from an external device. You need to be root to implant a rootkit.
Other places you could look for a rootkit: /proc/kallsyms and /etc/modules.

# Persistence Techniques:
I recommend looking at the following tool I helped developing:
https://github.com/MatheuZSecurity/D3m0n1z3dShell#

## process hiding
similar to the mount trick you can hide process by mounting an empty directory above their proc folder as ps is using the /proc folder to list all processes. For example:

<pre>mkdir /dev/shm/.hidden && mount -o bind /dev/shm/.hidden /proc/pid</pre>

## Troll

### Basics:
You can send nyancat to a user who has a pts(pseudo terminal slave):
download and compile nyancat from here: (https://github.com/klange/nyancat.git)

You can list users who have a pts using: <pre>w</pre>
https://man7.org/linux/man-pages/man1/w.1.html

Then you can send them nyacat by executing:

<pre>./nyacat > /dev/pts/yourpts</pre>

You can even execute commands in their terminals if the have a pts:
<pre>script -f /dev/pts/yourpts</pre>

This can all be bypassed by just not using a pts.

### Advanced:

You can manipulate bashrc files to run command before the user even gets into the terminal.
For example change /home/user/.bashrc and add a line like echo "Hello" and the user will be greeted with Hello on login.
I will leave the potential to your creativity here. There are also other rc files you could research.
To prevent getting targeted by this do not load the rc file on startup. For example you can disable the rc file on startup with:
<pre>ssh -t username@hostname /bin/sh</pre>

you can even put them into a shell you implemented yourself(f.e. trap "pre-prompt" or modify PROMPT_COMMAND, PS1 ...).

I also like to install bash insulter for command not found and stuff.
To monitor the ativity of other users i recommend using pspy(https://github.com/DominicBreuker/pspy)
I hope this ressource provides a good base for learning!

Terraminator
