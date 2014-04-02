#The Decline to State recording studio

| Donate to support this free software |
|:------------------------------------:|
| <img width="164" height="164" title="" alt="" src="doc/bitcoin.png" /> |
| [1K1ZDMJss4W9eK3QNppgtvHq91xUpr78gp](bitcoin:1K1ZDMJss4W9eK3QNppgtvHq91xUpr78gp) |

This recording studio is an Ansible playbook and collection of related software that will
launch an EBS-backed Amazon EC2 instance with Ubuntu 12.04 on it, then
configure it fully into an Internet radio broadcasting studio that can also be
used to record high-quality podcasts:

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

This software is a companion to the
[Decline to State publishing studio](https://github.com/Rudd-O/dts-publishingstudio),
which you can use to edit recorded shows and publish them online.

#Setting up the software to run on your computer

* make sure your administrative Amazon credentials are in the environment
* install git on your computer
* install ansible on your computer
* clone this repository
* change directory into this repository

#Launching the instance

* create the necessary elastic IP through the EC2 console
* create an S3 bucket to store the recordings
* make sure that a default security group exists, and that it allows connections to VNC and SSH
  * this is the security group that will be used to create the studio
* create an IAM keypair
* give the IAM keypair access to list keys and upload files to said bucket
* change the settings section in the launch script
  * the correct elastic IP needs to be there
  * the correct hostname needs to be there, and resolving to the right IP
  * put the public SSH key that you want to use to access the server

#Configuring the instance to act as a studio

* The first time (to set a password, AWS access key ID, AWS secret key)
  * `./converge --extra-vars "changepassword=<new password> accesskeyid=<AWS access key ID> secretaccesskey=<AWS secret key>"`
  * The password must be up to eight characters (VNC limitations).
* Right after that step, SSH as the user ubuntu into the newly-setup studio
  * Once in, run the `setup-recordbroadcast` program and answer its questions

From this point on, whenever the setup playbook changes, you can just run`./converge` to upgrade the studio.

#Connecting to the studio

You can connect with `./connect` and this will open Vinagre.  Alternatively
if you don't have that program, you can use a VNC client to connect to
the host name of your studio instance.

#Configuring media and applications

After configuring the instance for the first time, you need to set up Mumble
and Skype in it.  They should launch automatically.  Set them up to your
liking, and then pay attention to these important audio-related points:

- Both Skype and Mumble need to be configured to record and play through PulseAudio.
- Skype's automatic gain control (adjust input / microphone levels automatically) needs to be off.
- Mumble settings:
  - audio output should be set to Continuous,
  - Echo cancellation must be off,
  - Positional audio must be off, and
  - Amplification must be at 1.
- You may want to turn any sound effects and text-to-speech in both Mumble and Skype, so they don't interrupt you during the show.

Then exit the applications so they get a chance to save their configuration to disk.

After that, copy your show intro to `/home/ubuntu/Studio/Clips/Intro.wav`
(must be 48 KHz mono) and then copy your show outro to
`/home/ubuntu/Studio/Clips/Outro.wav`.  This completes the setup of the
media necessary to run the live show's intro and outro commands.

Then you need to set up the record and broadcast facilities.  In a terminal,
run command `recordbroadcast` and answer the questions related to your Icecast
server that you will be asked.

After you're done with these steps, either reboot or log out and log back in
to the EC2 instance (it is normal to get your VNC session disconnected when
you log off -- just connect right back and you should be able to log in).  

You only need to perform this configuration once (or when circumstances
demand that you update the configuration).

#Using the studio

Once you log back in from your first reboot, you should be able to see the
JACK suite of applications, mixer, Mumble and Skype already running. Audio
coming from Mumble will be relayed to Skype, and vice versa, so people
calling in from Skype will be able to talk with Mumble channel participants.

In addition to that, the recording and broadcast can now be started
in the background.  When you type the command `recordbroadcast`, this will
promptly broadcast your show to your Icecast server, and record it
into `/home/ubuntu/Studio/Recordings` as an Ogg Vorbis high-quality
floating-point 48 KHz audio file.

Finally, the `intro` and `outro` commands (which you can run on the terminal)
will play back on the live stream and into the Mumble and Skype audio inputs,
but not on the recorded show.

To stop recording and broadcast, run the command `pkill -INT gst-launch`.  It
is advisable to stop the recording and broadcast before downloading the file.

To start the recording and broadcast once again, run the command
`recordbroadcast`.  This starts the gst-launch GStreamer processes that
broadcast and record locally.

To download the recordings, retrieve the files in
`/home/ubuntu/Studio/Recordings` by copying them to your computer with an SFTP
client (the Decline to State publishing studio has inbuilt support for this).
In either of those cases, your SFTP client or SSH configuration must
have the private SSH key that corresponds to the public SSH key that you used
when launching the studio for the first time.  Alternatively, you could generate
a new key pair and then deploy the public key to the `ubuntu` user's SSH
`/home/ubuntu/.ssh/authorized_keys` file in the studio.

#Checklist for testing the studio functionality

There are a number of things you need to test before you can trust that the
studio will run without any technical issues.

Application test:

* Test that Mumble is:
  * started,
  * running,
  * connected to the right server,
  * joined in the right channel,
  * and broadcasting (the mouth icon is red rather than grey)
* Test that Skype is:
  * started,
  * connected,
  * logged in,
  * available
* Check the process list to see if two `gst-launch` processes are running:
  * one for the recording,
  * one for the broadcast;
  * if one is missing, try running `recordbroadcast` on the terminal and check again;
  * if one is still missing, check the credentials you supplied to the `setup-recordbroadcast` command; they are stored in the hidden file `/home/ubuntu/.recordbroadcast`.

Functionality test:

* Test that the audio from Mumble participants is going through the mixer.
  * Check the mixer window.  The Mumble in should move when people in the Mumble channel talk.
* Test that the `intro` and `outro` clips play back into the Mumble channel.
* Test that the audio from Skype callers is going through the mixer.
  * Check the mixer window.  The Skype in should move when you make a call and the other person speaks.
* Test that audio goes back and forth between Mumble and Skype.
  * Skype echo / call testing service usually does the job.
  * If Mumble participants can hear the Skype voice, and vice versa, it's all good.
* Test that the voice of everyone who is going to be in the show, actually echoes back from the Skype echo / call testing service.
  * Mumble is a bit buggy -- it will silence some people even though they do not show up as silenced.  If someone isn't being heard back on the echo / call testing service, chances are they need to be unmuted, un-server-muted or unsuppressed.
* Test that the audio is being recorded.
  * Download the latest file recorded in `/home/ubuntu/Studio/Recordings`.  You can download the audio file by using an SFTP client (which needs to have the private SSH key)
  * Open the downloaded file in an audio playback program.
  * Test that the expected audio is there.
  * Test that the `intro` and `outro` clips are not on the recording.
* Test that the audio is being broadcasted live.
  * Look at your radio station's Icecast home page.
  * Listen to the two mounts and verify that they are broadcasting.
  * Check that the `intro` and `outro` clips are played back here.

Quality test:

* Test that everyone coming in through Mumble have good sound quality.
  * You should not see the mixer column for Mumble light red and go above 0.0 dB.  This indicates that the mixer level for Mumble is too high, and needs to be reduced.  You can clear the clipping indicator on the mixer by clicking on it.
  * You should not hear people clipping (sounds like a horrible buzz) when they talk loudly.  This indicates that they have their microphone recording level too high, or they are speaking too close to the microphone.  Take corrective measures
* Test that callers coming in through Skype have good sound quality.
  * You should not see the mixer column for Skype light red and go above 0.0 dB.  This indicates that the mixer level for Skype needs to be reduced.  A good level to prevent clipping usually is around -6.1 dB.
  * You should not hear callers clipping either.  This indicates that they're speaking too close to their microphone, or that their recording level on their computer / phone is too high.  Take corrective measures.
