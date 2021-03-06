---
layout: top-level
title: Your First Playbook
prev: '<a href="preparing-a-computer.html">Prev: Preparing A Computer For Ansible</a>'
next: '<a href="organising-your-ansible-files.html">Next: Organising Your Ansible Files</a>'
---

# Your First Playbook

For your first playbook, we're going to automate installing [Curl](http://curl.haxx.se/) onto your computer.  Although it is a simple example, to build it you're going to perform many of the tasks that you need to master to work with Ansible.

Rather than download a ready-made example repo, I want you to create your first playbook by hand, by following the instructions below.  Doing so will help you learn what goes where.  (It's fine to cut and paste to avoid typing errors, btw.)

<div class="callout info" markdown="1">
#### Used Ansible Before?

If you've used Ansible before, this playbook might look a little different to how you currently do things.  Stick with it.  I promise that what I'm showing you is worth your time.

Everything I show you in your first playbook, you'll be able to continue using throughout this book and in all of your playbook repos.  Get into the habit of using these practices straight away.  We'll build on them in each chapter.  You'll build on them yourself after you've finished reading this book.
</div>

## Create Your First Playbook Repo

Throughout these examples, I'm assuming that you're storing your playbook repo in `~/ansible-playbooks`.  If you decide to create it somewhere else, you'll need to adjust the examples accordingly.

### 1. Create The Structure

The first thing you need to do is to create a folder structure to store your playbook in.

<pre>
mkdir ~/ansible-playbooks
cd ~/ansible-playbooks
mkdir -p inventory/group_vars
mkdir -p inventory/host_vars
mkdir plays
mkdir roles
</pre>

<div class="callout info" markdown="1">
#### What This Does

* __inventory__ holds information about the target computers
* __plays__ contains the list of what changes to make to target computers
* __roles__ contains the instructions on how to make changes to target computers

</div>

Save the following into the file `~/ansible-playbooks/ansible.cfg`:

<pre>
[defaults]
hostfile=inventory/
roles_path=roles
</pre>

<div class="callout info" markdown="1">
#### What This Does

__ansible.cfg__ is loaded when you run any Ansible commands:

* __hostfile=inventory__ tells Ansible where to find the Inventory
* __roles_path=roles__ tells Ansible to search the `roles/` folder when looking for a role

If you already know a little bit of Ansible, the __roles_path__ setting might seem to be unnecessary.  It's a little trick that allows you to [run any single play on its own](miscellaneous-tips.html#run_a_single_play_on_a_single_computer).
</div>

### 2. Create The Curl Role

Create the folder where the [tasks](key-concepts.html#tasks) for the role will live:

<pre>
mkdir -p ~/ansible-playbooks/roles/stuartherbert.curl/tasks
</pre>

Save the following into the file `~/ansible-playbooks/roles/stuartherbert.curl/tasks/main.yml`:

<pre>
---
- include: ubuntu_install.yml
  when: "ansible_distribution == 'Ubuntu'"
</pre>

Save the following into the file `~/ansible-playbooks/roles/stuartherbert.curl/tasks/ubuntu_install.yml`:

<pre>
---
- name: "Ubuntu: install Curl"
  apt:
    pkg: curl
    state: latest
  sudo: true
</pre>

<div class="callout info" markdown="1">
#### What This Does

The __stuartherbert.curl__ role tells Ansible how to install Ubuntu's _curl_ package:

* When the __stuart.curl__ role runs, Ansible executes the instructions in `tasks/main.yml` inside the `roles/stuartherbert.curl/` folder.
* `tasks/main.yml` tells Ansible to load the file `tasks/ubuntu_install.yml` if the target computer is an Ubuntu box.  (This creates the space to [add instructions for other operating systems](multiple-operating-systems.html) later).
* `tasks/ubuntu_install.yml` tells Ansible to install Ubuntu's _curl_ package using Ansible's [apt module](http://docs.ansible.com/apt_module.html).
</div>

<div class="callout info" markdown="1">
#### Why Is The Role Called stuartherbert.curl?

[AnsibleWorks Galaxy](http://galaxy.ansibleworks.com) is a registry for downloadable roles.  On there, role names are namespaced by adding the author's GitHub username as the prefix.  My GitHub username is [stuartherbert](https://github.com/stuartherbert/).

Get in the habit of prefixing your roles all the time, even if you do not plan to use third-party roles from AnsibleWorks Galaxy for now.
</div>

### 3. Create The Web-Dev Play

Save the following into the file `~/ansible-playbooks/plays/web-dev.yml`

<pre>
---
- hosts: web-dev
  roles:
  - stuartherbert.curl
</pre>

<div class="callout info" markdown="1">
#### What This Does

`plays/web-dev.yml` tells Ansible to apply the __stuartherbert.curl__ role to every target computer in the __web-dev__ group.
</div>

### 4. Add The Play To The Playbook

Save the following into the file `~/ansible-playbooks/site.yml`

<pre>
---
- include: plays/web-dev.yml
</pre>

<div class="callout info" markdown="1">
#### What This Does

`site.yml` tells Ansible to load the file `plays/web-dev.yml`.

Ansible does not automatically search for plays on disk.  You have to add all your plays to `site.yml` if you want Ansible to load them.
</div>

### 5. Add Your Computer To The Inventory

Save the following into the file `~/ansible-playbooks/inventory/hosts`

<pre>
[web-dev]
localhost
</pre>

<div class="callout info" markdown="1">
#### What This Does

This tells Ansible that __localhost__ is a computer in the group __web-dev__.

Plays like `plays/web-dev.yml` that we created earlier state which group that they should be run against.
</div>

Save the following into the file `~/ansible-playbooks/inventory/host_vars/localhost.yml`

<pre>
---
ansible_connection: local
</pre>

<div class="callout info" markdown="1">
#### What This Does

This tells Ansible that __localhost__ is the same computer that you're running Ansible on, and that it doesn't need to try and login via SSH to run plays on __localhost__.  This makes Ansible a bit faster when you're using it to provision software on your own computer.
</div>

Congratulations.  You've just created your first playbook.

## Run Your First Playbook

Run your first Ansible playbook from the command-line:

<pre>
cd ~/ansible-playbooks
ansible-playbook -K site.yml
</pre>

Ansible will prompt you for your `sudo` password.  Type it in, and press ENTER to continue.

If all has gone well, you will see this on the screen:

<pre>
sudo password:

PLAY [web-dev] ****************************************************************

GATHERING FACTS ***************************************************************
ok: [localhost]

TASK: [stuartherbert.curl | Ubuntu: install Curl] *****************************
changed: [localhost]

PLAY RECAP ********************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0
</pre>

Congratulations.  You've just run your first playbook.

## What Did We Just Do?

We've performed all of the common tasks that you'll do when working with Ansible.

1. Created a folder structure to hold all of our files for Ansible:

   This is something that you normally only do once, when you create a new playbook repo.

1. Created an Ansible role called 'stuartherbert.curl':

   A role is the building block of Ansible.  You create roles to install software, and then pull them into plays to combine them into provisioning instructions.

   In this case, the 'stuartherbert.curl' role uses the [apt module](http://docs.ansible.com/apt_module.html) to install the latest version of the package called 'curl' onto the target computer - if and only if the target computer is running Ubuntu.  If the target computer is running anything else, nothing will happen.

1. Created the 'web-dev' play:

   A play is a list of roles to apply to any target computer listed in the Inventory that matches the `hosts` string at the top of the file.  You create a new play for every different purpose a target computer can have.  In the Inventory, you tell Ansible which plays to run against a target computer; you can run more than one play against the same computer.

   In this case, the 'web-dev' play says to apply the 'stuartherbert.curl' role to every member of the 'web-dev' group in the Inventory.

1. Added the 'web-dev' play to the playbook:

   By convention, the playbook file in Ansible is called `site.yml`, and lives in the top-level folder of your playbook repo.  It contains a list of all of the plays that are available, and you use the Inventory to say which plays should run against which target computers.

1. Added 'localhost' to the Inventory:

   The file `inventory/hosts` is a .ini file that contains a list of which hosts to run which plays against, and the file `inventory/host_vars/localhost.yml` contains a list of variables to use when provisioning the target computer called 'localhost'.

   In this case, we've added 'localhost' to the 'web-dev' group.  Every computer listed in the 'web-dev' group will have the 'web-dev' play run against it.  We've told Ansible to treat 'localhost' as the current computer, so that it doesn't login via SSH to act.

1. Executed the playbook:

   Finally, we've run the playbook, by telling Ansible where our playbook is.  This has executed the 'web-dev' play, which has applied the 'stuartherbert.curl' role to the target computer 'localhost'.

These are the same actions that you will perform over and over as you build your own playbook and inventory.  Working with Ansible is a cycle of creating roles, combining them into plays, updating the Inventory with new plays and target computers, and running Ansible to execute your changes.