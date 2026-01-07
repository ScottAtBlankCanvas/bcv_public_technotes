# Getting started developing with gstreamer

## Resources

### Articles

- [Good development getting started](https://rosemary-crypto.github.io/blog/2024/build-with-gstreamer-Lesson-1/)

### Docs

- [GStreamer FAQ about running gst](https://gstreamer.freedesktop.org/documentation/frequently-asked-questions/using.html?gi-language=c)



## Installing Binaries (MacOS)

### Installing on Mac (problems and cleaning up)

- Tried following the [official Mac install docs](https://gstreamer.freedesktop.org/documentation/installing/on-mac-osx.html?gi-language=c)
  - Although most people online said to use homebrew

- Installed (both runtime and development pkgs)[https://gstreamer.freedesktop.org/download/#macos]
  - Not able to run gst-launch-1.0.  Not on PATH.  (fixed)
  - But then several dependent library issues (gobject, glib, others)

- Realizing I had gst installed from some previous work, and that brew on binaries likely conflicted, I uninstalled everything

```
# forget installed pkg
pkgutil --pkgs | grep gstreamer | xargs -I % sudo pkgutil --forget %

# Remove from disk
sudo rm -rf /Library/Frameworks/GStreamer.framework

# remove brew install
brew uninstall --ignore-dependencies gstreamer

```

### Installing on Mac (brew)

- Now install using brew and try to run a simple command
```
brew install gstreamer
which gst-launch-1.0

# try a command
gst-launch-1.0 filesrc location=/path/sample.mp4 ! decodebin ! autovideosink
# Error: gst-launcher Failed to load shared library 'libgobject-2.0.0.dylib’ and others

# install with brew
brew install glib gobject-introspection

# runs!
gst-launch-1.0 filesrc location=/path/sample.mp4 ! decodebin ! autovideosink

```

### Installing on Mac (binaries)

OK, let's see if we can get running using pkg installer

- Uninstall everything

```
# forget installed pkg
pkgutil --pkgs | grep gstreamer | xargs -I % sudo pkgutil --forget %

# Remove from disk
sudo rm -rf /Library/Frameworks/GStreamer.framework

# remove brew install
brew uninstall --ignore-dependencies gstreamer glib gobject-introspection

which gst-launch-1.0
```

- OK, now install just the (runtime pkg file)[https://gstreamer.freedesktop.org/download/#macos]
- Does not add to path, so try with explicit path:
```
# Plays video!
/Library/Frameworks/GStreamer.framework/Versions/1.0/bin/gst-launch-1.0 filesrc location=/Users/scott/code/videos/content/sample.mp4 ! decodebin ! autovideosink

```

- Even tho it plays, I do get this error:
** (gst-plugin-scanner:29788): CRITICAL **: 09:55:54.343: pygobject initialization failed: could not import gobject (error was: ModuleNotFoundError("No module named 'gi'"))
- Of course, missing glib and gobject:

```
# Provide missing glib and gobject
brew install glib gobject-introspection
```

- Try again... running with no error!!


## Setting up Dev Environment

Following [these instructions](https://gstreamer.freedesktop.org/documentation/installing/on-mac-osx.html?gi-language=c)

### Using develop binary

- OK, now install just the (develop pkg file)[https://gstreamer.freedesktop.org/download/#macos]

- Clone gst-docs subproject

```
cd ~/code/gstreamer/
git clone https://gitlab.freedesktop.org/gstreamer/gstreamer.git
cd subprojects/gst-docs
```

## Hello world

Let's try to compile a simple hello world.  [here](https://gstreamer.freedesktop.org/documentation/installing/on-mac-osx.html?gi-language=c)

- First we need to add the packages to pkg-config and put things on the PATH
```
export PKG_CONFIG_PATH="/Library/Frameworks/GStreamer.framework/Libraries/pkgconfig:$PKG_CONFIG_PATH"
export PATH="/Library/Frameworks/GStreamer.framework/Versions/1.0/bin:$PATH"
# Also potentially needed for linking:
export DYLD_FALLBACK_LIBRARY_PATH="/Library/Frameworks/GStreamer.framework/Libraries"
```

Created main.c per the example (and added a printf)
```
#include <stdio.h>
#include <gst/gst.h>

int main(int argc, char *argv[])
{
  gst_init(NULL, NULL);

  printf("main: hello world!\n");

  return 0;
}
```

Compile, link, and run:
```
clang -c main.c -o main.o `pkg-config --cflags gstreamer-1.0`
clang -o main main.o `pkg-config --libs gstreamer-1.0`
otool -L main
# runs
./main
```

## Simple video Player

Now let's build gthe example found [here](https://rosemary-crypto.github.io/blog/2024/build-with-gstreamer-Lesson-1/)

I found several issues compiling and trying to run:
- The callback needed moved before the main, as it needs to be declared
- Missing includes
- Mac requires some additional code as it cannot do a UI loop as written
- Change hard-coded path to file

```
#include <stdio.h>
#include <gst/gst.h>
#ifdef __APPLE__
#include <TargetConditionals.h>
#endif

// Callback function to dynamically link pads
void on_pad_added(GstElement* src, GstPad* new_pad, GstElement* sink) {
    GstPad* sink_pad = gst_element_get_static_pad(sink, "sink");

    // Check if the sink pad is already linked
    if (gst_pad_is_linked(sink_pad)) {
        g_object_unref(sink_pad);
        return;
    }

    // Attempt to link the newly created pad with the sink pad
    GstPadLinkReturn ret = gst_pad_link(new_pad, sink_pad);
    if (GST_PAD_LINK_FAILED(ret)) {
        g_printerr("Type is '%s' but link failed.\n",
            gst_structure_get_name(gst_caps_get_structure(gst_pad_get_current_caps(new_pad), 0)));
    }

    g_object_unref(sink_pad);
}

int internal_main(int argc, char* argv[]) {
    GMainLoop* loop;
    GstBus* bus;
    guint bus_watch_id;

    // Initialize GStreamer
    gst_init(&argc, &argv);

    // Create a main loop, which will run until we explicitly quit
    loop = g_main_loop_new(NULL, FALSE);

    // Create the elements
    GstElement* pipeline = gst_pipeline_new("video-player");
    GstElement* source = gst_element_factory_make("filesrc", "file-source1");
    GstElement* decoder = gst_element_factory_make("decodebin", "decode-bin1");
    GstElement* autovideosink = gst_element_factory_make("autovideosink", "video-output1");

    // Check that all elements were created successfully
    if (!pipeline || !source || !decoder || !autovideosink) {
        g_printerr("Not all elements could be created.\n");
        return -1;
    }

    // Set the source file location
    g_object_set(G_OBJECT(source), "location", "../big_buck_bunny_scene.mp4", NULL);

    // Add all elements to the pipeline
    gst_bin_add_many(GST_BIN(pipeline), source, decoder, autovideosink, NULL);

    // Link the source to the decoder (decoder will dynamically link to sink)
    if (!gst_element_link(source, decoder)) {
        g_printerr("Source and decoder could not be linked.\n");
        gst_object_unref(pipeline);
        return -1;
    }

    // Connect the pad-added signal for the decoder
    g_signal_connect(decoder, "pad-added", G_CALLBACK(on_pad_added), autovideosink);

    // Set the pipeline to the playing state
    gst_element_set_state(pipeline, GST_STATE_PLAYING);

    // Start the main loop and wait until it is quit (e.g., via EOS or an error)
    g_main_loop_run(loop);

    // Clean up: set the pipeline to NULL state, remove bus watch, and unref
    gst_element_set_state(pipeline, GST_STATE_NULL);
    gst_object_unref(GST_OBJECT(pipeline));
    g_main_loop_unref(loop);

    return 0;
}

int
main (int argc, char *argv[])
{
#if defined(__APPLE__) && TARGET_OS_MAC && !TARGET_OS_IPHONE
  return gst_macos_main ((GstMainFunc) internal_main, argc, argv, NULL);
#else
  return internal_main (argc, argv);
#endif
}
```

```
clang++ -c simple_player.cpp -o simple_player.o `pkg-config --cflags gstreamer-1.0`
clang++ -o simple_player simple_player.o `pkg-config --libs gstreamer-1.0`
./simple_player
```
