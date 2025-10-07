# Deploy_And_Configure_Nginx_Web_Server_Using_Ansible

## Project: Deploy and Configure Nginx Web Server using Ansible

## Task 1: Install and Configure Ansible
On your control node (e.g., Ubuntu):
sudo apt update
sudo apt install ansible -y
ansible --version

- Set up SSH access

Ensure you can SSH into the target servers (e.g., web1) without a password:
ssh-copy-id user@web1

## Task 2: Set Up Ansible Inventory File

Create the file /etc/ansible/hosts or in your project folder (e.g., inventory.ini):
"[web_servers]
web1 ansible_host=192.168.1.10 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa"

Replace the IP, username, and SSH key path with your actual values.

- Verify connection:
ansible all -m ping -i inventory.ini

## Task 3: Create Ansible Playbook to Install Nginx

Create a file called install_nginx.yml:
" ---
- name: Install Nginx on the server
  hosts: web_servers
  become: yes
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Ensure Nginx is running
      service:
        name: nginx
        state: started
        enabled: yes" 

- Run it:
ansible-playbook -i inventory.ini install_nginx.yml

## Task 4: Configure a Custom Nginx Website using Ansible

Create another playbook called configure_website.yml:
" ---
- name: Configure Nginx website
  hosts: web_servers
  become: yes
  tasks:
    - name: Create website root directory
      file:
        path: /var/www/mywebsite
        state: directory
        mode: '0755'

    - name: Deploy HTML content
      copy:
        content: |
          <html>
          <head><title>Welcome to My Website</title></head>
          <body>
          <h1>Hello from Nginx!</h1>
          </body>
          </html>
        dest: /var/www/mywebsite/index.html

    - name: Configure Nginx server block
      copy:
        content: |
          server {
              listen 80;
              server_name _;
              root /var/www/mywebsite;
              index index.html;
              location / {
                  try_files $uri $uri/ =404;
              }
          }
        dest: /etc/nginx/sites-available/mywebsite

    - name: Enable the Nginx server block
      file:
        src: /etc/nginx/sites-available/mywebsite
        dest: /etc/nginx/sites-enabled/mywebsite
        state: link

    - name: Remove default Nginx server block
      file:
        path: /etc/nginx/sites-enabled/default
        state: absent

    - name: Reload Nginx
      service:
        name: nginx
        state: reloaded"

- Run it:
ansible-playbook -i inventory.ini configure_website.yml

## Task 5: Verify Nginx Deployment

After running the playbooks, verify the setup:

On the target server:
sudo systemctl status nginx
ls /var/www/mywebsite/
cat /etc/nginx/sites-enabled/mywebsite

- From your browser or terminal:

Open your server IP in a browser: http://<server_ip>

- You should see:
<html>
<head><title>Welcome to My Website</title></head>
<body>
<h1>Hello from Nginx!</h1>
</body>
</html>

