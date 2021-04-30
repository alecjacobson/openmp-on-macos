# Testing OpenMP 🧵 on MacOS 

> **TL;DR** `brew install libomp` and `-Xpreprocessor -fopenmp -lomp`

IIRC, it used to be that the only way to get OpenMP working on macos was to
install GCC or a non-apple-build version of clang. Mixing compilers can be a
nightmare (maybe this is easier now with Conda etc.).

Happily, it appears that using OpenMP on macos is [now much easier](https://stackoverflow.com/questions/43555410/enable-openmp-support-in-clang-in-mac-os-x-sierra-mojave/60564952#60564952).

If you haven't yet installed openmp, consider the `test.cpp` included here with
contents:

```cpp
#include <omp.h>
#include <stdio.h>
int main() 
{
    #pragma omp parallel
    printf("Hello from thread %d, nthreads %d\n", omp_get_thread_num(), omp_get_num_threads());
}
```

Try compiling this with the default clang:

    clang++ test.cpp

You may see something like:

    test.cpp:1:10: fatal error: 'omp.h' file not found
    #include <omp.h>
             ^
    1 error generated.

Using homebrew we can install openmp (note: we're not replacing clang or
installing another compiler):

    brew install libomp

Now, if you try to recompile you'll get a bit farther:

    clang++ test.cpp

may produce:

    Undefined symbols for architecture x86_64:
      "_omp_get_num_threads", referenced from:
          _main in test-885e44.o
      "_omp_get_thread_num", referenced from:
          _main in test-885e44.o
    ld: symbol(s) not found for architecture x86_64
    clang: error: linker command failed with exit code 1 (use -v to see invocation)

This linker error is fixed by adding the `-lomp` flag:

    clang++ test.cpp -lomp

However, running the executable:

    ./a.out

reveals that we're not really getting multi-threading:

    Hello from thread 0, nthreads 1

On seemingly every other compiler we'd need to add `-fopenmp`, for builtin macos
clang we need `-Xpreprocessor -fopenmp`. So, finally compiling with:

    clang++ test.cpp -lomp -Xpreprocessor -fopenmp

will produce a binary `./a.out` which produces something like:

    Hello from thread 0, nthreads 8
    Hello from thread 3, nthreads 8
    Hello from thread 4, nthreads 8
    Hello from thread 1, nthreads 8
    Hello from thread 7, nthreads 8
    Hello from thread 2, nthreads 8
    Hello from thread 5, nthreads 8
    Hello from thread 6, nthreads 8
