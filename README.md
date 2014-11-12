Testing Secure Element API
==========
- Download QEMU-OPTEE based on [QEMU Script].
- You need a modified Qemu, which is shipped with SmartCard Simulator support
```sh
# install PC/SC Lite(pcscd) and development library
$ sudo apt-get install pcscd libpcsclite-dev
$ cd qemu
$ git remote add sy_repo https://github.com/m943040028/qemu.git
$ git fetch sy_repo
$ git checkout -b smart_card_emul sy_repo/smart_card_emul
$ ./configure --target-list="arm-softmmu" --enable-pcsc-passthru
$ make
```
- Install Vpcd, which is used to conduct the pcscd and SmartCard Simulator
```sh
$ git clone https://github.com/frankmorgner/vsmartcard.git
$ cd vsmartcard/virtualsmartcard/src/vpcd
$ make
$ sudo make install
(It will be installed to /usr/lib/pcsc/drivers/serial/libifdvpcd.so)
```
- Install and run the [JavaCard simulator]
```sh
# install java development kit
$ sudo apt-get install default-jdk
$ wget https://github.com/m943040028/jcardsim/releases/download/preview/jcardsim.jar
$ java -cp jcardsim.jar org.linaro.seapi.VpcdClient
```
- Get the SE API implementation source code
```sh
$ cd optee_os
$ git remote add sy_repo https://github.com/m943040028/optee_os.git
$ git fetch sy_repo
$ git checkout -b se_api sy_repo/se_api
```
- Build everything and run it.
```sh
$ ./build.sh
$ ./serial_0.sh (in other terminal)
$ ./serial_1.sh (in other terminal)
$ ./run_qemu.sh
(qemu) cont
```

[QEMU Script]:https://github.com/OP-TEE/optee_os/blob/master/scripts/setup_qemu_optee.sh
[JavaCard simulator]:https://github.com/m943040028/jcardsim/tree/se_api
