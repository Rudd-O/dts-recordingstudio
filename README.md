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
  * Listen to your radio station's Icecast URL.
* Test that the audio clips played back by the `intro` and `outro` commands get played back through the live broadcast, but not on the recording.
