# 虚拟机主要用于在centos机器上跑一些代码，不强求与线上版本一致
# 
# 统计一下需要ssh登录上去手动修改的部分
# 1. 关闭selinux
# 2. 修改mysql的root密码
# 
# 
Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"

  config.vm.box_check_update = false

  # vbguest配置
  config.vbguest.auto_update = false

  # 代理端口
  config.vm.network "private_network", type: "dhcp"
  config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1" # 用于nginx的后端
  config.vm.network "forwarded_port", guest: 3000, host: 8081, host_ip: "127.0.0.1" # 可以用于node服务端开发

  # 共享目录
  config.vm.synced_folder ".", "/vagrant"
  config.vm.synced_folder "/files/SyncedFoldersVagrant", "/data/www"
  config.vm.synced_folder "/files/express", "/data/express"
  # config.vm.synced_folder "./config/nginx/conf.d", "/etc/nginx/conf.d", type: "virtualbox"


  # VirtaulBox相关配置
  config.vm.provider "virtualbox" do |vb|
    vb.name = "kickflip"
    vb.cpus = 1
    vb.memory = "2048"
  end

  config.vm.provision "shell", inline: <<-SHELL
    sudo yum update -y
    # 安装虚拟增强工具，为了共享文件夹
    # 在virtualbox中将增强工具iso加入虚拟光驱
    sudo mount /dev/cdrom /media
    sudo yum install kernel-devel gcc
    cd /media
    sudo ./VBoxLinuxAdditions.run
    sudo yum install kernel-devel kernel-devel-3.10.0-693.11.6.el7.x86_64
    sudo ./VBoxLinuxAdditions.run
    # hand vagrant reload
    sudo mount -t vboxsf -o uid=1000,gid=1000 vagrant /vagrant

    cd ~
    # 安装基本库
    sudo yum install -y gcc gcc-c++
    
    # 下载安装工具
    sudo yum install -y wget

    # 安装nginx
    sudo yum update -y
    rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
    sudo yum install nginx -y
    sudo systemctl start nginx.service
    sudo systemctl enable nginx.service
    #sudo cp /vagrant/config/nginx/nginx.conf /etc/nginx/nginx.conf # 拷贝配置文件到虚拟机内
    sudo cp -R /vagrant/config/nginx/conf.d /etc/nginx/conf.d

    $ 安装nodejs
    wget https://nodejs.org/dist/v9.8.0/node-v9.8.0.tar.gz
    tar -xzf node-v9.8.0.tar.gz
    cd node-v9.8.0.tar.gz
    ./configure && make && sudo make install
    sudo ln -s /usr/local/bin/node /usr/bin/node
    sudo ln -s /usr/local/bin/npm /usr/bin/npm

    # 安装docker
    sudo yum install docker -y
    sudo systemctl start docker.service
    sudo systemctl enable docker.service

    # 安装php版本管理工具
    sudo rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
    sudo rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
    sudo yum install -y php72w php72w-cli php72w-fpm php72w-mysql
    sudo systemctl restart nginx.service php-fpm.service # php-fpm重启后需要重新启动
    sudo systemctl enable nginx.service php-fpm.service

    # 安装mysql
    wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm # 下载源
    sudo rpm -Uvh mysql57-community-release-el7-11.noarch.rpm
    yum repolist all | grep mysql
    sudo yum install -y mysql-community-server # 安装最新的GA版本
    sudo systemctl start mysqld.service
    # 接下来需要重新设置一下密码,必须ssh登录进去进行配置，1.找到临时密码；2.修改密码 3.如果无法使用临时密码登录可以是skip-grant-tables方法进去再修改

    # 安装一些工具
    sudo yum install -y ruby ruby-devel make gcc 	# 安装gem需要的一些ruby依赖
    sudo yum install -y gem
    gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/

    #################### php开发工具 #################

    #全局安装composer
    curl -sS https://getcomposer.org/installer | php
    sudo mv composer.phar /usr/local/bin/composer

    #################### node开发工具 #################
    sudo npm install -g pm2
    # 需要将pm2设为开机自启动
    # pm2 startup 执行生成出的命令
  SHELL
end
