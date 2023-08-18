# DISCLAIMER: This is not a fork to be merged into PCem; its entire purpose is to temporarily remedy this major problem with 8 year old code in hopes PCem/86box devs finally care to actually fix it
## What is this?
This fork does one very simple thing: it reimplements memory cache from PCem v11 (which was removed in v12's dev, see [this](https://github.com/sarah-walker-pcem/pcem/commit/f311c9e17fbf6b0e0f6478c1c6f896b95858b02c)).

Why? Because this cache turns out to be essential in avoiding MAJOR slowdowns that occur in PCem, particularly on the Windows desktop.
While removing this cache has led to various speedups on the guest side of things, it interestingly enough leads to a speed drop of roughly 20-30% in the emulator. 

Note: I don't really understand how PCem even works, let alone why this occurs; all I know is that this slowdown bug has been neglected by devs of PCem & all it's derivatives to the point where I have been using v10.1 for a long while simply because it didn't exhibit it, but eventually I traced the bug down to its origins and this is what I came up with.

### Test Scenario
- Find a high-clocked CPU which you can run at 100% guaranteed in PCem v11 (for me, I found the Pentium MMX 200 to be a good choice).
- Install NT4, followed by SP3 and IE4 _in PCem v11_ (SP4+ don't work unless you [patch IDE](https://github.com/sarah-walker-pcem/pcem/commit/13496ba7b9d449e4547c7b9705ac49a4cfeceba5)).
- Try to open IE4, "Explore" My Computer, or just browse files in general. Everything should work normally and no slowdowns should be observed.
- Use the exact same config in PCem latest (I just grabbed the latest dev build).
- Attempting to do the above actions would result in major (but momentary) slowdowns. On the long run, this has caused my clock to desynch around 5 mins on the guest in around 5 hours of real world time. This doesn't seem much; however, when the internet is involved, things get corrupted fast.
- Attempting to use this build observes the same behavior in PCem v11, despite being a little slower in operation than latest PCem. However, reliability is to be preffered over speed.

In perspective, the same concept should work on 86box too (to be tested).

### Building on Windows (MSYS2)
Make sure to use MINGW32 (32-bit is fastest).
You also obviously need the toolchain before building this.

#### Libraries
- SDL2
- wxWidgets 3.x
- OpenAL
- CMake
- Ninja
- PCAP

Use this command to build in an MSYS2 MINGW32 terminal 
```
cmake -G "Ninja" -DMSYS=TRUE -DCMAKE_BUILD_TYPE=Release .
ninja
```

then `./src/pcem` to run.

You can specify the Display Engine using `-DPCEM_DISPLAY_ENGINE=` The options you have are wxWidgets, and Qt
configure options are :
```
  -DCMAKE_BUILD_TYPE=Release : Generate release build. Recommended for regular use.
  -DCMAKE_BUILD_TYPE=Debug   : Compile with debugging enabled.
  -DUSE_NETWORKING=ON        : Build with networking support.
  -DUSE_PCAP_NETWORKING=ON   : Build with pcap networking support. (Needs USE_NETWORKING to compile) Requires libpcap.
  -DUSE_ALSA=OFF             : Build with support for MIDI output through ALSA. Requires libasound. (Linux Only)
  -DFORCE_X11=ON             : Enables a hack to force X11 on Wayland systems. See #128 for details. (Linux Only)
  -DPLUGIN_ENGINE=OFF        : Build with plugin support. Builds libpcem-plugin-api and links PCem with it. 
```

If you are using -DCMAKE_BUILD_TYPE=Debug, there are some more debug options you can enable if needed
```
  -DPCEM_SLIRP_DEBUG=ON           : Build PCem with SLIRP_DEBUG debug output
  -DPCEM_RECOMPILER_DEBUG=ON      : Build PCem with RECOMPILER_DEBUG debug output
  -DPCEM_NE2000_DEBUG=ON          : Build PCem with NE2000_DEBUG debug output
  -DPCEM_EMU8K_DEBUG_REGISTERS=ON : Build PCem with EMU8K_DEBUG_REGISTERS debug output
  -DPCEM_SB_DSP_RECORD_DEBUG=ON   : Build PCem with SB_DSP_RECORD_DEBUG debug output
  -DPCEM_MACH64_DEBUG=ON          : Build PCem with MACH64_DEBUG debug output
  -DPCEM_DEBUG_EXTRA=ON           : Build PCem with DEBUG_EXTRA debug output
```

If you are using -DCMAKE_BUILD_TYPE=RelWithDebInfo, there are additional options you can do
```
  -DPCEM_RELDEB_AS_RELEASE=ON     : Builds RelWithDebInfo with debugging logging enabled when this is off
```

They are some extra modules you can add if you build with `-DUSE_EXPERIMENTAL=ON`. These modules are untested.
incomplete, and may or may not be in a future build of PCem. We do not provide builds with these enabled as
well. It is also possible they may not even build.
```
  -DUSE_EXPERIMENTAL_PGC=ON       : Build PCem with Professional Graphics Controller support.
  -DUSE_EXPERIMENTAL_PRINTER=ON   : Build PCem with Printer support. Requires freetype.
```
