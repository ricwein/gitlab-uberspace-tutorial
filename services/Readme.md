# daemontools service scripts for uberspace

This folder contains two scripts to manage the services needed by gitlab.


## usage
if you haven't done it already, create a bin folder in your home directory:

    mkdir ~/bin

copy `gitlab` `sidekiq` and `gitlab-workhorse` in this `bin` folder.
make all three scripts executable with `chmod +x [filename]`
**IMPORTANT** if your gitlab instance is __not__ installed in ~/gitlab you need to edit the service-scripts and set the path according to your needs!

Setup deamons according to [uberspace documentation](https://wiki.uberspace.de/system:daemontools):

    uberspace-setup-service sidekiq ~/bin/sidekiq
    uberspace-setup-service gitlab ~/bin/gitlab
    uberspace-setup-service gitlab-workhorse ~/bin/gitlab-workhorse

That's it! Happy coding :-)

## update from git-http-server to gitlab-workhorse
    cd ~/service/git-http-server
    rm ~/service/git-http-server
    svc -dx . log
    rm -rf ~/etc/run-git-http-server
    cd ~/bin
    mv git-http-server.sh gitlab-workhorse.sh
    uberspace-setup-service gitlab-workhorse ~/bin/gitlab-workhorse.sh
