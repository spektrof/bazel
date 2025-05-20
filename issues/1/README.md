# Issue#1: Virtual headers symlink action for C++ compilation action

To reproduce the problem, the project must have a specific layout.

## Using 'include_prefix'

Add a library target to `workspace_root/prefix/BUILD.bazel` with `include_prefix="prefix"`.

This case the virtual file and the original file will have the same path but no virtual headers are generated.

``` shell
$ bazel run //bin:app --config dep_mode2
prefix/foo.cpp:1:10: error: 'prefix/foo.hpp' file not found with <angled> include; use "quotes" instead
    1 | #include <prefix/foo.hpp>
      |          ^~~~~~~~~~~~~~~~
```

Even the command line includes `-Ibazel-out/darwin_x86_64-fastbuild/bin/prefix/_virtual_includes/lib` but no files are there.

``` shell
$ ls bazel-out/darwin_x86_64-fastbuild/bin/prefix/_virtual_includes/lib
ls: bazel-out/darwin_x86_64-fastbuild/bin/prefix/_virtual_includes/lib: No such file or directory
```

Moving the library target to a subfolder is working.

``` shell
$ bazel run //bin:app --config dep_mode4 
Target //bin:app up-to-date:
  bazel-bin/bin/app
INFO: Elapsed time: 1.897s, Critical Path: 1.60s
INFO: 8 processes: 6 action cache hit, 4 internal, 4 darwin-sandbox.
INFO: Build completed successfully, 8 total actions
INFO: Running command line: bazel-bin/bin/app
My lucky number: 42
```

## Using 'strip_include_prefix'

Add a library target to workspace_root/BUILD.bazel with `strip_include_prefix="."`.

``` shell
$ bazel run //bin:app --config dep_mode1
prefix/foo.cpp:1:10: error: 'prefix/foo.hpp' file not found with <angled> include; use "quotes" instead
    1 | #include <prefix/foo.hpp>
```

Moving the library target to a subfolder is working.

``` shell
$ bazel run //bin:app --config dep_mode3
INFO: Found 1 target...
Target //bin:app up-to-date:
  bazel-bin/bin/app
INFO: Elapsed time: 1.104s, Critical Path: 0.87s
INFO: 4 processes: 10 action cache hit, 3 internal, 1 darwin-sandbox.
INFO: Build completed successfully, 4 total actions
INFO: Running command line: bazel-bin/bin/app
My lucky number: 42
```
## Fix:

https://github.com/spektrof/bazel/commit/601874d31c2b734dc327fa6922ad646311e4d848
