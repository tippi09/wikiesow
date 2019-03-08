# Basic macOS setup

In this guide I will explain my basic macOS setup which includes my [Vim](https://www.vim.org) and zsh configuration, as well as the installment of useful software like [Homebrew](https://brew.sh/index_de).

## Install Homebrew

Sadly macOS does not include a preinstalled package manager. This problem can be solved with Homebrew, which can be installed easily by running:
```console
# /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
When the installation process has finished you can install and uninstall software simple and clean by using `brew install` or `brew install`. To search for a software package, just type `brew search`.

## Install thefuck

With Homebrew, we can now install [thefuck](https://github.com/nvbn/thefuck), a handy tool which is able to correct your previous console command.
```console
# brew install thefuck
```

??? note "Configure thefuck in your shells startup script"
    This step is only necessary if you do not want to use my zsh setup described here. In that case, you have to add the following command in your `.bash_profile`, `.bashrc`, `.zshrc` or another startup script:
    ```console
    # eval $(thefuck --alias)
    ```
    Now you should be able to type `fuck` in your terminal and get your commands corrected if they fail.
The evaluation of `thefuck` is already contained in the following `.zshrc` configuration I am using. If you think `fuck` is a too vulgar alias for this command, you are of course free to change it to anything you like.

## Configure Vim and zsh

For my Vim and zsh configuration I am using the [.dotfiles repository by rngcntr](https://github.com/rngcntr/.dotfiles). First we have to clone the repository:
```console
# cd ~
# git clone https://github.com/rngcntr/.dotfiles.git .dotfiles
# cd .dotfiles/
# git submodule init
# git submodule update
```

Now we can create symlinks to use the configuration files in our `~/.dotfiles` directory. First we set our `.vimrc`, install [Vundle](https://github.com/VundleVim/Vundle.vim) with plugins and afterwards link the `.zshrc`:
```console
# cd ~
# git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
# ln -s .dotfiles/.vimrc .vimrc
# vim +PluginInstall +qall
# ln -s .dotfiles/.zshrc .zshrc
```

You can check if the links work by running `ls -l .vimrc` and `ls -l .zshrc` in your home directory. You should see an output that looks similar to this:
```
.vimrc -> .dotfiles/.vimrc
.zshrc -> .dotfiles/.zshrc
```
Now we improve the terminal experience by installing and configuring [rngcntr](https://github.com/rngcntr)'s fork of [oh-my-zsh](https://github.com/rngcntr/oh-my-zsh) ([original by robbyrussel](https://github.com/robbyrussell/oh-my-zsh)):
```console
# cd ~
# ln -s .dotfiles/.oh-my-zsh .oh-my-zsh
# ./.oh-my-zsh/tools/install.sh
```

After installing oh-my-zsh the zsh should be set as your default shell. If this is not the case, just run `chsh -s $(which zsh)`.
