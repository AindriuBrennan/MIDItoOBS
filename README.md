# Midi OBS what???
This script let's you use one or multiple MIDI controller like the Novation Launchpad, Ableton Push, Akai LPD or the Korg nanoPAD to switch scenes, start/stop recording/streaming, control volume/delay/transition time and more in [obs-studio](https://github.com/obsproject/obs-studio).

**Important**: If you upgrade to this version, your old configuration file is no longer usable and you have to re-create the entire mapping!! This is because of the new multi-device feature. (If this is a huge problem for you let me know and i'll tell you the steps to migrate by hand)

## Requirements

- Obviously a (USB) MIDI controller
- Python 3
- A few pip packages
  - [TinyDB](https://github.com/msiemens/tinydb)
  - [mido](https://github.com/olemb/mido)
  - [python-rtmidi](https://pypi.python.org/pypi/python-rtmidi)
  - [websocket-client](https://github.com/websocket-client/websocket-client)
- The [obs-websocket plugin](https://github.com/Palakis/obs-websocket/releases) (Version >= 4.6.1)
  
## Setup Part 1

- Install Python 3.x.x (whatever the latest version is)
  - On Windows: Make sure you trick "Add Python 3.x to PATH" in the setup
- Make sure you also install pip
- For instructions how to install TinyDB click [here](https://tinydb.readthedocs.io/en/latest/getting-started.html#installing-tinydb)
- For instructions how to install mido and python-rtmidi click [here](https://github.com/olemb/mido#installing)
- For instructions how to install websocket-client click [here](https://github.com/websocket-client/websocket-client#installation)

If you want to install all packages in one go, run "pip install -r requirements.txt"

## Setup Part 2 OBS Websocket

- Download the installer and run it
- Start OBS, open the "Tools" menu and select "websocket server settings"
- Make sure that "Enable Websocket server" is checked, "Server Port" is 4444, "Enable authentification" is unchecked and "Enable System Tray Alerts" is unchecked(trust me, you don't want that on)

## Setup Part 3

- [Download the repository](https://github.com/lebaston100/MIDItoOBS/archive/master.zip) or clone it
- Connect your MIDI controller
- Launch obs-studio
- Launch the setup.py (Try double click or the "Run Setup.bat" if you are on Windows)
- If you run the setup for the first time and have no yet setup a device yet, it will automatically start the device configuration:
  - You will get a list of available MIDI devices. Type the number you want to select and press Enter
  - You will be asked if you want to ad another device.
  - If you only have a single device choose 2 and press enter, otherwise select 1 and you will get a list with remaining devices.
- Now you will be asked to press a button or move a fader on your MIDI controller, do that
- If your midi controller sends control change messages, you will also be asked for the type of the input(fader or button)
- Select an action from the list and press enter. The names represent the request-type in obs-websocket
- Depending on the action, you will also be asked for the scene and source name (selecting always works by typing in the number and pressing enter). If no source of that type is available and you are promted to "select 0--1:" then you know that is no such source available in obs and the script will crash trying to select anything. Just add the required object and restart the setup script in this case. (This is already on the todo list for a further update)
- Available for buttons:
  - SetCurrentScene: Switches to the scene
  - SetPreviewScene: Puts a scene into preview when in studio mode
  - TransitionToProgram: Transitions the current preview scene to program
  - SetCurrentTransistion: Sets the transition that is used with SetCurrentScene
  - SetSourceVisibility: Hides or unhides a source
  - ToggleSourceVisibility: Toggles the visibility of a source
  - ToggleMute: Toggles the mute status from a source
  - SetMute: Mutes or unmutes a source
  - StartStopStreaming: Toggles the Streaming
  - StartStreaming: Starts streaming
  - StopStreaming: Stops streaming
  - StartStopRecording: Toggles the Recording
  - StartRecording: Starts recording
  - StopRecording: Stops recording
  - StartStopReplayBuffer: Toggles the replay buffer
  - StartReplayBuffer: Starts the replay buffer
  - StopReplayBuffer: Stops the replay buffer
  - SaveReplayBuffer: Save the replay buffer
  - SetTransitionDuration: Sets the length of the currently selected transistion if supported(fade)(in ms) to a predefined value
  - SetCurrentProfile: Changes to the selected obs profile
  - SetCurrentSceneCollection: Changes to the selected obs scene collection
  - ResetSceneItem: Resets a scene item
  - SetTextGDIPlusText: Sets the text of a GDI text source
  - SetBrowserSourceURL: Sets the url of a BrowserSource
  - ReloadBrowserSource: Reloads a BrowserSource
  
- Available for faders
  - SetVolume: Sets the volume of a source (unlike other solutions this will actually make the fader move in a visual linear way inside obs(Like a % slider))
  - SetSyncOffset: Sets the sync offset of a source(in ns)
  - SetSourcePosition: Sets the x or y position of a source (in px)
  - SetSourceRotation: Sets the rotation of a source (in degree)
  - SetSourceScale: Sets the scale for x/y or both of a source (For the scaling 1 = original scale)
  - SetTransitionDuration: Sets the length of the currently selected transistion if supported(fade)(in ms)
- Now you can either setup another button/fader by repeating the steps above(except starting the script again) or just close the window to exit the configuration
  
For a detailed description of most of the commands see the [obs-websocket protocol documentation](https://github.com/Palakis/obs-websocket/blob/master/docs/generated/protocol.md)

### Device Management

If you run the setup another time after the inital configuration you will get a dialog at startup where you can select if you want to go to the device management (1) or just continue adding new button/fader assignments with the already configured devices.

If you select 1 you have a few options:
- 1: Delete all devices from the database without removing their mapping. This does exactly that and be warned, will cause a device mixup when you add more devices later. You'll be better of using option 2
- 2: Remove a single device and their assignments.
- 3: Add a new device. This allows you do add more devices.
- 4: Skip device configuration. This exits the device management without changing anything and continues with the assignment dialog.
  
### Understanding input scaling

A midi value can be something between 0-127. That is a very limited number.

You will only be asked for Input Scale setup if it's required for the function(SetSourcePosition, SetSourceRotation, SetSourceScale, SetSyncOffset, SetTransitionDuration).

The first value you have to enter(lower output value) is the value that will be sent when the fader is sending a 0. The second value you have to enter(higher output value) is the value that will be sent when the fader is sending a 127. The range between the 2 numbers will be interpolated linearly.

## Sidenotes:

- You can change the IP and Port by modifying the main.py(line 5/6) and setup.py(line 6/7) with a text editor if the script is running on another PC on the network. 

## Setting up "macros" (optional):

You can assign unlimited different actions to the same button. This requires editing the config. (Sounds harder then it is)

 - Setup the functions as described above on different buttons
 - Now stop the setup.py and open the config(config.json) with a text editor.
 - Change the "msgNoC" value of the buttons you want to combine to the value of the button you want to use. Make sure you have the entry with the right device ID.
 - Here are some pictures for better understanding: [Step 1](https://cdn.lebaston100.de/git/midiobs/miditoobs_1.png) [Step 2](https://cdn.lebaston100.de/git/midiobs/miditoobs_2.png)
 - Now save and close the config file
 - Start main.py and verify that it works

## Using it (!Very important!)

- If you're a first time use make sure to follow setup steps 1-3
  - You can launch setup.py anytime(as long as main.py is not running) to change the configuration of a single button/fader without reconfiguring the whole controller.
- Always make sure that obs is running before launching any of the scripts
- Launch the main.py file (Try double click or the "Run Main.bat" if you are on Windows)
- The console gives you information when it successfully connects to OBS
- Also, if there is an error it will be printed out(If you ignore case sensitive fields or the scene doesn't exist)
- Third, it prints out a message every time you press a button that is setup
- Now just leave it running in the background
- To stop the program simply close the window (or CTRL + C)

## Troubleshooting

A user has reported that under certain circumstances the script(setup and main) will crash after start on Windows with "ImportError: DLL load failed: The specified module could not be found".
If this happens to you, plase install the Visual C++ Redistributable from Microsoft. Make sure you get the x86 version if you are using python 32bit(Which is default) ([Download](https://aka.ms/vs/15/release/vc_redist.x86.exe))

If you have any other problem, just open a Github issue or DM me @lebaston100 (Twitter/Instagram)

## Contributors

I had never imagined that so "many" people would contribute something to the project. Thanks to everyone who submitted a bug report or pull request.
Special thanks to:

- [ptitodd](https://github.com/ptitodd) (Adding program_change message handling)
- [asquelt](https://github.com/asquelt) (making it work in python2)
- [Alex-Dash](https://github.com/Alex-Dash) (make the volume control linear)
- [imcrazytwkr](https://github.com/imcrazytwkr) (completly refactoring the main.py)
- [juliscrazy](https://github.com/juliscrazy) (fix typo in readme)


### Tested on/with:

- Win 10 Build 18362
- Python 3.7.0:1bf9cc5093
- obs-studio 23.2.1
- obs-websocket 4.6.1
- KORG nanoPAD
- Hercules DJ Control MP3
- Behringer X-Touch Mini (tested by [me-vlad](https://github.com/me-vlad))

Let me know if you had success with your device.
