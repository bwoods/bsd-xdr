# XDR ![Build Status](https://travis-ci.org/bwoods/bsd-xdr.svg?branch=master)

This package contains a port of Sun's XDR library. It was derived from the implementation in the libtirpc package (version 0.1.10–7) from Fedora 11. That version was relicensed with explicit permission from the copyright holder (Sun Microsystems) to a BSD license. See the LICENSE file for more information.

## The goals of this port

  - Maintain the BSD license rather than copylefting it, as
    in portableXDR and various other versions.
  - Avoid autotools.
  - Avoid "config.h" pollution (or similar) in public header files.
  - Compile successfully at highest reasonable warning level,
    with warnings treated as errors.
  - Compile library as both static and shared.
  - Provide thorough tests of all libxdr facilities.
  - Where feasible, link test programs against both static and shared
    libraries.
  - Verify identical output on a variety of platforms, especially 32-
    and 64-bit hosts.


## Notes

Many xdr implementations are not 64bit clean. The original SunRPC implementation relied heavily on core transport-specific methods `xdr_putlong` and `xdr_getlong` — and assumed that “longs” were, in fact, exactly 32 bits. For 64-bit platforms this fails in many wonderful ways.

The implementation here addresses this difficulty in two ways:

1. the original integer primitives whose sizes can by problematic (`xdr_[u_]int`, `xdr_[u_]long`) take special care to do the right thing -- so long as the value can be represented in 32 bits. If the native type (`[unsigned] int` or `[unsigned] long`) is bigger than 32 bits, *but* the value contained can be represented using just those 32 bits, then just those 32 bits are xdr’ed. Otherwise, attempts to xdr the value return `FALSE` (instead of blindly xdr’ing the lower 32 bits to represent the value).
2. New size-specific primitives (`xdr_intN`, `xdr_uintN`, `xdr_u_intN`, where `N` is 8, 16, 32, or 64) are provided. These use transport-specific implementation methods `xdr_putint32` and `xdr_getint32` that are insensitive to variations of the width of `int`/`long`/`short` on various platforms. New code should use these primitives. Most modern implementations of xdr support them.

The supplied tests are very thorough with regards to:

- memory buffer transport
- stdio FILE* transport

in that every core primitive is tested on both input and output. Try `v` (or `-v -v -v -v`) when running the test programs. All of the tested platforms produce identical output for all cases (except for 64-bit linux; but only because it skips two known-to-fail tests: `xdr_long` and `xdr_u_long` with `MAX_LONG` and `MAX_U_LONG`. On 64-bit platforms, `MAX_LONG` and `MAX_U_LONG` cannot be represented with just 32 bits, so `xdr_[u_]long` returns `FALSE` and the test fails — but in fact, this actually demonstrates that the “protect against assuming longs are 32 bits” code is working as desired.

The `sizeof` test measures the size of a structure that includes a primitive of every type, and also the size of a populated linked list. (Yes, xdr can serialize data structures that have pointers, under certain conditions:

- there are no circular references — so doubly-linked lists are out, and 
- every object to be serialized is pointed-to by exactly one pointer — so no ‘sentinel nodes’ or ‘every element has a reference to HEAD/TAIL’ schemes.


See comments in `src/test/test_data.c` concerning `xdr_pgn_node_t_RECURSIVE`, `xdr_pgn_list_t_RECURSIVE`, and `xdr_pgn_list_t` for more information).
