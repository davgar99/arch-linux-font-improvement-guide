# Arch Linux Font Improvement Guide

![Arch Linux Logo](images/arch_linux_logo.svg)

# Table of Contents
  - [Synopsis](#synopsis)
  - [Improving Font Rendering and Compatibility on Arch Linux](#improving-font-rendering-and-compatibility-on-arch-linux)
    - [Step 1: Install Recommended Fonts](#step-1-install-recommended-fonts)
      - [Recommended Fonts](#recommended-fonts)
      - [Optional but Highly Recommended Fonts](#optional-but-highly-recommended-fonts)
      - [Available on the AUR](#available-on-the-aur)
      - [Popular Monospaced Fonts](#popular-monospaced-fonts)
    - [Step 2: Create XML Configuration File](#step-2-create-xml-configuration-file)
      - [Example XML File](#example-xml-file)
    - [Step 3: Disable Bitmap Fonts](#step-3-disable-bitmap-fonts)
    - [Step 4: Configure X11 Settings (Only for X11)](#step-4-configure-x11-settings-only-for-x11)
    - [Step 5: Create Symbolic Links](#step-5-create-symbolic-links)
    - [Step 6: Edit freetype2.sh File](#step-6-edit-freetype2sh-file)
    - [Step 7: Refresh Font Cache](#step-7-refresh-font-cache)
    - [Step 8: Restart Applications](#step-8-restart-applications)
    - [Optional Steps](#optional-steps)
  - [Sources](#sources)
  - [License](#license)

## Synopsis

After installing Arch Linux, you may wonder why the fonts in Arch Linux look so bland compared to Windows and macOS. The reason is that out of the box, Arch Linux doesn't implement many font rendering techniques to make the fonts look clear and legible. Essentially, there isn't much happening behind the scenes, so the text appears rather plain. Additionally, some apps or websites may display tofu (□) due to missing font support. Fortunately, these issues are relatively easy to fix, and this guide will discuss the solutions.

> [!TIP]
> Arch has actually started enabling a couple of these improvements by default in recent years. Basic hinting (`hintslight`) and LCD filtering now come pre-enabled out of the box on a fresh install. So fonts aren't quite as bare as they used to be, but there's still plenty of room for improvement, which is what the rest of this guide covers.

<figure>
  <img src="images/tofu_example.png" alt="Tofu Example">
  <figcaption>Example of Tofu</figcaption>
</figure>

> [!CAUTION]
> The following tweaks should work fine for most people, but as with anything in life, results may vary. If you need further assistance, feel free to leave a comment or consult the Arch Wiki. Furthermore, if you wish not to have any emoji support, be sure to ignore packages ending in or containing the word "emoji" and remove them if present.

> [!NOTE]
> These tips were created specifically for Arch Linux, but they should work for other Linux distributions with slight modifications. If something doesn't work for your particular distro, please use your distro's official documentation or support forum(s).

## Improving Font Rendering and Compatibility on Arch Linux

### Step 1: Install Recommended Fonts

Download and install the recommended fonts.

> [!TIP]
> If the fonts aren't available in the main repositories, check the AUR.

#### Recommended Fonts

```sh
sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji noto-fonts-extra
```

#### Optional but Highly Recommended Fonts

```sh
sudo pacman -S ttf-liberation ttf-dejavu ttf-roboto
```

#### Available on the AUR
> [!TIP]
> For AUR packages you'll have to either install them manually or use an AUR helper.
> In this guide, I'll be using paru, but feel free to use whatever you want.

> [!CAUTION]
> `ttf-symbola`'s upstream file is hosted on the font author's personal site, which frequently goes down or 404s. If the build fails, try building it again later.

```sh
paru -S ttf-symbola
```

#### Popular Monospaced Fonts

```sh
sudo pacman -S ttf-jetbrains-mono ttf-fira-code ttf-hack adobe-source-code-pro-fonts
```

### Step 2: Create XML Configuration File

Create a local or global XML file to apply font rendering effects.

**Global directory:** `/etc/fonts/local.conf`

**Per user directory:** `~/.config/fontconfig/fonts.conf`

#### Example XML File

> [!TIP]
> Feel free to modify this XML file if you don't need emoji support or certain features.
> If you don't know what you're doing, just leave it as is.

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
  <!-- Noto Color Emoji allows for emojis to render in all apps including the terminal, remove it if not needed -->
  <alias>
    <family>serif</family>
    <prefer>
      <family>Noto Serif</family>
      <family>Noto Color Emoji</family>
    </prefer>
  </alias>
  <alias>
    <family>sans-serif</family>
    <prefer>
      <family>Noto Sans</family>
      <family>Noto Color Emoji</family>
    </prefer>
  </alias>
  <alias>
    <family>sans</family>
    <prefer>
      <family>Noto Sans</family>
      <family>Noto Color Emoji</family>
    </prefer>
  </alias>
  <alias>
    <family>monospace</family>
    <prefer>
      <family>Noto Sans Mono</family>
      <family>Noto Color Emoji</family>
    </prefer>
  </alias>
  <alias>
    <family>mono</family>
    <prefer>
      <family>Noto Sans Mono</family>
      <family>Noto Color Emoji</family>
    </prefer>
  </alias>
</fontconfig>
```

### Step 3: Disable Bitmap Fonts

Bitmap fonts are used as fallbacks for some fonts. This can lead to some very blurry, pixelated, or abnormally large fonts. Some users have reported Microsoft fonts being affected by this, and therefore it's recommended for users to disable bitmap fonts on a per font basis or globally. Be careful with disabling it globally as it may break some fonts (additional testing is required). According to the Arch Wiki, users may use the `70-no-bitmaps-except-emoji.conf` preset to disable this behavior or use an XML file instead.

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

> [!NOTE]
> This excerpt was taken directly from the Arch Wiki so all credit goes to the Arch Wiki and all of its contributors.

> [!TIP]
> Newer `fontconfig` versions ship a ready-made preset for exactly this case, so you can skip writing XML by hand:
> ```sh
> sudo ln -s /usr/share/fontconfig/conf.avail/70-no-bitmaps-except-emoji.conf /etc/fonts/conf.d/
> ```
> Use the XML version below instead if you want finer control over which bitmap fonts (besides emoji) get kept.

> [!TIP]
> If emojis stop working after using the previous XML file, then feel free to use this one instead:

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
    <edit name="embeddedbitmap" mode="assign">
      <bool>true</bool>
    </edit>
  </match>
</fontconfig>
```

### Step 4: Configure X11 Settings (Only for X11)

> [!NOTE]
> If you are using Wayland, you can skip this step as these settings are handled by Fontconfig or the compositor directly.

Install `xorg-xrdb` (if needed).

```sh
sudo pacman -S xorg-xrdb
```

Edit the **~/.Xresources** file *OR* create one if not present.

> [!TIP]
> Backup the file just in case.

```sh
vim ~/.Xresources
```

> [!TIP]
> Replace `vim` with your preferred text editor.

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

> [!NOTE]
> On current Arch installs, `10-hinting-slight.conf` and `11-lcdfilter-default.conf` are usually already enabled by default (fontconfig now pre-links some presets via `/usr/share/fontconfig/conf.default/`). If you see `File exists` for those two, that just means they're already active, so there's nothing to fix. `10-sub-pixel-rgb.conf` is typically the only one still missing. The `-f` flag below makes all three commands safe to run regardless.

```sh
sudo ln -sf /usr/share/fontconfig/conf.avail/10-sub-pixel-rgb.conf /etc/fonts/conf.d/
sudo ln -sf /usr/share/fontconfig/conf.avail/10-hinting-slight.conf /etc/fonts/conf.d/
sudo ln -sf /usr/share/fontconfig/conf.avail/11-lcdfilter-default.conf /etc/fonts/conf.d/
```

### Step 6: Edit freetype2.sh File

> [!NOTE]
> `truetype:interpreter-version=40` has been Arch's default since FreeType 2.7, so on a stock system this step currently has no visible effect. It's still worth doing explicitly if you (or a different guide) previously set a different interpreter version and want to reset it back to the default.

Edit the `freetype2.sh` file.

```sh
sudo vim /etc/profile.d/freetype2.sh
```

Uncomment the following line from the file.

```sh
export FREETYPE_PROPERTIES="truetype:interpreter-version=40"
```

### Step 7: Refresh Font Cache

Fontconfig-aware software normally refreshes the cache when needed, so this step is usually unnecessary. If you want to rebuild the cache manually, run:

```sh
fc-cache -fv
```

### Step 8: Restart Applications

Restart any applications that were already running so they load the new Fontconfig configuration. A full system reboot is normally unnecessary.

### Optional Steps

Add the following line to your "**/etc/environment**" file.

```sh
FREETYPE_PROPERTIES="cff:no-stem-darkening=0 autofitter:no-stem-darkening=0"
```

> [!NOTE]
> This command enables **stem darkening**, which slightly thickens font "stems" to improve contrast and readability on low-DPI monitors. This results in a "richer" look similar to macOS font rendering.
> - `cff:no-stem-darkening=0` enables it for OpenType/CFF fonts.
> - `autofitter:no-stem-darkening=0` enables it for other fonts (like TrueType) when using the auto-hinter.
> - Setting these to `0` (false) enables the feature because the property is named "**no**-stem-darkening".

## Sources

<https://wiki.archlinux.org/title/Font_configuration>

<https://wiki.manjaro.org/index.php/Improve_Font_Rendering>

<https://www.freetype.org/freetype2/docs/reference/ft2-properties.html#no-stem-darkening>

## License

[Attribution-ShareAlike 4.0 International](LICENSE)


## 🌐 Web Resources & Aesthetic Symbols Index
- [SYM 1D42D](https://aestheticsymbols.io/symbol/sym-1d42d/)
- [SPRING TULIP BLOSSOM](https://aestheticsymbols.io/symbol/spring-tulip-blossom/)
- [PISCES ZODIAC FISHES](https://aestheticsymbols.io/symbol/pisces-zodiac-fishes/)
- [SYM 2764 FE0F 200D 1F525](https://aestheticsymbols.io/symbol/sym-2764-fe0f-200d-1f525/)
- [SYM 26E8](https://aestheticsymbols.io/symbol/sym-26e8/)
- [SYM 26FF](https://aestheticsymbols.io/symbol/sym-26ff/)
- [HOLLOW STAR](https://aestheticsymbols.io/symbol/hollow-star/)
- [SYM 1F60E](https://aestheticsymbols.io/symbol/sym-1f60e/)
- [SYM 1D41B](https://aestheticsymbols.io/symbol/sym-1d41b/)
- [TIKTOK CAPTIONS](https://aestheticsymbols.io/ja/tiktok-captions/)
- [SYM 1F643](https://aestheticsymbols.io/symbol/sym-1f643/)
- [SYM 1F978](https://aestheticsymbols.io/symbol/sym-1f978/)
- [SYM 26E4](https://aestheticsymbols.io/symbol/sym-26e4/)
- [KHANDA EMBLEM](https://aestheticsymbols.io/symbol/khanda-emblem/)
- [ANTICLOCKWISE OPEN CIRCLE ARROW](https://aestheticsymbols.io/symbol/anticlockwise-open-circle-arrow/)
- [SYM 26D5](https://aestheticsymbols.io/symbol/sym-26d5/)
- [SYM 1D473](https://aestheticsymbols.io/symbol/sym-1d473/)
- [SYM 1F601](https://aestheticsymbols.io/symbol/sym-1f601/)
- [SYM 1F47D](https://aestheticsymbols.io/symbol/sym-1f47d/)
- [SYM 1D492](https://aestheticsymbols.io/symbol/sym-1d492/)
- [SYM 265D](https://aestheticsymbols.io/symbol/sym-265d/)
- [RIGHT MATHEMATICAL WHITE SQUARE BRACKET](https://aestheticsymbols.io/symbol/right-mathematical-white-square-bracket/)
- [SYM 1F60A](https://aestheticsymbols.io/symbol/sym-1f60a/)
- [SPARKLE DOT FLARE](https://aestheticsymbols.io/symbol/sparkle-dot-flare/)
- [DISCORD STATUS](https://aestheticsymbols.io/es/discord-status/)
- [SYM 1D4A4](https://aestheticsymbols.io/symbol/sym-1d4a4/)
- [BRACKETS](https://aestheticsymbols.io/brackets/)
- [CUPID FEATHERY ARROW](https://aestheticsymbols.io/symbol/cupid-feathery-arrow/)
- [EIGHT POINTED STAR](https://aestheticsymbols.io/symbol/eight-pointed-star/)
- [SYM 2724](https://aestheticsymbols.io/symbol/sym-2724/)
- [SYM 1F973](https://aestheticsymbols.io/symbol/sym-1f973/)
- [ZODIAC CELESTIAL](https://aestheticsymbols.io/ru/zodiac-celestial/)
- [SYM 26B0](https://aestheticsymbols.io/symbol/sym-26b0/)
- [SYM 2660](https://aestheticsymbols.io/symbol/sym-2660/)
- [SYM 26BE](https://aestheticsymbols.io/symbol/sym-26be/)
- [BLUSHING SOFT SMILE KAOMOJI](https://aestheticsymbols.io/symbol/blushing-soft-smile-kaomoji/)
- [SYM 26DC](https://aestheticsymbols.io/symbol/sym-26dc/)
- [ROBLOX NAMES](https://aestheticsymbols.io/roblox-names/)
- [SYM 26C8](https://aestheticsymbols.io/symbol/sym-26c8/)
- [NATURE FLOWERS](https://aestheticsymbols.io/nature-flowers/)
- [SYM 1D44E](https://aestheticsymbols.io/symbol/sym-1d44e/)
- [SYM 2722](https://aestheticsymbols.io/symbol/sym-2722/)
- [SYM 26D0](https://aestheticsymbols.io/symbol/sym-26d0/)
- [SYM 1D478](https://aestheticsymbols.io/symbol/sym-1d478/)
- [SYM 26AC](https://aestheticsymbols.io/symbol/sym-26ac/)
- [SYM 1F627](https://aestheticsymbols.io/symbol/sym-1f627/)
- [SYM 1D47A](https://aestheticsymbols.io/symbol/sym-1d47a/)
- [SYM 26BA](https://aestheticsymbols.io/symbol/sym-26ba/)
- [TIKTOK CAPTIONS](https://aestheticsymbols.io/es/tiktok-captions/)
- [SYM 26C5](https://aestheticsymbols.io/symbol/sym-26c5/)
- [FREEFIRE NAMES](https://aestheticsymbols.io/vi/freefire-names/)
- [RINGED PLANET SATURN](https://aestheticsymbols.io/symbol/ringed-planet-saturn/)
- [SYM 1D406](https://aestheticsymbols.io/symbol/sym-1d406/)
- [UPWARD DIAGONAL ARROW](https://aestheticsymbols.io/symbol/upward-diagonal-arrow/)
- [MUSIC FLAT SIGN](https://aestheticsymbols.io/symbol/music-flat-sign/)
- [SYM 2671](https://aestheticsymbols.io/symbol/sym-2671/)
- [SYM 2628](https://aestheticsymbols.io/symbol/sym-2628/)
- [TRENDING](https://aestheticsymbols.io/ja/trending/)
- [SYM 1F64A](https://aestheticsymbols.io/symbol/sym-1f64a/)
- [SYM 1F495](https://aestheticsymbols.io/symbol/sym-1f495/)
- [SYM 2658](https://aestheticsymbols.io/symbol/sym-2658/)
- [SYM 1D44F](https://aestheticsymbols.io/symbol/sym-1d44f/)
- [ROTATED FLORAL HEART](https://aestheticsymbols.io/symbol/rotated-floral-heart/)
- [SYM 1F920](https://aestheticsymbols.io/symbol/sym-1f920/)
- [SYM 1F97A](https://aestheticsymbols.io/symbol/sym-1f97a/)
- [SYM 26B8](https://aestheticsymbols.io/symbol/sym-26b8/)
- [ZODIAC CELESTIAL](https://aestheticsymbols.io/pt/zodiac-celestial/)
- [SYM 1F928](https://aestheticsymbols.io/symbol/sym-1f928/)
- [WHITE HEART](https://aestheticsymbols.io/symbol/white-heart/)
- [SKULL AND CROSSBONES](https://aestheticsymbols.io/symbol/skull-and-crossbones/)
- [SYM 1D450](https://aestheticsymbols.io/symbol/sym-1d450/)
- [FREEFIRE NAMES](https://aestheticsymbols.io/ja/freefire-names/)
- [SIXTEEN POINTED STAR](https://aestheticsymbols.io/symbol/sixteen-pointed-star/)
- [BEAMED EIGHTH NOTES](https://aestheticsymbols.io/symbol/beamed-eighth-notes/)
- [SYM 1D45D](https://aestheticsymbols.io/symbol/sym-1d45d/)
- [SYM 1D498](https://aestheticsymbols.io/symbol/sym-1d498/)
- [SYM 26B5](https://aestheticsymbols.io/symbol/sym-26b5/)
- [SYM 26B9](https://aestheticsymbols.io/symbol/sym-26b9/)
- [STARS](https://aestheticsymbols.io/stars/)
- [SYM 1F92F](https://aestheticsymbols.io/symbol/sym-1f92f/)
- [SYM 1F92A](https://aestheticsymbols.io/symbol/sym-1f92a/)
- [TIKTOK CAPTIONS](https://aestheticsymbols.io/tiktok-captions/)
- [SYM 1F628](https://aestheticsymbols.io/symbol/sym-1f628/)
- [SYM 26E6](https://aestheticsymbols.io/symbol/sym-26e6/)
- [SYM 26EE](https://aestheticsymbols.io/symbol/sym-26ee/)
- [SYM 1D480](https://aestheticsymbols.io/symbol/sym-1d480/)
- [SYM 1D495](https://aestheticsymbols.io/symbol/sym-1d495/)
- [SYM 1D47D](https://aestheticsymbols.io/symbol/sym-1d47d/)
- [ZODIAC CELESTIAL](https://aestheticsymbols.io/vi/zodiac-celestial/)
- [SYM 2656](https://aestheticsymbols.io/symbol/sym-2656/)
- [SYM 1F638](https://aestheticsymbols.io/symbol/sym-1f638/)
- [SYM 26C7](https://aestheticsymbols.io/symbol/sym-26c7/)
- [SYM 26BD](https://aestheticsymbols.io/symbol/sym-26bd/)
- [SYM 2672](https://aestheticsymbols.io/symbol/sym-2672/)
- [CUTE BUNNY RABBIT FACE](https://aestheticsymbols.io/symbol/cute-bunny-rabbit-face/)
- [SYM 26E0](https://aestheticsymbols.io/symbol/sym-26e0/)
- [CIRCLED STAR](https://aestheticsymbols.io/symbol/circled-star/)
- [SYM 1F625](https://aestheticsymbols.io/symbol/sym-1f625/)
- [SYM 1D49C](https://aestheticsymbols.io/symbol/sym-1d49c/)
- [SYM 1D45C](https://aestheticsymbols.io/symbol/sym-1d45c/)
- [SYM 26DD](https://aestheticsymbols.io/symbol/sym-26dd/)
- [SYM 1D43C](https://aestheticsymbols.io/symbol/sym-1d43c/)
- [SYM 1F9D0](https://aestheticsymbols.io/symbol/sym-1f9d0/)
- [BORDERS DIVIDERS](https://aestheticsymbols.io/vi/borders-dividers/)
- [SYM 26EC](https://aestheticsymbols.io/symbol/sym-26ec/)
- [SYM 1D40D](https://aestheticsymbols.io/symbol/sym-1d40d/)
- [NATURE FLOWERS](https://aestheticsymbols.io/vi/nature-flowers/)
- [SYM 1D41A](https://aestheticsymbols.io/symbol/sym-1d41a/)
- [SYM 1F608](https://aestheticsymbols.io/symbol/sym-1f608/)
- [SYM 1D46D](https://aestheticsymbols.io/symbol/sym-1d46d/)
- [SYM 2642](https://aestheticsymbols.io/symbol/sym-2642/)
- [SYM 1F606](https://aestheticsymbols.io/symbol/sym-1f606/)
- [SYM 1F641](https://aestheticsymbols.io/symbol/sym-1f641/)
- [EIGHT POINTED BLACK STAR](https://aestheticsymbols.io/symbol/eight-pointed-black-star/)
- [SYM 1F47F](https://aestheticsymbols.io/symbol/sym-1f47f/)
- [SYM 1D444](https://aestheticsymbols.io/symbol/sym-1d444/)
- [WINGED ANGELIC COQUETTE HEART](https://aestheticsymbols.io/symbol/winged-angelic-coquette-heart/)
- [PINWHEEL STAR](https://aestheticsymbols.io/symbol/pinwheel-star/)
- [SYM 1F62F](https://aestheticsymbols.io/symbol/sym-1f62f/)
- [SYM 26C4](https://aestheticsymbols.io/symbol/sym-26c4/)
- [SYM 2682](https://aestheticsymbols.io/symbol/sym-2682/)
- [SYM 26FB](https://aestheticsymbols.io/symbol/sym-26fb/)
- [SYM 1D433](https://aestheticsymbols.io/symbol/sym-1d433/)
- [SYM 262E](https://aestheticsymbols.io/symbol/sym-262e/)
- [SYM 26FC](https://aestheticsymbols.io/symbol/sym-26fc/)
- [SYM 2733](https://aestheticsymbols.io/symbol/sym-2733/)
- [SYM 1D4A1](https://aestheticsymbols.io/symbol/sym-1d4a1/)
- [SYM 268C](https://aestheticsymbols.io/symbol/sym-268c/)
- [SYM 1F63E](https://aestheticsymbols.io/symbol/sym-1f63e/)
- [SYM 1F642 200D 2195 FE0F](https://aestheticsymbols.io/symbol/sym-1f642-200d-2195-fe0f/)
