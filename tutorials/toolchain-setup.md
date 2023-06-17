# Toolchain Setup

Let’s say we’ve written a C program that we want to run on our MCU. 
The first thing we need to do is compile the source code into an executable. 
To do that we need a cross-compiler. I’m going to use the GNU Arm embedded toolchain.

## Installing for Windows

Download from https://developer.arm.com/downloads/-/gnu-rm.

Follow the installation instructions. Ensure that the toolchain
is added to the path. There should be a checkbox near the end of installation.

## Installing for Linux

I would recommend installing through your package manager, if possible. 
On Ubuntu it will look something like:

```
sudo apt-get install gcc-arm-none-eabi-bin arm-none-eabi-gdb
```

on Arch:

```
pacman -S gcc-arm-none-eabi-bin arm-none-eabi-gdb
```

Otherwise, you can also download a tarball from the Arm website above.

## Testing the Toolchain

Once your toolchain is installed you can test that it generates an executable.

Write the following to a file called `test.c`:

```
int main() {

    while (1)
        ;
}
```

Then compile it with the following flag:

```
$ arm-none-eabi-gcc test.c -nostdlib
```

If it worked properly, you should see a file called `a.out` in your current directory. 

## Prep for Flashing

There's one last thing we need to do before we can flash our program to the MCU and run it.
The output of compilation, `a.out` is formatted as an ELF file, but the MCU expects it to be 
organized in a plain sequence of instructions. We can convert `a.out` to such a file using one of the 
binary utilities installed:

```
$ arm-none-eabi-objcopy -O binary a.out a.bin
```

This will generate a flat binary `a.bin`. This is the type of file 
you'll actually be flashing to your MCU (although don't expect this particular file to
do anything other than crash it).

## Toolchain Overview

Although we often refer to compilation as a single step, it’s really a multi-stage process with compilation being just one of the steps.

    Source -> preprocessing -> compilation -> assembly -> linking

The toolchain is a suite of programs that perform each of these steps to turn source code into an executable. 
If you poke around the directory where you installed the toolchain, you’ll see dozens of individual programs. 
Luckily, we don’t have to interact with each of them manually. The one you’ll interact with most is `arm-none-eabi-gcc`. 
This is the compiler driver, a top-level program that facilitates the multi-stage compilation process. 

`arm-none-eabi-gcc` invokes the *actual* compiler, a program called `cc1`. 
`cc1` takes your C source files as input and outputs the assembly representation. 
You don’t see the output files because they’re only stored temporarily during the process. 

`arm-none-eabi-gcc` then invokes `arm-none-eabi-as`, the assembler, 
which takes the assembly files as input and for each one outputs the equivalent object file, the actual binary code itself. 

Lastly, `arm-none-eabi-gcc` invokes `arm-none-eabi-ld`, the loader,
 which combines the individual object files into a single executable.

In our case, we also performed one more step: run the output ELF file through `arm-none-eabi-objcopy` to 
get a flat binary version of the file.

Programs like `as`, `ld`, `objcopy` and many others included in the toolchain make up a separate package referred
to as `binutils`.

