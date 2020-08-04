# Redundant Eth 2.0 Staking on a Raspberry Pi Cluster

The purpose of this project is to provide a (relatively) easy way to deploy a fault-tolerant Eth 2.0 staking Kubernetes cluster on Raspberry Pi 4's using Ansible. Familiarity with Kubernetes, or at least a willingness to learn, is recommended so that you can use the `kubectl` commands to troubleshoot your deployments if anything goes sideways.

#### So who exactly is this for...
- People who want to create a cost effective, yet redudant, staking setup
- People who want to learn a bit about Ethereum, Ansible, Kubernetes, and GlusterFS
- People who are just curious to see how Kubernetes can be used to self-heal Ethereum deployments

#### ⚠️ CAUTION ⚠️
**Your Ether is your responsibility.** Reading the code and understanding how this deployment works BEFORE committing any Ether (test or otherwise) will increase your likelyhood of success in troubleshooting. While this project was created for the Medalla testnet, it can easily be adapted for mainnet usage in the future, if you choose to take that risk. It is recommended that for a mainnet deployment you increase storage capacity and invest in redudant backup power supplies, as well as a backup source of internet connection, among other redundancies.

**THIS GUIDE IS PURELY INFORMATIONAL. PLEASE DO YOUR RESEARCH IF YOU WISH TO IMPLEMENT THIS WITH REAL ETHER WHEN PHASE 0 LAUNCHES.**

## Hardware
 - 3 (or more) Raspberry Pi 4s with at least 4GB of RAM. ([Canakit](https://www.canakit.com/raspberry-pi-4-4gb.html))
 - 3 (or more) MicroSD cards ([SanDisk Extreme](https://www.amazon.com/SanDisk-Ultra-microSDXC-Memory-Adapter/dp/B073JWXGNT/))
 - 5-port gigabit unmanaged switch ([Linksys one I bought](https://www.amazon.com/Linksys-Business-LGS105-Unmanaged-Enclosure/dp/B00FV12VSW/))
 - Ethernet cables ([Amazon](https://www.amazon.com/gp/product/B01J8KFTB2/))
 - (Optional - But highly recommended) USB Wall Charger ([Anker](https://www.amazon.com/gp/product/B00P936188/))
    - Also get: USB-A to USB-C cables ([Amazon](https://www.amazon.com/gp/product/B07K24TKL6/))
 - (Optional) Raspberry Pi tower enclosure ([Amazon one with built in fans!](https://www.amazon.com/gp/product/B07MW24S61/))

#### If you are running your own Eth 1.0 node
 - (Optional) 3 (or more) USB 3.0 SSD (external storage for each pi)
 	- Medalla testnet: Does not require a lot of space (50GB per drive should be more than enough)
 	- Mainnet: Will probably require around 1TB per drive when Phase 0 launches

## Raspberry Pi 4 setup

### Flash the OS
- Download the Ubuntu 18.04 64-bit image for Raspberry Pi 4 ([Link](https://ubuntu.com/download/raspberry-pi))
- Download Balena Etcher ([Link](https://www.balena.io/etcher/))
- Open up Balena Etcher and follow the instructions to flash the Ubuntu image to your microSD card

### Enable SSH
- Once flashed, unplug and re-plug the microSD card back into your machine
- Add an emtpy file named `ssh` on the `system-boot` partition

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

✏️ Take note of each IP address, you will need it for the Ansible step later.

### SSH into each Pi and update the password
- From your host machine, open a terminal and type:
  
  `ssh ubuntu@xxx.xxx.x.x` (replace the x's with the IP address of the Pi)
- The default password is `ubuntu`. Upon successfully logging in you will be prompted to update the password.

*Note: You will only be using password login during this step. For the rest of the guide, we will setup SSH keys and disable password login to increase security.*


## Ansible host setup
1. Install [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu)
2. Install a few dependencies for the Ansible modules we are using:
   
   `pip install jmespath`
3. Type this command to see if you have a set of SSH keys already:
   
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

- Edit the `hosts.ini` file. You will need the IP for one main "master" node and the rest will be "workers"
- Edit the file in `group_vars/all.yml` to suit your needs. Head to the `group_vars` directory to see the README for more details

## Generate your validator keys

For ease, we will be leveraging the (Eth2 Launchpad)[https://medalla.launchpad.ethereum.org/] to generate our validator keys and submit our deposits.

Once you've successfully followed all the instructions from the launchpad, you should end up with a directory called `validator_keys` that contains all the information and files you'll need to load into your validator.

Copy the contents of `validator_keys` into **ONLY ONE** of the client folders under `eth2_launchpad_files` in this directory. For example, if you are using the Lighthouse validator, copy the contents into `eth2_launchpad_files/lighthouse/validator_keys`.

**IT IS IMPERATIVE THAT YOU ONLY HAVE ONE COPY OF THE VALIDATOR KEYS IN ONE CLIENT FOLDER. HAVING THE SAME VALIDATOR KEYS IN MULTIPLE CLIENTS WILL CAUSE YOUR DEPOSIT TO BE SLASHED AND YOU WILL LOSE ETHER**

## Run the setup playbook

While at the root level of the repository, run this command:

`ansible-playbook -i hosts.ini playbooks/initial_setup.yml`

You will see the playbook run and you can monitor each step as it completes. The playbook end to end takes about 10 minutes to run.

If you are running into errors, you can run the above command with the `-v`, `-vv`, or `-vvv` command to see more details about each step as it runs.


### Other playbooks

- `remove_gluster_volume.yml`: Remove the mounted glusterfs volume and associated bricks and delete any directories associated with it
- `update_deployment.yml`: Redeploy kubernetes. Run after making any edits to the `group_vars/all.yml` file that may change what services are being run.

