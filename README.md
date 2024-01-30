## Module 3 Container Operations in Google Cloud Shell

**Access Google Cloud Shell:**  
[https://shell.cloud.google.com/](https://shell.cloud.google.com/)

**Install Asciinema:**  
`sudo apt-get install asciinema`

**Enter a New Namespace for Isolation:**  
`sudo unshare -i -m -n -p -u -U --mount-proc -f sh`

**Check Namespace:**  
`ip a`  
`ps xa`  
`ls -la /`  
`exit`

**Create a New Root Filesystem Directory:**  
`mkdir rootfs`

**Work with Docker:**  
- Run a BusyBox Docker container: `docker run busybox`  
- List Docker images: `docker images`  
- List all Docker containers: `docker ps -a`  

**Export Docker Image to the New Root Filesystem:**  
`docker export [ImageName] | tar xf - -C rootfs`  
- Check contents: `ll rootfs`  
- Check binary contents: `ll rootfs/bin`

**Enter Isolated Filesystem with Unshare:**  
`sudo unshare -i -m -n -p -u -U --mount-proc -R rootfs -f sh`

**Check Inside the Isolated Environment:**  
`ip a`  
`ps xa`  
`ls -la /`  
`exit`

**Prepare for Running with runc:**  
- Generate a spec file: `runc spec`  
- Check contents: `ll` and `ll rootfs`

**Run with runc:**  
`sudo runc run demo`  
- Inside the container, check: `ip a`, `ps xa`, `ls -la /`, `ls -la /bin`

**Change Configuration in runc:**  
- Edit the configuration file: `nano config.json`  
- Disable the terminal: Set `terminal` to `false`  
- Change command: `"sh", "-c", "while true; do { echo -e 'HTTP/1.1 200 OK\n\n Version: 1.0.0'; } | nc -l -p 8080; done"`

**Exit the Session and Kill the Container:**  
- Exit the editing session  
- Kill the container: `sudo runc kill demo KILL`

**Adjust Network Configuration in runc:**  
- Edit `config.json`: Modify `"namespaces"` to include `network` and set `"path"` to `"/var/run/netns/runc"`
- Add a network namespace: `sudo ip netns add runc`  
- Run the container again: `sudo runc run demo`

**Set Up Network Bridge and Interface:**  
- Enter sudo bash: `sudo bash`  
- Create and configure a bridge:  
  - `brctl addbr runc0`  
  - `ip link set runc0 up`  
  - `ip addr add 192.168.0.1/24 dev runc0`  
- Create a veth pair and attach it to the bridge and namespace:  
  - `ip link add name veth-host type veth peer name veth-guest`  
  - `ip link set veth-host up`  
  - `brctl addif runc0 veth-host`  
  - `ip link set veth-guest netns runc`  
- Configure network settings inside the namespace:  
  - `ip netns exec runc ip link set veth-guest name eth1`  
  - `ip netns exec runc ip addr add 192.168.0.2/24 dev eth1`  
  - `ip netns exec runc ip link set eth1 up`  
  - `ip netns exec runc ip route add default via 192.168.0.1`  
- Exit sudo bash: `exit`

**Test the Configuration:**  
- Run the container: `sudo runc run demo`  
- In a new terminal, test connectivity: `curl 192.168.0.2:8080`

**Record the Session with Asciinema:**  
[https://asciinema.org/a/P6zI6LleMjfjV6oXujxSkhiIi](https://asciinema.org/a/P6zI6LleMjfjV6oXujxSkhiIi)
