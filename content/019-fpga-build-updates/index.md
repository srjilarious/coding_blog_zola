+++
date = 2022-11-14
title = "FPGA design for Software Engineers - Build System Updates, ECP5 Support"

[taxonomies]
tags = ["FPGA"]
+++

It's been a while since the [last article](/post/018_fpga_docker_build) and I've recently come back to playing with my FPGA repo.  Given the amount of time that has passed there were some updates needed for the `Dockerfile` and a couple of improvements I wanted to make.

<!--more-->

### Article Series

1. [Verilog and State Machines](@/014-fpga-start/index.md)
2. [Simulation and Build Tools](@/015-fpga-start2/index.md)
3. [Seven Segment Displays](@/016-fpga-start3/index.md)
4. [Docker Builds](@/018-fpga-docker-build.md)
5. Build System Updates, ECP5 Support
6. [Time-Multiplexed Seven Segment Displays](@/022-shift-reg-multiplex/index.md)


# Build System Updates

I updated the `Dockerfile` to use Ubuntu 22.04 as the base image, and cleaned up the package installation so that it does `apt install` just once rather than having it scattered around.  Also thanks to [@asahsieh](https://github.com/asahsieh) for the PR that fixed being able to `git clone` from veripool without disabling SSL.

While doing this I removed a bunch of half-finished projects or ones that I haven't yet written up blog posts about yet.

## Conanfile.txt No More

I recently started using `conan.cmake` in other CMake based projects for using conan.  It removes the extra step of doing `conan install ..` and instead puts everything into your CMake files, which is nice.  I had been using the `conan_cmake_run` macro, but it seems that it's now deprecated, moving towards a bit more verbose setup, but one that should be more future proof for Conan 2.0.

So now in a demo/example, you'll see a block like the following to setup the extra simulation packages:

```cmake
conan_cmake_configure(
    REQUIRES 
      spdlog/1.9.2
      sfml/2.5.1
    GENERATORS 
      cmake_find_package
    IMPORTS "bin, *.dll -> ./bin"
    IMPORTS "lib, *.dylib* -> ./bin"
    OPTIONS sfml:graphics=True
    OPTIONS sfml:shared=True
  )
conan_cmake_autodetect(settings)
conan_cmake_install(
    PATH_OR_REFERENCE .
    BUILD missing
    REMOTE conancenter
    SETTINGS ${settings}
  )

find_package(spdlog)
find_package(SFML)
```

The generator has changed to `cmake_find_package`, which means you now can use the standard `find_package` system from CMake for your dependencies, rather than the `${CONAN_LIBS_XYZ}` variables that were created before.

# Adding Lattice ECP5 Support

I heard about the ECP5 line of FPGAs from Lattice while keeping an eye on the TinyFPGA EX board.  Unfortunately that project seems to be dead.  I found another board called the [ULX3S](https://www.crowdsupply.com/radiona/ulx3s) which has a few options for the size of the ECP5 you want to get, and these can have up to 84,000 LUTs versus the Ice40 on the TinyFPGA-BX which has only 8,000.

Since that gives the opportunity for doing much larger designs in the future, I wanted to add support for the ECP5 line to the build system since Yosys also has support for them.

After adding in the new tools to the Docker file, some copy/pasting and messing with a new `yosys_ecp5.cmake` script, You can now target the ECP5 FPGAs as well as the Ice40 FPGAs.

To handle this, it made sense to move away from the `fpga_project` macro that did both the simulation and synthesis target setup and break it out more explicitly for each project.  This is because you will need a different top-level verilog for each FPGA you target, so rather than going crazy with parameters to the `fpga_project` macro, I just split it out into different target calls.

You can see the `00_blinky` example for how this will look for supporting both ECP5 and Ice40 flows. Its `CMakeLists.txt` file now looke like:

```cmake
cmake_minimum_required(VERSION 3.10)
project(blinky VERSION 1.0.0 LANGUAGES CXX)

# Include our cmake script that defines an fpga_project macro
include(../cmake/fpga_project.cmake)

set(SIM_SRC_FILES 
    main.cpp
 )

# Does setup, finding verilator and setting up conan.cmake macros.
fpga_project_setup()

conan_cmake_configure(
  REQUIRES 
    spdlog/1.9.2
  GENERATORS 
    cmake_find_package
  )
conan_cmake_autodetect(settings)
conan_cmake_install(PATH_OR_REFERENCE .
                    BUILD missing
                    REMOTE conancenter
                    SETTINGS ${settings})

find_package(spdlog)

fpga_simulation_project(
    TARGET blinky_ice40_sim
    TOP_LEVEL_VERILOG blinky_ice40.v
    SIM_SRC_FILES main_ice40.cpp
    LINK_LIBS spdlog::spdlog
  )

fpga_simulation_project(
    TARGET blinky_ecp5_sim
    TOP_LEVEL_VERILOG blinky_ecp5.v
    SIM_SRC_FILES main_ecp5.cpp
    LINK_LIBS spdlog::spdlog
  )

ice40_synthesis(
    TARGET blinky_ice40
    TOP_LEVEL_VERILOG blinky_ice40.v
    PCF_FILE ${CMAKE_SOURCE_DIR}/../support/tiny_fpga_bx_pins.pcf 
  )

ecp5_synthesis(
    TARGET blinky_ecp5
    TOP_LEVEL_VERILOG blinky_ecp5.v
    LPF_FILE ${CMAKE_SOURCE_DIR}/../support/ulx3s_pins.lpf
  )

```

Notice that here I've shown creating separate simulation targets for the ECP5 and the Ice40 versions.  As I work through other examples, I will instead have the actual logic in a main verilog file and separate top level verilog files that just map the board pins into the design.

With ECP5, there is a different pin constraint file format used, called `.lpf`, and I've moved the Ice40 one and it into the `support/` folder and renamed them to match the board they go with.

## Uploading a Design to the ULX3S

Looking at the [manual](https://github.com/emard/ulx3s/blob/master/doc/MANUAL.md) from ULX3S, I ended up cloning the `fujprog` [repo](https://github.com/kost/fujprog) and building it from source.  It was really fast and worked out of the box without issues.  I also tried `openFPGALoader` but it failed to upload the bitstream right away and I haven't investigated it any further so far.


## Blinky Updates

For `00_blinky` on the ULX3S, since there are 8 LEDs available, I map a larger part of the counter to the LEDs.  I also added a check for the second user button being down to pause the counter.

{{ img(alt="ULX3S Blinky", src="ulx3s_blinky.jpg") }}

## State Machine Updates

`01_state_machine` has been updated so that the state machine itself is contained in its own `state_machine.v` module.  The Ice40 and ECP5 top level modules then just do the mapping of their pins into that module.  

One nice benefit of this setup is that the state_machine module can be parameterized for simulation by default, and then overriden in the board specfic top-level modules, removing the need for the `` `ifdef SIMULATION `` blocks from before.

For the ECP5 version, the state machine LED is mapped to all 8 of the user leds:

{{ img(alt="ULX3S State Machine", src="ulx3s_state_machine.jpg") }}

## TestBench.h changes

In order to make the simulations for different boards work properly, I had to have a way to cycle the clock pin for each board, but they are named differently depending on the board.

Since that had always been a bit hacky anyways, I reworked the `TestBench` template class to expect to be subclassed to provide a `setClock` method.  This lets us refer to `CLK` on a TinyFPGA-BX and to `i_clk` on the ULX3S.

I looked at different ways to handle this with template magic - thinking maybe passing a reference to the clock signals member from the verilated module would work.  The problem ended up being that the signals are implemented as references in the verilated module, and it turns out you can't have a pointer to a member that is a reference, so that approach ended up not working.

So now instead of creating a pointer to your testbench module like this:

```cpp
int main(int argc, char **argv) 
{
    Verilated::commandArgs(argc, argv);
    auto tb = std::make_unique<TestBench<Vstate_machine>>();

    /* ... */
}
```

You instead subclass the Testbench template like this:

```cpp
struct TB : public TestBench<Vstate_machine>
{
    virtual ~TB() = default;

    // We overload the setClock signal method so we can reference different
    // clock signals for different FPGA boards.
    void setClock(uint8_t val) { mCore->i_clk = val; }
};

int main(int argc, char **argv) 
{
    Verilated::commandArgs(argc, argv);
    TB tb;

    /* ... */
}
```

# Conclusion

Now with the build system cleaned up and able to target different FPGAs and boards more easily, I'm looking forward to cleaning up and finally releasing the multiple seven segment display article I talked about [two years ago](/post/016_fpga_design_p3).

<div id="commento"></div>
<script src="https://cdn.commento.io/js/commento.js"></script>
