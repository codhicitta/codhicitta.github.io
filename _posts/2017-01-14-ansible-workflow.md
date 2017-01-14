---
layout: post
title: "Ansible Workflow"
date: 2017-01-14 13:51:00 -0500
categories: Ansible DevOps
---
Here's how I get things done with Ansible. As in all things, YMMV.

## Project Workflow
When starting a new project, I start by cloning my [anseedble project](https://github.com/dheles/anseedble). This gives me a good starting point <sup id="a1">[1](#f1)</sup> to create a VM in Vagrant with the basic prereqs needed to install and configure things with Ansible.

    cd ~/Source
    git clone https://github.com/dheles/anseedble my-project

Since I’m not actually planning to add to anseedble, I need to break off its git-lationship <sup id="a2">[2](#f2)</sup> and start a new one:

    cd my-project
    rm -rf .git
    git init

Then I do at least the minimum configuration to give the project its own identity:

Edit README.md (...obvi)

Set some vars <sup id="a3">[3](#f3)</sup>:

    # inventory/group_vars/all/vars.yml
    project:        "my-project"
    version:        "0.1.0"

and <sup id="a4">[4](#f4)</sup>:

    # inventory/group_vars/dev/vault.yml
    vault_login_user_passphrase

And then I create a new repo for the project on GitHub (using the buttons, because easy). GitHub gives me that handy snippet to associate my project with the new repo. It's so handy, I can never remember it; so to help with that (maybe), here it is:

    git remote add origin https://github.com/dheles/ansible-role-my-role.git
    git push -u origin master

Then there's more of the

    hack hack hack

-ing that often takes the form of my:

## Role Workflow
Previously, I started by creating a playbook, with the idea of refactoring it into a role at some point down the line. Now, I have come to favor starting a piece of work as a role whenever possible. Obviously, there will still be some tasks which will be best left as playbooks, but I think these will be the exception than the rule. Part of the reason for this is that I found myself creating a lot of file and folder relationships to support the playbooks that will come for free to roles. Add to this the reusability of roles (for myself, as well as others) & this seems like a clear win to me.

So, to create a new role:

    cd roles
    ansible-galaxy init my-role
    cd my-role
    git init

Wait. git init? What?

I .gitignore the roles folder in anseedble (and any derivative projects).<sup id="a5">[5](#f5)</sup> So, after init-ing it, I can manage the role as its own project. So, I set it up and then create and link up a GitHub repo for it (much as described above). Following [Jeff Geerling's](https://github.com/geerlingguy) <sup id="a6">[6](#f6)</sup> lead, I name my ansible role projects in GitHub like, "ansible-role-my-role," so as to keep them easily identifiable. Within the projects where I use the role, I drop the prefix, so it'll just be "my-role."

So, add it to a playbook in the usual way:

    - name: use my role
      hosts: all
      become: true

      roles:
      - { role: my-role }

Then iterate over it til it works satisfactorally. Then, check it in to GitHub. <sup id="a7">[7](#f7)</sup>

OK. **Hold up a sec.** Once you do what we're about to do, **your local copy of your role will get overwritten**, so do please make sure your work has been checked in first.

So, after your work has been checked in properly, <sup id="a8">[8](#f8)</sup> add it to requirements.yml:

    - src: https://github.com/dheles/ansible-role-my-role
    name: my-role

If this is your first role in this project, and you're using Vagrant, you may need to comment in the following line in the Vagrantfile:

    ansible.galaxy_role_file = "requirements.yml"

Otherwise, install your role manually:

    ansible-galaxy install -r requirements.yml

### Revisions
Sooner or later, I realize a role I am using in a project is in need of revision. This is particularly true, since I tend to start out with just-what-I-need, rather than building in lots of options and configurability at the outset. When it happens, I simply replace the installed version with a clone of the role's repo from GitHub, make my changes, check in, and reinstall. Like so:

    cd roles
    rm -rf my-role
    git clone https://github.com/dheles/ansible-role-my-role my-role
    cd my-role

Again, its important to remember to comment out the role in requirements.yml, to avoid overwriting changes.

## Notes
<strong id="f1">1</strong> An Ansible seed, if you will. Geddit? An-<em>seed</em>-ble? Ok, you can go now: [↩](#a1)

<strong id="f2">2</strong> Not an actual word, BTW. I just made that up. [↩](#a2)

<strong id="f3">3</strong> "project" is used by my [login-user role](https://github.com/dheles/ansible-role-login-user), which is included in anseedble. "version" just seems like a good idea. [↩](#a3)

<strong id="f4">4</strong> As per Ansible's [best practices](https://docs.ansible.com/ansible/playbooks_best_practices.html#variables-and-vaults) [↩](#a4)

<strong id="f5">5</strong> Except any roles prefixed "local." As the workflow I'm now using (described here) does not take advantage of this exception, maybe it is worth dropping it. [↩](#a5)

<strong id="f6">6</strong> I highly recommend his book, [Ansible for DevOps](https://leanpub.com/ansible-for-devops). [↩](#a6)

<strong id="f7">7</strong> And possibly [Ansible Galaxy](https://galaxy.ansible.com/), though I have yet to do so, myself. [↩](#a7)

<strong id="f8">8</strong> Do I sound like someone who may once <sup id="a9">[9](#f9)</sup> <sup id="a10">[10](#f10)</sup> have had bite marks on his posterior that were a positive match for his own dental records? Yes, I suppose I do. [↩](#a8)

<strong id="f9">9</strong> Or more than once, perhaps. I can admit it. We're among friends here, right? [↩](#a9)

<strong id="f10">10</strong> Does that footnote have a footnote? In a *wiki*? No. Of course not.
...
It has two. [↩](#a10)
