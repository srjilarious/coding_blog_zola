+++
tags = ["C++"]
date = 2019-09-02
title = "Bitfield widths in C/C++ structures"
Categories = ["Cpp"]
+++


Recently, I learned about a feature in C and C++ that I hadn't run into before: bit widths for fields in structures.  The idea is that you can specify how many bits a particular field should have allocated to it.  For example, you can have a field that is only 5 bits wide and therefore only takes values in [0, 32).  You could then have a field that is 3 bits wide that is placed next to the first field in memory.

I decided to play around with them a bit, trying to create a packed [RGB 565](https://en.wikipedia.org/wiki/RGB565) structure using bitfield widths rather than doing the bitmasking and shifting by hand.  I noticed a couple of interesting things in the process:

1. It's implementation dependent as to what order the fields show up; left-to-right or right-to-left in memory.
2. It's implementation dependent as to whether fields span across type size boundaries.

You can read more in depth details [here on cppreference](https://en.cppreference.com/w/cpp/language/bit_field)

## An example

Here is the code I hacked together to see how it works.  There are three versions of basically the same RGB565 structure, but with minor changes, first with field type, and then with order:

    #include <cstdio>

    struct rgb565_a {
        unsigned char r : 5;
        unsigned char g : 6;
        unsigned char b : 5;
    };

    struct rgb565_b {
        unsigned short r : 5;
        unsigned short g : 6;
        unsigned short b : 5;
    };

    struct rgb565_c {
        unsigned short b : 5;
        unsigned short g : 6;
        unsigned short r : 5;
    };

    template <typename T, typename RawVal>
    void printStructInfo() {
        printf("-----------------------------\n");
        printf("sizeof %s = %lu\n", __PRETTY_FUNCTION__, sizeof(T));
        
        T val;
        val.r = 15;
        val.g = 63;
        val.b = 31;

        printf("packed struct contains: %d, %d, %d\n", val.r, val.g, val.b);

        RawVal rawVal = *(RawVal*)&val;
        printf("raw value is 0x%x\n\n\n", rawVal);
    }

    int main() {
        printStructInfo<rgb565_a, unsigned short>();
        printStructInfo<rgb565_b, unsigned short>();
        printStructInfo<rgb565_c, unsigned short>();
    }

Running this with GCC 7.4.0 you get:

     $ g++ ./bitfield_tests.cpp 
    
     $ ./a.out
    -----------------------------
    sizeof void printStructInfo() [with T = rgb565_a; RawVal = short unsigned int] = 3
    packed struct contains: 15, 63, 31
    raw value is 0x3f6f


    -----------------------------
    sizeof void printStructInfo() [with T = rgb565_b; RawVal = short unsigned int] = 2
    packed struct contains: 15, 63, 31
    raw value is 0xffef


    -----------------------------
    sizeof void printStructInfo() [with T = rgb565_c; RawVal = short unsigned int] = 2
    packed struct contains: 15, 63, 31
    raw value is 0x7fff

What you see is that the first structure is having each field placed into its own byte, not spanning the `g` field across the first and second bytes.

The second structure shows that the structure is packed little endian on my machine, meaning `r` is in the lower bits, `g` in the middle, and `b` in the upper bits, so actually a BGR565 structure.

The last one is what I actually wanted: 16 bits packed tightly together, with red in the upper bits and blue in the bottom.

## Summary

Letting the compiler do the bitmasking and shifting for you looks like a useful feature on the surface, but I'd be careful to have a unit test when using this in production.  I'd want to make sure that on any particular architecture I build for, that I would get the raw values that I expect so I wouldn't end up with red and blue switched around when changing compilers or targets.

<div id="commento"></div>
<script src="https://cdn.commento.io/js/commento.js"></script>
