# How To Install and Configure Ansible on Ubuntu 20.04
* https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-ansible-on-ubuntu-20-04

```
sudo apt-add-repository ppa:ansible/ansible
```

```


```
sudo apt update
sudo apt install ansible
```

```
ansible --version
```

## Working with Ansible

```
ansible-playbook -i inventory/hostfile file.yml --switch


```
--swich:

-v --> verbose
--tag=tagname --> Run only tagname into tasks/main.yaml
--syntax-check
--skip-tags "tagname1,tagname2, .."


## ANsible ADD-HOC commands:
something that you might type into do something really quick but dont want to save for later 
run command with ansible and widout ansible-playbook 

ansible [option] -m [module] -a "[module option]"
For example
```
ansible -m user -a "name=usertest state present" all
ansible -m user -a "name=usertest state present" 192.168.0.13
## please insert 192.168.0.13 in /etc/ansible/hosts
ansible -m yum -a "name=net-tools state present" all
ansible -m file -a "dest=/opt/file mode=600" all
```

## Default host for ansible
/etc/ansible/hosts
you can add group or hosntane and ip in the file 
After that you can use them in add-hoc command 

what is stete in ansible add-hoc commands?













