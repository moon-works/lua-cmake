CMake patches for Lua 5.x
=========================

This is patchset that implements cmake build system for lua 5.x.

Repository layout
-----------------

Git repo has main lua branches (`lua-5.1`, `lua-5.2`...) and topic branches
(`t/lua-cmake/5.1`, `t/lua-cmake/5.2`...).
Branch like `t/lua-cmake` is topic branch, controlled by topgit.

Building
--------

To build Lua, you need to run these commands:

```
mkdir build
cd build
cmake ..
make
```

Developing
----------

Clone repo:

```
git clone https://github.com/moon-works/lua-cmake.git
tg populate --remote
git checkout t/lua-cmake/5.1
# Now we on lua-cmake/5.1 topic branch
tg info
```

Exporting patchset:

```
tg export > lua-cmake-5.1.patch
```

Don't forget `tg push` after changing topic branch!

