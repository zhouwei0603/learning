# Install/Update WSL
```
wsl --update
```

# Start WSL
```
wsl
```

# Install Ubuntu
Install Ubuntu from *Microsoft Store"

# Run Ubuntu
Run from Windows Start Menu
or
```
# Base on your Ubuntu version
wsl Ubuntu20.04
```

## Check installed WSL
```
wsl -l -v
```

# Update Ubuntu
```
sudo apt update
sudo apt upgrade -y
```

# Install Desktop
```
sudo apt install xubuntu-desktop -y
```

# Install xrdp (Remote Desktop)
```
sudo apt install xrdp -y
```

# Configure Ubuntu

## Required configurations
```
echo "xfce4-session" > ~/.xsession
sudo echo "xfce4-session" > /home/<user>/.xsession

# Optional: Change RDP port. The default port is 3389.
sudo vi /etc/xrdp/xrdp.ini

sudo systemctl restart xrdp
sudo systemctl start xrdp
sudo systemctl enable xrdp
```

## Fix IBux issue for Chinese input
```
sudo nano /etc/xrdp/startwm.sh
```

### Edit startwm.sh (Add the following code at the beginning, after `#!/bin/sh`, of the file)
```
unset DBUS_SESSION_BUS_ADDRESS
unset XDG_RUNTIME_DIR
# 如果你使用的是Xfce，确保最后是启动xfce4-session
exec xfce4-session
```

### Restart RDP
```
sudo systemctl restart xrdp
```

# Use Windows RDP with address `localhost` to access Ubuntu




