---
title: "Simple, complete, free, and automated network configuration backup setup"
date: 2020-03-14
categories: 
  - "uncategorised"
tags: 
  - "ansible"
  - "free"
  - "network-backup"
  - "python"
---

I've been asked about how to automate a network configuration backup and I read a lot of articles on the internet, but I didn't find a simple, quick, complete and free setup.

So, I made this one.

You only need a Linux with Python3 (installed by default) check your version with:

```
$ python3 --version 
```

or easy upgradable with:

```
$ python3 --version 
$ sudo apt-get update
$ sudo apt-get install python3.6
```

and Ansible, very easy to [install](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).

```
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo apt-add-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible
```

Now this setup I made has one master Playbook for:

- Support IOS, ASA, NxOS, and F5
- Keeps 10 historic configurations, with format hostname-date-time
- Only keep the backup if the configuration is different from the previous one
- Send an email or Slack message if the backup fails

Get the code with:

```
$ git clone https://github.com/aegiacometti/netconf-backup 
```

Add your devices to the Ansible `hosts` file, and execute the backup with:

```
$ ansible-playbook ./netconf-backup.yml
```

Optionally, if you want the alerts to be sent when a configuration backup fails, set to **"yes"** the variables `alert_mail` and/or `alert_slack` at the master Playbook `netconfig-backup.yml`. And set your mail details and/or Slack webhook at the playbooks `playbooks/netconfig-backup-send-mail.yml` and/or `playbooks/netconfig-backup-msg-slack.yml`

Add the Playbook execution to `crond` with:

```
$ crontab -e
```

Add a line like:

```
0 0 * * * ansible-playbook ~/your_dir_to/netconf-backup.yml
```

That's it, you have an automated network configuration backup every day at 00:00.

And if something goes wrong you will have an email or a Slack message with the failed backup hostname.

Later with only using the Linux command `diff` you can have the difference between the configuration files.

Now, if you need, you could expose those files with a simple web server or shared folder setup, but please add a user/password login, you don't want the configurations to be public.

### Special SSH connectivity notes

If normal prompt ssh connection don't work, it will not work with Ansible either. So first check the normal ssh connection from command line, and if you have problems, check these two configurations to add to your Linux.

- Depending on the OS of your network devices you might need to enable other SSH parameters. lines with `sudo vi /etc/ssh/ssh_config`.

```
#Legacy changes
 KexAlgorithms diffie-hellman-group1-sha1,curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp5 21,diffie-hellman-group-exchange-sha256,diffie-hellman-group14-sha1
 Ciphers aes128-cbc,aes128-ctr,aes256-ctr
```

- On the Ansible side, analyse the addition of these two parameters in your `.ansible.cfg`.

```
[defaults]
# uncomment this to disable SSH key host checking
host_key_checking = False

[paramiko_connection]
# When using persistent connections with Paramiko, the connection runs in a
# background process.  If the host doesn't already have a valid SSH key, by
# default Ansible will prompt to add the host key.  This will cause 
# connections
# running in background processes to fail.  Uncomment this line to have
# Paramiko automatically add host keys.
host_key_auto_add = True
```

In the next post, I will explore how to use a configuration version system like [Git](https://en.wikipedia.org/wiki/Git) (locally) and [GitHub](https://github.com/) which is the same concept as Git, but in the cloud and with a web GUI of course, this would be the entrance to a bigger and interesting world of possibilities about configuration management.

Cheers.
