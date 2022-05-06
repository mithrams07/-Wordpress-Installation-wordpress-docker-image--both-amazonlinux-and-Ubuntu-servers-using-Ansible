# -Wordpress-Installation (docker image)---both-amazonlinux-and-Ubuntu-servers-using-Ansible
Wordpress Installation (using docker image of wordpress) on both Amazon linux and Ubuntu servers using Ansible

## Ansible playbook
vi main.yml
```
---
- name: "Installing Wordpress on amazonlinux"
  hosts: all
  become: true

  vars:
    user_amazon: ec2-user
    wpnet: word-net
    MYSQL_ROOT_PASS: mysqlroot@123
    MYSQL_DB: wordpress
    MYSQL_USR: wpuser
    MYSQL_PASS: wpuser@123

  tasks:
  
    - name: "Install docker on amazon linux"
      when: ansible_os_family == "RedHat" and ansible_distribution == "Amazon"
      yum:
        name:
          - docker
          - python2-pip
        state: present


    - name: "Installing docker client for python on amazon linux"
      when: ansible_os_family == "RedHat" and ansible_distribution == "Amazon"
      pip:
        name: docker-py
        state: present   


    - name: "Restarting and enabling docker"
      service:
        name: docker
        state: restarted
        enabled: yes
        

    - name: "Adding ec2-user to docker group on amazon linux"
      when: ansible_os_family == "RedHat" and ansible_distribution == "Amazon"
      user:
        name: "{{user_amazon}}"
        groups: docker
        append: yes

    - name : "Creating network for wordpress"
      docker_network:
        name : "{{wpnet}}"


    - name: "Create a database container for wordpress"
      docker_container:
        name: database
        image: mysql:5.7
        volumes:
          - mysql-vol:/var/lib/mysql
        env: 
          MYSQL_ROOT_PASSWORD: "{{MYSQL_ROOT_PASS}}"
          MYSQL_DATABASE: "{{MYSQL_DB}}"
          MYSQL_USER: "{{MYSQL_USR}}"
          MYSQL_PASSWORD: "{{MYSQL_PASS}}"
        networks: 
          - name: "{{wpnet}}"


    - name: "Create a wordpress container"
      docker_container:
        name: wordpress
        image: wordpress
        env: 
          WORDPRESS_DB_HOST: database
          WORDPRESS_DB_USER: "{{MYSQL_USR}}"
          WORDPRESS_DB_PASSWORD: "{{MYSQL_PASS}}"
          WORDPRESS_DB_NAME: "{{MYSQL_DB}}"

        volumes:
          - wp-vol:/var/www/html/
        ports:
          - "80:80"
        networks:
          - name: "{{wpnet}}"
```
        
        
   ## Output wil be as below
         
