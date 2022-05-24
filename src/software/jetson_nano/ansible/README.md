
# Ansible & Systemd User Guide

Ansible allows us to run actions on multiple robots at once. Actions are communicated through YAML files called playbooks.
For a more detailed look at how Ansible works, [see the RFC](https://docs.google.com/document/d/1hN3Us2Vjr8z6ihqUVp_3L7rrjKc-EZ-l2hZJc31gNOc/edit)

Systemd is a low level service that manages the automattic running of processes on a single robot. We also use it enable remote flashing. To learn more about how it works, [see the RFC](https://docs.google.com/document/d/1hN3Us2Vjr8z6ihqUVp_3L7rrjKc-EZ-l2hZJc31gNOc/edit)



### Ansible Setup
Ansible is integrated in our Bazel build environment. All playbooks must be added to the `src/jetson_nano/ansible/playbooks` directory. 
Before running any command, ensure that: 

1) all hosts (robots/jetsons) and the control node (machine that is running ansible) are on the same network. 

2) The ip addresses for all robots/jetsons are known. If you are unsure of the IP Addresses, connect to the tbots router and go to 192.168.0.1. Look under basic/wireless clients to find the jetson IP addresses. The jetsons should be called "robot". 
If you don't see the Jetsons, connect the Jetson to the router with an ethernet cable and check 192.168.0.1 again. The jetsons should be under wired clients. With a wired connection, you can ssh and use nmtui to connect it to the wifi. 

3) all hosts have installed the necessary dependencies and have setup systemd. This can be done by running the `setup_nano.sh` script through Ansible via the command 
`bazel run :run_ansible --cpu=jetson_nano -- -pb setup_nano.yml -ho {ip addresses of hosts} -pwd {robot_ssh_password} `

Notes: 
- two tags (dependencies, systemd) can be applied to specify what kind of setup to perform. 
- Output of the dependency setup script is streamed to the robot's `/tmp/setup.log` file

### How To Run an Ansible Playbook

To run a playbook called `playbook.yml`, use the command 
``bazel run :run_ansible --cpu=jetson_nano -- -pb playbook.yml -ho [list of ip addrs] -pwd {ssh_pass}`` 

If desired, the `-ho` argument can be replaced with `-p`, defining a port to listen to for Announcements from hosts. 

Different arguments might need to be added to support different scenarios (ex tags, list of hosts). To view a list of supported arguments, run 
``bazel run :run_ansible --cpu=jetson_nano -- -h`` 


### Remote Flashing

The `remote_flash.yml` playbook stops services, syncs new binaries to hosts and restarts services. The "--tags" argument can be used to specify which actions to perform and on which services. 
Possible service tags: thunderloop, announcement, display, redis. 
Possible action tags: stop, sync, start

if no service/action tags are specified, all service/actions will be used. 

For example, to remote flash (ie stop+sync+start) thunderloop and redis, run the command: 
 ``bazel run :run_ansible --cpu=jetson_nano -- -pb remote_flash.yml -ho [list of ip addrs] -pwd {ssh_pass} -t thunderloop redis`` 
 
To just stop the thunderloop and redis services, use: 
 ``bazel run :run_ansible --cpu=jetson_nano -- -pb remote_flash.yml -ho [list of ip addrs] -pwd {ssh_pass} -t thunderloop redis stop`` 

**warning** : the 'sync' action involves replacing a service's binary file, which might not work if the service is running at the time of replacement. Thus, it is best not to run the sync action unless accompanied with a stop action.

### Miscellaneous Tasks
Individual miscellaneous tasks (ex reboot, shutdown, rtt test) can be run through the `misc.yml` playbook by specifying the corresponding tag. 