---
layout: default
---

[Table of Contents](toc.md) > Your first package

# Your First Package

We will begin walking you through building a simple package.  This will introduce you to a basic software build from source, and then convert that into a package to be installed by the Pacman package manager.

## Hello!

All good programming experiences start with a simple "Hello, world!".  So will your packaging journey.

[GNU Hello](https://www.gnu.org/software/hello/) is an almost-trivial free software program that prints the phrase "Hello, world!" or a translation thereof to the screen. This is primarily to serve as an example of the GNU coding standards, and serve as template for more serious software projects. We shall use it as an introduction to the concept of package software.

## Building From Source

We will first walk through the steps of manually building the software.

### Obatining the Source

From the project website, we can follow the [link](https://ftp.gnu.org/gnu/hello/) through to download the source. We will select the latest version of the software to build, which at the time of writing is hello 2.10.

````shell
$ wget https://ftp.gnu.org/gnu/hello/hello-2.10.tar.gz
````

We have downloaded the source file (`hello-2.10.tar.gz`). Note that a cryptographic signature (`hello-2.10.tar.gz.sig`) is also available for download and could be use to verify the integrity of the source file. We will skip this for now, but verifying source files will be covered in more detail later.

Now we can move onto building the software.

### Building the Software

The first step is to extract the source tarball and change into the source directory:

````shell
$ tar -xf hello-2.10.tar.gz
$ cd hello-2.10
````

For this example, inside the source directory you will find a file named `INSTALL` with instructions of how to build and install the software. Other time you may need to follow instructions on a software's wiki page. Following these instructions we can build the software with the following commands:

````shell
$ ./configure
$ make
````

We can then test the build worked correctly with:

````shell
$ ./hello
Hello, world!
````
We could then install the software to our system.  It is not advisable to install software to the system without a package manager, so **do not run** this example command.  It would also need run as the `root` user to have access to the system directory.

````shell
$ make install
````

That is all it takes to build this piece of software. Most (but not all) will be slightly more complicated than this!

## Creating a PKGBUILD

