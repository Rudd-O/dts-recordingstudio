!Setting up the software to run on your computer

# clone this repository
# git submodule update
# install ansible dependencies (PyYAML, jinja2, paramiko, python-zmq, python-keyczar)
# source ansible/hacking/env-setup

!Launching the instance

# make sure your amazon credentials are in the environment
# create the necessary elastic IP through the EC2 console
# change the settings section in the launch script
## the correct elastic IP needs to be there
## the correct hostname needs to be there, and resolving to the IP
# change the hosts file to point to the elastic IP address of the studio
## originally it says studio.declinefm.com

!Configuring the instance
# The first time (to set a password)
## `./converge --extra-vars "changepassword=yournewpassword"`
# From then on
## `./converge`

After configuring the instance for the first time, you need to set up Mumble
and Skype. 

After that, Copy your show intro to /home/ubuntu/Music/Intro.wav
(must be 48 KHz mono) and then copy your show outro to
/home/ubuntu/Music/Outro.wav.  That completes the setup of the
media necessary to run the live show's intro and outro commands.

!Connecting to the studio

You can connect with `./connect` and this will open Vinagre.  Alternatively
if you don't have that program, you can use a VNC client to connect to
the host name of your studio instance.
