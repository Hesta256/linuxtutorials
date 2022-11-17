# Before you start

This is a simple guide (that actually makes sense) to setup CARLA 0.9.13 on Ubuntu 20.04. For this guide, you need the following prerequisites:

- Fresh Ubuntu 20.04 installation
- Computer with dedicated **nvidia** graphics driver
- Patience (and losing all hope for humanity)

---

# Dependencies
## Graphics driver and its dependencies
Check if you need to install the graphics driver by running

> `sudo lshw -numeric -C display`

If this shows **\*-display UNCLAIMED**, you need to install the driver. While you're at it, also install the vulkan driver, since CARLA 0.9.13 does not support openGL for desktop applications anymore. Start with

> `sudo apt-get install mesa-utils`  
> `apt search nvidia-driver`

Then check the latest Nvidia driver (at this time it is 520), install it with

>`sudo apt-get install nvidia-driver-520 nvidia-dkms-520`

Ensure it was installed correctly by running

>`sudo lshw -numeric -C display`

with the result showing **\*-display** without the word **UNCLAIMED**. Reboot to let the graphics driver load correctly

>`sudo reboot`

Now, we install vulkan driver and dependencies with

>`sudo add-apt-repository ppa:graphics-drivers/ppa`  
>`sudo apt-get install libvulkan1 mesa-vulkan-drivers vulkan-utils`  
>`sudo apt-get install nvidia-settings vulkan`

Update (for good measure)

>`sudo apt-get update && sudo apt-get -y upgrade`

Finally, install the Nvidia CUDA drivers

>`sudo apt-get install cuda-drivers`  
>`wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin`  
>`sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600`  
>`wget https://developer.download.nvidia.com/compute/cuda/11.8.0/local_installers/cuda-repo-ubuntu2004-11-8-local_11.8.0-520.61.05-1_amd64.deb`  
>`sudo cp /var/cuda-repo-ubuntu2004-11-8-local/cuda-*-keyring.gpg /usr/share/keyrings/`  
>_`# This avoids an error that the gpg key cannot be found by copying the key to the location where it is supposed to be`_  
>`sudo dpkg -i cuda-repo-ubuntu2004-11-8-local_11.8.0-520.61.05-1_amd64.deb`

Update and install CUDA

>`sudo apt-get update`  
>`sudo apt-get -y install cuda`

Now reboot and hopefully everything works fine. If Ubuntu does not start correctly (stuck at boot), you may need to [take some drastic measures](https://askubuntu.com/questions/668130/ubuntu-not-starting-post-cuda-installation). For now, you can boot in text-only mode (Ctrl+Alt+F1) and remove the _`nvidia-*`_ and _`cuda-*`_ drivers and try to see what went wrong.


## CARLA (0.9.13) dependencies

CARLA needs a lot, like **A LOT** of stuff to run correctly. To start

>`sudo apt-get update && `  
>`sudo apt-get install wget software-properties-common && `  
>`sudo add-apt-repository ppa:ubuntu-toolchain-r/test && `  
>`wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key|sudo apt-key add - && `  
>`sudo apt-add-repository "deb http://apt.llvm.org/xenial/ llvm-toolchain-xenial-8 main" && `  
>`sudo apt-get update`

Install _cmake_, _clang_, _python_, _boost_, ...

>`sudo apt-add-repository "deb http://apt.llvm.org/focal/ llvm-toolchain-focal main"`  
>`sudo apt-get install build-essential clang-10 lld-10 g++-7 cmake ninja-build libvulkan1 python python-dev python3-dev python3-pip libpng-dev libtiff5-dev libjpeg-dev tzdata sed curl unzip autoconf libtool rsync libxml2-dev git`

To avoid compatibility issues between Unreal Engine and the CARLA dependencies, use the same compiler version and C++ runtime library to compile everything. Change the default clang version to compile Unreal Engine and the CARLA dependencies.

>`sudo update-alternatives --install /usr/bin/clang++ clang++ /usr/lib/llvm-10/bin/clang++ 180 &&`  
>`sudo update-alternatives --install /usr/bin/clang clang /usr/lib/llvm-10/bin/clang 180`

Remember to add _`pip`_ to your PATH. One way to do so is to edit your profile file

>`~/.profile`

and at the end of it add the line

>`PATH="$HOME/.local/bin:$PATH"`

Now you need to restart the terminal to refresh your PATH.

CARLA uses different versions of python. Check if _`pip`_ and _`pip3`_ refer to Python2 and Python3 respectively with _`pip -V`_ and _`pip3 -V`_. If _`pip`_ uses Python3, force it to refer to Python2 with

>`curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip.py`  
>`sudo python2 get-pip.py`

**Note that _`pip`_ officially dropped support for Python2**, but we do this nonetheless to avoid possible errors with CARLA.

If you need to upgrade

>`pip3 install --upgrade pip && pip install --upgrade pip`

Continue to install python dependencies with:

>`sudo apt-get install python3-testresources`  
>`pip install --user setuptools && `  
>`pip3 install --user -Iv setuptools==47.3.1 && `  
>`pip install --user distro && `  
>`pip3 install --user distro && `  
>`pip install --user wheel && `  
>`pip3 install --user wheel auditwheel`


## Unreal Engine (4.26)

Starting with version 0.9.12, CARLA uses a modified fork of Unreal Engine 4.26. This fork contains patches specific to CARLA, so you need to download this fork of Unreal Engine. To start, you need to follow [this guide](https://www.unrealengine.com/en-US/ue-on-github) which sums up to

1. Create a GitHub account
2. Create an Epic Games account (and set a weekly reminder to download the free game every week on Saturday)
3. Log in to your Epic Games account, go to Account->Connections->Accounts->Connect GitHub. Follow the instructions till you link your github to your Epic Games account
4. Log in to your GitHub account, go to Settings->Developer Settings-> Personal Access Tokens->Tokens(classic). Generate a token with all scopes and any expiration day (we just need it to download Unreal Engine, you can delete it afterwards). Save the token before you close the webpage

Now we can download the fork of Unreal Engine for CARLA with

>`git clone --depth 1 -b carla https://github.com/CarlaUnreal/UnrealEngine.git ~/UnrealEngine_4.26`

and enter your GitHub credentials (username and token) to start your download. After a couple of minutes (or hours, depending on your internet), navigate to the download directory

>`cd ~/UnrealEngine_4.26`

and now make the build. This may take an hour or two (or four or more) depending on your system

>`./Setup.sh && ./GenerateProjectFiles.sh && make`

Meanwhile, you can download the CARLA release. **Note: Do NOT build CARLA**, it literally takes hours and isn't really worth it. Just download the release directly from GitHub and go on with your life. You can find [the download links here](https://github.com/carla-simulator/carla/releases/tag/0.9.13). Download _CARLA_0.9.13.tar.gz_ and _AdditionalMaps_0.9.13.tar.gz_, then extract both archives to the same directory using something like

>`mkdir -p ~/CARLA_0_9_13 && `  
>`tar -xvzf ~/Downloads/CARLA_0.9.13.tar.gz -C ~/CARLA_0_9_13 && `  
>`tar -xvzf ~/Downloads/AdditionalMaps_0.9.13.tar.gz -C ~/CARLA_0_9_13`



Now back to Unreal Engine. By now you may see a prompt to link the Unreal Engine externsions to UE4, just press yes. After Unreal Engine finishes building, you need to ensure it starts correctly, do so with

>`cd ~/UnrealEngine_4.26/Engine/Binaries/Linux && ./UE4Editor`

which will take, like another 10-15 minutes to build shaders and stuff, but should only happen for the first time you run UE4.

And you're finally done! Update (for good measure)

>`sudo apt-get update && sudo apt-get -y upgrade`

and reboot to start a fresh session. Now navigate to the directory where you extracted the CARLA archives, and try to run CARLA with

>`cd ~/CARLA_0_9_13 && ./CarlaUE4.sh`

If everything was done correctly, the default scene should load properly and you can traverse it using your mouse and the WASD keys. If the visualization is [too bright](https://github.com/carla-simulator/carla/issues/5278), you may try to reinstall your _`cuda_drivers`_ to solve this problem

>`sudo apt-get --reinstall install cuda-drivers`

Alternatively, if you have dual GPUs, CARLA may be launching using the integrated graphics card instead of the high-end card. Solve this by

>`export VK_ICD_FILENAMES="/usr/share/vulkan/icd.d/nvidia_icd.json"`

If this works, remember to add it to your _~/.bashrc_ file (and source the file for this session), so it gets permanently fixed.

---

# Start developing with CARLA

Now you can start using CARLA! Check the [First steps](https://carla.readthedocs.io/en/0.9.13/core_concepts/) for an introduction to the most important concepts.

You can also try to play around with the manual_control example. To do so, you need to install pygame using pip (and pip3)

>`pip install pygame numpy && pip3 install pygame numpy`

Go to your CARLA installation folder and start the server

>`./CarlaUE4.sh`

Then open a new terminal and run the manual_control example

>`cd ./PythonAPI/examples && ./manual_control.py`

If you get an error that pygame is not installed/found, "reinstall" it also using apt-get

> `sudo apt-get install python-pygame`

Now re-run the manual_control.py script and it should work just fine. Finally, you can change the simulation parameters using the config.py file. For example, change the weather conditions by opening a new terminal in your CARLA installation folder and executing

>`cd ./PythonAPI/util`  
>`python config.py --weather ClearNight`

to get a beautiful simulation of a clear night. Now you can finally go to sleep while your computer is slowly overheating from running CARLA.
