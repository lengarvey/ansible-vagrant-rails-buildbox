[Buildbox](http://buildbox.io) is a continuous integration provider similar to [Travis](https://travis-ci.com/), [CircleCI](https://circleci.com/) or
[Semaphore](https://semaphoreapp.com/), except that it runs the tests on your own hardware. This means that
your code never leaves your own servers (and Github if you happen to use that).

BuildBox is like a hybrid between Travis and [Jenkins](http://jenkins-ci.org/).
More secure and enterprise friendly than Travis and less smug and timewasting than Jenkins.

The annoying thing about Buildbox is that it's a bit of a pain to try out. You
need to have a server to run the Buildbox agent on and the server needs to be
able to run the tests for your code. This means taking advantage of that free
trial is a little difficult.

**This is a Vagrant/Ansible setup which will build a Rails capable VM ready to run
your Buildbox tests upon. This README is your guide on how to do that. Lets
go!**

## Steps to trying out Buildbox

1. Sign up to Buildbox at https://buildbox.io/signup
2. Click the "Create an Agent" button to make your agent
   ![create an agent](http://i.imgur.com/Y0IGzF2.png)
   and record the access token provided
3. Configure your project settings and set the Script Path to `./bin/buildbox-run`. You'll need to add this script to your repository too.
   this script specifies the commands to run your test suite. So something like:

   ```sh
   #!/bin/bash
   set -e
   set -x

   chruby-exec `cat .ruby-version` -- bundle install
   chruby-exec `cat .ruby-version` -- bundle exec rake db:create db:schema:load
   chruby-exec `cat .ruby-version` -- bundle exec rspec spec
   ```

4. Download and install VirtualBox - https://www.virtualbox.org/wiki/Downloads
5. Download and install Vagrant (don't use the gem version!) - http://www.vagrantup.com/downloads.html
6. Open your terminal and `git clone https://github.com/lengarvey/ansible-vagrant-rails-buildbox.git` (feel free to
fork if it you prefer to do so)
7. `cd ansible-vagrant-rails-buildbox` to change into the directory
8. run: `BUILDBOX_ACCESS_TOKEN=<YOUR BUILDBOX ACCESS TOKEN HERE> vagrant up` to create your virtual machine
9. Trigger a build in the Buildbox web interface by clicking the Trigger Build
button.

Done!

## Assumptions

This makes a few assumptions about your Rails application.

1. It uses Postgres as the database (you could easily configure it to use MySQL if preferred)
2. It runs on Ruby 2.1.0 or 2.1.1 (easily configurable)
3. It doesn't depend on sidekiq, or redis or resque.
4. Your tests should use Poltergeist or PhantomJS instead of capybara-webkit or
selenium
5. You have a `config/database.buildbox.yml` file similar to the
`config/database.travis.yml`. This is copied to `config/database.yml` by the
custom bootstrap.sh file provided.
6. Your github access key is `~/.ssh/id_rsa.pub`

All these things can be altered by forking the repo and adding extra stuff into the playbook.

For example:

* to add MySQL you'd need to add a new mysql role and switch our the postgres role.
* to change the Ruby version's available just add them to `provisioning/roles/ruby/defaults/main.yml`

## Troubleshooting

* Occasionally the `vagrant up` command will fail to provision due to an ssh error. Simply run `BUILDBOX_ACCESS_TOKEN=<YOUR BUILDBOX ACCESS TOKEN HERE> vagrant provision` and it should fix itself.
* Occasionally the buildbox agent won't do the next step after bundling. Just restart your build.
* If you want to tweak something the playbook is designed to be either idempotent (and fast) or to check to see if steps are required.

## Credit

Much credit goes to BuildBox for providing a [Docker file](https://github.com/buildboxhq/buildbox-docker/blob/master/Dockerfile) which I cribbed most of the Ansible playbook from.

Also [reInteractive](http://reinteractive.net) has provided me the time to work on Open source stuff and learning on Fridays. Which is pretty fantastic. If you're a Rails developer or designer in Australia and looking for a good place to work. Talk to me: [@lgarvey](https://twitter.com/lgarvey)

## TODO:

1. My Ansible skills are pretty bad.
2. It would be way nicer if it prompted you for your buildbox token rather than it having to passed via ENV variable.
3. Determine if the `buildbox-run` script requires `chruby-exec` or if it has access to the chruby function.
