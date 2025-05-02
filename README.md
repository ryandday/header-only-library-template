# Myproj

# Dependencies

Depends on header-only [__boost::pfr__](https://github.com/boostorg/pfr) for reflection capabilities.

Make sure you have boost installed on your system before you build.

# Build

```
cmake . -B build
cmake --build build --parallel
```

## Dev mode (use static analyzers, use warnings, build tests, etc.)

```
cmake . -B build -DMYPROJ_DEV_MODE=ON
cmake --build build
```

# Install and use

To install onto system after building (linux / osx):

``` 
sudo cmake --install build
```

Then to include in another cmake project:

```
find_package(myproj)
target_link_libraries(YourProject myproj::myproj)
```

# TODO

Clang format

Pipeline - build run test

gtest
