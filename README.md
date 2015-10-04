OpenEmbedded  layer for GStreamer 1.0
=====================================

This layer provides UNOFFICIAL support for the
[GStreamer 1.0 framework](http://gstreamer.freedesktop.org/) for use with
OpenEmbedded and/or Yocto. It is used for GStreamer recipe backports, to
provide older OE versions with support for newer GStreamer versions, and
also as a staging ground for recent recipe upgrades which haven't yet made
it into OE-core (for example, because a new GStreamer version was released
just recently).


Dependencies
------------

* URI: git://git.openembedded.org/openembedded-core
* branch: master
* revision: HEAD

Additionally, the meta-multimedia and meta-oe layers of the meta-openembedded repo at
git://git.openembedded.org/meta-openembedded are necessary for optional plugins
(see below).


Package names and coexistence with 0.10
---------------------------------------

Since 1.0 was designed to be able to coexist with 0.10 in the same system, the names
of the recipes for GStreamer 1.0 reflect that. The names are:

* gstreamer1.0
* gstreamer1.0-plugins-base
* gstreamer1.0-plugins-good
* gstreamer1.0-plugins-bad
* gstreamer1.0-plugins-ugly
* gstreamer1.0-libav
* gstreamer1.0-omx


Building git versions
---------------------

By default, release tarballs are unpacked and built. It is possible, however, to build from the GStreamer
git repositories instead. It is generally better to use the releases, however for bleeding edge features
the git versions might be necessary.

To let Bitbake know that the git version of a package is preferred, add to local.conf:

    PREFERRED_VERSION_<packagename> = "git"

For example, to use git versions of all packages, add to local.conf:

    PREFERRED_VERSION_gstreamer1.0 = "git"
    PREFERRED_VERSION_gstreamer1.0-plugins-base = "git"
    PREFERRED_VERSION_gstreamer1.0-plugins-good = "git"
    PREFERRED_VERSION_gstreamer1.0-plugins-bad = "git"
    PREFERRED_VERSION_gstreamer1.0-plugins-ugly = "git"
    PREFERRED_VERSION_gstreamer1.0-libav = "git"
    PREFERRED_VERSION_gstreamer1.0-omx = "git"


Enabling plugins with dependencies
----------------------------------

By default, in the base/good/bad/ugly recipes, only dependency-less plugins and plugins with dependencies
that are supported by OE-core (i.e. recipes for them exist in OE-core) are always enabled.

These are:
* gstreamer1.0-plugins-base : ivorbis (Tremor), ogg, theora, vorbis, pango, gio-unix-2.0
* gstreamer1.0-plugins-good : cairo, flac, gdk-pixbuf, gudev, jpeg, libpng, soup, speex, taglib
* gstreamer1.0-plugins-bad : curl, uvch264, neon, sndfile, hls, sbc, dash, bz2, smoothstreaming, rsvg, dtls
* gstreamer1.0-plugins-ugly : a52dec, lame, mad, mpeg2dec

With the X11, Wayland, ALSA, BlueZ, DirectFB, OpenGL, and PulseAudio plugins, the situation is a bit different.
They are built depending on the contents of the DISTRO\_FEATURES value (in other words, depending on what the OE
distribution supports).

The rest is disabled by default, and can be enabled by appending to the respective PACKAGECONFIG values.
For example, to enable vpx and wavpack support in gstreamer1.0-plugins-good , add to local.conf:

    PACKAGECONFIG_append_pn-gstreamer1.0-plugins-good = "vpx wavpack"

Note that after enabling a plugin this way, it must be ensured that recipes for the plugin's dependencies
are available. In the example above, recipes for vpx and wavpack must exist. This typically means that
additional OE layers must be used (often meta-oe or meta-multimedia).

This is also how Orc support is enabled internally. Since version 1.6.0, Orc 0.4.23 is included in this layer,
and enabled by default. (Orc 0.4.23 is present in OE-Core, but to make GStreamrer 1.6 buildable with older
layers, its recipe is included.)

Below is a list of all configuration values for enabling additional plugins and features in the packages.

* gstreamer1.0-plugins-good
    * `jack` : JACK audio system plugins
    * `vpx` : plugins for en- and decoding VP8 video streams, using Google's libvpx
    * `wavpack` : WavPack plugins
* gstreamer1.0-plugins-bad
    * `assrender` : ASS/SSA subtitle renderer plugins
    * `faac` : AAC encoding plugins using the FAAC library
    * `faad` : AAC decoding plugins using the FAAD library
    * `libmms` : Microsoft Multimedia Stream plugins
    * `modplug` : Decoder plugins for module files (MOD/S3M/XM/IT/..) using the ModPlug library
    * `mpg123` : MPEG-1 layer 1/2/3 audio decoder plugin using the mpg123 library
    * `opus` : Opus audio decoder plugin
    * `flite` : Flite speech synthesizer plugins
    * `opencv` : OpenCV image processing plugins
    * `openal` : OpenAL audio plugins
    * `fluidsynth` : FluidSynth plugins
    * `schroedinger` : Dirac video codec plugins using the schroedinger library
    * `webp` : WebP plugins
    * `rtmp` : Real Time Messaging Protocol (RTMP) plugins
    * `libssh2` : Enable libssh2 support in cURL plugins
    * `gtk` : GTK+3 plugins
    * `qt5` : Qt5 QML plugins
* gstreamer1.0-plugins-ugly
    * `cdio` : Compact Disc audio plugins using libcdio
    * `dvdread` : DVD source plugins using libdvdread
    * `x264` : h.264/AVC encoder plugin using libx264
* gstreamer1.0-libav
    * `libav` : builds the package using the system's libav instead of the included one (*not recommended* unless you really know what you are doing!)
    * `gpl` : build the package in GPL mode (enables GPL elements)


OpenMAX IL support
------------------

The gstreamer1.0-omx package adds support for OpenMAX IL. By default, the
recipe is configured to use the bellagio OpenMAX IL implementation that ships
with OE-Core. BSP layers are encouraged to add .bbappend files which set the
`GSTREAMER_1_0_OMX_TARGET` variable to make gstreamer1.0-omx use the device
specific OpenMAX IL support. Currently, these device specific targets are
supported:

* `rpi` : Raspberry Pi OpenMAX IL implementation

If the value of `GSTREAMER_1_0_OMX_TARGET` is changed by a .bbappend file to
a device specific value, the recipe automatically sets the `PACKAGE_ARCH` of
gstreamer1.0-omx to `MACHINE_ARCH`.

Furthermore, if a device specific .bbappend file is written, it is recommended
to also set the value of `GSTREAMER_1_0_OMX_CORE_NAME` in it. This value
specifies the filename of the OpenMAX core (a shared library) that needs to be
used. With the Bellagio OpenMAX implementation, its value is:
`${libdir}/libomxil-bellagio.so.0`. The gstreamer1.0-omx recipe needs this value
for adjusting the `core-name` entries in the `gstomx.conf` configuration file.
