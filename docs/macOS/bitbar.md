# BitBar: Menu bar plugins
With [BitBar](https://getbitbar.com) you are able to put anything in your macOS menu bar. I have a [repository](https://github.com/tippi09/bitbar-plugins) containing all the BitBar plugins I am using, including a plugin which makes it easy to control my Kodi configuration from [this guide](../Raspberry_Pi/semihead.md#configure-kodi).

First of all it is recommended to install BitBar by using [Homebrew](https://brew.sh/index_de):
```console
# brew cask install bitbar
```
After installing you can create a directory for BitBar in `/Applications/` to keep your plugins organized with the app itself:
```console
# cd /Applications/
# mkdir BitBar && mv BitBar.app BitBar/
# cd BitBar/
```
Now clone my [bitbar-plugin repository](https://github.com/tippi09/bitbar-plugins) in a directory called `plugins-repository`:
```console
# git clone git@github.com:tippi09/bitbar-plugins.git plugins-repository
```
To separate your own plugins from my repository we first create another directory called `plugins-enabled`: 
```console
# mkdir plugins-enabled && cd plugins-enabled
```
Now create symlinks from the `plugins-repository` to the `plugins-enabled` directory. You can choose if you would like to use all of my BitBar plugins:
```console
# ln -s ../plugins-repository/* .
```
Or just some of them like my Kodi addon:
```console
# ln -s ../plugins-repository/kodi.sh .
```
**Now start BitBar for the first time** and set `/Applications/BitBar/plugins-enabled` as plugin folder. You can add your own plugins in the `plugins-enabled` directory and update my plugin repository in `plugins-repository`.
