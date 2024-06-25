# Chef-Project-Nginx-NodeJS-MySQL

Sure! Here are examples of Chef cookbooks to manage different components of a web application stack on Amazon Linux 2023. We will create three cookbooks: one for setting up Nginx as a web server, one for deploying a Node.js application, and one for setting up MySQL as the database server.

### Cookbook 1: Nginx

#### Step 1: Generate the Nginx Cookbook
```bash
chef generate cookbook nginx
```

#### Step 2: Edit the default Recipe (nginx/recipes/default.rb)
```ruby
package 'nginx' do
  action :install
end

service 'nginx' do
  action [:enable, :start]
end

template '/etc/nginx/nginx.conf' do
  source 'nginx.conf.erb'
  notifies :reload, 'service[nginx]', :delayed
end
```

#### Step 3: Create a Template for Nginx Configuration (nginx/templates/default/nginx.conf.erb)
```erb
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        location / {
            proxy_pass http://localhost:3000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

#### Step 4: Upload the Nginx Cookbook to the Chef Server
```bash
knife cookbook upload nginx
```

### Cookbook 2: Node.js Application

#### Step 1: Generate the Node.js Application Cookbook
```bash
chef generate cookbook myapp
```

#### Step 2: Edit the default Recipe (myapp/recipes/default.rb)
```ruby
package 'git'
package 'nodejs'
package 'npm'

git '/srv/myapp' do
  repository 'https://github.com/your-repo/your-app.git'
  revision 'master'
  action :sync
end

execute 'install_dependencies' do
  command 'npm install'
  cwd '/srv/myapp'
  action :run
end

execute 'start_app' do
  command 'npm start &'
  cwd '/srv/myapp'
  action :run
end
```

#### Step 3: Upload the Node.js Application Cookbook to the Chef Server
```bash
knife cookbook upload myapp
```

### Cookbook 3: MySQL

#### Step 1: Generate the MySQL Cookbook
```bash
chef generate cookbook mysql
```

#### Step 2: Edit the default Recipe (mysql/recipes/default.rb)
```ruby
package 'mysql-server' do
  action :install
end

service 'mysqld' do
  action [:enable, :start]
end

execute 'create_database' do
  command 'mysql -e "CREATE DATABASE myapp;"'
  not_if 'mysql -e "SHOW DATABASES LIKE \'myapp\';"'
end
```

#### Step 3: Upload the MySQL Cookbook to the Chef Server
```bash
knife cookbook upload mysql
```

### Running the Cookbooks on a Node

#### Step 1: Bootstrap the Node
```bash
knife bootstrap NODE_IP -x ec2-user -i /path/to/key.pem --sudo --node-name NODE_NAME
```

#### Step 2: Assign the Cookbooks to the Node's Run List
```bash
knife node run_list add NODE_NAME 'recipe[nginx]'
knife node run_list add NODE_NAME 'recipe[myapp]'
knife node run_list add NODE_NAME 'recipe[mysql]'
```

#### Step 3: Run Chef Client on the Node
```bash
knife ssh "name:NODE_NAME" "sudo chef-client" -x ec2-user -i /path/to/key.pem
```

### Testing with Test Kitchen

#### Step 1: Install Test Kitchen
```bash
chef gem install kitchen
chef gem install kitchen-vagrant
chef gem install kitchen-inspec
```

#### Step 2: Create a `.kitchen.yml` File
```yaml
---
driver:
  name: vagrant

provisioner:
  name: chef_zero

platforms:
  - name: amazonlinux-2

suites:
  - name: default
    run_list:
      - recipe[nginx::default]
      - recipe[myapp::default]
      - recipe[mysql::default]
    attributes:
```

#### Step 3: Run Test Kitchen
```bash
kitchen converge
kitchen verify
kitchen destroy
```

### Integration with Jenkins Pipeline

#### Step 1: Create a Jenkinsfile
```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your-repo/your-cookbook.git'
            }
        }
        stage('Test') {
            steps {
                sh 'kitchen test'
            }
        }
        stage('Deploy') {
            steps {
                sh 'knife cookbook upload your-cookbook'
                sh 'knife ssh "name:NODE_NAME" "sudo chef-client"'
            }
        }
    }
}
```

### Monitoring and Reporting

#### Step 1: Install and Configure Chef Automate
Follow the [Chef Automate installation guide](https://docs.chef.io/automate/install/).

#### Step 2: Configure Reporting
```bash
chef-server-ctl enable-reporting
```

#### Step 3: Set Up Node Monitoring
Use tools like InSpec to write compliance profiles and run them on nodes.

### Summary

This project involves generating and uploading cookbooks for Nginx, a Node.js application, and MySQL. We bootstrap a node, assign the cookbooks to the node's run list, and run the Chef client on the node. We also set up testing with Test Kitchen and integrate the deployment process with Jenkins for CI/CD. Finally, we enable reporting and monitoring with Chef Automate. This setup ensures that your web application stack on Amazon Linux 2023 is automated, tested, and continuously integrated.
