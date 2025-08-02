# Arch Linux Font Improvement Guide

![Arch Linux Logo](https://github.com/davgar99/arch-linux-font-improvement-guide/blob/main/images/arch_linux_logo.svg)

# Table of Contents
  - [Synopsis](#synopsis)
    - [Disclaimer](#disclaimer)
  - [Improving Font Rendering and Compatibility on Arch Linux](#improving-font-rendering-and-compatibility-on-arch-linux)
    - [Step 1: Install Recommended Fonts](#step-1-install-recommended-fonts)
      - [Recommended fonts](#recommended-fonts)
      - [Optional but highly recommended fonts](#optional-but-highly-recommended-fonts)
      - [Available on the AUR](#available-on-the-aur)
      - [Popular Monospaced Fonts](#popular-monospaced-fonts)
    - [Step 2: Create XML Configuration File](#step-2-create-xml-configuration-file)
      - [Example XML file](#example-xml-file)
    - [Step 3: Disable Bitmap Fonts](#step-3-disable-bitmap-fonts)
    - [Step 4: Configure X11 Settings (Only for X11)](#step-4-configure-x11-settings-only-for-x11)
    - [Step 5: Create Symbolic Links](#step-5-create-symbolic-links)
    - [Step 6: Edit freetype2.sh File](#step-6-edit-freetype2sh-file)
    - [Step 7: Refresh Font Cache](#step-7-refresh-font-cache)
    - [Step 8: Reboot](#step-8-reboot)
  - [Sources](#sources)
  - [License](#license)

## Synopsis

After installing Arch Linux, you may wonder why the fonts in Arch Linux look so bland compared to Windows and macOS. The reason is that out of the box, Arch Linux doesn't implement many font rendering techniques to make the fonts look clear and legible. Essentially, there isn't much happening behind the scenes, so the text appears rather plain. Additionally, some apps or websites may display tofu (□) due to missing font support. Fortunately, these issues are relatively easy to fix, and this guide will discuss the solutions.

<figure>
  <img src="https://github.com/davgar99/arch-linux-font-improvement-guide/blob/main/images/tofu_example.png" alt="Tofu Example">
  <figcaption>Example of Tofu</figcaption>
</figure>

### Disclaimer

> The following tweaks should work fine for most people, but as with anything in life, results may vary. If you need further assistance, feel free to leave a comment or consult the Arch Wiki. Furthermore, if you wish not to have any emoji support, be sure to ignore packages ending in or containing the word "emoji" and remove them if present.

> **Side Note:** these tips were created specifically for Arch Linux, but they should work for other Linux distributions with slight modifications. If something doesn't work for your particular distro, please use your distro's official documentation or support forum(s).

## Improving Font Rendering and Compatibility on Arch Linux

### Step 1: Install Recommended Fonts

Download and install the recommended fonts.

> **Tip:** If the fonts are not available in the main repositories, check the AUR.

#### Recommended fonts

```sh
sudo pacman -S noto-fonts
sudo pacman -S noto-fonts-cjk
sudo pacman -S noto-fonts-emoji
sudo pacman -S noto-fonts-extra
```

#### Optional but highly recommended fonts

```sh
sudo pacman -S ttf-liberation
sudo pacman -S ttf-dejavu
sudo pacman -S ttf-roboto
```

#### Available on the AUR

```sh
paru -S ttf-symbola
```

#### Popular Monospaced Fonts

```sh
sudo pacman -S ttf-jetbrains-mono
sudo pacman -S ttf-fira-code
sudo pacman -S ttf-hack
sudo pacman -S adobe-source-code-pro-fonts
```

### Step 2: Create XML Configuration File

Create a local or global XML file to apply font rendering effects.

**Global directory:** `/etc/fonts/local.conf`

**Per user directory:** `XDG_CONFIG_HOME/fontconfig/fonts.conf`

#### Example XML file

> **Side note:** Please make sure to include fallback fonts and address other necessary criteria, as this XML file is minimal and may not cover all potential use cases. Furthermore, this XML file adds emoji support to many apps (including the terminal) so feel free to modify the XML file if you don't need certain features.

```xml
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
    <edit name="rgba" mode="assign">
      <const>rgb</const>
    </edit>
    <edit name="hintstyle" mode="assign">
      <const>hintslight</const>
    </edit>
    <edit name="lcdfilter" mode="assign">
      <const>lcddefault</const>
    </edit>
  </match>
  <!-- Configure default fonts & fallback fonts -->
  <!-- Replace fonts with preferred fonts -->
  <!-- Noto Emoji allows for emojis to render in all apps including the terminal, remove if not needed  -->
  <alias>
    <family>serif</family>
    <prefer>
      <family>Noto Serif</family>
      <family>Noto Color Emoji</family>
      <family>Noto Emoji</family>
    </prefer>
  </alias>
  <alias>
    <family>sans-serif</family>
    <prefer>
      <family>Noto Sans</family>
      <family>Noto Color Emoji</family>
      <family>Noto Emoji</family>
    </prefer>
  </alias>
  <alias>
    <family>sans</family>
    <prefer>
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

### Step 3: Disable Bitmap Fonts

Bitmap fonts are used as fallbacks for some fonts. This can lead to some very blurry, pixelated, or abnormally large fonts. Some users have reported Microsoft fonts being affected by this, and therefore it's recommended for users to disable bitmap fonts on a per font basis or globally. Be careful with disabling it globally as it may break some fonts (additional testing is required). According to the Arch Wiki, users may use the `70-no-bitmaps.conf` preset to disable this behavior or use an XML file instead.

**Path:** `~/.config/fontconfig/conf.d/20-no-embedded.conf`

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "urn:fontconfig:fonts.dtd">
<fontconfig>
  <match target="font">
    <edit name="embeddedbitmap" mode="assign">
      <bool>false</bool>
    </edit>
  </match>
</fontconfig>
```

> **Note:** This excerpt was taken directly from the Arch Wiki so all credit goes to the Arch Wiki and all of its contributors.

**Possible Fix:** If emojis stop working after using the previous XML file, then feel free to use this one instead:

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "urn:fontconfig:fonts.dtd">
<fontconfig>
  <match target="font">
    <edit name="embeddedbitmap" mode="assign">
      <bool>false</bool>
    </edit>
  </match>
  <match target="font">
    <test name="family" qual="any">
      <string>Noto Color Emoji</string>
    </test>
    <edit name="embeddedbitmap">
      <bool>true</bool>
    </edit>
  </match>
</fontconfig>
```

### Step 4: Configure X11 Settings (Only for X11)

Install `xorg-xrdb` (if needed).

```sh
sudo pacman -S xorg-xrdb
```

Edit the **~/.Xresources** file *OR* create one if not present.

> **Tip:** Backup the file just in case.

```sh
vim ~/.Xresources
```

> **Tip:** Replace `vim` with your preferred text editor.

Add the following lines to that file and save changes.

```sh
Xft.lcdfilter: lcddefault
Xft.hintstyle: hintslight
Xft.hinting: 1
Xft.antialias: 1
Xft.rgba: rgb
```

Run this command when finished.

```sh
xrdb -merge ~/.Xresources
```

### Step 5: Create Symbolic Links

Create required symbolic links for text rendering effects to work:

```sh
sudo ln -s /usr/share/fontconfig/conf.avail/10-sub-pixel-rgb.conf /etc/fonts/conf.d/
sudo ln -s /usr/share/fontconfig/conf.avail/10-hinting-slight.conf /etc/fonts/conf.d/
sudo ln -s /usr/share/fontconfig/conf.avail/11-lcdfilter-default.conf /etc/fonts/conf.d/
```

### Step 6: Edit freetype2.sh File

Edit the `freetype2.sh` file.

```sh
sudo vim /etc/profile.d/freetype2.sh
```

Uncomment the following line from the file.

```sh
export FREETYPE_PROPERTIES="truetype:interpreter-version=40"
```

### Step 7: Refresh Font Cache

Refresh the font cache files by running the following command.

```sh
sudo fc-cache -fv
```

### Step 8: Reboot

Reboot your PC and enjoy better-looking fonts! 😁

## Sources

<https://wiki.archlinux.org/title/Font_configuration>

<https://wiki.manjaro.org/index.php/Improve_Font_Rendering>

## License

[Attribution-NonCommercial-ShareAlike 4.0 International](https://github.com/davgar99/arch-linux-font-improvement-guide/blob/main/LICENSE)
