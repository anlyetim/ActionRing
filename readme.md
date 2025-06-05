
# ActionRing - Linux

<p align="center">
  <img src="https://img.shields.io/badge/Platform-Linux-turquoise">
  <img src="https://img.shields.io/badge/Language-Python-blue">
</p>

Action Ring is a “Quick Action Ring” that opens around the mouse cursor.  
It gives you instant access to your favorite apps, saves time and is a pleasure to look at.

<p align="center">
  <img src="https://github.com/anlyetim/ActionRing/blob/main/ActionRing.gif" width="300" height="300" />
</p>


## ✨ Features

- Application ring design
- Quick access with shortcuts
- Customizable structure (icons and apps)
- Written in Python

>[!NOTE]
>You can open the settings quickly by pressing the **S** key while the Action Ring is active.


<p align="center">
<img src="https://github.com/anlyetim/ActionRing/blob/main/ActionRing_Settings.png" width="300" height="300" />
</p>

## 📦 Requirements

The following packages must be installed for this application to work:


*for Debian based:*
```bash
sudo apt install python3 python3-gi gir1.2-gtk-3.0
```

*for Arch based:*
```bash
sudo pacman -Syu python python-pip gtk3
```

**Note:** This version of the application also uses Webcord and Zapzap via Flatpak. If you want to add different shortcuts or delete existing shortcuts, you can make changes via `settings.json`.

Or you can install Zapzap and Webcord via Flathub if you don't have these apps.**(Flatpak must be installed)**

*for Webcord:*
```bash
flatpak install flathub io.github.spacingbat3.webcord
```
*for Zapzap:*
```bash
flatpak install flathub com.rtosta.zapzap
```
## 🚀 Installation

```bash
git clone https://github.com/anlyetim/ActionRing.git
cd ActionRing
python3 action_ring.py
```

> [!IMPORTANT] 
> This application was only tested on an Arch-based Hyprland desktop. I have no idea how it will work on KDE, Gnome, etc.
#

> [!TIP]
> If you want to use this application on Hyprland like me, you can bind the executable in your `hyprland.conf` file by adding a line like this:
```bash
bind = MOD, <key>, exec, /full/path/to/actionring
```
Save the config and reload Hyprland to apply the changes.

#

Alternatively, if you'd like to run the executable file actionring, make sure it's marked as executable first:
```bash
chmod +x /home/(your_username)/ActionRing/actionring
```
Here’s the exact keybind I personally use:
```bash
bind = SUPER, F2, exec, /home/(your_username)/ActionRing/actionring
```
