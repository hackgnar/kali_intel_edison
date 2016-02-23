### Manually building Kali Linux from Scratch for Intel Edison
This method is the most complex of all three methods in this series for getting Kali Linux rolling on your Intel Edison board.  However, this method provides the most flexibility for customization.  If you are new to Intel Edison or building Linux distros, you may want to check out the follow up posts in this series (coming soon) as they are quicker and easier methods for getting Kali Linux on your Edison board.

#### Install Build Dependancies
* Once the source files are downloaded and uncompressed, we must install some build dependancies before creating our Kali image.
```` bash
sudo apt-get update
sudo apt-get -y install build-essential git diffstat gawk chrpath texinfo libtool gcc-multilib debootstrap u-boot-tools debian-archive-keyring dfu-util screen
````

#### Create a Directory for Building the Image
* This step is optional, but I find it useful to create a directory for keeping my builds organized.  If you chose to skip this step or alter it, just note that you may have to substitute directory references later in this documentation.
```` bash
mkdir -p ~/src/edison
cd ~/src/edison
````

#### Download the Latest Edison Source Package
* The source packages Intel provides are periodically updated.  Because of this, it is wise to manually download them from Intels official download page.  As of this writing you can find the source package downloads on [this page](https://software.intel.com/en-us/iot/hardware/edison/downloads).

* Download the latest "Linux source files" on the page listed above.  As of this documentation, the link was ["Linux source files"](http://downloadmirror.intel.com/25028/eng/edison-src-ww25.5-15.tgz).

* If you are not interested in grabbing the latest source package and want to use the latest package at the time of this writing, you can run the following.
```` bash
curl -O http://downloadmirror.intel.com/25028/eng/edison-src-ww25.5-15.tgz
````

#### Uncompress the Source Files
* If you manually downloaded the source files in the step above, make sure you move the tar file to your build directory ~/src/edison.  Once it is there, you can uncompress it with the following command.
```` bash
tar xfvz edison-src-ww25.5-15.tgz
````

#### Download the Kali Linux Build Scripts
* This step pulls my custom build scripts for the Intel Edison Kali Linux along with the Kali Linux debootstrap files you will need later on to build the base distro image.
```` bash
git clone https://github.com/hackgnar/kali_intel_edison.git 
````

#### Copy the Kali build files to the proper locations
* Before we get building, we need to copy some files from the git repo we just cloned to the proper locations.
```` bash
cp ./kali_intel_edison/build_script/create-kali-image.sh ./edison-src/meta-intel-edison/utils/create-kali-image.sh
chmod 755 ./edison-src/meta-intel-edison/utils/create-kali-image.sh
sudo cp ./kali_intel_edison/debootstrap_scripts/kali-rolling /usr/share/debootstrap/scripts/kali-rolling
sudo chmod 644 /usr/share/debootstrap/scripts/kali-rolling
sudo chown root:root /usr/share/debootstrap/scripts/kali-rolling
````

#### Edit the Stock Edison Image Root File System Size
* This step is necisarry to make the root filesystem on the image you will build in later steps larger.
* If you do not do this step, you will run out of space pretty quickly when installing applications in Kali Linux.
```` bash
sed -i -e 's/524288/1400000/' ~/src/edison/edison-src/meta-intel-edison/meta-intel-edison-distro/recipes-core/images/edison-image.bb
````

#### (Optional) Speed Up The Build With Parallelization
* Before moving on, if your computer has more than 4 CPU cores, you can speed up the make process by telling the build scripts to build using parallelization.  By default, the Edison build configuration is set to build 4 packages at a time with 4 make threads.  If you have a machine with 8 or more CPU cores, set the following variables to the amount of CPU cores on your machine.

* Here is a little tip.  If you dont have access to a fast machine with tons of CPU cores, you can spin up an Amazon ec2 m4.10xlarge instance (it has 40 CPU cores) to build an image from scratch in roughly 20 minutes.  When I use the m4.10xlarge, I set both ````parallel_make```` and ````bb_number_thread```` to a value of 40.
````
export SETUP_ARGS="--parallel_make=8 --bb_number_thread=8"
````

#### Initialize the Build Environment
* The following commands prepare our build directory.  You only need to run this command prior to your first build.  You will not have to run this again on consecutive builds unless you completely wipe the edison-src directory.
```` bash
cd edison-src
make setup
````

#### Build the Base Edison Image and Debian Package Dependancies
* As Kali Linux is a Debian based system, we need this step to create the deb packages we will install to our Kali Image.  It will not actually create a Debian image.  It only creates the base edison image and deb packages we will need in later steps.
* When this build process completes, ignore any output telling you to run the script ````create-debian-image.sh````.  That script is for building Debian images.  We are simply going to replace that step with the custom ````create-kali-image.sh```` script.
* Note, this step can take a very long time.  When you are ready to kick off the build, run the following:
```` bash
make debian_image
````

#### Build the Kali Linux Image
* This step will build the base Kali linux image from the custom ````create-kali-image.sh```` script.  Once complete, your image will be in ````~/src/edison/edison-src/out/current/build/toFlash````.
* I do a lot of builds on high CPU ec2 images, so I find it useful to tar up the ````toFlash```` directory once this step is complete.  I can then transfer the contents of toFlash to the actual system I use to flash my edison.  When doing this, I dont use the directions in the following step.  I simply run the ````flashall.sh```` script in the toFlash directory.
```` bash
sudo ./meta-intel-edison/utils/create-kali-image.sh --build_dir=./out/current/build
````

#### Flash the Kali Image to the Edison Board.
* If you are customizing you base image filesystem before flashing, you can skip this step and just run ````./flashall.sh```` from the toFlash directory as you do in the manual Yacto build process. 
* Once the image is built, you can install it to the Edison board by running the following:
```` bash
make flash
````

#### Logging into your Kali Image
* You can log into your Kali Image via a terminal just like Edison Yacto or Debian images
* When logging into your Kali image, the login username and password is set to Username:````user```` Password:````edison````
````
screen /dev/USBDEVICE 115200
````

