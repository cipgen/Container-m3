# module_3_container  

https://shell.cloud.google.com/  

sudo apt-get install asciinema  

sudo \  

unshare -i -m -n -p -u -U --mount-proc -f sh  
check namespace  
ip a  
ps xa  
ls -la /  
exit  
mkdir rootfs  
docker run busybox  
docker imasges  
docker ps -a  

docker export  [ImageName] | tar xf - -C rootfs   
ll rootfs  
ll rootfs/bin  
sudo \  
unshare -i -m -n -p -u -U --mount-proc -R rootfs -f sh  

-=Check=-  
ip a  
ps xa  
ls -la /  

exit  
runc spec   
ll  
ll rootfs  

sudo runc run demo  
ip a  
ps xa  
ls -la /  
ls -la /bin  
_  

-change config-  

nano config.json   
- terminal  false  
"sh", "-c", "while true; do { echo -e 'HTTP/1.1 200 0K\n\n Version: 1.0.0'; } | nc-Vlp 8080; done"  

exit session  
- sudo runc kill demo KILL  

-change config-  

nano config.json   
	- "namespaces": network  
	- "path": "/var/run/netns/runc"  

sudo ip netns add runc  
sudo runc run demo  

sudo bash  
brctl addbr runc0  
ip link set runc0 up  
ip addr add 192.168.0.1/24 dev runc0  
ip link add name veth-host type veth peer name veth-guest  
ip link set veth-host up  
brctl addif runc0 veth-host  
ip link set veth-guest netns runc  
ip netns exec runc ip link set veth-guest name eth1  
ip netns exec runc ip addr add 192.168.0.2/24 dev eth1  
ip netns exec runc ip link set eth1 up  
ip netns exec runc ip route add default via 192.168.0.1  
exit  

-test-  
sudo runc run demo  
new terminal  

curl 192.168.0.2:8080  

 https://asciinema.org/a/P6zI6LleMjfjV6oXujxSkhiIi  
 
