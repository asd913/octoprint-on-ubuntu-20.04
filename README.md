# octoprint-on-ubuntu-20.04
These are the steps and files I used to get octoprint running on ubuntu 20.04 LTS

These instructions are based on the guide developed by Chris Riley for his video "Octoprint On Linux - Install - How To - Chris's Basement" https://www.youtube.com/watch?v=fimVwRXarf4

Chris' instructions were for ubuntu 18.04. I've updated them for 20.04.

The word doc file contains all the instructions and code. Change every instance of <user> to your username. Everything in BOLD copy and run in the terminal, everything in italics, copy and paste into config files.
  
Overall, the steps are similar to Chris' video. The following is the small list of the changes I made:
  
  1.) webcam - I added an option for restarting the webcam (for testing new settings entered into webcamDaemon)
  
  2.) webcamDaemon - I've also changed his webcam output to be 1280x720. You will need to refer to the mjpg-streamer documentation to understand and choose your own settings: https://github.com/jacksonliam/mjpg-streamer/blob/master/mjpg-streamer-experimental/plugins/input_uvc/README.md
