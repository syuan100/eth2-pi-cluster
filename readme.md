# Redundant Eth 2.0 Staking on a Raspberry Pi Cluster

The purpose of this project is to provide a (relatively) easy way to deploy a fault-tolerant Eth 2.0 staking cluster on Raspberry Pi 4's using Ansible. Familiarity with Kubernetes, or at least a willingness to learn, is recommended so that you can use the `kubectl` commands to troubleshoot your deployments if anything goes sideways.

#### What does this project contain?
- Full hardware setup instructions for a Raspberry Pi cluster
- Single-line command for a self-healing deployment of geth, beacon, and validator nodes
- A single variables file `group_vars/all.yml` that controls your deployment
- Additional playbooks to update your deployment

#### Work in progress
- Adding additional clients
  - [x] Lighthouse
  - [ ] Prysm
  - [x] Teku
  - [ ] Nimbux
  - [ ] Lodestar (?)
- Logging and monitoring
- Including a setting for POAP [`graffiti`](https://beaconcha.in/poap)

#### Potential improvements
- Other distributed storage options other than GlusterFS
- Triggering backup deployments to the cloud

#### Open source to the core

Please feel free to fork, make PRs, and open issues! This is all done completely on the side in my free time as a way to benefit the Ethereum community, so I may not have the velocity to make changes quickly.

---

#### So who exactly is this for...
- People who want to create a cost effective, yet redudant, staking setup
- People who want to learn a bit about Ethereum, Ansible, Kubernetes, and GlusterFS
- People who are just curious to see how Kubernetes can be used to self-heal Ethereum deployments

#### ⚠️ CAUTION ⚠️
**Your Ether is your responsibility.** Reading the code and understanding how this deployment works BEFORE committing any Ether (test or otherwise) will increase your likelyhood of success in troubleshooting. While this project was created for the Medalla testnet, it can easily be adapted for mainnet usage in the future, if you choose to take that risk. It is recommended that for a mainnet deployment you increase storage capacity and invest in redudant backup power supplies, as well as a backup source of internet connection, among other redundancies.

**THIS GUIDE IS PURELY INFORMATIONAL. PLEASE DO YOUR RESEARCH IF YOU WISH TO IMPLEMENT THIS WITH REAL ETHER WHEN PHASE 0 LAUNCHES.**

#### A small shout out to the pre-made ansible playbooks that I used:

- [Ubuntu Ansible playbooks by Digital Ocean](https://github.com/do-community/ansible-playbooks)
- [k3s-ansible by Rancher](https://github.com/rancher/k3s-ansible)
- [gluster-ansible by Gluster](https://github.com/gluster/gluster-ansible)

## Hardware
 - 3+ Raspberry Pi 4s with at least 4GB of RAM. ([Canakit](https://www.canakit.com/raspberry-pi-4-4gb.html))
 - 3+ MicroSD cards ([SanDisk Extreme](https://www.amazon.com/SanDisk-Ultra-microSDXC-Memory-Adapter/dp/B073JWXGNT/))
 - 5-port gigabit unmanaged switch ([Linksys one I bought](https://www.amazon.com/Linksys-Business-LGS105-Unmanaged-Enclosure/dp/B00FV12VSW/))
 - Ethernet cables ([Amazon](https://www.amazon.com/gp/product/B01J8KFTB2/))
  - (Optional - But highly recommended) 3+ USB 3.0 SSD (external storage for each pi)
 	- Medalla testnet: Does not require a lot of space (50GB per drive should be more than enough)
 	- Mainnet: Will probably require at least 1TB per drive when Phase 0 launches
 - (Optional - But highly recommended) USB Wall Charger ([Anker](https://www.amazon.com/gp/product/B00P936188/))
    - Also get: USB-A to USB-C cables ([Amazon](https://www.amazon.com/gp/product/B07K24TKL6/))
 - (Optional) Raspberry Pi tower enclosure ([Amazon one with built in fans!](https://www.amazon.com/gp/product/B07MW24S61/))

## Raspberry Pi 4 setup

### Flash the OS
- Download the Ubuntu 18.04 64-bit image for Raspberry Pi 4 ([Link](https://ubuntu.com/download/raspberry-pi))
- Download Balena Etcher ([Link](https://www.balena.io/etcher/))
- Open up Balena Etcher and follow the instructions to flash the Ubuntu image to your microSD card

### Enable SSH
- Once flashed, unplug and re-plug the microSD card back into your machine
- Add an emtpy file named `ssh` on the `system-boot` partition (Note: Some images may already have the file there upon flashing)

You can now insert the microSD card into the Raspberry Pis.

### Connect everyting to your network

Your setup should look something like this:

[ Modem * ]---[ Router * ]---[ Switch ]---[ Raspberry Pis ]

_* You should already have these_

#### ⚠️ SECURTIY WARNING ⚠️
**This guide assumes you have taken all the proper and necessary steps to secure your network traffic through the router. IMPROPER CONFIGURATION OR CARELESS PORT FORWARDING/OPENING CAN RESULT IN YOUR DEVICES GETTING HACKED AND POTENTIALLY LOSING YOUR STAKED ETHER.**

Check out the page below for tips on how to test your router security
[https://www.routersecurity.org/testrouter.php](https://www.routersecurity.org/testrouter.php)

### Write down the local IP address for each Pi
Once you've properly connected everyting and the Raspberry Pis have completed booting, log into your router to see what each Pi's IP address is.

You may want to setup IP reservations for each Pi so that you don't need to check everytime you reboot or unplug your Pi. Most routers support this through their UI, or [read this article for general instructions on how to do that](https://lifehacker.com/how-to-set-up-dhcp-reservations-and-never-check-an-ip-5822605).

✏️ Take note of each IP address, you will need it for the Ansible step later.

### SSH setup
1. From your host machine, open a terminal and type:
  
  `ssh ubuntu@xxx.xxx.x.x` (replace the x's with the IP address of the Pi)
2. The default password is `ubuntu`. Upon successfully logging in you will be prompted to update the password.

  *Note: You will only be using password login during this step. For the rest of the guide, we will setup SSH keys and disable password login to increase security.*
3. Go back to your host machine and type this command to see if you have a set of SSH keys already:
   
   `ls -al ~/.ssh` 
   
   If you see something like `id_rsa.pub` as one of the files then you can skip step 4.
4. If you don't have an SSH key already for your host machine, generate an SSH key:
    
    `ssh-keygen`
    
    Follow the instructions. Note: This process will be faster if you don't use a passphrase.
    - If you DO use a passphrase on your key, follow the instructions [here](https://docs.ansible.com/ansible/latest/user_guide/connection_details.html#ssh-key-setup) for how to use `ssh-agent` to store your passphrase.
5. Copy the SSH keys to your Raspberry Pis:
   
   `ssh-copy-id ubuntu@xxx.xxx.x.x`
   
   Once your keys are successfully copied, you should be able to login without using the password:
   - Test it out! `ssh ubuntu@xxx.xxx.x.x`


## Ansible setup
1. Install [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu)
2. Install a few dependencies for the Ansible modules we are using:
   
   `pip install jmespath`

## Replicated storage setup
*Note: Attaching a USB stick or SSD drive to your Raspberry Pi will be easier and more performant than partitioning a portion of your SD card.*

1. Attach your SSD drive to the upper blue USB 3.0 port of each Pi
2. `ssh` into your Pis and check what the device path is by running this command:
   
   `lsblk`
   
   You should see an output similar to this:
   
   ```
   NAME                             MAJ:MIN   RM   SIZE RO TYPE MOUNTPOINT
    sda                                 8:0    1  58.4G  0 disk
    └─sda1                              8:1    1  58.4G  0 part
    mmcblk0                           179:0    0 238.3G  0 disk
    ├─mmcblk0p1                       179:1    0   256M  0 part /boot/firmware
    └─mmcblk0p2                       179:2    0   238G  0 part /
   ```
   
   On *most* Pis, your USB SSD will show up as `sda1`. This means your device path will be `/dev/sda1`. If you already have a USB port occupied, it's possible that your path may be `sdb1` or `sdc1`. The important part is that it follows the convention of `sdx#`.
   
   ✏️Take note of the device path, you'll need it for the Ansible step later.


## Ansible playbook setup

- In the repository, edit the `hosts.ini` file. You will need an IP for one main "master" Pi node and the rest will be "worker" node Pis

  ```
  [master]
  ; Put the IP address for your main Kubernetes server
  192.168.xx.xxx

  [nodes]
  ; Put the IP addresses for the rest of the nodes
  192.168.xx.xxx
  192.168.xx.xxx
  192.168.xx.xxx
  ```

- Edit the file in `group_vars/all.yml` to suit your needs. Head to the `group_vars` directory to see the README for more details

## Generate your validator keys

For ease, we will be leveraging the [Eth2 Launchpad](https://medalla.launchpad.ethereum.org/) to generate our validator keys and submit our deposits.

Once you've successfully followed all the instructions from the launchpad, you should end up with a directory called `validator_keys` that contains all the information and files you'll need to load into your validator.

Copy the contents of `validator_keys` into **ONLY ONE** of the client folders under `eth2_launchpad_files`. For example, if you are using the Lighthouse validator, copy the contents into `eth2_launchpad_files/lighthouse/validator_keys`.

**IT IS IMPERATIVE THAT YOU ONLY HAVE ONE COPY OF THE VALIDATOR KEYS IN ONE CLIENT FOLDER. HAVING THE SAME VALIDATOR KEYS IN MULTIPLE CLIENTS WILL CAUSE YOUR DEPOSIT TO BE SLASHED AND YOU WILL LOSE ETHER.**

## Run the setup playbook

While at the root level of the repository, run this command:

`ansible-playbook -i hosts.ini playbooks/initial_setup.yml`

You will see the playbook run and you can monitor each step as it completes. The playbook end to end takes about 10 minutes to run.

If you are running into errors, you can run the above command with the `-v`, `-vv`, or `-vvv` flag to see more details about each step as it runs.

If you've deployed successfully, SSH into your master node and run this command:

`sudo kubectl get all -o wide`

You should see something like this:

```
NAME                                        READY   STATUS    RESTARTS   AGE     IP            NODE     NOMINATED NODE   READINESS GATES
pod/svclb-geth-4m6vk                        2/2     Running   0          4h9m    10.42.0.32    rpi-01   <none>           <none>
pod/svclb-lighthouse-beacon-kdr5p           3/3     Running   0          4h9m    10.42.0.33    rpi-01   <none>           <none>
pod/svclb-geth-udp-cr8ns                    1/1     Running   0          3h58m   10.42.0.34    rpi-01   <none>           <none>
pod/svclb-lighthouse-beacon-udp-jbx5k       1/1     Running   0          3h58m   10.42.0.35    rpi-01   <none>           <none>
pod/svclb-lighthouse-beacon-udp-7m7hx       1/1     Running   0          3h58m   10.42.1.177   rpi-02   <none>           <none>
pod/svclb-geth-udp-hdh7v                    1/1     Running   0          3h58m   10.42.1.176   rpi-02   <none>           <none>
pod/svclb-geth-mdszx                        2/2     Running   0          4h9m    10.42.1.172   rpi-02   <none>           <none>
pod/svclb-lighthouse-beacon-ffrkv           3/3     Running   0          4h9m    10.42.1.173   rpi-02   <none>           <none>
pod/svclb-lighthouse-beacon-4r8wm           3/3     Running   0          4h9m    10.42.2.189   rpi-03   <none>           <none>
pod/svclb-geth-udp-w6h2j                    1/1     Running   0          3h58m   10.42.2.192   rpi-03   <none>           <none>
pod/svclb-geth-2tn9x                        2/2     Running   0          4h9m    10.42.2.188   rpi-03   <none>           <none>
pod/svclb-lighthouse-beacon-udp-659f4       1/1     Running   0          3h58m   10.42.2.193   rpi-03   <none>           <none>
pod/lighthouse-validator-856c4df647-hlrnf   1/1     Running   0          3h32m   10.42.1.178   rpi-02   <none>           <none>
pod/geth-6c88bff8d8-js7fp                   1/1     Running   0          3h32m   10.42.2.194   rpi-03   <none>           <none>
pod/svclb-geth-udp-s2dtq                    1/1     Running   0          3h58m   10.42.3.244   rpi-04   <none>           <none>
pod/svclb-geth-2wwb2                        2/2     Running   0          4h9m    10.42.3.239   rpi-04   <none>           <none>
pod/svclb-lighthouse-beacon-twq22           3/3     Running   0          4h9m    10.42.3.240   rpi-04   <none>           <none>
pod/svclb-lighthouse-beacon-udp-8dn5s       1/1     Running   0          3h58m   10.42.3.245   rpi-04   <none>           <none>
pod/lighthouse-beacon-59b7c579ff-n7bwr      1/1     Running   0          3h25m   10.42.1.179   rpi-02   <none>           <none>

NAME                            TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                                        AGE     SELECTOR
service/kubernetes              ClusterIP      10.43.0.1       <none>           443/TCP                                        7d7h    <none>
service/lighthouse-beacon       LoadBalancer   10.43.205.225   192.168.xx.xxx   5052:30452/TCP,5053:32564/TCP,9000:30583/TCP   4h9m    app=lighthouse-beacon
service/geth                    LoadBalancer   10.43.40.149    192.168.xx.xxx   8545:31424/TCP,30303:30263/TCP                 4h9m    app=geth
service/geth-udp                LoadBalancer   10.43.88.38     192.168.xx.xxx   30303:30831/UDP                                3h58m   app=geth
service/lighthouse-beacon-udp   LoadBalancer   10.43.133.3     <pending>        9000:31752/UDP                                 3h58m   app=lighthouse-beacon

NAME                                         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE     CONTAINERS                               IMAGES                                                                          SELECTOR
daemonset.apps/svclb-geth-udp                4         4         4       4            4           <none>          3h58m   lb-port-30303                            rancher/klipper-lb:v0.1.2                                                       app=svclb-geth-udp
daemonset.apps/svclb-geth                    4         4         4       4            4           <none>          4h9m    lb-port-8545,lb-port-30303               rancher/klipper-lb:v0.1.2,rancher/klipper-lb:v0.1.2                             app=svclb-geth
daemonset.apps/svclb-lighthouse-beacon       4         4         4       4            4           <none>          4h9m    lb-port-5052,lb-port-5053,lb-port-9000   rancher/klipper-lb:v0.1.2,rancher/klipper-lb:v0.1.2,rancher/klipper-lb:v0.1.2   app=svclb-lighthouse-beacon
daemonset.apps/svclb-lighthouse-beacon-udp   4         4         4       4            4           <none>          3h58m   lb-port-9000                             rancher/klipper-lb:v0.1.2                                                       app=svclb-lighthouse-beacon-udp

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS             IMAGES                              SELECTOR
deployment.apps/lighthouse-validator   1/1     1            1           20h   lighthouse-validator   syuan100/lighthouse:latest_arm64    app=lighthouse-validator
deployment.apps/geth                   1/1     1            1           8h    geth                   syuan100/go-ethereum:1.9.16-arm64   app=geth
deployment.apps/lighthouse-beacon      1/1     1            1           20h   lighthouse-beacon      syuan100/lighthouse:latest_arm64    app=lighthouse-beacon
```

It's normal that some containers may crash and restart at the beginning of the deployment. Give it a few minutes to properly deploy.

### Useful `kubectl` commands

#### Check logs for each deployment

`sudo kubectl logs -l app=app-name-here -f --since=10m`

Replace `app-name-here` with the appropriate app label for the deployment you want to inspect (i.e. `geth`, `lighthouse-beacon`, etc)

#### Execute commands in a container

`sudo kubectl -it exec lighthouse-beacon-59b7c579ff-n7bwr sh`

Replace `lighthouse-beacon-59b7c579ff-n7bwr` with whatever container you wish to login to. You can also replace `sh` with `/bin/bash` if you prefer.


### Other playbooks

Run all of the below with `ansible-playbook -i hosts.ini`

- `remove_gluster_volume.yml`: Remove the mounted glusterfs volume and associated bricks and delete any directories associated with it.
- `update_deployment.yml`: Redeploy kubernetes. Run after making any edits to the `group_vars/all.yml` file that may change how services are being run.
