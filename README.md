Testing Secure Element API
==========
- Download QEMU-OPTEE based on [QEMU Script].
- You need a [Modified QEMU], which is shipped with SmartCard Simulator support; it is simply a device that acts a client of [PC/SC Lite] and provided a very simple hardware/software interface to the guest machine.
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
- Install Vpcd, which is used to conduct the [PC/SC Lite] and [JavaCard simulator]
```sh
$ git clone https://github.com/frankmorgner/vsmartcard.git
$ cd vsmartcard/virtualsmartcard/src/vpcd
$ make
$ sudo make install
(It will be installed to /usr/lib/pcsc/drivers/serial/libifdvpcd.so)
```
- Install and run the Enhanced [JavaCard simulator] (It's enhanced to support GlobalPlatform logical channel management)
```sh
# install java development kit
$ sudo apt-get install default-jdk
$ wget https://github.com/m943040028/jcardsim/releases/download/release1/jcardsim.jar
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
- I have an integration test built at **core/tee/se/reader/passthru_reader/test.c**, it should be passed without error (I know it is tricky, but it's very convenient at this stage. I am planned to move it to a static TA). If you got any error, please feel free to contact me.

Technical Background
==========
In order to let you understand what is happened in the test case, some terminologies need to be introduced
- **Section:** The only way that a TA can gain access to a card reader is through session. Different TAs can have a session opened on the same reader. It's SE manager's responsibility to demux the request from different TAs. Upon a seesion is opened by a TA, the card is power-up and ready to accept commands.
- **Applet:** An applet is an application runs on a SmartCard OS. It's possbile to have multiple applets running on the system, and each is identified by **AID**. The only method to communicate with applet is through **Logical Channel**
- **Logical Channel:** It's used for host application(in our case, a TA) to communcate with applets on the SmartCard. [GlobalPlatform Card Specification] defined maximum 20 logical channels, numbered from 0~19. Channel number 0 is so-called **Basic logical channel**, or in short, **Basic channel**. A channel can be opened or closed by a host application (TA, in our case). It's the SmartCard OS's responsbility to manage the state of logical channels. Basic channel is always opened and cannot be closed. A channel must **select** an applet, which means the command passed through the channel will be process by the selected applet. GlobalPlatform required a default applet must be selected on basic channel after system reset. Host application can select different applet by issues a **SELECT command** on basic channel. Other logical channels (numbered 1~19) can be opened with or without a given **AID**. If AID is not given, the applet selected on basic channel will be selected on the just opened logical channel.
- **MultiSelectable and Non-MultiSelectable Applet:** A applet can be MultiSelectable or Non-MultiSelectable. For a Non-MultiSelectable applet, it can only be selected by a channel, further **SELECT command** on other channel that targeting to the applet will fail to success. **MultiSelectable** applet can be selected by multiple channels, the applet can decide maximum number of channels it's willing to accept.

[QEMU Script]:https://github.com/OP-TEE/optee_os/blob/master/scripts/setup_qemu_optee.sh
[Modified QEMU]:https://github.com/m943040028/qemu/tree/smart_card_emul
[JavaCard simulator]:https://github.com/m943040028/jcardsim/tree/se_api
[PC/SC Lite]:https://pcsclite.alioth.debian.org/
[GlobalPlatform Card Specification]:http://www.globalplatform.org/specificationscard.asp
