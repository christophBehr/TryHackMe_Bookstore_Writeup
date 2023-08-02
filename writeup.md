# Try Hack Me Bookstore Writeup

## A Beginner level box with basic web enumeration and REST API Fuzzing

### Start
After starting the victim machine I started my Parrot VM on VirtualBox connected via openVPN to the tryhackme network and checked if I'm really connected.
With a Ping I ensured that it is visible and the connection is established.

### Enumeration

First I started with `nmap [victim machine IP]`

<img title="nmap scan" alt="standart nmap scan" src="/assets/nmap.png">

It shows a possible AIP on port 5000,
A second nmap run `nmap -sC -sV [victim machine IP]` gives a lot more detail.
I was curious when I saw Werkzeug on port 5000. It seems some sort of a debugger involeved with a 
Patreon Hack in 2015. 
But first I tried every button and function on http://[victim machine IP]/ with no results, 
everything is basically a dummy.
I searched with gobuster for interesting sub-directories but had no luck. A quick
```
gobuster dir -u http://[victim machine IP] -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
shows nothing interesting.

<img title="gobuster scan"  alt="nothing interesting" src="/assets/gobuster.png">

I checked http://[victim machine IP]:5000, it reveals the used API called Foxy API.
<img tittle="Foxy API" alt="Foxy REST API v.2.0" src="/assets/foxyREST.png">
I google searched the documentation of the API, but there wasn't anything helpfull here. 
There was some kind of exploit with this API but I haven't tried that.
Then I did another gobuster scan, this time on [victim machine IP]:5000.
```
gobuster dir -u http://[victim machine IP]:5000 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

The scan gave me with /api and /console two interesting sub-directories which I check out immediately.
<img title="gobuster on port 5000" alt="/api and /console" src="/assets/gobuster_port5000.png">
http://[victim machine IP]:5000/api/ gave me some Documentation, a nice find to play around with.
<img title="/api" alt="API Documentation" src="/assets/port5000_api.png">

http://[victim machine IP]:5000/console/ on the other hand gave me, what seems like a python console in the browser.
<img title="/console" alt="Pythonshell locked with pin" src="/assets/port5000_console.png">
Unfortunately the console is locked with a pin code.

### Gaining initial foothold 

I tried to play around with the findings on the API Documentation page. But again had no luck. I tried wfuzz on the API's but it showed nothing new.
While playing around I noticed /api/v2/ in the URl's and I wondered if v2 = version 2 and if so, some rests of version 1 still existing somewhere.
So I kept fuzzing around until I had success with 
```
wfuzz -u http://[vicitim machine IP]:5000/api/v1/resources/books?FUZZ=1 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --hc 404
``` 
which gave me a new parameter that I could try out.

<img title="wfuzz" alt="finally success with wfuzz" src="/assets/wfuzz.png">

Maybe LFI (Local File Inclusion) is working, so I tried to get the bash history
http://[victim machine IP]:5000/api/v1/resources/books?show=.bash_history 
which actually worked and gave me the needed pin for the console.
<img title="Console PIN="the needed Pin"  src="/assets/pin2.png">

After I inserted the pin I had access to a python console to play with.
A reversed shell would be much appreciated now. I set up a netcat listener on port 1234 on my attack machine and started googling for a python reversed shell.
I found a a nice manual here: [Python_Shell](https://www.linuxfordevices.com/tutorials/shell-script/reverse-shell-in-python)
And was granted with a shell as sid. With `ls` I found the first flag in user.txt

### Becoming Root

The other files where mostly uninteresting except for the try-harder. Trying `./try-harder` gave me a prompt to insert the magic number which I didn't  have, so I guess I have to try harder...
I kept searching for clues what the magic number might be, but there where none.
Then I opened a simple python server on the victim machine and wget it to my attack machine.
I knew that with Ghidra you can reverse engineer Software. I didn't  have any experience with Ghidra or reverse engineering.
As. like many times, YouTube for the rescue. 
This [Ghidra_Tutorial](https://www.youtube.com/watch?v=fTGTnrgjuGA&t=488s) gave me enough information to reverse engineer the try-harder script.
The magic number is the result of three XOR'd HEX numbers found in the code.
Running the try-harder script again on the victim machine an inserting the magic number granted me root.
in /root/root.txt I found the root and final flag.






