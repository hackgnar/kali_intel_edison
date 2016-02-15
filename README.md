# Kali for Intel Edison

* need to edit the following script
````
sudo ~/src/edison/edison-src/meta-intel-edison/utils/create-debian-image.sh --build_dir=~/src/edison/edison-src/out/current/build
````

* to build
````
cp ~/kali_intel_edison/build_script/create-debian-image.sh ./meta-intel-edison/utils/
sudo ./meta-intel-edison/utils/create-kali-image.sh --build_dir=./out/current/build
````
