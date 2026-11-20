# Maszyna i narzędzia

Wszystkie wyniki pochodzą z jednej maszyny, zwykłego komputera biurkowego bez izolacji pomiarowej.

| element | wartość |
|---|---|
| procesor | Intel Core i3-6100 (Skylake), 3,7 GHz, 2 rdzenie / 4 wątki, L2 512 KB, L3 3 MB |
| pamięć | 32 GB DDR4-2133 (2 × 16 GB), bez ECC; odnotowane rzadkie przekłamania pojedynczych bitów (`METHODOLOGY.md`) |
| płyta główna | MSI B150M PRO-VDH |
| GPU | Intel HD Graphics 530 (zintegrowana, 23 jednostki, OpenCL 3.0) |
| system | Windows 10 Pro (build 19045) |
| Java (JVM) | OpenJDK 27 (HotSpot, kompilator C2), flagi z pliku produkcyjnego: sterta 384 MB, Serial GC |
| Java, Kotlin, Scala (obrazy natywne) | Oracle GraalVM 25.0.4 (`native-image`, optymalizacja `-O3` pod procesor tej maszyny, z profilem kompilacji i bez) |
| OpenJ9 | IBM Semeru 25.0.5 (OpenJ9 0.63.0) |
| Kotlin / Scala | 2.4.21 / 3.10.0 |
| Rust | 1.99.0, toolchain MSVC, `target-cpu=x86-64-v3`, alokator mimalloc, z profilem kompilacji i bez |
| C# | .NET SDK 10.0.402 (JIT i NativeAOT) |
| Go | 1.27.2, `GOAMD64=v3`, z profilem kompilacji i bez |
| C++ | MSVC 14.51, C++23, `/O2 /GL /arch:AVX2`, z profilem kompilacji |
| TypeScript | Node 24.21.0 (sterta V8 ograniczona do 256 MB) i Bun 1.4.2, bez kroku kompilacji |
| WebAssembly | wasmtime 50.0.0 |
| Python | 3.14.4 (wywołania przez `ctypes`) |
