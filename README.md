# ansible-ubuntu (vagrant)

Requires Ansible 2.1 or greater.

* 2.1.4 - Updating Java install to work - Oracle JDK via webupd8team is now broken
* 2.1.3 - Adding mongo driver to phpt
* 2.1.2 - Patch for checking on /var/run in php
* 2.1.1 - Do not install mongo as part of php, and some Elasticsearch tweaks
* 2.1.0 - Certbot
* 2.0.2 - Testing this and future versions on *Ubuntu 16*
* 1.7.0 - Testing 1.x versions on *Ubuntu 14*
    * PHP 7

## What is this?

A set of Ansible roles for Ubuntu you can `npm install` locally.

To have Ansible check `node_modules/ansible-ubuntu` for roles add a file called `ansible.cfg` into the root of your project:

```
[defaults]

roles_path = node_modules/ansible-ubuntu/ansible
hash_behaviour = merge
```

Note that the merge hash behavior [allows you to override on value in a dictionary](http://stackoverflow.com/a/25131711/186636).
http://docs.ansible.com/ansible/intro_configuration.html#hash-behaviour

Then create a `Vagrantfile` (you can use `Vagrantfile.sample` to get started).

In your `site.yml` and in your meta dirs you can refer to the `ansible-ubuntu` roles, e.g.:

```
---

- name: Setup Solid Aggregator Stack
  hosts: vagrant
  become: yes
  roles:
    - common/oh-my-zsh
    - common/elasticsearch
```

If you add project level roles into your directory, then they can pull in these general purpose roles as follows:

`project/nginx/tasks/meta/main.yml`

```
---
dependencies:
   - { role: 'common/nginx'}
```

The roles have set defaults for the variables you can modify. Just look in the `defaults` dir of each node.

There is a `provision` binary that is included. `-h` for help.

Once this is setup, you can just `vagrant up`. The initial `vagrant up` will also provision things. Once the box exists
you can `vagrant provision` to reprovision.

This is not meant to be a replacement for [Ansible Galaxy](https://galaxy.ansible.com/). This is just a suite of Ansible roles I find useful for working with Ubuntu.

## Dependencies

You need vagrant and ansible installed:

http://vagrantup.com

Ansible:

```
sudo easy_install pip
sudo pip install ansible
```

Some of these depend on each other

* Certbot
    * Getting certbot to work fully automated is a little tricky, since it requires working http virtual hosts for all domain, and then you have to switch to working https configs.
    * You must fill out `certbot.email`
    * You must fill out `certbot.domains`
    * If you want to run the certbot command yourself, set `certbot.create_certs` to `false`.
* Elasticsearch
    * For ES 5.x, you can adjust memory in `/etc/elasticsearch/jvm.options`: defaults: `-Xms2g` `Xmx2g`

    ```
    elasticsearch:
      bind: 127.0.0.1
      version: 5.4.1
    ```  

* Git (2.3+)
* Grasshopper - sets up [grasshopper-cli](https://github.com/Solid-Interactive/grasshopper-cli)
* Hostname

    ```
    server:
        hostname: "{{ ansible_hostname }}"
    ```

* Java 8
* Kibana4
* MySQL
* Nginx
* Node via Nvm

    ```
    nvm:
        version: v0.31.0
        node_version: v5.0.0
    ```

* Oh My ZSH

    ```
    ohmyzsh:
        theme: avit
    ```

* PHP Fpm
    * define the version you want - all others will be removed
    
        ```
        php:
            version: 7.1
        ```
* ruby (2.2)
* Sharp - with needed libvips install

* Mongo
    * Will skip mongo installation if mongo already installed
    * Single Instance
    * Authorization enabled

        ```
        mongo:
            journaling: "false"
            auth: "enabled"
            root_admin_name: "root"
            root_admin_password: "sample root password"
            backup_name: "backup"
            backup_password: "sample backup password"
            version: "3.4.7" # version has to be 3.0.X, 3.2.X, or 3.4.X
        ```

* Nginx
    * If you want to use the `nginx.conf` from this repo, set `nginx.replace_template` to `True`. The default is `False`
    
        ```
        nginx:
            replace_template: False
        ```
        
* LEMP
* postfix

    ```
    postfix:
        main_mailer_type: Internet Site
        mailname: "{{ ansible_hostname }}"
        protocol: ipv4
    ```

## Trouble shooting

If tasks are timing out due to a slow connection, add:

```
  async: 1800
```

You will have to break apart tasks `with_items` to indvidual ones.

Since this uses node via nvm, you have to source ~/.zshrc (to load nvm) before any node or npm related task.
