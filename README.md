# linux-aarch64-flippy-cross-bin

A middle ground between [linux-aarch64-flippy](https://aur.archlinux.org/packages/linux-aarch64-flippy) and [linux-aarch64-flippy-bin](https://aur.archlinux.org/packages/linux-aarch64-flippy-bin), different from non-bin where you have to build the kernel and package it on an AArch64 host, and different from -bin where you have no freedom on deciding the kernel configs

You'll need to build the kernel and package the built binaries to archives on a powerful x86 host, and re-pack them to AArch64 packages on a normal AArch64 host.
## Preparation on x86-64
1. Install cross-compile tools
    ```
    sudo pacman -Syu base-devel aarch64-linux-gnu-gcc
    ```
2. Create a folder and populate with build script and config file. Only files in the `builder` folder in the repo is useful for x86-64.
    - Just clone the project and get into `linux-aarch64-flippy-cross-bin/builder` if you don't need a clean folder structure. Otherwise, clean it up a little bit as how you like it.
        ```
        PROJECT=linux-aarch64-flippy-cross-bin
        git clone https://github.com/7Ji/${PROJECT}
        cd ${PROJECT}/builder
        unset PROJECT
        ```
3. Download flippy's kernel source  
    - Either the latest source
        ```
        wget https://github.com/unifreq/linux-6.0.y/archive/refs/heads/main.tar.gz -O source.tar.gz
        ```
        Replace `linux-6.0.y` with other repos if you want to use older kernels instead
    - Or if you want a specific commit
        ```
        wget https://github.com/unifreq/linux-6.0.y/archive/ffada480851de173cb57188cee9447596051e014.tar.gz -O source.tar.gz
        ```
        Replace `linux-6.0.y` and commit with other repos and corresponding commit if you want to use older kernel/commit instead
    - Or if you want to use a kernel source from kernel.org
        ```
        wget https://git.kernel.org/torvalds/t/linux-6.2-rc1.tar.gz -O source.tar.gz
        ```
## One-command build on x86-64
You could just run the script to unpack, build and re-pack in one go:
```
./builder.sh all
```
After this, you will get `kernel.tar.gz`, `dtbs.tar.gz` and `headers.tar.gz` in the current folder, download & upload them to your AArch64 device

## Exact build on x86-64
The build could also be done in a few splitted steps if you want to go deep, so you could adjust some settings here and there, especially useful when debugging things
1. Unpack the source archive
    ```
    ./builder.sh unpack
    ```
2. Setting up the cross-compile environment
    ```
    . builder.sh setup
    ```
    (note it's `.`, not `./`,  the script must be included as part of the current Bash session so the environment could be effective)
3. Prepare the configuration and version info
    ```
    ./builder.sh prepare
    ```
4. Optionally, if you want to adjust the kernel settings, do it now
    ```
    cd builddir
    make nconfig
    cd ..
    ```
    (If you prefer `menuconfig`, you could always do it that way)
5. Actually build the kernel
    ```
    ./builder.sh build
    ```
6. Pack stuffs into archives
    ```
    ./builder.sh package
    ```
After this, you will get `kernel.tar.gz`, `dtbs.tar.gz` and `headers.tar.gz` in the current folder, download & upload them to your AArch64 device

## Preparation on AArch64
1. Install the base development tools
    ```
    sudo pacman -Syu base-devel
    ```
2. Create a folder and populate with essential package metainfos. Only files in the `packager` folder in the repo is useful for AArch64.
    - Just clone the project and get into `linux-aarch64-flippy-cross-bin/packager` if you don't need a clean folder structure. Otherwise, clean it up a little bit as how you like it.
        ```
        PROJECT=linux-aarch64-flippy-cross-bin
        git clone https://github.com/7Ji/${PROJECT}
        cd ${PROJECT}/packager
        unset PROJECT
        ```
## Re-pack on AArch64
1. Make sure you've transferred `kernel.tar.gz`, `dtbs.tar.gz` and `headers.tar.gz` from the x86-64 build host to the packager, the current work folder should contain the following files:
    - dtbs.tar.gz
    - headers.tar.gz
    - kernel.tar.gz
    - PKGBUILD
2. Just re-pack
    ```
    makepkg
    ```
3. Optionally install the packages and test them
    ```
    sudo pacman -U linux-aarch64-flippy-cross-bin-6.0.15-1-aarch64.pkg.tar.xz linux-aarch64-flippy-cross-bin-dtb-amlogic-6.0.15-1-aarch64.pkg.tar.xz linux-aarch64-flippy-headers-6.0.15-1-aarch64.pkg.tar.xz
    ```