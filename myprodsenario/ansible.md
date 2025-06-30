

to change all node dns with ansible and playbook

vi inventory.ini

      [all_nodes]
      zizi1 ansible_host=192.168.1.101 ansible_user=zizi
      zizi2 ansible_host=192.168.1.102 ansible_user=zizi
      zizi3 ansible_host=192.168.1.103 ansible_user=zizi
      zizi4 ansible_host=192.168.1.104 ansible_user=zizi
      zizi5 ansible_host=192.168.1.105 ansible_user=zizi
      zizi6 ansible_host=192.168.1.106 ansible_user=zizi
      
      [exclude_nodes]
      zizi0 ansible_host=192.168.1.100 ansible_user=zizi



vi dns_update.yml
        
        ---
        - name: Update DNS on all nodes except zizi0
          hosts: all_nodes
          become: yes
          tasks:
            - name: Update /etc/resolv.conf
              copy:
                content: |
                  nameserver 8.8.8.8
                  nameserver 127.0.0.1
                  nameserver 78.157.42.100
                  nameserver 78.157.42.101
                dest: /etc/resolv.conf
                owner: root
                group: root
                mode: '0644'
        


copy ssh key to all node and recommanded:

ssh-copy-id zizi@192.168.144.40 to 46

if you want do it with ssh pass :

ansible-playbook -i inventory.ini update_dns.yml --ask-pass --ask-become-pass




      ansible-playbook -i inventory.ini update_dns.yml --limit 'all_nodes:!zizi0'








