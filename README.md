# RuRaNoPuPoRe: A Stupidly Named Ubuntu 12.04 LTS Ruby, Rails+Nginx, Node.js, Puma, Postgres, and Redis Ansible Playbook Collection

While this playbook works for the most part, it's essentially a work in progress and something I've put together while learning all of Ansible, Capistrano, and Vagrant so.. you've been warned! :-)

Ruby, Nginx + Puma, Node, Postgres, and Redis (with, optionally, ElasticSearch) form the basis for the stack of significant webapps I work on. This playbook brings them all together with a role for each and should be easy to customize for your own situations. I currently have this playbook working fine against Vagrant in a local development environment.

Before I move on, remember.. this is all used at your own risk and this stuff is primary up here for my own use, but if it helps you out or you want to learn stuff from it, be my guest :-)

## To add ElasticSearch

https://github.com/Traackr/ansible-elasticsearch is a superb Ansible playbook for Elasticsearch and it's easy to integrate. Get RuRaNoPuPoRe set up on your app however you like and then introduce ansible-elasticsearch as a submodule:

    cd roles
    git submodule add git://github.com/traackr/ansible-elasticsearch.git ./ansible-elasticsearch
    git submodule update --init
    git commit ./submodule -m "Added ansible-elasticsearch as submodule"

Then include `ansible-elasticsearch` as a role in `site.yml`.

## Vagrant bits and pieces

Want to run ansible-playbook directly instead of using `vagrant provision`? There's a (long winded) way:

    ansible-playbook -i .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory --private-key=~/.vagrant.d/insecure_private_key -u vagrant provisioning/site.yml

If using Vagrant and all of this stuff is in `provisioning`, you can set things up in the `Vagrantfile` with:

    config.vm.provision "ansible" do |ansible|
      ansible.playbook = "provisioning/site.yml"
    end

## Capistrano bits and pieces

If you're using Capistrano 3.2+ for deployment, some useful things here I've been using.

If using Vagrant for staging/development, this is a nice way to use the shared folder for git checkout rather than a repository on GitHub, say:

    set :repo_url, 'file:///vagrant'
    set :local_repository, 'file://.'

And a variety of Puma related stuff some of which I figured out, some of which was taken from other projects and makes it all work:

    set :linked_files, %w{config/database.yml}
    set :linked_dirs, %w{tmp/sockets log config/puma}
    set :sockets_path, Pathname.new("#{fetch(:deploy_to)}/shared/tmp/sockets/")

    set :ssh_options, {:forward_agent => true }
    set :puma_roles, :app
    set :puma_socket, "unix://#{fetch(:sockets_path).join('puma_' + fetch(:application) + '.    sock')}"
    set :pumactl_socket, "unix://#{fetch(:sockets_path).join('pumactl_' + fetch(:application) + '.    sock')}"
    set :puma_state, fetch(:sockets_path).join('puma.state')
    set :puma_log, -> { shared_path.join("log/puma-#{fetch(:stage )}.log") }
    set :puma_flags, nil

    set :bundle_flags, '--deployment'

    namespace :deploy do
      task :restart do
        invoke 'puma:restart'
      end
    end

    after 'deploy:publishing', 'deploy:restart'

## Credits

Quite a few bits and pieces have been cribbed, inspired by, or ripped from a variety of places, including:

* https://github.com/jprichardson/ansible-redis
* https://github.com/radar/ansible-rails-app .. which in turn took from https://github.com/dodecaphonic/ansible-rails-app
* https://github.com/jgrowl/ansible-playbook-ruby-from-src
* https://gist.github.com/acrookston/9148401
* https://github.com/aenglund/nodejs-ansible
