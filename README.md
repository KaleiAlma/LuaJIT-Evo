# LuaJIT for LuaSTG Evo

This is a customized version of LuaJIT for LuaSTG Sub, which mainly includes the following modifications:
* Added CMake support (not really)
* `io`, `os` libraries now support `utf-8` encoding
* Ported `utf8` from Lua 5.4
* Ported `string.pack` and `string.unpack` from Lua 5.4

Limitations:

* I am lazy, so updates may not be timely.

Credit goes to https://github.com/Demonese for most of the LuaJIT modifications, and obviously Mike Pall for writing LuaJIT to start with.
