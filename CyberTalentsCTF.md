## Quals: Saudi and Oman National CTF
This weekend our team decided to give a shot at the aforementioned CTF. I believe this CTF was used to advertise their CyberTalents.com website. The CTF only featured nine challenges and turned out to be more difficult than we expected. Below are the write-ups for the challenges we were able to solve (some completed after the CTF had already closed).

### Just another conference
This one was the easiest. The clue given was a regular conference hosted by OWASP with the answer being `AppSec`.

### I love images
This one actually took me way longer than I'd like to admit. After using both `strings` and `exiftool` on the image `godot.png` I was quickly able to find the string 'IZGECR33JZXXIX2PNZWHSX2CMFZWKNRUPU======'. It looked like a Base64 string, so I tried decoding it and got nothing of value and did the same with Base32. Unfortunately, I only discovered the flag after the fact, turns out the Base32 decoder I used didn't decode it properly. When I tried again on a different decoder I finally found the flag:
```
FLAG{Not_Only_Base64}
```
But this was after **3 hours** of needless searching with `binwalk`,`stegsolve`, and various other tools. Sadly I didn't find the flag until after the competition had closed.
