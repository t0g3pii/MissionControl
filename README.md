<p align="left">
<img alt="GitHub" src="https://img.shields.io/github/license/ndeadly/MissionControl">
<img alt="GitHub release (latest by date)" src="https://img.shields.io/github/v/release/ndeadly/MissionControl">
<img alt="GitHub All Releases" src="https://img.shields.io/github/downloads/ndeadly/MissionControl/total">
<a href="https://ko-fi.com/J3J01BZZ6">
    <img border="0" alt="ko-fi" src="https://www.ko-fi.com/img/githubbutton_sm.svg" align="right">
</a>
</p>

# MissionControl

Use controllers from other consoles natively on your Nintendo Switch via Bluetooth. No dongles or other external hardware neccessary.

### Features
* Supports all firmware versions
* Pair Bluetooth controllers natively via `Change Grip/Order` screen
* Native button remapping (firmware 10.0.0+)
* `btdrv-mitm` sysmodule adds extension IPC commands that can be used to interact with the `bluetooth` process without interfering with the state of the system. 

### Supported Controllers
 Currently, the following controllers are supported. If you have a third-party variant of one of the below, or a Bluetooth controller that isn't listed, consider submitting an issue with the controller details, including vid/pid, to request support.

* __Nintendo Wii Remote + extensions (Nunchuck, Classic Controller, Classic Controller Pro)__
* __Nintendo WiiU Pro Controller__
* __Sony DualShock4 Controller__
* __Microsoft Xbox One S Controller__*

**Not all Xbox One wireless controllers support Bluetooth. Older variants use a proprietary 2.4Ghz protocol and cannot be used with the Switch. See [here](https://support.xbox.com/help/hardware-network/accessories/connect-and-troubleshoot-xbox-one-bluetooth-issues) for information on identifying the Bluetooth variant.*

### Requirements
* Hackable Nintendo Switch running the latest [Atmosphère](https://github.com/Atmosphere-NX/Atmosphere/releases) CFW
* Compatible Bluetooth controller

### Installation
Download the latest release .zip and extract to the root of your SD card. A console reboot is required in order for MissionControl to become active.

*__Note: Currently a modified boot2 is required to launch btdrv-mitm early enough to intercept Bluetooth initialisation. This will get overwritten any time Atmosphère is updated on SD, and will need to be replaced.__*

### Usage
Install MissionControl to your SD card, reboot the console and then pair controllers as you normally would via the `Controllers->Change Grip/Order` screen. Once paired, controllers will reconnect automatically when woken up.

Most native features *should* just work (with the exception of things like firmware update). If you find something that's broken please create an issue.

### How it works
MissionControl works by Man-In-The-Middling the `bluetooth` system module and intercepting its initialisation IPC commands and system events, and translating incoming/outgoing data to convince the Switch that it's communicating with an official Pro Controller.

To achieve this, the `btdrv-mitm` module obtains the handles to `bluetooth` system events and shared memory when the system attempts to initialise them over IPC via the `btm` and `hid` modules. It then creates its own secondary versions of these and passes their handles on instead of the original. This allows modifications to be made to any data buffers before notifying (or not) the system. Additionally, the `WriteHidData` IPC command is intercepted to translate or drop outgoing requests to the controller. In the case of the latter, fake responses can be written directly to the buffer in shared memory.

Intercepting initialisation IPC commands also allows homebrew to properly make use of the `bluetooth` service. Normally, calling any of the IPC commands that would initialise or finalise system events would either crash the console, or invalidate the event handles held by system processes. With `btdrv-mitm` we are able to hand out alternative event handles when homebrew attempts to initialise an interface, and redirect the real system events to those instead of the events held by system processes.

IPS patches to the `bluetooth` module are provided to (re)enable the passing of abitrary pincodes when Bluetooth legacy pairing is used (Nintendo hardcodes a value of `'0000'`, ignoring IPC arguments). This enables Wii(U) devices to be paired with the console.

### Building from source
First, clone the repository to your local machine and switch to the newly cloned directory
```
git clone --recurse-submodules https://github.com/ndeadly/MissionControl.git
cd MissionControl
```

MissionControl uses a custom fork of `libnx` that adds Bluetooth service wrappers and type definitions. This needs to be built and installed first

```
cd libnx
make && make install
```

Next build `Atmosphere-libs`
```
cd ../Atmosphere-libs
make
```

Finally, build and package the distribution .zip. This will build a custom `boot2`, `btdrv-mitm` and package them up with bluetooth exefs patches. 
```
cd ..
make dist
```

The resulting package can be installed as described above.

### Planned Features
* Controller management application
* Rumble support
* Motion support
* Keyboard and mouse support

### Known Issues and limitations
* Non-Switch controllers cannot be used to wake the system from sleep.
* Controllers that haven't had their hardware ID whitelisted for identification will not be recognised as connected. This can include some official Switch controllers. They will however still pair with the console and store their details to the database. If you encounter such a controller, please create an issue requesting support. 
* Wii(U) controllers can take a while to pair with the console. For some reason they are only detected at the end of a device discovery cycle. Be patient and re-press the sync button on the controller if neccessary.
* Xbox One button layout was changed at some point in a firmware update. Please ensure your controller firmware is up to date if you have issues with incorrect button mappings.

### Credits
* [__switchbrew__](https://switchbrew.org/wiki/Main_Page) for the extensive documention of the Switch OS.
* [__devkitPro__](https://devkitpro.org/) for the homebrew compiler toolchain.
* __SciresM__ for his dedicated work on the [Atmosphère](https://github.com/Atmosphere-NX) project, libstratosphere and general helpfulness with all things Switch related.
* __misson20000__ for his handy debug monitor [Twili](https://github.com/misson20000/twili) and IPC logger [Ilia](https://github.com/misson20000/ilia)
* __dekuNukem__, __CTCaer__, __shinyquagsire23__ and others for their work in reversing and documenting the switch controller communication protocol.
* __friedkeenan__ for helping to test Wii extension controller support.
* Everyone else over at the __ReSwitched__ discord server who helped answering technical questions.
