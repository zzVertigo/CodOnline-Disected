# CodOnline-Disected
### A repository dedicated to studying the workings of Call of Duty: Online

This repository is an on going work-in-progress and will be updated as I find more information out.

#### 2021-01-30

To begin I can __undoubtedly confirm__ that Call of Duty: Online does in fact make use of the DemonWare service in order to handle matchmaking. The proof all starts with the hostnames used in the game. If you have worked on any Call of Duty game in the past such as Modern Warfare 2 this will be of no surprise to you but Call of Duty makes use of Stun servers (in this case its stun1-3-tub.codol.qq.com but may vary throughout each game).

Knowing already from previous work done on the game it was very easy to predict which port the game would use and no surprise that it would be running on UDP sockets. Making use of Resource Monitor lead me to confirm this by looking at the active TCP connections.

![ResourceMonitor](https://i.imgur.com/oMmmOmL.png)

![Wireshark](https://i.imgur.com/8HyQWUa.png)

After confirming this the next step was to take a trip on to good ol' Wireshark to start monitoring all the traffic on that port. Sure enough the game makes a connection to the IP 121.51.232.95 and after doing a quick ping to the stun hostname I was able to connect the hostname to that IP.

The next step that I will be pursuing next is a MITM proxy to start dumping packets and analyzing any changes that may have been made although I am not expecting too many changes as when I dumped the binary (Will explore more on this later) I was not shocked to see a lot of remnents of MW2 and Black Ops game code left behind.

FYI: In order to dump the binary you will 110% need to use a Kernel level dump tool (In this case I used [KsDumper](https://github.com/EquiFox/KsDumper)). KsDumper made a pleasent experience and I recommened this tool all the way. After I was left with an unpacked executable and was shocked to see that the size had jumped from a miniscule 7mb all the way to 255mb.

After building the network proxy I was able to give my self some second guesses about the matchmaking backend. After carefully inspecting the packets its hard to say for sure whether it is using DemonWare of if its just an upgraded IWNet server. Some packets share similar characteristics to IWNet but the hostnames lean towards stun servers which are commonly used for DW.

Anyways, a few other things to keep in mind too is there is a web service as well which handles the lobby menu and your player stats. I have a few theories in mind for which of those could be holding my account data but I have yet to confirm anything yet. What I truly need to focus on at the moment is getting the client disconnected and running without the WeGame client. It in more simple terms requires you to have a QQ account in order to make use of the game and without it the game will complain about it needing to be launch with the client. I am unable to see if any parameters are passsed to the client due to the escalation in privileges on the executable but hopefully I can cirmcumvent that at some point. The good news however is chaning the stun servers is quite easy as they left a file called 'stun.ini' that actually allows you to change where the game connects to.

Silly developers...

#### 2021-02-04

I have now had time to work more on the game and discovered that the stun servers aren't as useful as I thought.. from my newest research on it it turns out the client on continously sends packets to it and that's basically it. The real stuff comes in when I began to investigate the LSG and AUTH servers both using the hostname formal-zone-(auth/lsg).codol.qq.com. Both servers make use of 2 servers (one HTTPS and one TCP server listening on port 3074). The client makes a quick connection to the HTTPS server and disconnects right away (probably preventing people from just randomly being able to connect sort of like a barrier before entering the actual LSG/AUTH server) after that the client makes a connection to the AUTH server (not much worth mentioning here) and it sends a few packets. Then we get to the LSG server where actual useful data is sent through like player data and stats alike. The first packet sent through is actually decrypted so it's quite clear that the game makes use of a stream cipher to encrypt data. I haven't been able to identify which crypto the server uses but rest assured it will be solved shortly after IDA finishes its sig scan for references to any cryptography being used. The key is 110% embedded in the client somewhere as the first packet sent actually contains 8 bytes of data that consistently changes on each connection as the data does too.

After that its unclear what the server and client send to each other till I am able to decrypt the data. Regardless, I was able to start at least figuring out the header and so far it seems it uses 4 bytes in the beginning to identify the amount of packet the data contains and after is encrypted data. I can only presume that within that encrypted data exists the true packet header.

More coming soon.
