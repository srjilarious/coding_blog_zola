+++
tags = ["FPGA"]
date = 2021-02-20
title = "FPGA Design for Software Engineers - Docker Builds"
+++

As I mentioned in the last [blog post](@/017-kubuntu/index.md) I recently switched my laptop over to using Linux as my main OS from Windows 10.  As part of that I was going through setting up all of my build tools for various projects and I realized the FPGA build system was a bit more cumbersome than it needed to be and, utilizing docker, it could be made more developer friendly.

<!--more-->

### Article Series

1. [Verilog and State Machines](@/014-fpga-start/index.md)
2. [Simulation and Build Tools](@/015-fpga-start2/index.md)
3. [Seven Segment Displays](@/016-fpga-start3/index.md)
4. Docker Builds
5. [Build System Updates, ECP5 Support](@/019-fpga-build-updates/index.md)
6. [Time-Multiplexed Seven Segment Displays](@/022-shift-reg-multiplex/index.md)

# Docker Build Setup

I only recently discovered how great docker can be for development, being something like 5 years behind others on the matter.  So I created a docker file for setting up a build container along with a helper script.  Since getting volume mapping and such just right can be difficult, the helper script is there to perform the setup as well as for building the individual projects.

The scripts are setup so that your user ID and group ID are used for build artifacts.  When building, a `build/` folder with the resulting artifacts is created within the container, but since the uid/gid match the host user's, you'll be able to access the files as you'd expect outisde of the container.

A neat feature of the setup is that the container's built conan libraries are mapped under the `build/conan_data` folder on the host, so that consecutive builds don't cause them to be rebuilt all the time.

## Setup Container

The first step to get going is to set up the docker container using the following command:

```bash
docker_build.sh setup
```

The `docker_build.sh` isn't very complicated, so feel free to peek inside to see how the user id and group id are being mapped.


## Building a project

```bash
docker_build.sh build <directory name of example>
```

For example: 
```bash
docker_build.sh build 01_state_machine
```

The artifacts in this example will show up under `build/01_state_machine`  Both the simulation project and the bitstream file will be built.

# Moving over to GitHub

I've recently started moving my main repositories over from GitLab to GitHub.  I still really like GitLab, but the way that commits are counted in the UTC timezone was pretty annoying, and since GitHub has provided private repos for a while I decided to make the move over.

I've also swapped to using `master` as the current state of the repo, including projects I have not yet written about.  Each article will still include a git branch you can use to check the best current state of the code for that article however.

You can find the fpga_start repo here: https://github.com/srjilarious/fpga_start

# Next Time

I recently got some time to finally build out the real hardware version of a time-multiplexed seven segment display circuit (`05_seven_seg_shift_reg_multi_time` if you want a sneak peek), and I planto write up the article on that -- soonish.

<div id="commento"></div>
<script src="https://cdn.commento.io/js/commento.js"></script>

