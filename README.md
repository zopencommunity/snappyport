# snappy for z/OS

[snappy](https://github.com/google/snappy) 1.2.2 — Google's compression
library, ported to z/OS.

snappy does not aim for the best compression ratio; it aims to be very fast.
That is why it is the default codec in Parquet, Cassandra and much of the
columnar-storage world, and it is one of the compression options Arrow C++
builds against.

## Installing

```sh
zopen install snappy
```

## Using it from another port

The port exports the usual flags, so a dependent build picks it up without
anything special:

```sh
export ZOPEN_STABLE_DEPS="... snappy"
```

which contributes `-I<prefix>/include`, `-L<prefix>/lib` and `-lsnappy`.

Directly:

```cpp
#include <snappy.h>

std::string packed;
snappy::Compress(input.data(), input.size(), &packed);

std::string unpacked;
snappy::Uncompress(packed.data(), packed.size(), &unpacked);
```

## Notes on the port

**No patches.** snappy builds clean on z/OS as it stands. It is self-contained
C++ with no OS-specific code beyond the standard library, so it also does not
need zoslib.

Two upstream options are turned off, because both default to ON and each pulls
in a dependency that is not ported:

| option | why |
| --- | --- |
| `SNAPPY_BUILD_TESTS=OFF` | needs googletest |
| `SNAPPY_BUILD_BENCHMARKS=OFF` | needs google/benchmark |

Turning off snappy's own tests means the port supplies its own. The check
compiles a program against the **installed** library and round-trips 215 KB of
compressible text through it, asserting that the output is smaller, that
`IsValidCompressedBuffer` accepts it, and that what comes back is byte-identical
to what went in. For a compressor that is the only claim that matters, and it
is checked against the artifact that actually ships rather than the build tree.
