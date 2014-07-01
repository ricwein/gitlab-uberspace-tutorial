# daemontools service scripts for uberspace

This folder contains two scripts to manage the services needed by gitlab. 


## usage
if you haven't done it already, create a bin folder in your home directory:

	mkdir ~/bin

copy `gitlab` and `sidekiq` in this `bin` folder.   
**IMPORTANT** if your gitlab instance is __not__ installed in ~/gitlab you need to edit both service scripts and set the path according to your needs!

Setup two deamons according to [uberspace documentation](https://wiki.uberspace.de/system:daemontools):

	uberspace-setup-service run-sidekiq ~/bin/sidekiq
	uberspace-setup-service run-gitlab ~/bin/gitlab

That's it! Happy coding :-)

## changelog

### 2014-07-27

initial version of this README. Test with [gitlab 7.0 stable](https://about.gitlab.com/) on [Uberspace](https://uberspace.de) CentOS 5 host.
