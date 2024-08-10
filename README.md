---
-
  gather_facts: true
  hosts: alies
  become: true

#   roles:
#     # sets up static nfs, snmp and fail2ban + iptables
#     - firewall

  tasks:

    - name: Install EPEL repository
      dnf:
        name: epel-release
        state: present

    - name: Install packages
      dnf:
        name:
          - git
          - rsync
          - curl
          - nano
          - screen
        state: latest

    ###############################################
    #
    #       configure & install nginx
    #
    ###############################################

    - name: Install Nginx mainline repo
      dnf_repository:
        name: nginx
        description: Nginx Mainline Repository
        baseurl: https://nginx.org/packages/centos/$releasever/$basearch/
        gpgcheck: 1
        gpgkey: https://nginx.org/keys/nginx_signing.key
        enabled: 1

    - name: Install Nginx
      dnf:
        name: nginx
        state: latest
        
    - name: Ensure Nginx is started and enabled
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Wait for Nginx to start
      wait_for:
        port: 80
        delay: 10

    - name: Check Nginx version with curl
      command: curl -I 127.0.0.1 | awk '/^Server:/ {print $0}'
      register: curl_result
      failed_when: curl_result.rc != 0

    - name: Print Nginx version
      debug:
        msg: "{{ curl_result.stdout }}"
      when: curl_result.rc == 0