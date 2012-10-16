#Introduction to the Studio

The Studio is an Ansible playbook and collection of related software that will
launch an EBS-backed Amazon EC2 instance with Ubuntu 12.04 on it, then
configure it fully into an Internet radio recording studio:

* using JACK Audio Connection Kit as a low-latency audio bus,
* using Mumble (routed through PulseAudio) as the studio set where radio hosts can talk,
* using Skype (routed through PulseAudio) to receive calls from callers and guests,
* using GStreamer to record the audio from the show and publish it to an Icecast server in both MP3 and Ogg Vorbis formats,
* using VNC to allow the studio techs and hosts to handle the show,
* in a matter of minutes.

We the peeps at [Decline to State](http://declinefm.com) use it to record our
radio show every Wednesday, and it works wonders.  We decided to release it
as open source software (under the AGPLv3) because this is software that can
power many Internet radio stations run by people across the globe.

#Setting up the software to run on your computer

* install git on your computer
* clone this repository
* `git submodule update`
* install ansible dependencies (PyYAML, jinja2, paramiko, python-zmq, python-keyczar)
* `source ansible/hacking/env-setup`

#Launching the instance

* make sure your amazon credentials are in the environment
* create the necessary elastic IP through the EC2 console
* change the settings section in the launch script
  * the correct elastic IP needs to be there
  * the correct hostname needs to be there, and resolving to the IP
* change the hosts file to point to the elastic IP address of the studio
  * originally it says studio.declinefm.com

#Configuring the instance to act as a studio

* The first time (to set a password)
  * `./converge --extra-vars "changepassword=yournewpassword"`
  * The password must be up to eight characters (VNC limitations).
* From then on
  * `./converge`

#Connecting to the studio

You can connect with `./connect` and this will open Vinagre.  Alternatively
if you don't have that program, you can use a VNC client to connect to
the host name of your studio instance.

#Configuring media and applications

After configuring the instance for the first time, you need to set up Mumble
and Skype in it.  They should launch automatically.

After that, copy your show intro to `/home/ubuntu/Studio/Clips/Intro.wav`
(must be 48 KHz mono) and then copy your show outro to
`/home/ubuntu/Studio/Clips/Outro.wav`.  This completes the setup of the
media necessary to run the live show's intro and outro commands.

Then you need to set up the record and broadcast facilities.  In a terminal,
run command `recordbroadcast` and answer the question related to your Icecast
server that you will be asked.  After running the command, you can start the
command `recordbroadcast`, which will promptly start broadcasting your show
to the icecast server, and recording it into `/home/ubuntu/Studio/Recordings`.

#Testing the instance

There are a number of things you need to test.

* Test that the audio from Mumble participants is recorded.
  * Check the mixer window.
* Test that the audio from Skype callers is recorded.
  * Check the mixer window.
* Test that audio goes back and forth between Mumble and Skype.
  * Skype echo / call testing service usually does the job.
* Test that the audio is being recorded.
  * Download the file recorded in `/home/ubuntu/Studio/Recordings`.
* Test that the audio is being broadcasted live.
  * Look at your radio station's Icecast home page.
  * Listen to the two mounts and verify that they are broadcasting.
* Test that the audio clips played back by the `intro` and `outro` commands get played back through the live broadcast, but not on the recording.
