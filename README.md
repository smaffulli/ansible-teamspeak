TeamSpeak server
===============

Boot a server on OpenStack cloud and start TeamSpeak collaboration
service.

Requirements
------------

This is a generic role to deploy and run Teamspeak3 server. It is
tested on Ubuntu and Fedora and should run on other, too.

Role Variables
--------------

Specify the version of Teamspeak3 server you want to run modifying the
vars/main.yaml file.

Dependencies
------------

There are no dependencies.

Example Playbook
----------------

An example playbook using the Ansible 2.x OpenStack module


    - hosts: localhost
      connection: local
      vars:
        ansible_python_interpreter: python
        key_name: mykey
        private_key: /home/reed/.ssh/new_id.rsa
      tasks:

        - name: Launch an OpenStack instance
          os_server:
            cloud: dreamcloud
            name: teamspeak-server
            state: present
            image: Ubuntu-14.04
            flavor_ram: 2048
            auto_floating_ip: yes
            key_name: "{{ key_name }}"
            nics:
              - net-name: private-network
          register: teamspeak_server

        - name: get facts about the server (including its public v4 IP address)
          os_server_facts:
            server: teamspeak
          until: teamspeak_server.server.public_v4 != ""
          retries: 5
          delay: 10

        - set_fact: public_v4="{{ teamspeak_server.server.public_v4 }}"

        - name: add the server to our ansible inventory
          add_host: hostname={{ public_v4 }} groups=teamspeak ansible_ssh_user=dhc-user ansible_ssh_private_key_file={{ private_key }}

        - name: wait for ssh to be available
          wait_for: host={{ public_v4 }} port=22 state=started

    - hosts: teamspeak
      sudo: True

      roles:
         - { smaffulli.teamspeak }

License
-------

GPLv3 or any later version.

Author Information
------------------

This role was created by [Stefano Maffulli](http://maffulli.net).
Latest versions, bug reports and pull requests on
[Github](https://github.com/smaffulli).
