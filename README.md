# p2p-udp-communication
## UDP Punch Hole + Reliable(ish) UDP

Server + Client software to enable communication between two peers using UDP punch hole and adding basic reliability capabilities (checksum+ack)

I made this so I could play some online games that was only possible through port forwarding, having no personal public IPv4 address available anymore prevents for those type of games

To get over that I did this software to enable peer to peer communication using the concept of UDP Punch Hole, and also adding some reliability to the UDP packets so the connection can be, at least, functional

This is how it works:

```
[HOST] ----- [SERVER] ----- [CLIENT]
```

The SERVER is responsible to function as the udp punch hole enabler, giving the HOST the CLIENT's IP and Port and CLIENT the HOST's IP and Port  
After HOST successfully communicates with CLIENT through the punch hole, the software will start exchanging packets

Then CLIENT will create a TCP Listen Socket to the IP 127.0.0.1 on port 52000  
This socket will work as the tunnel CLIENT will use to reach HOST

In my case, when I was hosting my minecraft server, my friends would connect to my server by direct connecting, on their side, to 127.0.0.1:52000

After they connect to it, the software (CLIENT side) will forward all packets through the punch hole to HOST, which will also create a connection to the server it is setup to connect, and in my case it was the minecraft server, so then the software (on HOST side) starts a new TCP socket and connects to 127.0.0.1:25565

This is a visual representation of all of this:

```
                                       UDP PUNCH HOLE (packet exchange)                                      
                        public ip:random port                      public ip:random port
                        
[MINECRAFT SERVER] ======== [HOST] ====================================== [CLIENT] ======== [MINECRAFT CLIENT]
   listening at             tcp socket                                    tcp socket           connected to
 127.0.0.1:25565            connecting to                                 listening at        127.0.0.1:52000
                            127.0.0.1:25565                               127.0.0.1:52000
```

On HOST side, every new packet coming from CLIENT is forwarded to 127.0.0.1:25565  
Every packet response coming from 127.0.0.1:25565 to HOST is forwarded to CLIENT, that receives it through the UDP PUNCH HOLE, and then it forwards to 127.0.0.1:52000, effectively creating a "fake" TCP connection through the UDP PUNCH HOLE

```
            [HOST SIDE]                   UDP PUNCH HOLE                 [CLIENT SIDE]
[127.0.0.1:25565] ===== [123.213.143.132:13516] >>>>>>>>>> [132.112.134.13:12463] ===== [127.0.0.1:52000]
                                                <<<<<<<<<<
```

I also did some basic hash verification and packet acknowledgment to make sure the packets coming from the UDP PUNCH HOLE were correct in its integrity, if not, then it requests a resend of the packet to the other side, it'll halt one path (IN or OUT packets from HOST or CLIENT side) of communication until this missed packet is recovered or the connection is deemed as lost, and in that case, sockets are shutdown

--------------------------

Some problems and how I managed to solve it:

PROBLEM: Some NAT don't work well with UDP PUNCH HOLE tech, depending on the NAT architecture, the punch hole will just not work.

For that I also make it so UDP PUNCH HOLE can be done using IPv6. Some home routers, even though you have a unique IPv6, you can't change settings in the router and enable traffic from its firewall, keeping you from port forwarding and communication using it. 

In my case, none of my friends had another NAT for our IPv6 so we were able to UDP PUNCH HOLE the router's firewall easily and exchange packets this way.

Even when using IPv6 to enable UDP PUNCH HOLE, that was just for the external communication. 
To connect internally HOST still connects to 127.0.0.1:25565 and CLIENT to 127.0.0.1:52000

```
            [HOST SIDE]                                                UDP PUNCH HOLE                                          [CLIENT SIDE]
[127.0.0.1:25565] ===== [2001:0db8:85a3:0000:0000:8a2e:0370:7334:13516] >>>>>>>>>> [2001:0db8:85a3:0000:0000:8a2e:0370:7334:12463] ===== [127.0.0.1:52000]
                                                                        <<<<<<<<<<
```

I only tried using Minecraft as game server but it should work with other games as well, I also did some basic testing with Apache and it also works.

### To-Do:

* Change SERVER code so it can handle multiple connections coming at the same time
* Add Encryption/Decryption to UDP packets for security and privacy
* Also, I don't think any of this code is well written or anything, so a code refactoring is also a good idea
* Yeah also, this only works for TCP servers (not even sure if a UDP one exists but... sure I guess it's an idea to implement UDP listeners)












