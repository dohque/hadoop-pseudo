# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.box = 'centos-65-x64-virtualbox-nocm'
  config.vm.box_url = 'http://puppet-vagrant-boxes.puppetlabs.com/centos-65-x64-virtualbox-nocm.box'

  config.vm.provider 'virtualbox' do |v|
    # v.memory = 8192
    v.memory = 4096
    v.cpus = 4
  end

  config.vm.network 'forwarded_port', guest: 8088,  host: 8088   # resource manager
  config.vm.network 'forwarded_port', guest: 8042,  host: 8042   # node manager
  config.vm.network 'forwarded_port', guest: 19888, host: 19888  # mapreduce history server
  config.vm.network 'forwarded_port', guest: 50070, host: 50070  # namenode
  config.vm.network 'forwarded_port', guest: 50075, host: 50075  # datanode

  config.omnibus.chef_version = '12.0.1'
  config.berkshelf.enabled = true
#  config.berkshelf.berksfile_path = "Berksfile"
  
  config.vm.provision :chef_zero do |chef|
    # chef.cookbooks_path = 'provisioning/cookbooks'
    chef.add_recipe 'java'
    chef.add_recipe 'python'
    chef.add_recipe 'hadoop::default'
    chef.add_recipe 'hadoop::hadoop_hdfs_namenode'
    chef.add_recipe 'hadoop::hadoop_hdfs_datanode'
    chef.add_recipe 'hadoop::hadoop_yarn_resourcemanager'
    chef.add_recipe 'hadoop::hadoop_yarn_nodemanager'
    chef.add_recipe 'hadoop::hadoop_mapreduce_historyserver'
    
    chef.json = {
      'java' => {
        'install_flavor' => 'oracle',
        'jdk_version' => 7,
        # 'java_home' => '/usr/lib/jvm/java',
        'oracle' => {
          'accept_oracle_download_terms' => true
        }
      },
       'hadoop' => {
         'distribution' => 'cdh',
         'hadoop_env' => {
           'java_home' => '/usr/lib/jvm/java',
         },
         'hdfs_site' => {
           'dfs.replication' => 1,
           # 'dfs.permissions' => false
         },
         'yarn_site' => {
           'yarn.resourcemanager.hostname' => '0.0.0.0',
           'yarn.nodemanager.delete.debug-delay-sec' => 14400,
           # 'hadoop.security.authorization' => 'false',
           # 'allowed.system.users' => 'vagrant',
           'yarn.nodemanager.aux-services' => 'mapreduce_shuffle',
           'yarn.nodemanager.aux-services.mapreduce_shuffle.class' => 'org.apache.hadoop.mapred.ShuffleHandler',
           'yarn.application.classpath' =>
'''
$HADOOP_CONF_DIR,$HADOOP_COMMON_HOME/*,$HADOOP_COMMON_HOME/lib/*,
$HADOOP_HDFS_HOME/*,$HADOOP_HDFS_HOME/lib/*,
$HADOOP_YARN_HOME/*,$HADOOP_YARN_HOME/lib/*,
/usr/lib/hadoop-mapreduce/*,/usr/lib/hadoop-mapreduce/lib/*
'''
         },
         'mapred_site' => {
           'mapreduce.framework.name' => 'yarn'
         }
      }
    }
  end

  $script = <<SCRIPT
sudo -u hdfs hdfs namenode -format -nonInteractive
service hadoop-hdfs-namenode start
service hadoop-hdfs-datanode start

pip install snakebite
sudo -u hdfs snakebite mkdirp /tmp
sudo -u hdfs snakebite chmod 1777 /tmp

sudo -u hdfs snakebite mkdirp /var
sudo -u hdfs snakebite mkdirp /var/log
sudo -u hdfs snakebite chmod -R 1775 /var/log
sudo -u hdfs snakebite chown yarn:mapred /var/log

sudo -u hdfs snakebite mkdirp /tmp/hadoop-yarn
sudo -u hdfs snakebite chown -R mapred:mapred /tmp/hadoop-yarn
sudo -u hdfs snakebite mkdirp /tmp/hadoop-yarn/staging/history/done_intermediate
sudo -u hdfs snakebite chown -R mapred:mapred /tmp/hadoop-yarn/staging
sudo -u hdfs snakebite chmod -R 1777 /tmp

sudo -u hdfs snakebite mkdirp /var/log/hadoop-yarn/apps
sudo -u hdfs snakebite chmod -R 1777 /var/log/hadoop-yarn/apps
sudo -u hdfs snakebite chown yarn:mapred /var/log/hadoop-yarn/apps

sudo -u hdfs snakebite mkdirp /user/vagrant
sudo -u hdfs snakebite chown vagrant /user/vagrant

service hadoop-yarn-resourcemanager start
service hadoop-yarn-nodemanager start
service hadoop-mapreduce-historyserver start
SCRIPT

  config.vm.provision :shell do |shell|
    shell.inline = $script
  end  
end
