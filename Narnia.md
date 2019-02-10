## OverTheWire: Narnia
After the frusration and lack of success in the Oman Quals CTF, I've decided to redirect my attention to some of the OverTheWire challenges. I've already completed both the Bandit and Leviathon series of challenges, but will not be doing write-ups on those due to the amount of resources already out there.

### Narnia0
After logging in with the username and password `narnia0` and reading the Welcome banner. It is discovered that the passwords for each level are stored in /etc/narnia_pass/narniaX and the programs to be exploited are in /narnia/ (as well as the source code for said programs).

After looking at the code for narnia0.c, we can see that we need the val variable to be equal to 0xdeadbeef which can be done with a buffer overflow since the buf variable that is read-in from user input is just before the val variable on the stack.

**To be continued**
