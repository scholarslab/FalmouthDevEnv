
require 'fileutils'
require 'vagrant'

################################
## These tasks set up the VM. ##
################################

namespace :setup do

  desc 'Clone OpsCode cookbook.'
  task :cookbooks do |t|
    unless File.directory?('cookbooks')
      puts 'Cloning OpsCode Chef Cookbooks.'
      system('git clone https://github.com/opscode/cookbooks.git cookbooks')
    end
  end

  desc 'Clone SLab cookbook.'
  task :slabcookbooks do |t|
    unless File.directory?('slab-cookbooks')
      puts 'Cloning Scholars\' Lab Chef Cookbooks.'
      system('git clone https://github.com/scholarslab/cookbooks.git slab-cookbooks')
    end
  end

  desc 'This initializes the VM.'
  task :initvm do
    env = Vagrant::Environment.new
    if !env.primary_vm.created?
      puts 'Starting VM.'
      env.cli('up')
    else
      puts 'VM already exists. Skipping.'
    end
  end

  desc 'Expands the archive directory\'s contents.'
  task :archive do
    puts 'Expanding archive files.'
    system('cd omeka ; tar xfj ../archive.tar.bz2')
  end

  desc 'This loads the Falmouth data into the database.'
  task :loaddb do
    puts 'Loading the initial database data.'
    system('gzip -cd falmouth-production.sql.gz | mysql -ufalmouth -pfalmouth --protocol=TCP --port=3334 falmouth')
  end

  desc 'This transfers the Solr config files from the SolrSearch plugin to the VM.'
  task :solrconf do
    env = Vagrant::Environment.new

    puts 'Transferring files from SolrSearch/solr-home to /var/solr/site.'
    env.primary_vm.ssh.execute do |ssh|
      ssh.exec!('sudo chmod -R a+rwx /var/solr/site/conf')
    end
    Dir['omeka/plugins/SolrSearch/solr-home/conf/*.*'].each do |src|
      dest = File.join('/var/solr/site/conf', File.basename(src))
      puts "\t#{src} => #{dest}"
      env.primary_vm.ssh.upload!(src, dest)
    end
  end

  desc 'This restarts Tomcat.'
  task :tomcatrestart do
    puts 'Restarting Tomcat.'
    env = Vagrant::Environment.new
    env.primary_vm.ssh.execute do |ssh|
      ssh.exec!('sudo /sbin/service tomcat6 restart ') do |channel, stream, data|
        print data
        $stdout.flush
      end
    end
  end

end

desc "Initialize the environment. This calls cookbooks, slabcookbooks, initvm,
archive, loaddb, solrconf, and tomcatrestart."
task :init => ['setup:cookbooks',
               'setup:slabcookbooks',
               'setup:initvm',
               'setup:archive', 
               'setup:loaddb',
               'setup:solrconf',
               'setup:tomcatrestart'] do
end

####################################################################
## Some useful tasks for working with the VM and the environment. ##
####################################################################

desc 'Do a safe halt on the VM.'
task :halt do
  env = Vagrant::Environment.new
  puts 'Halting VM.'
  env.primary_vm.ssh.execute do |ssh|
    ssh.exec!('sudo halt') do |channel, stream, data|
      print data
      $stdout.flush
    end
  end
end

desc 'Clobber everything. This deletes downloaded repositories and destroys the VM.'
task :clobber do
  FileUtils.rmtree %w{cookbooks omeka slab-cookbooks}, :verbose => true
  env = Vagrant::Environment.new
  puts 'Destroying VM.'
  env.cli('destroy')
end

desc "cat /tmp/vagrant-chef/chef-stacktrace.out."
task :chefst do
  env = Vagrant::Environment.new
  raise "Must run `vagrant up`" if !env.primary_vm.created?
  raise "Must be running!" if !env.primary_vm.vm.running?
  puts "Getting chef stacktrace."
  env.primary_vm.ssh.execute do |ssh|
    ssh.exec!("cat /tmp/vagrant-chef/chef-stacktrace.out") do |channel, stream, data|
      puts data
    end
  end
end

