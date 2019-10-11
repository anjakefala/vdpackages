# PyOxidizer

Evaluate the feasibility of using [PyOxidizer](https://pyoxidizer.readthedocs.io/en/latest/) to build standalone, statically linked binaries of VisiData for Linux & MacOS.

### Status

It is possible to get successful builds of VisiData with PyOxidizer on both MacOS and Ubuntu 18.04 so far. However, as of this writing those executables won't actually *run*. At this point, other packaging methods seem like a smoother path.

### Notes / Tweaks from Testing

#### General

* Due to numpy-related complications (see [this issue](https://github.com/indygreg/PyOxidizer/issues/65)), any loader with pandas/numpy needs to be excluded before VisiData will build successfully.

#### MacOS

* The `_curses` module is explicitly excluded from the linking pass ([reference](https://github.com/indygreg/PyOxidizer/blob/ec600e181ec9fd5ca272944500f358c99d152fcf/pyoxidizer/src/pyrepackager/repackage.rs#L55-L75)). This seems like a dealbreaker.

#### Ubuntu 18.04

* After a successful build, `pyoxidizer run` gives:

```AttributeError: 'NoneType' object has no attribute 'loader'```

Tweaking `pyoxidizer.toml` or trimming `requirements.txt` to the bare minimum don't seem to get past that error or help identify its source.
