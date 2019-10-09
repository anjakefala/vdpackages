# Project: Create a .dmg image for MacOS of VisiData

- Note from MacOS Catalina (when it is launched in late September) and higher. Apple is getting tighter and only Apple Developer signed code will run on future macOS revisions.

- once we have a .pkg
    - test it on a mac that lacks a python env
    - test it on a mac that has a homebrewed python env
- figure out where and how to publish it

## To build a .App
NOTE: the following must be run on MacOS

```
git clone git@github.com:saulpw/app-vd.git
cd app-vd
make git-init
(ensure that visidata is checked out for the branch that you want to deploy)
make build
```
- to deploy the .App on that particular Mac, then run

```
make deploy
```

## how to create a .dmg from a .App on a mac
- `make build`
- Open Disk Utility -> File -> New Image -> Image From Folder.
- Select the folder where `make build` ,App is placed. Give a name for the DMG and save.

## TODO
- when the .dmg is clicked on, you should get a prompt to drag VisiData.app to the Applications folder
      - package that directory in dmg, with symlink to /Applications and background
- ensure that the static/Info.plist contains all supported visidata format types
- right-click on .csv, launch with visidata should go to that .csv
- make window bigger by default (store previous size?)
- no wait on exit
- the Info.plist was copied over from Slack/Spotify and should be more carefully editted


## Internal details
- pyinstaller creates the contents of Contents/MacOS
    - pyinstaller must be run within the visidata repo
- currently when .dmg is clicked on, static/vdwrap.sh is run - that opens the terminal and runs visidata
- icons
    - create a folder visidata.iconset
    - in this folder the following sizes will be needed (USE the names below for the files)
        - icon_16x16.png
        - icon_16x16@2x.png
        - icon_32x32.png
        - icon_32x32@2x.png
        - icon_128x128.png
        - icon_128x128@2x.png
        - icon_256x256.png
        - icon_256x256@2x.png
        - icon_512x512.png
        - icon_512x512@2x.png
    - the following shell script uses imagemagick to convert icon_512x512@2x.png (size 1024x1024) to create the rest
    ```
     #!/bin/bash
     for size in 16 32 128 256 512; do convert 'icon_512x512@2x.png' -resize ${size}x${size} icon_${size}x${size}.png; done; for size in 16 32 128 256; do dub=$[ ${size} * 2 ]; convert 'icon_512x512@2x.png' -resize ${dub}x${dub} icon_${size}x${size}'@2x'.png; done
     ```
    - move the iconset folder to `/Applications/VisiData.app/Contents/Resources/`
    - to convert the iconset folder to an icns file, run `iconutil -c isns visidata.iconset`
    - in Info.plist set `CFBundleIconFile` to `visidata.icns`

## Differences between DMG and PKG
DMG:
* Disk image format, like ISO, etc.
* Is not an installer, just a way to store files.
* Supports encryption, password protection, and compression.
* Single file (ideal for distribution).

PKG:
* Installer package. Can install files to different locations, etc.
* Supports compression.
* Is just a folder, with multiple files inside (just like .app).
* Ideal for minimal click installation.

## Python modules we want
python-dateutil     # date type
requests            # http
lxml                # html/xml
openpyxl            # xlsx
xlrd                # xls
h5py                # hdf5
pyshp               # shapefiles
sas7bdat            # sas7bdat (SAS)
xport               # xpt (SAS)
savReaderWriter     # sav (SPSS)
PyYAML              # yaml/yml
