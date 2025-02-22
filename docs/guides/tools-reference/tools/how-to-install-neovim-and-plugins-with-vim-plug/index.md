---
slug: how-to-install-neovim-and-plugins-with-vim-plug
description: 'This guide shows you how to install NeoVim, a plugin manager, and plugins that help it replace the vim text editor.'
og_description: 'With refactored code and better, asynchronous plugin management, NeoVim is literally the future of vim. This guide shows you how to install NeoVim, a plugin manager, and plugins that help it replace vim to become your new favorite text editor.'
keywords: ["neovim", "text", "editor", "vim", "nvim", "plugins"]
aliases: ['/tools-reference/how-to-install-neovim-and-plugins-with-vim-plug/','/tools-reference/tools/how-to-install-neovim-and-plugins-with-vim-plug/']
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
published: 2017-10-02
modified: 2022-12-27
modified_by:
  name: Linode
title: How to Install NeoVim and Plugins with vim-plug
external_resources:
 - '[NeoVim official website](https://neovim.io)'
 - '[Neovim-completion-manager](https://github.com/roxma/nvim-completion-manager)'
 - '[Far.vim](https://github.com/brooth/far.vim)'
tags: ["linux"]
image: neovim-basics-title.jpg
authors: ["Gabriel Cánepa"]
---

## What is NeoVim?

If you are a system administrator or a software developer, a robust text editor is among the tools you use on a daily basis. It's likely that you've used the vi or vim editors that have served the Unix and Linux communities for decades.

Although vim is under active development, it includes some ~300k lines of [C89](https://en.wikipedia.org/wiki/ANSI_C#C89) code. In addition to being difficult to maintain, issues and new pull requests cannot be addressed very easily since Bram Moolenaar is the only person who maintains the large codebase of the program.

These difficulties, along with the lack of support for some desired features such as asynchronous plugins, motivated the birth of NeoVim as a fork of vim. The project's main objective is to completely refactor vim so that maintenance can be simplified, and new features and bug fixes can be quickly added to the source code.

## What To Expect From NeoVim?

During NeoVim's short life, two outstanding features have already been implemented: asynchronous plugins, and the Remote Procedure Call (RPC) API for controlling NeoVim programmatically.

Asynchronous plugins make it possible for plugins to run as background processes without interfering with the main editor process.

The RPC API allows GUI programs (and other software that speaks the *msgpack-rpc* messaging protocol) to connect to a running NeoVim instance. This means that you can integrate well-known text editors such as Atom, Visual Studio Code, or Sublime Text with NeoVim and have modifications sync bidirectionally between them. That way, you can leverage all the features of the GUI program while using nvim's engine behind the scenes.

This guide details the installation and configuration of NeoVim, along with two asynchronous plugins, `nvim-completion-manager` and `far.vim`.

## Before You Begin

1. If you have not already done so, create a Linode account and Compute Instance. See our [Getting Started with Linode](/docs/products/platform/get-started/) and [Creating a Compute Instance](/docs/products/compute/compute-instances/guides/create/) guides.

1. Follow our [Setting Up and Securing a Compute Instance](/docs/products/compute/compute-instances/guides/set-up-and-secure/) guide to update your system. You may also wish to set the timezone, configure your hostname, create a limited user account, and harden SSH access.

1. Install the necessary tools:

    - In CentOS and similar distributions, enable the [Extra Packages (EPEL) repository](https://fedoraproject.org/wiki/EPEL), then install the packages:

        ```command
        sudo yum install epel-release -y
        sudo yum install wget fuse fuse-libs git ack python34-pip -y
        ```

    - In Debian, Ubuntu, and their derivatives:

        ```command
        sudo apt-get install fuse libfuse2 git python3-pip ack-grep -y
        ```

In this tutorial we install NeoVim by downloading an appimage, a binary file that contains the application and its dependencies (with the exception of the [FUSE libraries](https://github.com/AppImage/AppImageKit/wiki/FUSE), which need to be installed separately). This method is distribution-agnostic and provides the latest version of the package.

Note that the NeoVim appimage is currently only available for 64-bit systems. If you are using a different architecture, or would like to see if NeoVim is available from your distribution's repositories, you may refer to the installation instructions in the [NeoVim wiki](https://github.com/neovim/neovim/wiki/Installing-Neovim).

While FUSE libraries make it possible for the appimage to be run by non-root users even from their home directories, we make the program available for all users by placing it inside `/usr/bin`. This will allow users to utilize the same application but configure it to meet their specific needs.

## Install the NeoVim Appimage

1. Download and install the appimage, use the `output-document` option to rename it to `nvim`:

    ```command
    wget --quiet https://github.com/neovim/neovim/releases/download/nightly/nvim.appimage --output-document nvim
    ```

1. Set the owner to `root`, and make nvim accessible to all users:

    ```command
    chmod +x nvim
    sudo chown root:root nvim
    ```

1. Move the binary file to `/usr/bin`:

    ```command
    sudo mv nvim /usr/bin
    ```

1. Move into your home directory and create the subfolder structure to store the configuration file:

    ```command
    cd ~
    mkdir -p .config/nvim
    ```

## Install the Vim-plug Plugin Manager

To make it easier to install plugins, use the Vim-plug plugin manager. This plugin manager uses `git` to manage most plugins:

```command
curl -fLo ~/.local/share/nvim/site/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

## (Optional) Import Existing vim Configuration

If you already have a vim configuration you enjoy, reutilize it for nvim. While vim and nvim can share the same configuration file, create a separate one for nvim to prevent errors if you don't have `Vim-plug` also configured for vim:

```command
ln -s ~/.vimrc ~/.config/nvim/init.vim
```

Or create a separate one beginning with the current contents of `~/.vimrc`:

```command
cp ~/.vimrc ~/.config/nvim/init.vim
```

## Run NeoVim

Launch nvim:

```command
nvim
```

To exit without saving changes, press the **ESC** key to enter Command mode, then:

```command
:q!
```

## Install NeoVim Plugins

### Nvim-completion-manager Plugin

*nvim-completion-manager* is a fast, extensible completion framework that supports a variety of programming languages and snippet solutions. Some of these are supported out of the box, while others require the installation of extra Python 3 modules to work. In this guide we illustrate the use of this plugin with [UltiSnips](https://github.com/SirVer/ultisnips), a robust snippet solution.

1. Install the NeoVim Python module:

    ```command
    pip3 install --user neovim
    ```

1. Add the following lines at the bottom of your `~/.config/nvim/init.vim` file to include the snippets available through UltiSnips and [vim-snippets](https://github.com/honza/vim-snippets):

    ```file {title="~/.config/nvim/init.vim" lang=aconf}
    call plug#begin()
    Plug 'ncm2/ncm2'
    Plug 'SirVer/ultisnips'
    Plug 'honza/vim-snippets'
    call plug#end()
    ```

1. Launch nvim, execute `PlugInstall`, update the plugins, and exit:

    ```command
    nvim
    :PlugInstall
    :UpdateRemotePlugins
    :q!
    :q!
    ```

1. The plugin will be ready for use after you restart nvim. To test it, create a `.py` file named `helloworld.py` as follows:

    ```command
    nvim helloworld.py
    ```

    Press **i** to enter Insert mode, and type `def`. You should be presented with a dropdown list similar to that shown in the image below. Highlight one of the options using the up and down arrows in your keyboard and press the *Tab* key. The code snippet will be inserted into the body of the file:

    ![Neovim autocomplete snippets](neovim-autocomplete-snippets.png)

    For more examples using other programming languages, refer to the [plugin documentation](https://github.com/SirVer/ultisnips/blob/master/doc/UltiSnips.txt).

### Far.vim Plugin

*far-vim* is a plugin for performing asynchronous search and replace operations on a set of files (typically within the same directory).

1. Insert the following line before `call plug#end()` in `~/.config/nvim/init.vim`:

    ```file {title="~/.config/nvim/init.vim"}
    Plug 'brooth/far.vim'
    ```

1. Open nvim and execute `PlugInstall`, update the plugins, and exit. The plugin will be available when you restart nvim:

    ```command
    nvim
    :PlugInstall
    :UpdateRemotePlugins
    :q!
    :q!
    ```

1. To test `Far.vim`, create a directory named `myproject` and create two files within the directory:

    ```command
    mkdir myproject
    cd myproject
    ```

    ```file {title="myproject/greeting.py" lang=python}
    def greet(name):
        print('Hello', name)
    ```

    and

    ```file {title="myproject/persons.py" lang=python}
    #!/usr/bin/python3
    from greeting import greet
    # Import the greet function from greeting.py
    # Pass a name to greet()
    greet('Jack')
    greet('Jill')
    greet('Bob')
    ```

1. Open either file with nvim and use Command mode:

    ```command
    :Far name nombre .py$ --source=acknvim
    ```

    The nvim window will split into two panes. The left one shows the file and the one in the right shows the results if the changes are applied in all the `.py` files inside the current directory. Apply the changes with `Fardo` (still in Command mode):

    ```command
    :Fardo
    ```

    To undo the changes:

    ```command
    :Farundo
    ```

### Neomake - An Alternative to make

You may also want to take a look at [neomake](https://github.com/neomake/neomake), a plugin similar to syntastic in vim. Where syntastic may freeze vim while it checks the syntax of a large file every time you save changes, neomake can perform the same function without causing any interruptions.

## Roadmap and Future

There are more than 300 developers contributing to NeoVim in GitHub and it is expected that the project will grow at a faster pace than it has done in the past. Visit the official GitHub to view [milestones](https://github.com/neovim/neovim/milestones) and a [roadmap](https://neovim.io/roadmap/) for the next version.
