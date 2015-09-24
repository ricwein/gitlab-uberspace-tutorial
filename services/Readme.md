# daemontools service scripts for uberspace

This folder contains two scripts to manage the services needed by gitlab.


## usage
if you haven't done it already, create a bin folder in your home directory:

    mkdir ~/bin

copy `gitlab` `sidekiq` and `git-http-server` in this `bin` folder.
make all three scripts executable with `chmod +x [filename]`
**IMPORTANT** if your gitlab instance is __not__ installed in ~/gitlab you need to edit the service-scripts and set the path according to your needs!

Setup deamons according to [uberspace documentation](https://wiki.uberspace.de/system:daemontools):

    uberspace-setup-service sidekiq ~/bin/sidekiq
    uberspace-setup-service gitlab ~/bin/gitlab
    uberspace-setup-service git-http-server ~/bin/git-http-server

That's it! Happy coding :-)
