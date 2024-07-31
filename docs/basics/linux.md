# 使用Linux服务器

## 命令行(CLI)、终端(Terminal)与Shell

与Windows或Mac OS使用的图形界面不同，命令行是Linux服务器中最主要的人机交互方式。顾名思义，在命令行中，我们使用以行为最小单位的文本指令来操作计算机。在这一使用方式下，我们需要通过终端向系统提交指令并接收反馈，而Shell则是系统用于解读文本指令的程序。

在历史上，终端曾是一种专用的输入设备，而在现代计算机中，我们通常把提供命令行交互界面的软件称为终端。终端软件运行在本地的计算机而不是服务器上，好的终端软件可以为我们提供更高效的人机交互和更美观易理解的界面，大家可以根据自己使用的操作系统选择合适的终端软件。

运行终端后，我们通过Shell与计算机交互，我们通过终端输入的文本指令被Shell接收后，计算机根据Shell解释的结果执行操作。当我们直接运行终端时，终端默认连接到本地的Shell，而当我们要使用远程服务器时我们会通过下文介绍的SSH连接到远程服务器的Shell，从而与远程服务器交互。

不同的Shell，如Linux中常用的Bash，zsh即不同的指令解释器，可能具有不同的指令和语法，支持不同的特性，但Linux和MacOS上的大多数Shell都遵循一套共同的语法标准，即[Unix shell language](https://en.wikipedia.org/wiki/Unix_shell)，我们只需要掌握这些语法即可满足几乎所有场景的使用需要。

!!! tip 
    在计算机术语中，本地(local)指你当前使用的计算机，远程(Remote)则指你通过网络连接的计算机。

=== "Windows10/11"

    我们推荐在windows上使用[Windows Terminal](https://github.com/microsoft/terminal?tab=readme-ov-file#installing-and-running-windows-terminal)，它由微软开发，提供了丰富的功能和高度可定制的外观。

    在Windows中默认的Shell是CMD或者PowerShell，它们也支持部分最基础的Unix shell language，但更多方面还是独立于Unix体系，自成一体。不过，当我们连接到远程主机上，我们使用的就是远程主机上的Shell，与本地的Shell无关，因此即使我们本地的计算机使用的是Windows系统，我们也无需掌握这部分知识。

=== "MacOS"

    TODO

## SSH连接

在上一节中我们提到，如果我们将本地的终端连接到远程服务器的shell，就可以与远程服务器交互。SSH就是一种用于连接远程shell的网络协议，这一小节我们将介绍如何使用本地的ssh工具连接到远程服务器。

### 使用OpenSSH

OpenSSH是Windows、MacOS和Linux上最常用的SSH工具，通常由系统自带，你可以在本地终端中输入以下指令确认软件是否存在:

```shell
$ ssh
usage: ssh [-46AaCfGgKkMNnqsTtVvXxYy] [-B bind_interface]
           [-b bind_address] [-c cipher_spec] [-D [bind_address:]port]
           [-E log_file] [-e escape_char] [-F configfile] [-I pkcs11]
           [-i identity_file] [-J [user@]host[:port]] [-L address]
           [-l login_name] [-m mac_spec] [-O ctl_cmd] [-o option] [-p port]
           [-Q query_option] [-R address] [-S ctl_path] [-W host:port]
           [-w local_tun[:remote_tun]] destination [command]
```

如果正确安装，你会看到以上信息，这段信息是对ssh指令的使用说明，具体含义我们会在下文中介绍。

!!! tip
    在unix shell相关的语境下，如果一行以`$`符号作为开头，则表示这个符号的后续内容是用户提交的指令，这个符号本身不需要你输入到终端中。反之，如果某行不以`$`符号起始，则表示这行是由系统输出的内容。

部分版本的Windows和Linux中没有预置OpenSSH，你需要手动安装。

=== "Ubuntu"

    ```shell
    sudo apt-get install openssh-client
    ```

=== "Windows7"

    在Windows10之前的版本中，系统并不预装OpenSSH，微软对windows7提供了官方的支持，可以参考[Win32-OpenSSH](https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH-Using-MSI)这一页面中的说明进行安装

### 主机(Host)、用户(User)与认证(Authentication)

使用SSH连接的基本指令是`sh [username@]hostname`，其中hostname称为主机名，用于指定你要访问的远程服务器；username称之为用户名，是远程服务器上已经注册的一个用户账户。

### 使用vscode

## 常用Linux命令和操作

## 文件系统和目录权限

## Shell和环境变量