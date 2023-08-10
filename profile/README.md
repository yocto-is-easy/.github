# Yocto is easy

Hi! That is my pet project, the goal of which to learn how Yocto works and what can I do with it.

### Provided features

- [x] Support of custom request responce library (LRRP - Lightweight request-response library)
- [x] Custom RPC framework (MIDF - Mocto interface definition framework)
- [x] Util to observe other services' state
- [x] Support of DS18B20 temperature sensor
- [x] Remote test execution
- [x] Logging service with ring buffer
- [ ] Support of plugin manager
- [ ] Wifi service support
- [ ] Support of initializing smart devices over serial(ESP32)

### Build

> :warning: The developer may work after 6 p.m. by Kyiv until midight. So, the sources may be not buildable during development process.

1. Get sources

We are using repo tool to manage all the yocto sources. As for the custom modules, they are placed on github and has been downloading during the Yocto build process.

How to install repo you can see [here](https://gerrit.googlesource.com/git-repo/+/refs/heads/master/README.md)

Init the manifest:
```bash
repo init -u git@github.com:yocto-is-easy/manifest.git
```

Do the sync of sources:
```bash
repo sync
```

2. Build docker image and run its container

Before doing this step it is recommended to include your user to the docker group.

It is created docker image to build Yocto without dependency to the development platform(Arch, Ubuntu, etc.)

From the top directory run:
```bash
source docker/env-setup.sh
```

Run the builder:
```bash
run.sh
```

3. Set the target machine

In workdir/build/conf/local.conf change the MACHINE variable:
```bash
MACHINE ??= "raspberrypi3-64"
```

If you have a board of another version (like Raspberry Pi 4), so, set it.

4. Build Yocto image

The project has several images:
- mocto-image - release image
- mocto-image-dbg - debug image(it includes tests for services and remote executor support)

```bash
bitbake mocto-image-dbg
```

5. Flush image to SD card

SD card image is placed in the `workdir/build/tmp/deploy/images/raspberrypi3-64`

Use the following command to flush the image:
```bash
dd if=mocto-image-dbg-raspberrypi3-64.rpi-sdimg of=/your/sd-card/path
```

6. Run Raspberry Pi

Put the card to your device, run it, connect over ssh and enjoy the result

Several commands to test image.

**Check the states of services:**
```bash
observe
```

result:
```bash
temp_check : online
test_service : online
```

**Run one of available tests:**
```bash
exec-local temp-check-tests
```

result:
```bash
Running main() from /home/build/workdir/build/workspace/googletest/googletest/src/gtest_main.cc
[==========] Running 2 tests from 1 test suite.
[----------] Global test environment set-up.
[----------] 2 tests from TempCheckTest
[ RUN      ] TempCheckTest.ok_temperature_format
[       OK ] TempCheckTest.ok_temperature_format (1647 ms)
[ RUN      ] TempCheckTest.get_temperature_time_1s
[       OK ] TempCheckTest.get_temperature_time_1s (832 ms)
[----------] 2 tests from TempCheckTest (2479 ms total)

[----------] Global test environment tear-down
[==========] 2 tests from 1 test suite ran. (2479 ms total)
[  PASSED  ] 2 tests.
```

**Run the client of temp-check-service (aka temp check client):**
```bash
tcc
```

result:
```bash
Current temperature is: 22.437 C
```
