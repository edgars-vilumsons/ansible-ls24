# Ansible Homework

## Preparations

### Install required software
- Install Ansible
  On Linux/MacOS install ansible for your OS.
  On Windows install ansible under WSL2 subsystem (e.g. Ubuntu).
- Install docker and docker-compose
- Install Git
- Rsync

### Prepare dev environment

1. Create `docker-compose.yml` file
    ```yml
    version: '3'
    services:
      webserver1:
        image: ilmlv/ubuntu2204-ssh-workspace:1.0.0
        ports:
          - "127.0.0.101:80:80/tcp"
          - "127.0.0.101:22:22/tcp"
          - "127.0.0.101:13371:13371/tcp"

      webserver2:
        image: ilmlv/ubuntu2204-ssh-workspace:1.0.0
        ports:
          - "127.0.0.102:80:80/tcp"
          - "127.0.0.102:22:22/tcp"
          - "127.0.0.102:13371:13371/tcp"
    ```
    Take notice of addresses being used for our containerized webservers `127.0.0.101` and `127.0.0.102`\
    SSH credentials is: `root:password`.

2. Start webservers
   ```
   docker-compose up
   ```
3. Save host fingerprint so ssh connection wouldn't need manual action
   ```bash
   ssh-keyscan -t rsa -H 127.0.0.101 >> ~/.ssh/known_hosts
   ssh-keyscan -t rsa -H 127.0.0.102 >> ~/.ssh/known_hosts
   ```

## Debugging

If having troubles to understand ansible failures you may run it with additional arguments `-v`, `-vv` or `-vvv` for increased log verbosity level.\
If webserver containers break while testing scripts then you may `docker-compose down` and `docker-compose up` again to start with fresh state.

## Task 1
- create ansible project with `ansible.cfg` configured to use current dir inventory file with group "public".
- public inventory group should have 3 hosts: google.com, github.com, youtube.com.
- prepare ansible cli command for pinging all "public" group hosts from your workstation (using ansible local connection).
- also create `README.md` file in project root directory and store ping command example in readme.

Hint: For this task inventory file should be configured to use local connection for public group. Because we want tasks to be performed from your machine without remote ssh connection.
```
[public:vars]
ansible_connection=local
```


## Task 2
- add inventory group "webservers" with two hosts "127.0.0.101" and "127.0.0.102".
- create ansible `playbook.yml` for "webservers" host group containing single role "webserver".
- role "webserver" should have separate `install-nginx.yml` file with tasks for nginx service installation and service start\
  N.B. as for this task we are using containerized environment "ansible.builtin.service" module wont work properly, instead use "ansible.builtin.shell: /etc/init.d/nginx start"
- execute playbook and save cli command example to `README.md` file.
- validate you have access to http://127.0.0.101 and http://127.0.0.102


## Task 3
- create inventory group "webservers" variable `project_dir_path=/var/www`
- for role "webserver" add new task file `install-webapp.yml`:
  - update nginx site webroot directory location to `project_dir_path` variable location and restart service
  - remove old nginx webroot directory `/var/www/html` with all contents
  - generate randomized string and save it to ansible variables.
  - under webserver role, create `index.html.j2` Jinja2 template file.
  - save template generated `index.html` file to `project_dir_path` variable location directory.
- execute playbook and verify that after each playbook run http://127.0.0.101 response changes and matches our template.


## Task 4
- for `playbook.yml` create new role "webserver-proxy":
  - change nginx listening port to 13371/tcp and restart service
  - create Caddyfile template, you can use example below.
  - create task to save template generated file to `/etc/caddy/Caddyfile`.
  - create task to download Caddy binary and run it in background (optional: run as service)
- verify responses of `curl http://127.0.0.101:13371` (nginx) is same as `curl http://127.0.0.101` (caddy)
- verify `curl --head http://127.0.0.101` has response header "Server: Caddy"

Caddyfile template example:
```
:80 {
  reverse_proxy http://127.0.0.1:13371
}
```


## Task 5
Create new ansible playbook `backup.yml` for "webserver" group to execution following tasks:
- for each inventory hostname, initialize Git repository in subfolder of `backups` directory, if not already initialized.
- with Rsync pull content of `project_dir_path` location to `backups` inventory hostname subfolder
- git commit all changes

Hint: this tasks also needs `ansible_connection=local` but this time it can be configured using `delegate_to: localhost` for each task
