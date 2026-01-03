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
