# Kali for Intel Edison

* need to edit the following script
````
sudo ~/src/edison/edison-src/meta-intel-edison/utils/create-debian-image.sh --build_dir=~/src/edison/edison-src/out/current/build
````

* to flash on osx
````
brew install coreutils findutils gnu-tar gnu-sed gawk gnutls gnu-indent gnu-getopt dfu-util
````

* to build
````
cp ~/kali_intel_edison/build_script/create-debian-image.sh ./meta-intel-edison/utils/
sudo ./meta-intel-edison/utils/create-kali-image.sh --build_dir=./out/current/build
````

* to attach to the edison from osx
````
screen /dev/tty.usbserial-DA017L19 115200
````
