
require 'fileutils'
require 'vagrant'

ARCHIVE_FILE = File.join(File.dirname(__FILE__), 'archive.tar.bz2')
SQL_DUMP = File.join(File.dirname(__FILE__), 'falmouth-production.sql.gz')
BROKEN_RECIPES = %w{opscode/windows
                    opscode/firewall
                    opscode/webpi
                    opscode/iis}

task :default => :usage

task :usage do
  puts "You forgot to tell the computer what to do; try one of these commands:"
  system("rake -T")
end

################################
## These tasks set up the VM. ##
################################

namespace :setup do

  desc 'Clone Chef cookbooks.'
  task :cookbooks do
    unless File.directory?("cookbooks")
      Dir.mkdir('cookbooks')
    end
    Rake::Task['setup:opscodebook'].invoke
    Rake::Task['setup:slabbook'].invoke
    Rake::Task['setup:cleankitchen'].invoke
  end

  desc 'Clone OpsCode cookbook.'
  task :opscodebook do
    system('git clone --branch=slab https://github.com/erochest/opscode-cookbooks.git cookbooks/opscode')
  end

  desc 'Clone SLab cookbook.'
  task :slabbook do
    system('git clone https://github.com/scholarslab/cookbooks.git cookbooks/slab')
  end

  desc "This removes recipes that are causing errors from the cookbook."
  task :cleankitchen do
    BROKEN_RECIPES.each do |recipe|
      if File.directory?("cookbooks/#{recipe}")
        FileUtils.rmtree "cookbooks/#{recipe}", :verbose => true
      end
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
    if File.exists?(ARCHIVE_FILE)
      puts 'Expanding archive files.'
      system("cd omeka ; tar xfj #{ARCHIVE_FILE}")
    else
      puts "Archive file (#{ARCHIVE_FILE}) does not exist. Skipping."
    end
  end

  desc 'This loads the Falmouth data into the database.'
  task :loaddb do
    if File.exists?(SQL_DUMP)
      puts 'Loading the initial database data.'
      system("gzip -cd #{SQL_DUMP} | mysql -ufalmouth -pXXXXXXXX --protocol=TCP --port=3334 falmouth")
    else
      puts "SQL dump file (#{SQL_DUMP}) does not exist. Skipping."
    end
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
  FileUtils.rmtree %w{cookbooks omeka}, :verbose => true
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
    ssh.exec!("cat /tmp/vagrant-chef-1/chef-stacktrace.out") do |channel, stream, data|
      puts data
    end
  end
end

