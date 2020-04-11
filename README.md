# Ansible UdelaRInterior.users role

> `UdelaRInterior.users` is an [Ansible](http://www.ansible.com) role, forked from [`weareinteractive.users`](https://github.com/weareinteractive/ansible-users), which:
>
> * manges users and groups
> * manages user's private key
> * manages user's authorized keys
> * manages user's password, profile and several settings
> * manages user's groups membership

## Installation

To be able to update later and eventually to modify it, prefer using `requirements.yml` with the git source:

```yaml
- name: users
  src: https://github.com/UdelaRInterior/ansible-users.git
  ```
And then download it with `ansible-galaxy`:

```shell
$ ansible-galaxy install -r requirements.yml -g
```

Using `git`, you'll have to be carefull to folder name :

```shell
$ git clone https://github.com/UdelaRInterior/ansible-users.git users
```

## Dependencies

* Ansible >= 2.4

## Variables

Here is a list of all the default variables for this role, which are also available in `defaults/main.yml`.

```yaml
---
# This role takes advantage of Ansible's user role.
# All user related properties will fall back to Ansible's default values.
# @see http://docs.ansible.com/ansible/user_module.html
#
# users:
# - username: foobar              #(required)
#   name: Foo Bar
#   uid: 1000
#   group: staff                  # the primary group of the user. Defaults to `users_group` variable if defined, else omit (i.e. `username` as primary group)
#   password: xxxxx               # a password, to be used in: user.password | password_hash( 'sha512', user.pwd_salt | default ( users_pwd_salt ) )
#   pwd_salt: yyyy                # a salt value for password hash creation
#   groups: ["adm", "www-data"]   # legacy way to pur users in groups, better now to use `os_group` structure
#   append: no                    #(only append groups, leave others)
#   home_mode: "0750"
#   home_create: yes
#   home: /path/to/user/home
#   home_files:
#     - "/path/to/user/home/.bashrc"
#     - "/path/to/user/home/.bash_profile"
#   system: no
#   authorized_keys:
#     - "xxx"
#     - "{{ lookup('file', '/path/to/id_rsa.pub') }}"
#   authorized_keys_exclusive: yes
#   ssh_key_type: rsa
#   ssh_key_bits: 2048
#   ssh_key_password: ""
#   ssh_key_generate: no
#   ssh_key: "xxx" or "{{ lookup('file', '/path/to/id_rsa') }}"
#   shell: /bin/bash
#   update_password: always      # if omited, update_password will be 'on_create', except if 'users_force_passwords' is defined to 'true'
#
# System groups (don't call it group! it's already used by ansible)
# os_groups:
# - name: superadmin
#   gid: XXXX
#   system: no
#   members:
#   - user1
#   - deploy
#   sodoer: without-password
#
# - name: adminsys
#   gid: YYYY
#   system: no
#   members:
#   - user1
#   - user2
#   - user3
#   - user4
#   sodoer: with-password
#

# list of users to add (see structure above)
users: []
# users default home directory
users_home: /home
# default user's primary group for users
users_group:
# default user's secondary groups (legacy, better to use `os_groups` structure)
users_groups: []
# default user's home directory permissions
users_home_mode: "0755"
# default user's ssh key type
users_ssh_key_type: rsa
# default user's ssh key bits
users_ssh_key_bits: 2048
# default user's setting for authorized keys exclusive (i yes, removes other existing keys)
users_authorized_keys_exclusive: no
# list of users to be removed
users_remove: []

## el "omit" no funciona para la definición del shell, lo que pone el campo a un valor vacío (se debería omitir el parámetro ansible, no solo su valor)
default_shell: /bin/bash

# This variable gives a default value if user.pwd_salt is not defined. 
# It should be overwriten with a vault value in calling playbooks. 
users_pwd_salt: '3T2Luq3TomaUdlhgOg16pn2xzKEiLZva'

# This variables forces to update all available individual passwords
users_force_passwords: false

# This is the name of the file where the password is written in user's account
users_password_file: password.txt
```

## Usage

This is an example playbook:

```yaml
---

- hosts: all
  become: yes
  roles:
  - udelarinterior.users
  vars:
    users:
    - username: foobar
      name: Foo Bar 1
    - username: foobar_authorized_keys
      authorized_keys:
        - "{{ lookup('file', 'tests/id_rsa.pub') }}"
      home_create: yes
    - username: foobar_nohome
      home_create: no
    - username: foobar2
      name: Foo Bar 2
      password: "{{ vault_password_foobar2 }}"
      pwd_salt: "{{ vault_salt_foobar1 }}"        
    - username: foobar3
    - username: foobar_home_mode
      home_mode: "0750"
    - username: foobar_key
      ssh_key: "{{ lookup('file', 'tests/id_rsa') }}"
    - username: foobar_key_generate
      ssh_key_generate: yes
      ssh_key_password: secret
    - username: foobar_system
      system: yes
    - username: foobar_file
      home_files:
      - "tests/.bashrc"

    os_groups:
    - name: admins
      gid: 1010
      system: no
      members:
      - foobar
      - foobar_key
      sodoer: with-password
    - name: staff
      members:
      - foobar
      - foobar2
      - foobar3
      - foobar_home_mode
      - foobar_file

    users_authorized_keys_exclusive: yes
    users_remove: [foobar4]

```

## Contributing
In lieu of a formal style guide, take care to maintain the existing coding style. Add unit tests and examples for any new or changed functionality.

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

*Note: To update the `README.md` file please install and run `ansible-role`:*

```shell
$ gem install ansible-role
$ ansible-role docgen
```

## License
Copyright (c):
- We Are Interactive, under the MIT license.
- UdelaR Interior, under GPL-v3
