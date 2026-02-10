---
title: "Quick start to Ansible Vault"
date: 2020-04-05
categories: 
  - "uncategorised"
tags: 
  - "ansible"
  - "ansible-vault"
  - "free"
---

Start using [Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html#ansible-vault) is pretty easy.

As usual, there is no magic here, you keep the key in mind or a file somewhere.

If you keep it in mind doesn't sound very practical because you are trying to automate tasks without human intervention and either way you don't want to type that long key each time.

_(You can use this method for any kind of file, Ansible or not, for a picture is you like.)_

Now, if you DO want to introduce the pass-key each time, then skip this section.

## Create the file with the key

Just create a file with the pass inside with `vi` or `echo`, like this

  `echo "my_password" >> .vault_pass` 

Even the name of the file can be whatever you want like "cakes", ".boring", etc.

Keep the file hidden in the hard disk using the normal file privileges from Linux. Sounds pretty traditional right. I think is good enough, a weird file somewhere in the disk that only you and root know about.

Next, use it as an environment variable in your user\_id profile $HOME/.profile or system-wide /etc/environment, and add it with:

 `export ANSIBLE_VAULT_PASSWORD_FILE="$HOME/.vault_pass"`

In this way, Ansible will automatically use it to encrypt and decrypt.

## Create, encrypt, view, and edit files

The next steps are very easy:

1.- create an encrypted file

 `ansible-vault create _your_file_` 

2.- encrypt an existing file

 `ansible-vault encrypt _your_file_` 

3.- view an encrypted file

 `ansible-vault view _your_file_` 

4.- edit an encrypted file

  `ansible-vault edit _your_file_` 

That's all.

Refer to Ansible Documentation at [https://docs.ansible.com/ansible/latest/user\_guide/vault.html](https://docs.ansible.com/ansible/latest/user_guide/vault.html)
