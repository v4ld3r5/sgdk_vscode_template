
PLEASE NOTE: Thanks for having a look. I'm not using this anymore as I changed my work environment to use:
* Toolchain: https://github.com/andwn/marsdev
* VSCode helper : https://marketplace.visualstudio.com/items?itemName=zerasul.genesis-code

--

# Basic SGDK & VS Code template

This `hello world` example contains a very basic VS Code setup aiming to work on SGDK projects to create Genesis/MegaDrive homebrew. 

## Usage


Since majority of tooling was developed with Windows in mind `Wine` is chosen to isolate the SGDK environment, so these steps would easily work for MacOS, *nix and Linux users.
If you work from Windows then no setup is required as the variables used are the same ones setup for SGDK itself. 

We also setup `Gens KMod` to test our progress and have it integrated as part of the project's build tasks.

## Setup

Following is my config on a MacBook Air Mid 2015.

### 1. Prepare directory path & environment variables

Create and set the necessary permissions:

```
sudo mkdir /opt/gendev
sudo chown $USER: /opt/gendev
```

Export the necessary variables:
* `GENDEV` for the root folder
* `GDK` and `GDK_WIN` as required by SGDK, they will also appear as part of Wine environment
* `GENS` for our emulator path

```
export GENDEV=/opt/gendev
export GDK='c:/sgdk'
export GDK_WIN='c:\sgdk'
export GENS=$GENDEV/gens/gens.exe
```

To make your changes permanent add them to your shell profile. I'm using ZSH nowadays:

```
echo “export GENDEV=/opt/gendev” >> .zshrc
echo “export GDK='c:/sgdk’” >> .zshrc
echo “export GDK_WIN=‘c:\sgdk’” >> .zshrc
```

### 2. Install Wine

Use your package manager to install Wine. On MacOS do:

```
brew cask install xquartz
brew install wine
```

Now let's prepare our environment. Under our root path we will set the Wine environment from which our tools will be used:

```
mkdir $GENDEV/wine
WINEDEBUG=-all WINEARCH=win32 WINEPREFIX=$GENDEV/wine wineboot
```

### 3. Install SGDK

I prefer to clone the project. Next step is **important**, that way SGDK is visible inside the Wine environment.
The benefit of doing this way is you have a good control of each component under your root path, while inside Wine you keep simple paths:

```
cd $GENDEV
git clone https://github.com/Stephane-D/SGDK.git
ln -sv $GENDEV/SGDK $GENDEV/wine/drive_c/sgdk
```

### 4. Install Java

This is required by SGDK. I'm using Oracle's JDK version 8. Download it and we'll then install from command line.
Next command will open the Windows terminal inside our Wine environment, and since our host filesystem is still reachable from `Z:` unit we can have this easily installed:

```
cd path/where/jdk/is/downloaded
WINEPREFIX=$GENDEV/wine wine cmd
```

The prompt will change to a Windows one, but you will still be in the same folder as the JDK, so to install simply do as:

```
Z:\> jdk-8u181-windows-i586.exe /s
```

Last but not least, you need to update the `%PATH%` variable on your wine environment so that it can find the installed Java binaries.
In my case I had to append: `;C:\Program Files\Java\jdk1.8.0_181\bin` . To open the registry do:

```
WINEPREFIX=$GENDEV/wine wine regedit
```

Navigate to: `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Session Manager\Environment`
Then edit the `PATH` key.

### 5. Compile SGDK

Ook, we are ready to compile `libmd` as instructed on SGDK instructions. Again we open a Windows terminal and run make:

```
WINEPREFIX=$GENDEV/wine wine cmd
Z:\> %GDK_WIN%\bin\make -f %GDK_WIN%\makelib.gen
```

### 5. Install GensK

At this point we will leave the emulator ready as well. Official webpage: https://gendev.spritesmind.net/page-gensK.html
We'll add it under our main path and make it also visible inside the Wine environment:

```
mkdir $GENDEV/gens
mv ~/Downloads/gensKMod_073/* $GENDEV/gens
ln -sv $GENDEV/gens $GENDEV/wine/drive_c/gens
```

### 6. Prepare project template

If everything went fine we are now ready to download the template. You might want to create a work directory under our root path as well:

```
mkdir $GENDEV/workdir
cd $GENDEV/workdir
git clone https://github.com/v4ld3r5/sgdk_vscode_template.git
```

Open the project folder in VSCode, hit `Cmd + Shift + B` and on the popup list you will see 3 tasks:
* `SGDK - Build & Test`: compiles the project, then launches Gens with the created ROM
* `SGDK - Clean`: removes all binaries
* `SGDK - Build`: just compiles the project

Enjoy!
