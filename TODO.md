# TODO: Porting build to CMake

## Targets

- `argon2`: executable
- ✅ `libargon2` shared/static library

## C Standard

- ✅ `C89/90`

## Options

- ✅ Threads support: if not enabled, set define ARGON2_NO_THREADS
- check if can build with optimizations (`opt.c`) or not (`ref.c`)

## pkg-config

- ✅ configure and install `libargon2.pc.in`
