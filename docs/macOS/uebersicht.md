# Übersicht: System monitor configuration
With [Übersicht](http://tracesof.net/uebersicht/) you are able to configure an active system monitor on your macOS desktop. I have a [repository](https://github.com/tippi09/uebersicht-widgets) containing all the Übersicht plugins I am using in my personal arrangement.

!!! note
    I do not own all of the widgets contained in my repository but mainly made adjustments to them. Most of the widgets I use can originally be found [here](http://tracesof.net/uebersicht-widgets/).
First of all it is recommended to install Übersicht by using [Homebrew](https://brew.sh/index_de):
```console
# brew cask install ubersicht
```
After installing you can create a directory for Übersicht in `/Applications/` to keep your plugins organized with the app itself:
```console
# cd /Applications/
# mkdir Übersicht && mv Übersicht.app Übersicht/
# cd Übersicht/
```
Now clone my [uebersicht-widgets repository](https://github.com/tippi09/uebersicht-widgets) in a directory called `widgets-repository`:
```console
# git clone git@github.com:tippi09/uebersicht-widgets.git widgets-repository
```
To separate your own plugins from my repository we first create another directory called `widgets-enabled`: 
```console
# mkdir widgets-enabled && cd widgets-enabled
```
Now create a symlink from the `widgets-repository` to the `widgets-enabled` directory.
```console
# ln -s ../widgets-repository/* .
```
**Now start Übersicht for the first time** and set `/Applications/Übersicht/widgets-enabled` as widget folder in the preferences. You can add your own plugins in the `widgets-enabled` directory and update my widget repository in `widgets-repository`.
