# SWarmprom-IMproved aka SWIM

SWarmprom-IMproved, also known as SWIM, is an updated version of the original [Swarmprom project](https://github.com/stefanprodan/swarmprom).

The aim for SWIM is to be as easy to deploy and use as the original Swarmprom, but use up to date versions of the software used, be easier to maintain, configure and extend for different situations.

SWIM uses the official distributed container images. This makes upgrading or downgrading to different versions is easy to do, as as switching to custom built images, and also to mix the different versions of the different programs depending on your needs.

The original Swarmprom used custom built container images that only supported AMD64 architecture and was locked to one specific version for all the programs used. SWIM makes it easier to deploy on ARM64 as well as AMD64 hardware. It should also be easier to deploy on new and less popular hardware, such as various ARM versions, RISC-V and other non-AMD64 architectures that are increasingly gaining a lot of popularity.

To deploy a different image than the default, you need to modify the compose file to use different container image(s). If the version of the software does not exist as an container image, or your hardware architecture is not supported, then you can in theory build them yourself from source since all the software used by SWIM is OSS with the Apache License, Version 2.0.

Please help improve SWIM by submitting PRs and/or add write suggestions on how to improve SWIM. For suggestions please use the GitHub discussion forum or create an GitHub issue.

Have a good day or evening wherever you are.