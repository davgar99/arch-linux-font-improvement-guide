# Arch Linux Font Improvement Guide

![Arch Linux Logo](https://archlinux.org/static/logos/archlinux-logo-dark-scalable.518881f04ca9.svg)

## Improving Font Rendering and Compatibility on Arch Linux

Fonts in Arch Linux don't look all that great, and the reason why probably won't surprise you. Furthermore, many people notice the infamous "tofu" everywhere due to a lack of font support. Fortunately, there are a few ways to fix this without having to install many packages, and most of the necessary components should already be included. Get ready to dive into the configuration files!

![Tofu Example](https://i.stack.imgur.com/LdjiI.png)

*Example of tofu*

### WARNING:
The following tweaks should work fine for most people, but as with anything in life, your experience may vary. If you need further assistance, feel free to leave a comment or consult the Arch Wiki.

### Synopsis
After installing Arch Linux, you may wonder why the fonts in Arch Linux look so bland compared to Windows and macOS. The reason is that out of the box, Arch Linux doesn't implement many font rendering techniques to make the fonts look clear and legible. Essentially, there isn't much happening behind the scenes, so the text appears rather plain. Additionally, some apps or websites may display tofu (□) due to missing font support. Fortunately, these issues are relatively easy to fix, and this guide will discuss the solutions.

### Step 1:
Download and install the recommended fonts.

**Tip:** If the fonts are not available in the main repositories, check the AUR.

#### Recommended fonts:

```
sudo pacman -S noto-fonts
```
```
sudo pacman -S noto-fonts-cjk
```
```
sudo pacman -S noto-fonts-emoji
```
```
sudo pacman -S noto-fonts-extra
```

#### Optional but highly recommended fonts:

```
sudo pacman -S ttf-liberation
```
```
sudo pacman -S ttf-dejavu
```
```
sudo pacman -S ttf-roboto
```
```
paru -S ttf-symbola
```

#### Popular Monospaced Fonts:

```
sudo pacman -S ttf-jetbrains-mono
```
```
sudo pacman -S ttf-fira-code
```
```
sudo pacman -S ttf-hack
```
```
sudo pacman -S adobe-source-code-pro-fonts
```

### Step 2:
Create a local or global XML file to apply font rendering effects.

**Global directory:** /etc/fonts/local.conf

**Per user directory:** XDG_CONFIG_HOME/fontconfig/fonts.conf

#### Example XML file:
##### Modify the code as needed.

```
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "urn:fontconfig:fonts.dtd">
<fontconfig>
	<!-- Apply text rasterization, hinting, and anti-aliasing -->
  <match target="font">
    <edit name="antialias" mode="assign">
      <bool>true</bool>
    </edit>
    <edit name="hinting" mode="assign">
      <bool>true</bool>
    </edit>
    <edit mode="assign" name="rgba">
      <const>rgb</const>
    </edit>
    <edit mode="assign" name="hintstyle">
      <const>hintslight</const>
    </edit>
    <edit mode="assign" name="lcdfilter">
      <const>lcddefault</const>
    </edit>
  </match>
  <!-- Configure default fonts & fallback fonts -->
  <!-- Replace fonts with prefered fonts -->
    <alias>
    <family>serif</family>
    <prefer>
     <family>Roboto</family>
     <family>Noto Serif</family>
     <family>Noto Color Emoji</family>
     <family>Noto Emoji</family>
    </prefer>
  </alias>
  <alias>
    <family>sans-serif</family>
    <prefer>
     <family>Roboto</family>
     <family>Noto Sans</family>
     <family>Noto Color Emoji</family>
     <family>Noto Emoji</family>
    </prefer>
  </alias>
  <alias>
    <family>sans</family>
    <prefer>
     <family>Roboto</family>
     <family>Noto Sans</family>
     <family>Noto Color Emoji</family>
     <family>Noto Emoji</family>
    </prefer>
  </alias>
  <alias>
    <family>monospace</family>
    <prefer>
     <family>JetBrainsMono</family>
     <family>Noto Mono</family>
     <family>Noto Color Emoji</family>
     <family>Noto Emoji</family>
    </prefer>
  </alias>
  <alias>
    <family>mono</family>
    <prefer>
     <family>JetBrainsMono</family>
     <family>Noto Mono</family>
     <family>Noto Color Emoji</family>
     <family>Noto Emoji</family>
    </prefer>
  </alias>
</fontconfig>
```

### Step 3:
Install xorg-xrdb.

```
sudo pacman -S xorg-xrdb
```

Edit the **~/.Xresources** file *OR* create one if not present.

**Tip:** Backup the file just in case.

```
vim ~/.Xresources
```

**Tip:** Replace VIM with your editor of choice.

Add the following lines to that file and save changes.

```
Xft.antialias: 1
Xft.hinting: 1
Xft.rgba: rgb
Xft.hintstyle: hintslight
Xft.lcdfilter: lcddefault
```

Run this command when finished.

```
xrdb -merge ~/.Xresources
```

### Step 4:
Create required symbolic links for text rendering effects to work:

```
sudo ln -s /usr/share/fontconfig/conf.avail/10-sub-pixel-rgb.conf /etc/fonts/conf.d/
```

```
sudo ln -s /usr/share/fontconfig/conf.avail/10-hinting-slight.conf /etc/fonts/conf.d/
```

```
sudo ln -s /usr/share/fontconfig/conf.avail/11-lcdfilter-default.conf /etc/fonts/conf.d/
```

### Step 5:
Edit the freetype2.sh file.

```
sudo vim /etc/profile.d/freetype2.sh
```

Uncomment the following line from the file.

```
export FREETYPE_PROPERTIES="truetype:interpreter-version=40"
```

### Step 6:
Refresh the font cache files by running the following command.

```
sudo fc-cache -fv
```

### Step 7:
Reboot your PC and enjoy better-looking fonts! 😁
