# EvlzCTF 2019

2/2/2019 43/176

Teammates: TheHeavyHorse, Seanr94

Spent about four hours total. Got flags for Sanity Check 1, WeTheUsers, and FindMe. Also got flag for DontBlink but had trouble inputting it.

### Find Me
**Type**: Reverse

Used 'strings findme' to find:
> dWdnYyUzQSUyRiUyRnJpeW0lN0JoZXlfZnJyemZfZWJnZ3JhX2p2Z3VfNjQlN0RwZ3MucGJ6


It looked like Base64 and after decoding we get:
> uggc%3A%2F%2Friym%7Bhey_frrzf_ebggra_jvgu_64%7Dpgs.pbz

This didn't help us til later. We next analyzed the code with Radare2 and realized the executable contains three functions. We found the first one to be a ROT13 cipher. Every time we tried running the code though, it would crash at the end of function one after a 'cdqe' command. TheHeavyHorse used *Bless* to get rid of the command and after further analysis the second function revealed that it encoded strings to URL. From here we inferred that the third function was Base64 encoder and decoded the above strings to the following to get the key:
'''
uggc://riym{hey_frrzf_ebggra_jvgu_64}pgs.pbz
http://evlz{url_seems_rotten_with}ctf.com
'''

### WeTheUsers
**Type**: Web

In this challenge we were given a [source file](https://github.com/adamstebbing/CTF_Writeups/evlzctf-web-chal.txt) which allowed us to figure out that if the password is written 'password, \'true\',' then the user would be granted admin privileges.
