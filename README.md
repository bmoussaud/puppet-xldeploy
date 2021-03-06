puppet-xldeploy
===============

####Table of Contents

1. [License](#license)

2. [Overview](#overview)

3. [Module Description - What the module does and why it is useful](#module-description)
4. [Setup - The basics of getting started with puppet-xldeploy](#setup)
    * [What puppet-xldeploy affects](#what-puppet-xldeploy-affects)
    * [Setup requirements](#setup-requirements)
    * [Beginning with puppet-xldeploy](#beginning-with-puppet-xldeploy)
5. [Usage - Configuration options and additional functionality](#usage)
6. [Reference - An under-the-hood peek at what the module is doing and how](#reference)
7. [Limitations - OS compatibility, etc.](#limitations)
8. [Development - Guide for contributing to the module](#development)

license
-------
THIS CODE AND INFORMATION ARE PROVIDED "AS IS" WITHOUT WARRANTY OF ANY KIND, EITHER EXPRESSED OR 
IMPLIED, INCLUDING BUT NOT LIMITED TO THE IMPLIED WARRANTIES OF MERCHANTABILITY AND/OR FITNESS 
FOR A PARTICULAR PURPOSE. THIS CODE AND INFORMATION ARE NOT SUPPORTED BY XEBIALABS.

overview
--------
The xldeploy module enables you to install and manage large XL Deploy enabled infrastructures.

module-description
------------------
XL Deploy is a kick-ass modular platform independant software deployment system. This module enables you to tie this deployment software into your puppet environment. It can take care of installing XL Deploy servers as well as integrate middleware components elsewhere in your infrastructure.

setup
-----
**what-puppet-xldeploy-affects:**

* installation/service/configuration files for XL Deploy
* listened-to ports
* backend storage filesystem/database
* xldeploy housekeeping
* xldeploy repository configuration items
* xldeploy repository roles/permissions
* xldeploy repository dictionaries
* xldeploy cli installation 
* installs compatible java version (optional)

**setup-requirements**

* This module makes use of exported resources (optional) if you descide to use this functionallity please configure this in your puppet.conf file on the master.
* This module makes use of the puppetdbquery module to distribute sshkeys across the infrastructure (optional). This functionality depends on puppetdb.


**beginning-with-puppet-xldeploy**

Basic usage when installing a xldeploy server

    class{xldeploy::server:}

For a more comprehensive setup in wich a supported java version is installed and a xldeploy license file is included try the example below.

    class{xldeploy::server:
        install_java                => true,
        install_license             => true,
        install_type                => 'download',
        download_user               => 'foobar',
        download_password           => 'notapassword',
        download_proxy_url          => 'http://some:user@companyproxy.evil.empire:8080'
     }


To install the command line interface (cli) software bundled with xldeploy use the seperate xldeploy::cli class like discribed below. 

    class{xldeploy::cli:
            install_java                => true,
            install_type                => 'netinstall',
            download_user               => 'foobar',
            download_password           => 'notapassword',
            download_proxy_url          => 'http://some:user@companyproxy.evil.empire:8080'
     }


From a potential xldeploy client machine using the module to register a ci in with the xldeploy server. 

  all in one go:

    class{xldeploy::client:
            http_context_root   => '/xldeploy',
            http_server_address => 'xldeploy.local.domain',
            http_port           => '4516',
            rest_user           => 'admin',
            rest_password       => 'dummy',
            ssl                 => false,
            cis                 => { 'project_folder' => { name => "/Infrastructure/projectx",
                                                           type => 'core.Directory',
                                                           },
                                     'host' => { name => "/Infrastructure/projectx/${hostname}_sshHost",
                                                 type => 'overthere.SshHost',
                                                 properties => { 'os' => 'UNIX',
                                                                 'port' => '22',
                                                                 'username' => 'deployit',
                                                                 'tags' => 'projectx',
                                                                 'connectionType' => 'SCP',
                                                                 'address' => '${hostname}' }
                                      }
                                     }
    }
  the above example uses the builtin create_resources construct to create the ci's specified in the array one by one. This construct is especially handy when used in conjunction with automatic data bindings from a Hiera backend.
  
  
  Users are also able to create ci's using the module's types and providers (setting 'use_exported_resources' to 'true'):

    xldeploy_ci{ 'Infrastructure/projectx':
          ensure             => present,
          type               => 'core.Directory',
          rest_url           => 'http://admin:password@xldeploy.domain.local:4516/xldeploy' }
    }
    
    xldeploy_ci{ "Infrastructure/projectx/${hostname}_sshHost":
          ensure             => present,
          type               => 'overthere.SshHost',
          properties         => { 'os' => 'UNIX',
                                  'port' => '22',
                                  'username' => 'deployit',
                                  'tags' => 'projectx',
                                  'connectionType' => 'SCP',
                                  'address' => '${hostname}' },
          rest_url           => 'http://admin:password@xldeploy.domain.local:4516/xldeploy' }
    }

  Both class parameters and types and providers are available for defining memberships,users, roles, dictionary_settings and role_permissions wich work in the same way as the above specified method for defining a ci.
  
usage
-----
  
  
  
**xldeploy::server**
######version                          
    specifies the version of xldeploy to install 
    version numbers below 4.0 will install deployit
######xldeploy_base_dir                 
    specifies the base dir under wich xldeploy will be installed. 
    default: /opt
######xldeploy_init_repo 
    Tells puppet to initialize the xldeploy repository upon first installation
    default: true
######tmp_dir 
    Tells puppet wich temporary directory to use during installation. This directory will be used for downloading the installation packages
    default: /var/tmp
######os_user       
    Specifies the default user for xldeploy to be installed under
    this user will be created by puppet
    default: pre_4 : deployit
             post_4 : xldeploy
######os_group            
    Specifies the group xldeploy will be installed under
    This group will be created by pupept
    default: pre_4 : deployit
             post_4 : xldeploy
######import_ssh_key               
    Tells puppet to use automagic to create and distribute an ssh key to all client systems for secure ssh communication
    default : false
######http_bind_address
    the http address to bind the xldeploy server to 
    default: 0.0.0.0 (xldeploy will listen on all interfaces) 
######http_port   
    the http port on wich xldeploy will listen for incoming traffic
    default: 4516
######http_context_root                
    the http context root xldeploy will use for incoming web traffic(both gui and rest interface)
    default: /deployit
######http_server_address
    the ipadress to contact the xldeploy server on (this is used in combination with the ci creation construct)
    defaults to the fqdn of the current host
######admin_password                   
    the admin password to be used for xldeploy
    default: admin
######jcr_repository_path      
    the relative path to put the jcr_repository
    default: repository
######importable_packages_path
    the relative path to the importable packages directory
    default: importablePackages
######install_type        
    the type of installation to use. Possible values: puppetfiles, download
    default: download
######puppetfiles_xldeploy_source
    when install_type is set to puppetfiles this parameter should contain the path to the xldeploy tar.gz packages
    default: undef
######download_user    
    a valid xldeploy download acount user
    default: undef
######download_password     
    a valid xldeploy download account user password
    default: undef
######download_proxy_url    
    optional proxy url (should contain user and password if needed) 
    we've included this because we hate modules that download stuff for you and then completely forget that most company's do not allow direct internet access from the production network
######use_exported_resources 
    use exported resources to import the various exported ci and other resources to the xldeploy service 
    default: false
######use_exported_keys      
    export a locally generated public key of a ssh key pair that can be used to import on client nodes to setup secure communication
    default: false
######client_propagate_key
   
######java_home              
######install_java           
######install_license        
######license_source         
######enable_housekeeping    
######ldap_server_id         
######ldap_server_url        
######ldap_server_root             
######ldap_manager_dn              
######ldap_manager_password             = $xldeploy::params::ldap_manager_password,
######ldap_user_search_filter           = $xldeploy::params::ldap_user_search_filter,
######ldap_user_search_base             = $xldeploy::params::ldap_user_search_base,
######ldap_group_search_filter          = $xldeploy::params::ldap_group_search_filter,
######ldap_group_search_base            = $xldeploy::params::ldap_group_search_base,
######ldap_role_prefix                  = $xldeploy::params::ldap_role_prefix,
######xldeploy_authentication_providers = $xldeploy::params::xldeploy_authentication_providers,
######repository_type                   = $xldeploy::params::repository_type,
######datastore_driver                  = $xldeploy::params::datastore_driver,
######datastore_url                     = $xldeploy::params::datastore_url,
######datastore_user                    = $xldeploy::params::datastore_user,
######datastore_password                = $xldeploy::params::datastore_password,
######datastore_databasetype            = $xldeploy::params::datastore_databasetype,
######datastore_schema                  = $xldeploy::params::datastore_schema,
######datastore_persistencemanagerclass = $xldeploy::params::datastore_persistencemanagerclass,
######datastore_jdbc_driver_url         = $xldeploy::params::datastore_jdbc_driver_url,
######gem_use_local                     = $xldeploy::params::gem_use_local,
######gem_hash                          = $xldeploy::params::gem_hash,
######gem_array                         = $xldeploy::params::gem_array,
######disable_firewall                  = $xldeploy::params::disable_firewall,
######custom_productname                = undef,
######custom_download_server_url        = undef,
######custom_download_cli_url           = undef,
######server_plugins                    = { } ,
######cis                               = { } ,
######memberships                       = { } ,
######users                             = { } ,
######roles                             = { } ,
######dictionary_settings               = { } ,
######role_permissions                  = { } ,
######xldeploy_default_settings         = { }

**xldeploy::client**

######os_user                           = $xldeploy::params::os_user,
######os_group                          = $xldeploy::params::os_group,
######os_user_home                      = $xldeploy::params::os_user_home,
######os_user_manage                    = $xldeploy::params::os_iser_manage,
######http_bind_address                 = $xldeploy::params::http_bind_address,
######http_port                         = $xldeploy::params::http_port,
######http_context_root                 = $xldeploy::params::http_context_root,
######http_server_address               = $xldeploy::params::http_server_address,
######ssl                               = $xldeploy::params::ssl,
######verifySsl                         = $xldeploy::params::verifySsl,
######rest_user                         = $xldeploy::params::rest_user,
######rest_password                     = $xldeploy::params::rest_password,
######client_sudo                       = $xldeploy::params::client_sudo,
######client_user_password              = $xldeploy::params::client_user_password,
######client_user_password_salt         = $xldeploy::params::client_user_password_salt,
######use_exported_resources            = $xldeploy::params::use_exported_resources,
######use_exported_keys                 = $xldeploy::params::use_exported_keys,
######client_propagate_key              = $xldeploy::params::client_propagate_key,
######gem_use_local                     = $xldeploy::params::gem_use_local,
######gem_hash                          = $xldeploy::params::gem_hash,
######gem_array                         = $xldeploy::params::gem_array,
######cis                               = { } ,
######memberships                       = { } ,
######users                             = { } ,
######roles                             = { } ,
######dictionary_settings               = { } ,
######role_permissions                  = { } ,