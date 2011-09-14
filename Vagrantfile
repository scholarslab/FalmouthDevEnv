require 'fileutils'

Vagrant::Config.run do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "falmouth"

  # The url from where the 'config.vm.box' box will be fetched if it
  # doesn't already exist on the user's system.
  config.vm.box_url = "./base-centos32.box"

  config.vm.provision :chef_solo do |chef|
    chef.cookbooks_path = ["cookbooks/opscode", "cookbooks/slab"]

    chef.add_recipe "omeka"
    chef.add_recipe "solr"

    # :omeka_version can be a version number tag (e.g., '1.4', '1.3.2') or
    # 'HEAD' or a branch name.
    chef.json.merge!({
      :omeka => {
        :version => 'stable-1.4',

        :username            => 'admin',
        :password            => 'XXXXXXXX',
        :super_email         => 'err8n@virginia.edu',
        :administrator_email => 'err8n@virginia.edu',
        :site_title          => 'Falmouth',
        :description         => 'Falmouth',
        :copyright           => '2011 The Board and Visitors of the University of Virginia',

        :mysql_user          => 'falmouth',
        :mysql_password      => 'XXXXXXXX',
        :mysql_db            => 'falmouth',

        :themes => [
          {:name => 'falmouth', :type => 'svn',
           :svn_username => 'err8n', :svn_password => 'XXXXXXXX',
           :url => 'https://subversion.lib.virginia.edu/repos/slab_rd/falmouthTheme'}
        ],
        :plugins => [
          {:name => 'CsvImport', :url => 'git://github.com/omeka/plugin-CsvImport.git'},
          {:name => 'SolrSearch', :url => 'git://github.com/scholarslab/SolrSearch.git'},
          {:name => 'VraCoreElementSet', :url => 'git://github.com/scholarslab/VraCoreElementSet.git'},
          {:name => 'Dropbox', :url => 'git://github.com/omeka/plugin-Dropbox.git',
           :revision => 'tags/1.3-0.5'}
        ]
      },
      :domain => [],
      :openldap => {}
    })
  end

  config.vm.forward_port('mysql', 3306, 3334)
  config.vm.forward_port('apache2', 80, 8050)
  config.vm.forward_port('tomcat', 8080, 8090)
end
