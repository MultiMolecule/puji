---
summary: Linux服务器使用基础
authors:
  - Gengyuan Hu
date: 2024-08-06
---

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

    MacOS自带的终端软件是Terminal.app，它直接支持Unix shell language。我们可以通过以下方式访问：

    - 直接在Spotlight搜索中输入“Terminal”打开
    - 或者通过Finder -> 应用程序 -> 实用工具找到并打开Terminal

    默认情况下，MacOS使用Bash作为Shell。如果你希望更换到更现代的zsh，可以参考官方文档完成安装和配置。

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

如果正确安装，你会看到以上信息，这段信息是对ssh指令的使用说明，重要的用法我们会在下文中介绍。

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

在SSH指令中，`ssh [username@]hostname`是基本格式，其中：

- `hostname`: 远程服务器的主机名或IP地址。
- `[username@]`: 可选部分，指定远程服务器上的用户名。

如果省略了`[username@]`，那么默认会使用在本地计算机上登录的用户账户作为远程服务器上的用户名。

通常，服务器需要以一定的方式验证你的身份来保证你的数据安全，这个过程称为认证。认证方式有两种：密码(passward)和密钥(SSH key)。密码认证是最常见的认证方式，你只需要在连接时输入正确的密码即可；而密钥认证通过一对软件生成的公钥和私钥来验证你的身份，只要你正确配置了密钥，在连接时ssh会自动使用你的私钥进行认证，无需你手动输入密码。出于安全和方便考虑，我们强烈建议大家使用密钥认证。

### 配置ssh Key

=== "Linux/MacOS"
    首先，你需要确认本地是否已经有`SSH key`，如果有，那么你就无需重新生成。在本地shell中输入以下指令:
    ```shell
    $ ls -al ~/.ssh
    drwxr-xr-x  2 username username 4096 Aug  5 21:49 .
    drwxr-x--- 21 username username 4096 Aug  5 21:52 ..
    -rw-------  1 username username 2602 Aug  5 20:13 id_rsa
    -rw-------  1 username username  568 Aug  5 20:13 id_rsa.pub
    ```
    如果存在`id_rsa`和`id_rsa.pub`文件，则说明你已经生成过SSH key了。如果不存在，则输入以下指令生成:
    ```shell
    $ ssh-keygen -t rsa
    Generating public/private rsa key pair.
    Enter file in which to save the key (/path/to/home/.ssh/id_rsa):    # 直接回车使用默认路径
    Enter passphrase (empty for no passphrase):                         # 直接回车不对SSH-key二次加密
    Enter same passphrase again:                                        # 直接回车确认密码为空
    Your identification has been saved in /path/to/home/.ssh/id_rsa.
    Your public key has been saved in /path/to/home/.ssh/id_rsa.pub.
    The key fingerprint is:
    SHA256:AFPEOVIUXhe09rwYXGzh3MvtBc5r20va+kBgkW5hLnQ user@localhost   # 这里显示的是公钥的指纹，帮助人快速识别公钥
    The key's randomart image is:                                       # 根据公钥生成的随机图像，帮助人快速识别公钥
    +---[RSA 2048]----+
    |    o**o..+oo    |
    |    .++. o E.o   |
    |     .o.. BoB o  |
    |       . +.O.+ + |
    |        S = o.= o|
    |           o.. o.|
    |          . ..o..|
    |             .=o |
    |             o++o|
    +----[SHA256]-----+
    ```

=== "Windows10/11"
    首先，你需要确认本地是否已经有`SSH key`，如果有，那么你就无需重新生成。在本地shell中输入以下指令:
    ```powershell
    PS C:\Users\username> ls ~/.ssh
    -a----         2024/8/5     20:13           2602 id_rsa
    -a----         2024/8/5     20:13            568 id_rsa.pub
    ```
    如果存在`id_rsa`和`id_rsa.pub`文件，则说明你已经生成过SSH key了。如果不存在，则输入以下指令生成:
    ```powershell
    PS C:\Users\username> ssh-keygen -t rsa
    Generating public/private rsa key pair.
    Enter file in which to save the key (C:\Users\username/.ssh/id_rsa):    # 直接回车使用默认路径
    Enter passphrase (empty for no passphrase):                         # 直接回车不对SSH-key二次加密
    Enter same passphrase again:                                        # 直接回车确认密码为空
    Your identification has been saved in C:\Users\username/.ssh/id_rsa.
    Your public key has been saved in C:\Users\username/.ssh/id_rsa.pub.
    The key fingerprint is:
    SHA256:AFPEOVIUXhe09rwYXGzh3MvtBc5r20va+kBgkW5hLnQ user@localhost   # 这里显示的是公钥的指纹，帮助人快速识别公钥
    The key's randomart image is:                                       # 根据公钥生成的随机图像，帮助人快速识别公钥
    +---[RSA 2048]----+
    |    o**o..+oo    |
    |    .++. o E.o   |
    |     .o.. BoB o  |
    |       . +.O.+ + |
    |        S = o.= o|
    |           o.. o.|
    |          . ..o..|
    |             .=o |
    |             o++o|
    +----[SHA256]-----+
    ```

现在，你在本地目录`.ssh`下至少拥有了两个文件:

- `id_rsa`: 私钥文件，保存在本地，用于身份验证。
- `id_rsa.pub`: 公钥文件，你需要将它上传到远程服务器上，用于身份验证。

你可以用文本编辑器查看它们的内容：

=== "id_rsa.pub"
    公钥文件的内容是一行字符串，形如:
    ```
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC6YhZoTIm2Jkf1pD3nVZL/ZuVF2NFAzOr6cSesI2HIaAnhj0GS4Bj84KYM+9+fDVFay4H3CG8Rf+puiSD+4XEMUuer6f66yQALqo51LzMhxjDwVYq5BQ4Ek4+bWK84p1JS6AuieWXzZMa2fkz2Kwqik7V53akp33lnw/vxBpP4pAjTLP9lmRCk9RnpUsTpWzewIZiqIXBrJpZv68CuO8jzW574nGzs8KSSF/kGZXGiCvoigj2uAfbSicRljedA+fOKWe8N0CKwEAzobCIDfHzp8zzWyENrzdu1cn4ZN19F9bt7CmYmVWQSmrqvouP3Pf+Ce0IgjSZBdsBFv2pdiTkT username@localhost
    ```

=== "id_rsa"
    私钥文件的内容是很多行字符串，形如:
    ```
    -----BEGIN RSA PRIVATE KEY-----
    MIIEpAIBAAKCAQEAumIWaEyJtiZH9aQ951WS/2blRdjRQMzq+nEnrCNhyGgJ4Y9B
    kuAY/OCmDPvfnw1RWsuB9whvEX/qbokg/uFxDFLnq+n+uskAC6qOdS8zIcYw8FWK
    uQUOBJOPm1ivOKdSUugLonll82TGtn5M9isKopO1ed2pKd95Z8P78QaT+KQI0yz/
    ZZkQpPUZ6VLE6Vs3sCGYqiFwayaWb+vArjvI81ue+Jxs7PCkkhf5BmVxogr6IoI9
    rgH20onEZY3nQPnzilnvDdAisBAM6GwiA3x86fM81shDa83btXJ+GTdfRfW7ewpm
    JlVkEpq6r6Lj9z3/gntCII0mQXbARb9qXYk5EwIDAQABAoIBAQCSi5cEsNFCd7zy
    pg3KO12WFQhGL+Dvq29CNQA1d6hlk2/ZevLbEfpzsgq9gLjl7Om/ku2AF5CE2Oex
    u35HCWkCgJkJcbVIlcvEYHkcKF1yu8s03H1zVkccUA2E3mj/CYhQCYVEXWFMyzr2
    uD24/ESjabIxvJhKhRyG+vC4JSzpPG9V8ZQrerkLAT7xSNZTXqkDCbSnspcI6yPb
    iOa7Bw0sQnSJIVciisT0sINteqKSd6S5TqSB9PsIgsmvbBk9fkCjynNE4bY3ewCC
    WShyliCO6RrBr2tShMLDfsxmoLUrZlxWBeoKYmVZRs+uWBKVdHjBWAlQMH+ivrV6
    8B4ADhQBAoGBAPDOIsHQhnKieTfFxTBNKekXzIctMytSONg5/E82vohGqsRKZXr+
    41Zag/M5vDA+JStBtSMO6IDU2Rw+Hn1gHnTPKnfRx/aCLIeytax//A5QooJ6ORYx
    7WuLk1mBZNNAoZwjdiKpnimNyWWrrPX4aIBVtkK3Vf2KwuZIyOI1R9sTAoGBAMYk
    11E0t6i6KYXZaRtZKdQCX1BjBqZLlliBfvl/fSwSBssDZdt1ucPN4Uun1mPgy0cT
    dXaMO7X7eaY6+IOo47tAiHk4VzqlNLk+Y35A+KikBzeNm7TD9DwJiBpOmPFwR1nP
    OqOjSSRusGirHOE3BcudV5o0h6rmogFMHIcsNuoBAoGBAOAEZIE5pFnwnCQucAtH
    Pb4CzdrTSc77ZraA+yAWJZpRY3vIWi/Z/1POUQJsq42Vwq5DKme67sErQe7sOyEX
    0j2InFFrb0L8RsDWl/wp9Cq9CPGpEoJ7YAu0hRe3MDz222GN+9CzStgNd1aGJxmM
    RmtdUXkvZWfBNx9Uhs0qE/bRAoGAAaOAXF2RP0X63e6EXgOIwwYZ/7Ix9eIeJjE7
    +ZhCUsD7aWZnyz7YAHSNbnC+5yiOxdG1YPub6s9fnC5Uq9ITwBKyjj4XCpcfLoED
    laG37L0eiikTppUQSgbSJ1WLEkQZcvaxx3SsQC7iKptvq7UmyR5OASp6DMHHiTc1
    7TBCbAECgYAYLiTd1V0xkzHxNLsQvVt/OiCBkIjxEHq+TD9Ap8DiFVQ2PEgd9nd0
    jryIZCX7jr77L1L7qUW0pGD1HDR2bYx5JNeI3hkBC+fgTiQ28ZCNgb2jKmRpVwNE
    lOOA05wlJ+PoRXd6Z8QuDZ5vmLrW2X/mZKuOF1u1d2c1xoi+s6UBhQ==
    -----END RSA PRIVATE KEY-----
    ```

!!! alert
    请不要直接复制使用这里的公钥和私钥，务必自己生成。公钥和私钥总是成对且独特的，公钥可以公开，私钥则需要妥善保管，不应在任何地方公开。

对于OpenSSH来说，所有和SSH相关的信息都会保存在`.ssh`目录下，在本地如此，在服务器上也如此。在SSH服务器上，系统通过`~/.ssh/authorized_keys`文件来管理哪些用户可以使用SSH登录到服务器。`authorized_keys`文件是一个文本文件，每一行代表一个公钥，当用户使用公钥进行身份验证时，系统会检查`authorized_keys`文件中是否存在对应的公钥，如果存在则允许登录，否则拒绝登录。因此，配置SSH密钥认证就是要将`id_rsa.pub`文件中的公钥内容追加到`authorized_keys`文件中。实验室的各个算力集群都提供了相应的网页控制台来管理密钥，你应当查阅相应的[服务器文档](../server_details/index.md)后在控制台中提交你的公钥来完成配置，而无需手动编辑`authorized_keys`文件。

如果你第一次链接到一个新的服务器，你可能会看到如下提示：
```
ECDSA key fingerprint is SHA256:ZUiMybZcoVMqJVOt7DiSKjAhMgJkhZ0nNc0SnvIoaVs.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```
这是服务器的公钥指纹，与你的公钥无关。它是每个服务器自身的标识，用于确认你要连接的服务器是否正确，输入yes并回车即可。之后，这个指纹就会保存到本地的`~/.ssh/known_hosts`文件中。这个文件里每一行都是一个曾经连接过的服务器，如果在某次连接时，目标服务器的指纹与保存的指纹不一致，连接就会自动断开。这个设计是防止你在不知情的情况下连接到了被替换的服务器上。

### 配置SSH Config

现在，我们重新查看一下SSH的用法:
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
其中，最常用的有：

- `-p port`: 指定SSH连接的端口，默认为22，与服务器的设置有关。是否需要修改可以在服务器文档中查询
- `-i identity_file`: 指定私钥文件，默认为`~/.ssh/id_rsa`。

!!! tip
    SSH实际上提供了很多其他功能，例如使用`SSH -L`选项可以指定本地端口转发，使用`SSH -R`选项可以指定远程端口转发，使用`scp`指令可以传输本地文件到服务器等，更多信息可以自行上网查阅。

对于常用的服务器，你可以把它们的配置写入`~/.ssh/config`文件中，这样你就可以通过简单的命令来管理它们了。例如，你可以将以下内容添加到你的`~/.ssh/config`文件中：
```
Host server1
    HostName server1.example.com
    Port 22
    User username
    IdentityFile ~/.ssh/id_rsa

Host server2
    HostName server2.example.com
    Port 22
    User username
    IdentityFile ~/.ssh/id_rsa
```
这样你就可以通过`ssh server1`或`ssh server2`来登录到相应的服务器了。

### 使用VSCode

除了命令行方式外，你还可以使用支持SSH的IDE来管理你的Linux服务器代码。VSCode是一款广泛使用的开发环境，它提供了Remote Development扩展包，允许你在本地编辑代码，并在远程服务器上运行和调试：

1. 打开VSCode。
2. 安装“Remote - SSH”扩展。
3. 在VSCode的左侧菜单栏中，点击“Remote Explorer”图标。
4. 如果你已经配置好了`~/.ssh/config`，你保存的服务器会自动出现在远程资源管理器中。如果没有，你可以点击“+”按钮来手动添加服务器。

!!! note
    使用VSCode第一次连接到一个新的远程服务器时，需要在远程服务器上安装vscode的后台服务。当远程服务器可以连接到公网时，这个过程是自动进行的，而当远程服务器无法连接公网，vscode会尝试在本地下载后通过`scp`传输到服务器上进行安装。然而，我们的S集群既无法连接公网也禁止`scp`传输，此时你需要先配置代理上网，具体操作请查阅集群相关文档。

## Linux入门

### 基本命令

在开始使用Linux服务器之前，熟悉一些常用的Linux命令是非常重要的。以下是一些基本的、你可能经常需要使用的命令：

| 命令       | 描述                                         |
|------------|----------------------------------------------|
| `ls`       | 列出当前目录下的文件和子目录                 |
| `cd <dir>` | 改变工作目录到 `<dir>`                       |
| `mkdir`    | 创建新的目录                                 |
| `rm`       | 删除一个文件或目录                           |
| `cp`       | 复制一个文件或目录                           |
| `mv`       | 移动或重命名文件                             |
| `touch`    | 创建空文件                                   |
| `echo`     | 输出文本到标准输出或者写入到一个文件         |
| `cat`      | 显示或合并文本文件内容                       |
| `grep`     | 在文件中搜索符合条件的行                     |
| `tar`      | 打包和压缩文件                               |
| `ps`       | 查看正在运行的进程                           |
| `top`      | 实时显示系统状态                             |
| `kill`     | 终止一个进程                                 |
| `sudo`     | 以root权限执行命令                           |

每个指令的具体使用方式可以通过`man <command>`或`<command> --help`来获取帮助文档。

### 文件系统(File System)和目录权限

Linux使用一种名为Unix File System (UFS)的文件系统，它有一个层次结构来组织文件。每一个文件或目录都有拥有者（owner）、组群（group）和其他用户（others）三个级别的权限。每个级别下有读（r）、写（w）和执行（x）三种基本权限。

- 读（r）: 可以查看内容
- 写（w）: 可以修改内容或创建新文件/目录
- 执行（x）: 对于目录，表示可以列出其内容；对于文件，则表示可运行

你可以使用`ls -l`命令来查看文件的详细信息和权限。例如：

```shell
drwxr-xr-x 2 user group 4096 Oct 13 08:12 directory_name
```

- 第一个字符 `d` 表示这是一个目录。
- 接下来的三组权限分别对应拥有者、所属组和其他用户的权限。

要更改文件或目录的权限，可以使用`chmod`命令。例如：
```shell
chmod u+x filename  # 给所有者增加执行权限
```

或者使用数值方式设置权限（4=读，2=写，1=执行，最终数值是三种权限之和）
```shell
chmod 755 filename  # 设置为所有者可读写执行，所属组和其他用户只可读和执行。
```
!!! tip
    你可能觉得这些数字看起来很奇怪，但如果把它们写为二进制，例如，`7 5 5`写为`111 101 101`，这些数字就变得非常直观了。每一组权限中的三个二进制位分别对应读、写和执行权限。

此外，如果你在远程服务器上使用了`ls -l`，你可能会感到奇怪，为什么ls无法看到之前提到的`~/.ssh`目录？因为这个目录是隐藏的，需要使用`ls -a`才能看到。在Linux中，以`.`开头的文件和目录都是隐藏的，它们不会被默认显示出来。此外，还有两个隐藏的特殊目录，分别是`.`和`..`，分别表示当前目录和上一级目录。

### Shell Scripts和环境变量

Shell作为人机交互的解释器，既可以从终端接受你的指令，也可以直接从文本文件中读取指令并执行。这种文本文件被称为Shell脚本（Shell Script）。你可以使用任何文本编辑器编写和保存这样的脚本，然后通过`bash <script_name>`或者给脚本加上可执行权限后直接运行来执行它。例如：

```shell
#!/bin/bash
string_to_be_printed="Hello, World!"
echo $string_to_be_printed
```
这个shell script由3行构成，第一行是“Shebang”，它告诉系统这个脚本应该由哪个解释器来执行。第二行是一个变量声明，第三行则是输出该变量的内容。运行这个文件，系统会在命令行中输出`Hello, World!`。相比于直接在终端输入指令，使用脚本的好处是你可以将一系列操作打包成一个可执行的文件，方便重复使用和分享。

!!! tip
    如果你使用`bash <script_name>`的方式运行脚本，shebang不会起作用，因为bash会直接执行脚本中的指令，而不是通过解释器执行。只有直接把脚本作为可执行文件运行时，shebang才会起作用。

除了你在脚本中定义的变量外，Shell还保存了一些预定义的变量，我们称为环境变量。环境变量在Linux中非常重要，几乎所有的命令行程序都会使用它们来获取信息或者传递信息。例如`$PATH`变量指定了shell应当在哪些路径搜索可执行文件，只有在这些路径下的文件可以不加前缀地运行，如`ssh`实际上是`/usr/bin`下的一个可执行文件，这是一个被`$PATH`默认包含的路径。你可以通过`env`指令来查看当前保存的全部环境变量。

Shell中直接声明的变量，如上面的`string_to_be_printed`只能在当前的shell或当前运行的脚本中使用，而无法被其他的程序读取，因此它们并不是环境变量。那么，如何改变环境变量呢？一种方式是显示地添加或删除，Linux中有专门的系统指令：

```shell
# 假设该文件名为test.sh
export HELLO_STRING1="Hello, World1!" # export用于添加环境变量
export HELLO_STRING2="Hello, World2!" # export用于添加环境变量
unset HELLO_STRING1                   # unset用于删除环境变量
```
通过`source test.sh`运行该脚本后，HELLO_STRING2就被添加到了环境变量中，而HELLO_STRING1则被添加后又删除了。

!!! tip
    `source <script name>`命令运行脚本和`bash <script name>`命令运行脚本的效果非常相似，但不同之处在于前者会直接在当前shell中运行脚本，而后者会在一个新的子shell中运行脚本。因此，如果你希望脚本中的变量能被保留，你应该使用`source <script name>`而不是`bash <script name>`。

另一种方式是对某一行指令临时添加，只需要在指令前声明变量，如：
```shell
CUDA_VISIABLE_DEVICES=0 python train.py
```
以这种方式声明的变量会以环境变量的形式作用在当前指令上，但是只对这行有效，不会保留。

然而，显式声明的环境变量也不是永久保存的，每次你打开一个新的shell，它们都会被重置。有没有办法永久保存一些环境变量，而无需每次都重新添加呢？当然！事实上，每一个shell都提供了一个特殊的脚本文件，如bash的`~/.bashrc`和zsh的`~/.zshrc`，它们会在你每次打开一个新的shell时自动`source`。因此，你可以把添加环境变量的指令写入到这些文件中，这样就可以永久保存这些环境变量了。

!!! tip
    通常，我们使用大写字母来命名环境变量，以区分普通变量。