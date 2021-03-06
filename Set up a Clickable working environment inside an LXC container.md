INTRODUCTION

Clickable is the Brian Douglass' program to build, manage, install and in general develop for the Ubuntu Touch platform without the need of the whole ubuntu SDK.
It can runs virtually in any linux environment (differently from the ubuntu SDK), but like me maybe also you encoutered problems setting up it in your favourite distro; here I am going to guide you throught setting up it in a LXC container.
In this way you aren't forced to change distro or run an expansive virtual machine, but you can use clickable almost like a native set up.
In this guide I consider using the apt package manager. Adapt the 'apt-get' commands to your circustances.

SET UP LXD

We are going to use an ubuntu 16.04LTS container. To use it we have to install lxd, the lxc container hypervisor deamon. With the following command:
"sudo apt-get install lxd"
You could be on a distro which hasn't in the repository LXD. No problem, if your distro supports snap packages you can install the snap version of LXD.
I used the snap package to achieve the final result, but I suggest use your distro LXD package if you can. So first install snapd, then install LXD snap:
"sudo apt-get install snapd"
"sudo snap install lxd"
Here is a deeper guide about the LXD snap package: https://stgraber.org/2016/10/17/lxd-snap-available/. If your are going to use the snap package, take a look there.
Now we have lxd installed, let's configure it:
"lxd init"
Just press enter until the end for a standard set up.
Troubleshooting:
if you stumble in any strange error when running lxd commands, check if your user is in the lxd group, if not add yourself there:
"usermod -a -G lxd username"
Another common issue could be to not have the lxd daemon running; if so just run:
"sudo systemctl start lxd.service"

SET UP THE CONTAINER

Now we'll set up the ubuntu container:
"lxc launch ubuntu:16.04 best_container_name_ever"
and lxc will automatically download and set up our new container.
Before enter the new environment, we need to change some container policy option to enable nested lxc container creation:
"lxc config set best_container_name_ever security.privilegied true"
"lxc config set best_container_name_ever security.nesting true"
now we are ready to enter the matr...emm shell of our ubuntu container:
"lxc exec best_container_name_ever -- /bin/bash"

INSIDE THE CONTAINER

Inside the container we have to do basically three things: basic configuration of the system, creation of a new not-root user and configuring clickable.
First of all we need some basic utility:
"apt-get update"
"apt-get install nano sudo git"
These packages may be already installed, but who know?
Next step is to configure a not root user. These commands should work, but I haven't tested them:
"useradd mr_nice_guy"
"passwd mr_nice_guy"
Enable him to run sudo
"usermod -a -G sudo mr_nice_guy" // (Maybe this is not the most clean way to do it?)
Now we are ready to being the nice guy.
"su mr_nice_guy"
Step in our home directory
"cd"
Let's install and configure LXD. See the previous 'SET UP LXD' section, here is the same story.

CONFIGURE CLICKABLE (INSIDE THE CONTAINER)

Here we are! Let's eventually download clickable:
"git clone https://github.com/bhdouglass/clickable.git"
Let's enter the master directory. Here we need to install the usdk-target executable to be able to run it.
Inside the container we need to run sudo with the '-S' option, because without it will fail.
"sudo -S cp usdk-target /bin"
"sudo -S cp usdk-target /usr/bin"
"sudo -S cp usdk-target /usr/sbin"
"sudo -S cp usdk-target /sbin"
I copyied it into all the four directoryes because I don't know in which one I should copy it; it's a quick-and-dirt solution, but I am too lazy.
Now we need to patch the clickable executable. I said before that in this environment we have to run sudo with the '-S' option.
"nano clickable"
Look for the line:
"subprocess.check_call(shlex.split('sudo {} usdk-target create -n {} -p {}'.format(env, name, fingerprint)))"
And add the -S option. Here it is the result:
"subprocess.check_call(shlex.split('sudo -S {} usdk-target create -n {} -p {}'.format(env, name, fingerprint)))"
Save with CTRL-O and exit nano.
You have succeffully set up clickable inside an lxd container!
Now you can run clickable inside the lxc container like a charm without problems.
For information about using clickable, see here: https://github.com/bhdouglass/clickable
Note that canonical server might not work, you can tell clickable to use a different image server with the 'USDK_TEST_REMOTE' environment variable.

FINAL THOUGHTS

Clickable is a wonderful tool to develop app for ubuntu touch, thanks to Brian Douglass and all the devs for manteining it and for all the help they gave me :D
Good coding everyone!!!