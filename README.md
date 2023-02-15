# Jenkins, Ansible and Vagrant
### Vagrantfile
1. First, create a new Vagrantfile with the following contents to define the virtual machine:
    ```sh
    Vagrant.configure("2") do |config|
        config.vm.box = "centos/8"
        config.vm.provider "virtualbox" do |vb|
          vb.memory = "2048"
        end
        config.vm.network "private_network", ip: "192.168.33.10"
        config.vm.provision "ansible" do |ansible|
          ansible.playbook = "playbook.yaml"
          ansible.become = true
        end
      end
    ```
    This will create a new Centos 8 virtual machine with 2GB of RAM, a private IP address of 192.168.33.10, and provision it with Ansible using the playbook.yaml playbook.
### playbook.yaml
2. Next, create a new playbook.yaml playbook with the following contents:
    ```sh
    ---
    - name: Install Jenkins
      hosts: all
      become: true
      tasks:
        - name: Configure repos for Centos
          shell: sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
    
        - name: Configure baseurl for Centos
          shell: sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
           
        - name: Update yum cache
          yum: update_cache=yes
    
        - name: Install fontconfig
          yum:
            name: fontconfig
            state: present
    
        - name: Add Jenkins APT key
          rpm_key:
            key: https://pkg.jenkins.io/redhat-stable/jenkins.io.key
            state: present
    
        - name: Add Jenkins Yum repository
          yum_repository:
            name: jenkins
            description: Jenkins repository
            baseurl: https://pkg.jenkins.io/redhat-stable/
            gpgcheck: yes
            gpgkey: https://pkg.jenkins.io/redhat-stable/jenkins.io.key
            enabled: yes
    
        - name: Install Java
          yum:
            name: java-11-openjdk-headless
            state: present
    
        - name: Install Jenkins
          yum:
            name: jenkins
            state: present
    
        - name: Enable and Start the Jenkins Service
          service:
            name: jenkins
            enabled: yes
            state: started
    
        - name: Jenkins initialAdminPassword
          command: sudo cat /var/lib/jenkins/secrets/initialAdminPassword
          register: command_output
    
        - name: Print initialAdminPassword to console
          debug: 
            msg : "{{command_output.stdout}}"
    ```
    This playbook will:
    - Add the Jenkins APT key to the system
    - Add the Jenkins APT repository to the system
    - Install Jenkins using the yum module
 ###  Create & Provision VM   
3. Finally, run vagrant up to create and provision the virtual machine
this will create a new virtual machine, install Jenkins, and make it available at http://192.168.33.10:8080/. You can access the Jenkins web interface by opening a web browser and navigating to that URL.

## Assumption and Prerequisites 
Assumptions made for this projects.
1. Vagrant is already installed and configured
2. Ansible is also installed on the Host machine
3. The IP configured is not in use by host machine (192.168.33.10)
