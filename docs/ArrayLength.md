# Length Benchmarks

Figuring out the most efficient way to compare 

<details><summary>SourceCode</summary>

```csharp
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Jobs;

namespace Benchmarkz;

[SimpleJob(RuntimeMoniker.Net48)]
[SimpleJob(RuntimeMoniker.Net60)]
[SimpleJob(RuntimeMoniker.Net80)]
[MemoryDiagnoser(displayGenColumns: true)]
public class AnyBenchmark
{
    private uint[] smallArray;
    private uint[] mediumArray;
    private uint[] bigArray;
    private List<uint> bigList;
    private HashSet<uint> bigHashset;
    private IEnumerable<uint> bigEnumerable;

    [GlobalSetup]
    public void GlobalSetup()
    {
        smallArray = FillArray(100).ToArray();
        mediumArray = FillArray(10000).ToArray();
        bigArray = FillArray(1000000).ToArray();
        bigList = FillArray(1000000).ToList();
        bigHashset = FillArray(1000000).ToHashSet();
        bigEnumerable = FillArray(1000000);
    }

    [Benchmark]
    public void AnySmall() => _ = smallArray.Any();

    [Benchmark]
    public void AnyMedium() => _ = mediumArray.Any();

    [Benchmark]
    public void AnyBig() => _ = bigArray.Any();

    [Benchmark]
    public void AnyBigList() => _ = bigList.Any();

    [Benchmark]
    public void AnyBigHashet() => _ = bigHashset.Any();

    [Benchmark]
    public void AnyBigEnum() => _ = bigEnumerable.Any();

    [Benchmark]
    public void LenghtSmall() => _ = smallArray.Length != 0;

    [Benchmark]
    public void LenghtMedium() => _ = mediumArray.Length != 0;

    [Benchmark(Baseline = true)]
    public void LenghtBig() => _ = bigArray.Length != 0;

    [Benchmark]
    public void LenghtBigList() => _ = bigList.Count != 0;

    [Benchmark]
    public void LenghtBigHashet() => _ = bigHashset.Count != 0;

    [Benchmark]
    public void LenghtBigEnum() => _ = bigEnumerable.Count() != 0;


    [Benchmark]
    public void TakeCountSmall() => _ = smallArray.Take(1).Count() == 1;

    [Benchmark]
    public void TakeCountMedium() => _ = mediumArray.Take(1).Count() == 1;

    [Benchmark]
    public void TakeCountBig() => _ = bigArray.Take(1).Count() == 1;

    [Benchmark]
    public void TakeCountBigList() => _ = bigList.Take(1).Count() == 1;

    [Benchmark]
    public void TakeCountBigHashet() => _ = bigHashset.Take(1).Count() == 1;

    [Benchmark]
    public void TakeCountBigEnum() => _ = bigEnumerable.Take(1).Count() == 1;

    [Benchmark]
    public void EnumeratorCheckSmall() => _ = EnumMagic(smallArray);

    [Benchmark]
    public void EnumeratorCheckMedium() => _ = EnumMagic(mediumArray);

    [Benchmark]
    public void EnumeratorCheckBig() => _ = EnumMagic(bigArray);

    [Benchmark]
    public void EnumeratorCheckBigList() => _ = EnumMagic(bigList);

    [Benchmark]
    public void EnumeratorCheckBigHashset() => _ = EnumMagic(bigHashset);

    [Benchmark]
    public void EnumeratorCheckBigEnumerable() => _ = EnumMagic(bigEnumerable);

    private static IEnumerable<uint> FillArray(uint count)
    {
        for (uint i = 0; i < count; i++)
            yield return i;
    }

// net48 lacks TryGetNonEnumeratedCount
#if NET6_0_OR_GREATER
    private static bool EnumMagic<T>(IEnumerable<T> source)
    {
        if (source.TryGetNonEnumeratedCount(out var count))
            return count > 1;

        using var enumerator = source.GetEnumerator();
        return enumerator.MoveNext();
    }
#else
    private static bool EnumMagic<T>(IEnumerable<T> source)
    {
        using var enumerator = source.GetEnumerator();
        return enumerator.MoveNext();
    }
#endif
}
```

</details>

```
// * Detailed results *
AnyBenchmark.AnySmall: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 5.519 ns, StdErr = 0.037 ns (0.67%), N = 35, StdDev = 0.218 ns
Min = 5.264 ns, Q1 = 5.335 ns, Median = 5.425 ns, Q3 = 5.734 ns, Max = 5.989 ns
IQR = 0.399 ns, LowerFence = 4.736 ns, UpperFence = 6.333 ns
ConfidenceInterval = [5.387 ns; 5.652 ns] (CI 99.9%), Margin = 0.133 ns (2.40% of Mean)
Skewness = 0.49, Kurtosis = 1.78, MValue = 2.44
-------------------- Histogram --------------------
[5.257 ns ; 5.432 ns) | @@@@@@@@@@@@@@@@@@
[5.432 ns ; 5.627 ns) | @@@@@@
[5.627 ns ; 5.893 ns) | @@@@@@@@@@
[5.893 ns ; 6.077 ns) | @
---------------------------------------------------

AnyBenchmark.AnyMedium: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 5.154 ns, StdErr = 0.002 ns (0.04%), N = 13, StdDev = 0.007 ns
Min = 5.142 ns, Q1 = 5.150 ns, Median = 5.156 ns, Q3 = 5.158 ns, Max = 5.170 ns
IQR = 0.008 ns, LowerFence = 5.138 ns, UpperFence = 5.171 ns
ConfidenceInterval = [5.146 ns; 5.163 ns] (CI 99.9%), Margin = 0.009 ns (0.17% of Mean)
Skewness = 0.29, Kurtosis = 2.58, MValue = 2
-------------------- Histogram --------------------
[5.138 ns ; 5.174 ns) | @@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyBig: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 5.639 ns, StdErr = 0.004 ns (0.06%), N = 15, StdDev = 0.014 ns
Min = 5.613 ns, Q1 = 5.630 ns, Median = 5.638 ns, Q3 = 5.646 ns, Max = 5.667 ns
IQR = 0.016 ns, LowerFence = 5.606 ns, UpperFence = 5.670 ns
ConfidenceInterval = [5.624 ns; 5.653 ns] (CI 99.9%), Margin = 0.015 ns (0.26% of Mean)
Skewness = 0.33, Kurtosis = 2.38, MValue = 2
-------------------- Histogram --------------------
[5.606 ns ; 5.674 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyBigList: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 3.467 ns, StdErr = 0.002 ns (0.07%), N = 13, StdDev = 0.009 ns
Min = 3.447 ns, Q1 = 3.465 ns, Median = 3.465 ns, Q3 = 3.475 ns, Max = 3.479 ns
IQR = 0.010 ns, LowerFence = 3.450 ns, UpperFence = 3.490 ns
ConfidenceInterval = [3.456 ns; 3.477 ns] (CI 99.9%), Margin = 0.010 ns (0.30% of Mean)
Skewness = -0.57, Kurtosis = 2.8, MValue = 2
-------------------- Histogram --------------------
[3.442 ns ; 3.484 ns) | @@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyBigHashet: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 3.441 ns, StdErr = 0.003 ns (0.07%), N = 15, StdDev = 0.010 ns
Min = 3.417 ns, Q1 = 3.436 ns, Median = 3.442 ns, Q3 = 3.447 ns, Max = 3.457 ns
IQR = 0.012 ns, LowerFence = 3.419 ns, UpperFence = 3.465 ns
ConfidenceInterval = [3.431 ns; 3.452 ns] (CI 99.9%), Margin = 0.010 ns (0.30% of Mean)
Skewness = -0.58, Kurtosis = 3.15, MValue = 2
-------------------- Histogram --------------------
[3.412 ns ; 3.463 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyBigEnum: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 11.739 ns, StdErr = 0.022 ns (0.19%), N = 14, StdDev = 0.083 ns
Min = 11.629 ns, Q1 = 11.682 ns, Median = 11.713 ns, Q3 = 11.785 ns, Max = 11.889 ns
IQR = 0.103 ns, LowerFence = 11.528 ns, UpperFence = 11.939 ns
ConfidenceInterval = [11.645 ns; 11.833 ns] (CI 99.9%), Margin = 0.094 ns (0.80% of Mean)
Skewness = 0.56, Kurtosis = 1.85, MValue = 2
-------------------- Histogram --------------------
[11.583 ns ; 11.934 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.LenghtSmall: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 0.000 ns, StdErr = 0.000 ns (NaN%), N = 15, StdDev = 0.000 ns
Min = 0.000 ns, Q1 = 0.000 ns, Median = 0.000 ns, Q3 = 0.000 ns, Max = 0.000 ns
IQR = 0.000 ns, LowerFence = 0.000 ns, UpperFence = 0.000 ns
ConfidenceInterval = [0.000 ns; 0.000 ns] (CI 99.9%), Margin = 0.000 ns (NaN% of Mean)
Skewness = NaN, Kurtosis = NaN, MValue = 2
-------------------- Histogram --------------------
[-0.500 ns ; 0.500 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.LenghtMedium: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 0.000 ns, StdErr = 0.000 ns (100.00%), N = 12, StdDev = 0.002 ns
Min = 0.000 ns, Q1 = 0.000 ns, Median = 0.000 ns, Q3 = 0.000 ns, Max = 0.006 ns
IQR = 0.000 ns, LowerFence = 0.000 ns, UpperFence = 0.000 ns
ConfidenceInterval = [-0.002 ns; 0.002 ns] (CI 99.9%), Margin = 0.002 ns (443.70% of Mean)
Skewness = 2.65, Kurtosis = 8.48, MValue = 2
-------------------- Histogram --------------------
[-0.001 ns ; 0.001 ns) | @@@@@@@@@@@
[ 0.001 ns ; 0.003 ns) |
[ 0.003 ns ; 0.005 ns) |
[ 0.005 ns ; 0.007 ns) | @
---------------------------------------------------

AnyBenchmark.LenghtBig: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 0.000 ns, StdErr = 0.000 ns (74.06%), N = 15, StdDev = 0.001 ns
Min = 0.000 ns, Q1 = 0.000 ns, Median = 0.000 ns, Q3 = 0.000 ns, Max = 0.003 ns
IQR = 0.000 ns, LowerFence = 0.000 ns, UpperFence = 0.000 ns
ConfidenceInterval = [-0.001 ns; 0.001 ns] (CI 99.9%), Margin = 0.001 ns (306.64% of Mean)
Skewness = 2.65, Kurtosis = 9, MValue = 2
-------------------- Histogram --------------------
[-0.000 ns ; 0.001 ns) | @@@@@@@@@@@@@
[ 0.001 ns ; 0.001 ns) | @
[ 0.001 ns ; 0.002 ns) |
[ 0.002 ns ; 0.003 ns) | @
---------------------------------------------------

AnyBenchmark.LenghtBigList: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 0.000 ns, StdErr = 0.000 ns (100.00%), N = 15, StdDev = 0.001 ns
Min = 0.000 ns, Q1 = 0.000 ns, Median = 0.000 ns, Q3 = 0.000 ns, Max = 0.002 ns
IQR = 0.000 ns, LowerFence = 0.000 ns, UpperFence = 0.000 ns
ConfidenceInterval = [-0.001 ns; 0.001 ns] (CI 99.9%), Margin = 0.001 ns (414.05% of Mean)
Skewness = 3.13, Kurtosis = 11.39, MValue = 2
-------------------- Histogram --------------------
[-0.000 ns ; 0.000 ns) | @@@@@@@@@@@@@@
[ 0.000 ns ; 0.001 ns) |
[ 0.001 ns ; 0.002 ns) |
[ 0.002 ns ; 0.002 ns) |
[ 0.002 ns ; 0.003 ns) | @
---------------------------------------------------

AnyBenchmark.LenghtBigHashet: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 0.002 ns, StdErr = 0.001 ns (22.84%), N = 13, StdDev = 0.002 ns
Min = 0.000 ns, Q1 = 0.001 ns, Median = 0.002 ns, Q3 = 0.003 ns, Max = 0.006 ns
IQR = 0.003 ns, LowerFence = -0.004 ns, UpperFence = 0.008 ns
ConfidenceInterval = [0.000 ns; 0.005 ns] (CI 99.9%), Margin = 0.002 ns (98.60% of Mean)
Skewness = 0.26, Kurtosis = 1.74, MValue = 2
-------------------- Histogram --------------------
[-0.001 ns ; 0.001 ns) | @@@@
[ 0.001 ns ; 0.004 ns) | @@@@@@
[ 0.004 ns ; 0.006 ns) | @@@
---------------------------------------------------

AnyBenchmark.LenghtBigEnum: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 2.227 ms, StdErr = 0.004 ms (0.19%), N = 15, StdDev = 0.016 ms
Min = 2.190 ms, Q1 = 2.219 ms, Median = 2.233 ms, Q3 = 2.237 ms, Max = 2.244 ms
IQR = 0.019 ms, LowerFence = 2.190 ms, UpperFence = 2.266 ms
ConfidenceInterval = [2.209 ms; 2.244 ms] (CI 99.9%), Margin = 0.017 ms (0.78% of Mean)
Skewness = -1, Kurtosis = 2.83, MValue = 2
-------------------- Histogram --------------------
[2.181 ms ; 2.252 ms) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountSmall: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 22.853 ns, StdErr = 0.050 ns (0.22%), N = 13, StdDev = 0.181 ns
Min = 22.520 ns, Q1 = 22.801 ns, Median = 22.863 ns, Q3 = 23.003 ns, Max = 23.174 ns
IQR = 0.202 ns, LowerFence = 22.498 ns, UpperFence = 23.306 ns
ConfidenceInterval = [22.635 ns; 23.070 ns] (CI 99.9%), Margin = 0.217 ns (0.95% of Mean)
Skewness = -0.16, Kurtosis = 2.11, MValue = 2
-------------------- Histogram --------------------
[22.418 ns ; 23.276 ns) | @@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountMedium: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 24.489 ns, StdErr = 0.060 ns (0.25%), N = 15, StdDev = 0.234 ns
Min = 24.029 ns, Q1 = 24.358 ns, Median = 24.508 ns, Q3 = 24.580 ns, Max = 24.895 ns
IQR = 0.222 ns, LowerFence = 24.025 ns, UpperFence = 24.913 ns
ConfidenceInterval = [24.238 ns; 24.739 ns] (CI 99.9%), Margin = 0.250 ns (1.02% of Mean)
Skewness = -0.02, Kurtosis = 2.26, MValue = 2
-------------------- Histogram --------------------
[23.904 ns ; 25.020 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountBig: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 22.727 ns, StdErr = 0.058 ns (0.25%), N = 15, StdDev = 0.224 ns
Min = 22.127 ns, Q1 = 22.640 ns, Median = 22.805 ns, Q3 = 22.895 ns, Max = 22.943 ns
IQR = 0.255 ns, LowerFence = 22.258 ns, UpperFence = 23.277 ns
ConfidenceInterval = [22.487 ns; 22.966 ns] (CI 99.9%), Margin = 0.240 ns (1.05% of Mean)
Skewness = -1.29, Kurtosis = 3.93, MValue = 2
-------------------- Histogram --------------------
[22.008 ns ; 22.597 ns) | @@
[22.597 ns ; 23.063 ns) | @@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountBigList: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 20.223 ns, StdErr = 0.026 ns (0.13%), N = 14, StdDev = 0.097 ns
Min = 19.935 ns, Q1 = 20.209 ns, Median = 20.251 ns, Q3 = 20.279 ns, Max = 20.314 ns
IQR = 0.069 ns, LowerFence = 20.105 ns, UpperFence = 20.383 ns
ConfidenceInterval = [20.114 ns; 20.333 ns] (CI 99.9%), Margin = 0.110 ns (0.54% of Mean)
Skewness = -1.79, Kurtosis = 5.74, MValue = 2
-------------------- Histogram --------------------
[19.882 ns ; 20.367 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountBigHashet: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 30.938 ns, StdErr = 0.042 ns (0.14%), N = 15, StdDev = 0.162 ns
Min = 30.469 ns, Q1 = 30.865 ns, Median = 31.001 ns, Q3 = 31.050 ns, Max = 31.090 ns
IQR = 0.184 ns, LowerFence = 30.589 ns, UpperFence = 31.326 ns
ConfidenceInterval = [30.765 ns; 31.112 ns] (CI 99.9%), Margin = 0.173 ns (0.56% of Mean)
Skewness = -1.55, Kurtosis = 4.92, MValue = 2
-------------------- Histogram --------------------
[30.383 ns ; 31.177 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountBigEnum: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 26.098 ns, StdErr = 0.096 ns (0.37%), N = 15, StdDev = 0.370 ns
Min = 25.253 ns, Q1 = 25.977 ns, Median = 26.034 ns, Q3 = 26.210 ns, Max = 26.809 ns
IQR = 0.233 ns, LowerFence = 25.628 ns, UpperFence = 26.560 ns
ConfidenceInterval = [25.702 ns; 26.494 ns] (CI 99.9%), Margin = 0.396 ns (1.52% of Mean)
Skewness = 0, Kurtosis = 3.28, MValue = 2
-------------------- Histogram --------------------
[25.056 ns ; 25.795 ns) | @
[25.795 ns ; 26.912 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckSmall: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 5.959 ns, StdErr = 0.009 ns (0.15%), N = 15, StdDev = 0.034 ns
Min = 5.906 ns, Q1 = 5.936 ns, Median = 5.956 ns, Q3 = 5.981 ns, Max = 6.017 ns
IQR = 0.045 ns, LowerFence = 5.868 ns, UpperFence = 6.049 ns
ConfidenceInterval = [5.924 ns; 5.995 ns] (CI 99.9%), Margin = 0.036 ns (0.60% of Mean)
Skewness = 0.08, Kurtosis = 1.76, MValue = 2
-------------------- Histogram --------------------
[5.888 ns ; 6.035 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckMedium: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 5.965 ns, StdErr = 0.008 ns (0.13%), N = 15, StdDev = 0.031 ns
Min = 5.924 ns, Q1 = 5.945 ns, Median = 5.952 ns, Q3 = 6.001 ns, Max = 6.012 ns
IQR = 0.056 ns, LowerFence = 5.861 ns, UpperFence = 6.086 ns
ConfidenceInterval = [5.932 ns; 5.998 ns] (CI 99.9%), Margin = 0.033 ns (0.55% of Mean)
Skewness = 0.36, Kurtosis = 1.4, MValue = 2
-------------------- Histogram --------------------
[5.907 ns ; 6.029 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBig: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 6.496 ns, StdErr = 0.014 ns (0.21%), N = 15, StdDev = 0.053 ns
Min = 6.436 ns, Q1 = 6.451 ns, Median = 6.489 ns, Q3 = 6.549 ns, Max = 6.595 ns
IQR = 0.099 ns, LowerFence = 6.303 ns, UpperFence = 6.697 ns
ConfidenceInterval = [6.439 ns; 6.554 ns] (CI 99.9%), Margin = 0.057 ns (0.88% of Mean)
Skewness = 0.4, Kurtosis = 1.52, MValue = 2
-------------------- Histogram --------------------
[6.408 ns ; 6.623 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBigList: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 4.062 ns, StdErr = 0.006 ns (0.16%), N = 15, StdDev = 0.025 ns
Min = 4.025 ns, Q1 = 4.047 ns, Median = 4.057 ns, Q3 = 4.074 ns, Max = 4.104 ns
IQR = 0.027 ns, LowerFence = 4.007 ns, UpperFence = 4.114 ns
ConfidenceInterval = [4.036 ns; 4.089 ns] (CI 99.9%), Margin = 0.027 ns (0.65% of Mean)
Skewness = 0.4, Kurtosis = 1.97, MValue = 2
-------------------- Histogram --------------------
[4.015 ns ; 4.117 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBigHashset: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 4.069 ns, StdErr = 0.006 ns (0.14%), N = 14, StdDev = 0.021 ns
Min = 4.041 ns, Q1 = 4.054 ns, Median = 4.066 ns, Q3 = 4.085 ns, Max = 4.117 ns
IQR = 0.031 ns, LowerFence = 4.007 ns, UpperFence = 4.132 ns
ConfidenceInterval = [4.046 ns; 4.093 ns] (CI 99.9%), Margin = 0.024 ns (0.58% of Mean)
Skewness = 0.6, Kurtosis = 2.38, MValue = 2
-------------------- Histogram --------------------
[4.029 ns ; 4.128 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBigEnumerable: Job-FQEJOX(Runtime=.NET 7.0, Toolchain=net70)
Runtime = .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 14.742 ns, StdErr = 0.022 ns (0.15%), N = 15, StdDev = 0.084 ns
Min = 14.518 ns, Q1 = 14.714 ns, Median = 14.758 ns, Q3 = 14.801 ns, Max = 14.849 ns
IQR = 0.087 ns, LowerFence = 14.583 ns, UpperFence = 14.933 ns
ConfidenceInterval = [14.653 ns; 14.832 ns] (CI 99.9%), Margin = 0.089 ns (0.61% of Mean)
Skewness = -1.02, Kurtosis = 3.94, MValue = 2
-------------------- Histogram --------------------
[14.473 ns ; 14.894 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnySmall: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 5.416 ns, StdErr = 0.032 ns (0.59%), N = 16, StdDev = 0.128 ns
Min = 5.279 ns, Q1 = 5.311 ns, Median = 5.354 ns, Q3 = 5.520 ns, Max = 5.665 ns
IQR = 0.209 ns, LowerFence = 4.997 ns, UpperFence = 5.834 ns
ConfidenceInterval = [5.286 ns; 5.546 ns] (CI 99.9%), Margin = 0.130 ns (2.40% of Mean)
Skewness = 0.55, Kurtosis = 1.65, MValue = 2
-------------------- Histogram --------------------
[5.253 ns ; 5.387 ns) | @@@@@@@@@
[5.387 ns ; 5.611 ns) | @@@@@@
[5.611 ns ; 5.731 ns) | @
---------------------------------------------------

AnyBenchmark.AnyMedium: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 5.254 ns, StdErr = 0.006 ns (0.11%), N = 14, StdDev = 0.021 ns
Min = 5.217 ns, Q1 = 5.242 ns, Median = 5.259 ns, Q3 = 5.265 ns, Max = 5.293 ns
IQR = 0.024 ns, LowerFence = 5.206 ns, UpperFence = 5.301 ns
ConfidenceInterval = [5.230 ns; 5.278 ns] (CI 99.9%), Margin = 0.024 ns (0.46% of Mean)
Skewness = -0.3, Kurtosis = 2.17, MValue = 2
-------------------- Histogram --------------------
[5.206 ns ; 5.304 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyBig: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 5.253 ns, StdErr = 0.006 ns (0.11%), N = 14, StdDev = 0.021 ns
Min = 5.223 ns, Q1 = 5.236 ns, Median = 5.255 ns, Q3 = 5.266 ns, Max = 5.292 ns
IQR = 0.030 ns, LowerFence = 5.192 ns, UpperFence = 5.310 ns
ConfidenceInterval = [5.230 ns; 5.277 ns] (CI 99.9%), Margin = 0.023 ns (0.45% of Mean)
Skewness = 0.01, Kurtosis = 1.9, MValue = 2
-------------------- Histogram --------------------
[5.212 ns ; 5.304 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyBigList: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 2.140 ns, StdErr = 0.006 ns (0.27%), N = 14, StdDev = 0.022 ns
Min = 2.116 ns, Q1 = 2.126 ns, Median = 2.138 ns, Q3 = 2.153 ns, Max = 2.195 ns
IQR = 0.027 ns, LowerFence = 2.085 ns, UpperFence = 2.194 ns
ConfidenceInterval = [2.116 ns; 2.165 ns] (CI 99.9%), Margin = 0.025 ns (1.16% of Mean)
Skewness = 0.94, Kurtosis = 3.2, MValue = 2
-------------------- Histogram --------------------
[2.104 ns ; 2.149 ns) | @@@@@@@@@@
[2.149 ns ; 2.207 ns) | @@@@
---------------------------------------------------

AnyBenchmark.AnyBigHashet: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 2.137 ns, StdErr = 0.005 ns (0.24%), N = 15, StdDev = 0.020 ns
Min = 2.113 ns, Q1 = 2.122 ns, Median = 2.132 ns, Q3 = 2.151 ns, Max = 2.180 ns
IQR = 0.029 ns, LowerFence = 2.079 ns, UpperFence = 2.194 ns
ConfidenceInterval = [2.115 ns; 2.158 ns] (CI 99.9%), Margin = 0.021 ns (1.00% of Mean)
Skewness = 0.6, Kurtosis = 2.11, MValue = 2
-------------------- Histogram --------------------
[2.103 ns ; 2.190 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyBigEnum: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 10.308 ns, StdErr = 0.024 ns (0.23%), N = 15, StdDev = 0.092 ns
Min = 10.028 ns, Q1 = 10.287 ns, Median = 10.320 ns, Q3 = 10.366 ns, Max = 10.401 ns
IQR = 0.079 ns, LowerFence = 10.168 ns, UpperFence = 10.485 ns
ConfidenceInterval = [10.209 ns; 10.406 ns] (CI 99.9%), Margin = 0.099 ns (0.96% of Mean)
Skewness = -1.69, Kurtosis = 5.82, MValue = 2
-------------------- Histogram --------------------
[9.979 ns ; 10.450 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.LenghtSmall: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 0.000 ns, StdErr = 0.000 ns (47.98%), N = 12, StdDev = 0.000 ns
Min = 0.000 ns, Q1 = 0.000 ns, Median = 0.000 ns, Q3 = 0.000 ns, Max = 0.001 ns
IQR = 0.000 ns, LowerFence = -0.000 ns, UpperFence = 0.000 ns
ConfidenceInterval = [-0.000 ns; 0.000 ns] (CI 99.9%), Margin = 0.000 ns (212.89% of Mean)
Skewness = 1.11, Kurtosis = 2.54, MValue = 2.25
-------------------- Histogram --------------------
[-0.000 ns ; 0.000 ns) | @@@@@@@@@
[ 0.000 ns ; 0.001 ns) | @@
[ 0.001 ns ; 0.001 ns) | @
---------------------------------------------------

AnyBenchmark.LenghtMedium: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 0.000 ns, StdErr = 0.000 ns (86.80%), N = 14, StdDev = 0.000 ns
Min = 0.000 ns, Q1 = 0.000 ns, Median = 0.000 ns, Q3 = 0.000 ns, Max = 0.001 ns
IQR = 0.000 ns, LowerFence = 0.000 ns, UpperFence = 0.000 ns
ConfidenceInterval = [-0.000 ns; 0.001 ns] (CI 99.9%), Margin = 0.000 ns (366.38% of Mean)
Skewness = 2.87, Kurtosis = 9.94, MValue = 2
-------------------- Histogram --------------------
[-0.000 ns ; 0.000 ns) | @@@@@@@@@@@@@
[ 0.000 ns ; 0.001 ns) |
[ 0.001 ns ; 0.001 ns) |
[ 0.001 ns ; 0.002 ns) | @
---------------------------------------------------

AnyBenchmark.LenghtBig: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 0.001 ns, StdErr = 0.000 ns (25.66%), N = 15, StdDev = 0.001 ns
Min = 0.000 ns, Q1 = 0.000 ns, Median = 0.001 ns, Q3 = 0.002 ns, Max = 0.003 ns
IQR = 0.002 ns, LowerFence = -0.002 ns, UpperFence = 0.004 ns
ConfidenceInterval = [-0.000 ns; 0.002 ns] (CI 99.9%), Margin = 0.001 ns (106.24% of Mean)
Skewness = 0.42, Kurtosis = 1.7, MValue = 2
-------------------- Histogram --------------------
[-0.000 ns ; 0.001 ns) | @@@@@@@@@
[ 0.001 ns ; 0.002 ns) | @@@@
[ 0.002 ns ; 0.003 ns) | @@
---------------------------------------------------

AnyBenchmark.LenghtBigList: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 0.000 ns, StdErr = 0.000 ns (46.56%), N = 15, StdDev = 0.000 ns
Min = 0.000 ns, Q1 = 0.000 ns, Median = 0.000 ns, Q3 = 0.000 ns, Max = 0.001 ns
IQR = 0.000 ns, LowerFence = -0.000 ns, UpperFence = 0.001 ns
ConfidenceInterval = [-0.000 ns; 0.001 ns] (CI 99.9%), Margin = 0.000 ns (192.76% of Mean)
Skewness = 1.48, Kurtosis = 3.87, MValue = 2.36
-------------------- Histogram --------------------
[-0.000 ns ; 0.000 ns) | @@@@@@@@@@@
[ 0.000 ns ; 0.001 ns) |
[ 0.001 ns ; 0.001 ns) | @@
[ 0.001 ns ; 0.001 ns) | @
[ 0.001 ns ; 0.002 ns) | @
---------------------------------------------------

AnyBenchmark.LenghtBigHashet: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 0.000 ns, StdErr = 0.000 ns (100.00%), N = 12, StdDev = 0.000 ns
Min = 0.000 ns, Q1 = 0.000 ns, Median = 0.000 ns, Q3 = 0.000 ns, Max = 0.000 ns
IQR = 0.000 ns, LowerFence = 0.000 ns, UpperFence = 0.000 ns
ConfidenceInterval = [-0.000 ns; 0.000 ns] (CI 99.9%), Margin = 0.000 ns (443.70% of Mean)
Skewness = 2.65, Kurtosis = 8.48, MValue = 2
-------------------- Histogram --------------------
[-0.000 ns ; 0.000 ns) | @@@@@@@@@@@
[ 0.000 ns ; 0.000 ns) |
[ 0.000 ns ; 0.000 ns) | @
---------------------------------------------------

AnyBenchmark.LenghtBigEnum: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 661.041 us, StdErr = 3.048 us (0.46%), N = 16, StdDev = 12.193 us
Min = 641.523 us, Q1 = 653.061 us, Median = 661.278 us, Q3 = 667.464 us, Max = 688.361 us
IQR = 14.403 us, LowerFence = 631.456 us, UpperFence = 689.068 us
ConfidenceInterval = [648.627 us; 673.455 us] (CI 99.9%), Margin = 12.414 us (1.88% of Mean)
Skewness = 0.43, Kurtosis = 2.54, MValue = 2
-------------------- Histogram --------------------
[638.124 us ; 664.195 us) | @@@@@@@@@@@
[664.195 us ; 678.795 us) | @@@@
[678.795 us ; 694.712 us) | @
---------------------------------------------------

AnyBenchmark.TakeCountSmall: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 16.587 ns, StdErr = 0.071 ns (0.43%), N = 13, StdDev = 0.256 ns
Min = 16.354 ns, Q1 = 16.381 ns, Median = 16.504 ns, Q3 = 16.688 ns, Max = 17.267 ns
IQR = 0.307 ns, LowerFence = 15.920 ns, UpperFence = 17.149 ns
ConfidenceInterval = [16.281 ns; 16.894 ns] (CI 99.9%), Margin = 0.307 ns (1.85% of Mean)
Skewness = 1.31, Kurtosis = 4.07, MValue = 2
-------------------- Histogram --------------------
[16.325 ns ; 16.920 ns) | @@@@@@@@@@@@
[16.920 ns ; 17.410 ns) | @
---------------------------------------------------

AnyBenchmark.TakeCountMedium: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 16.467 ns, StdErr = 0.028 ns (0.17%), N = 15, StdDev = 0.109 ns
Min = 16.273 ns, Q1 = 16.395 ns, Median = 16.483 ns, Q3 = 16.525 ns, Max = 16.703 ns
IQR = 0.130 ns, LowerFence = 16.199 ns, UpperFence = 16.720 ns
ConfidenceInterval = [16.351 ns; 16.584 ns] (CI 99.9%), Margin = 0.117 ns (0.71% of Mean)
Skewness = 0.26, Kurtosis = 2.47, MValue = 2
-------------------- Histogram --------------------
[16.215 ns ; 16.761 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountBig: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 17.432 ns, StdErr = 0.033 ns (0.19%), N = 15, StdDev = 0.128 ns
Min = 17.227 ns, Q1 = 17.349 ns, Median = 17.426 ns, Q3 = 17.535 ns, Max = 17.643 ns
IQR = 0.186 ns, LowerFence = 17.069 ns, UpperFence = 17.815 ns
ConfidenceInterval = [17.295 ns; 17.569 ns] (CI 99.9%), Margin = 0.137 ns (0.78% of Mean)
Skewness = -0.01, Kurtosis = 1.77, MValue = 2
-------------------- Histogram --------------------
[17.227 ns ; 17.712 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountBigList: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 12.623 ns, StdErr = 0.030 ns (0.24%), N = 13, StdDev = 0.108 ns
Min = 12.475 ns, Q1 = 12.560 ns, Median = 12.594 ns, Q3 = 12.655 ns, Max = 12.885 ns
IQR = 0.096 ns, LowerFence = 12.416 ns, UpperFence = 12.799 ns
ConfidenceInterval = [12.493 ns; 12.753 ns] (CI 99.9%), Margin = 0.130 ns (1.03% of Mean)
Skewness = 1, Kurtosis = 3.25, MValue = 2
-------------------- Histogram --------------------
[12.457 ns ; 12.946 ns) | @@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountBigHashet: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 22.107 ns, StdErr = 0.052 ns (0.24%), N = 15, StdDev = 0.203 ns
Min = 21.500 ns, Q1 = 22.052 ns, Median = 22.138 ns, Q3 = 22.219 ns, Max = 22.343 ns
IQR = 0.167 ns, LowerFence = 21.803 ns, UpperFence = 22.469 ns
ConfidenceInterval = [21.891 ns; 22.324 ns] (CI 99.9%), Margin = 0.217 ns (0.98% of Mean)
Skewness = -1.64, Kurtosis = 5.63, MValue = 2
-------------------- Histogram --------------------
[21.393 ns ; 22.451 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountBigEnum: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 19.493 ns, StdErr = 0.055 ns (0.28%), N = 13, StdDev = 0.198 ns
Min = 18.940 ns, Q1 = 19.468 ns, Median = 19.498 ns, Q3 = 19.533 ns, Max = 19.826 ns
IQR = 0.065 ns, LowerFence = 19.369 ns, UpperFence = 19.631 ns
ConfidenceInterval = [19.256 ns; 19.730 ns] (CI 99.9%), Margin = 0.237 ns (1.21% of Mean)
Skewness = -1.24, Kurtosis = 5.41, MValue = 2
-------------------- Histogram --------------------
[18.830 ns ; 19.293 ns) | @
[19.293 ns ; 19.936 ns) | @@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckSmall: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 5.201 ns, StdErr = 0.007 ns (0.14%), N = 15, StdDev = 0.027 ns
Min = 5.131 ns, Q1 = 5.188 ns, Median = 5.199 ns, Q3 = 5.223 ns, Max = 5.233 ns
IQR = 0.035 ns, LowerFence = 5.135 ns, UpperFence = 5.277 ns
ConfidenceInterval = [5.172 ns; 5.231 ns] (CI 99.9%), Margin = 0.029 ns (0.56% of Mean)
Skewness = -0.87, Kurtosis = 3.26, MValue = 2
-------------------- Histogram --------------------
[5.117 ns ; 5.248 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckMedium: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 5.208 ns, StdErr = 0.013 ns (0.25%), N = 15, StdDev = 0.051 ns
Min = 5.118 ns, Q1 = 5.168 ns, Median = 5.212 ns, Q3 = 5.244 ns, Max = 5.276 ns
IQR = 0.076 ns, LowerFence = 5.054 ns, UpperFence = 5.357 ns
ConfidenceInterval = [5.154 ns; 5.262 ns] (CI 99.9%), Margin = 0.054 ns (1.04% of Mean)
Skewness = -0.08, Kurtosis = 1.55, MValue = 2
-------------------- Histogram --------------------
[5.114 ns ; 5.304 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBig: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 5.455 ns, StdErr = 0.035 ns (0.64%), N = 24, StdDev = 0.170 ns
Min = 5.231 ns, Q1 = 5.327 ns, Median = 5.411 ns, Q3 = 5.566 ns, Max = 5.813 ns
IQR = 0.239 ns, LowerFence = 4.969 ns, UpperFence = 5.924 ns
ConfidenceInterval = [5.324 ns; 5.586 ns] (CI 99.9%), Margin = 0.131 ns (2.40% of Mean)
Skewness = 0.59, Kurtosis = 2.17, MValue = 2
-------------------- Histogram --------------------
[5.225 ns ; 5.482 ns) | @@@@@@@@@@@@@
[5.482 ns ; 5.637 ns) | @@@@@@@
[5.637 ns ; 5.819 ns) | @@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBigList: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 1.633 ns, StdErr = 0.006 ns (0.35%), N = 13, StdDev = 0.020 ns
Min = 1.597 ns, Q1 = 1.623 ns, Median = 1.631 ns, Q3 = 1.646 ns, Max = 1.664 ns
IQR = 0.024 ns, LowerFence = 1.587 ns, UpperFence = 1.682 ns
ConfidenceInterval = [1.609 ns; 1.657 ns] (CI 99.9%), Margin = 0.024 ns (1.49% of Mean)
Skewness = -0.03, Kurtosis = 1.74, MValue = 2
-------------------- Histogram --------------------
[1.586 ns ; 1.670 ns) | @@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBigHashset: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 1.900 ns, StdErr = 0.005 ns (0.25%), N = 15, StdDev = 0.018 ns
Min = 1.864 ns, Q1 = 1.888 ns, Median = 1.905 ns, Q3 = 1.907 ns, Max = 1.931 ns
IQR = 0.019 ns, LowerFence = 1.860 ns, UpperFence = 1.935 ns
ConfidenceInterval = [1.881 ns; 1.920 ns] (CI 99.9%), Margin = 0.020 ns (1.03% of Mean)
Skewness = -0.13, Kurtosis = 2.23, MValue = 2
-------------------- Histogram --------------------
[1.854 ns ; 1.941 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBigEnumerable: Job-DISSLE(Runtime=.NET 8.0, Toolchain=net80)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 9.411 ns, StdErr = 0.027 ns (0.28%), N = 15, StdDev = 0.104 ns
Min = 9.150 ns, Q1 = 9.363 ns, Median = 9.408 ns, Q3 = 9.469 ns, Max = 9.580 ns
IQR = 0.106 ns, LowerFence = 9.203 ns, UpperFence = 9.628 ns
ConfidenceInterval = [9.301 ns; 9.522 ns] (CI 99.9%), Margin = 0.111 ns (1.18% of Mean)
Skewness = -0.63, Kurtosis = 3.53, MValue = 2
-------------------- Histogram --------------------
[9.095 ns ; 9.279 ns) | @
[9.279 ns ; 9.635 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnySmall: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 8.917 ns, StdErr = 0.014 ns (0.15%), N = 14, StdDev = 0.052 ns
Min = 8.766 ns, Q1 = 8.901 ns, Median = 8.928 ns, Q3 = 8.950 ns, Max = 8.975 ns
IQR = 0.049 ns, LowerFence = 8.827 ns, UpperFence = 9.024 ns
ConfidenceInterval = [8.859 ns; 8.976 ns] (CI 99.9%), Margin = 0.058 ns (0.65% of Mean)
Skewness = -1.62, Kurtosis = 5.48, MValue = 2
-------------------- Histogram --------------------
[8.737 ns ; 9.003 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyMedium: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 8.902 ns, StdErr = 0.009 ns (0.10%), N = 15, StdDev = 0.034 ns
Min = 8.847 ns, Q1 = 8.884 ns, Median = 8.900 ns, Q3 = 8.924 ns, Max = 8.971 ns
IQR = 0.040 ns, LowerFence = 8.823 ns, UpperFence = 8.985 ns
ConfidenceInterval = [8.866 ns; 8.938 ns] (CI 99.9%), Margin = 0.036 ns (0.40% of Mean)
Skewness = 0.06, Kurtosis = 2.19, MValue = 2
-------------------- Histogram --------------------
[8.830 ns ; 8.989 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyBig: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 8.706 ns, StdErr = 0.011 ns (0.13%), N = 15, StdDev = 0.043 ns
Min = 8.640 ns, Q1 = 8.680 ns, Median = 8.696 ns, Q3 = 8.735 ns, Max = 8.791 ns
IQR = 0.055 ns, LowerFence = 8.598 ns, UpperFence = 8.817 ns
ConfidenceInterval = [8.661 ns; 8.752 ns] (CI 99.9%), Margin = 0.046 ns (0.52% of Mean)
Skewness = 0.29, Kurtosis = 1.99, MValue = 2
-------------------- Histogram --------------------
[8.617 ns ; 8.814 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyBigList: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 10.897 ns, StdErr = 0.024 ns (0.22%), N = 15, StdDev = 0.093 ns
Min = 10.658 ns, Q1 = 10.881 ns, Median = 10.916 ns, Q3 = 10.950 ns, Max = 11.004 ns
IQR = 0.069 ns, LowerFence = 10.777 ns, UpperFence = 11.054 ns
ConfidenceInterval = [10.798 ns; 10.996 ns] (CI 99.9%), Margin = 0.099 ns (0.91% of Mean)
Skewness = -1.16, Kurtosis = 3.67, MValue = 2
-------------------- Histogram --------------------
[10.609 ns ; 11.053 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyBigHashet: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 11.554 ns, StdErr = 0.022 ns (0.19%), N = 15, StdDev = 0.085 ns
Min = 11.387 ns, Q1 = 11.520 ns, Median = 11.541 ns, Q3 = 11.607 ns, Max = 11.704 ns
IQR = 0.087 ns, LowerFence = 11.390 ns, UpperFence = 11.737 ns
ConfidenceInterval = [11.462 ns; 11.645 ns] (CI 99.9%), Margin = 0.091 ns (0.79% of Mean)
Skewness = -0.04, Kurtosis = 2.27, MValue = 2
-------------------- Histogram --------------------
[11.341 ns ; 11.750 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyBigEnum: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 9.628 ns, StdErr = 0.007 ns (0.07%), N = 14, StdDev = 0.026 ns
Min = 9.563 ns, Q1 = 9.616 ns, Median = 9.633 ns, Q3 = 9.648 ns, Max = 9.663 ns
IQR = 0.032 ns, LowerFence = 9.569 ns, UpperFence = 9.696 ns
ConfidenceInterval = [9.599 ns; 9.658 ns] (CI 99.9%), Margin = 0.029 ns (0.31% of Mean)
Skewness = -0.93, Kurtosis = 3.3, MValue = 2
-------------------- Histogram --------------------
[9.549 ns ; 9.677 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.LenghtSmall: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 0.087 ns, StdErr = 0.003 ns (3.07%), N = 15, StdDev = 0.010 ns
Min = 0.067 ns, Q1 = 0.084 ns, Median = 0.088 ns, Q3 = 0.093 ns, Max = 0.105 ns
IQR = 0.009 ns, LowerFence = 0.070 ns, UpperFence = 0.106 ns
ConfidenceInterval = [0.076 ns; 0.098 ns] (CI 99.9%), Margin = 0.011 ns (12.70% of Mean)
Skewness = -0.27, Kurtosis = 2.23, MValue = 2.5
-------------------- Histogram --------------------
[0.065 ns ; 0.075 ns) | @@@
[0.075 ns ; 0.081 ns) |
[0.081 ns ; 0.092 ns) | @@@@@@@@
[0.092 ns ; 0.105 ns) | @@@@
---------------------------------------------------

AnyBenchmark.LenghtMedium: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 0.080 ns, StdErr = 0.003 ns (3.91%), N = 15, StdDev = 0.012 ns
Min = 0.056 ns, Q1 = 0.071 ns, Median = 0.081 ns, Q3 = 0.088 ns, Max = 0.101 ns
IQR = 0.017 ns, LowerFence = 0.045 ns, UpperFence = 0.115 ns
ConfidenceInterval = [0.067 ns; 0.093 ns] (CI 99.9%), Margin = 0.013 ns (16.20% of Mean)
Skewness = -0.31, Kurtosis = 2.07, MValue = 3
-------------------- Histogram --------------------
[0.050 ns ; 0.060 ns) | @
[0.060 ns ; 0.073 ns) | @@@@
[0.073 ns ; 0.078 ns) |
[0.078 ns ; 0.091 ns) | @@@@@@@@
[0.091 ns ; 0.104 ns) | @@
---------------------------------------------------

AnyBenchmark.LenghtBig: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 0.089 ns, StdErr = 0.003 ns (3.64%), N = 15, StdDev = 0.012 ns
Min = 0.068 ns, Q1 = 0.081 ns, Median = 0.091 ns, Q3 = 0.095 ns, Max = 0.112 ns
IQR = 0.013 ns, LowerFence = 0.061 ns, UpperFence = 0.115 ns
ConfidenceInterval = [0.075 ns; 0.102 ns] (CI 99.9%), Margin = 0.013 ns (15.06% of Mean)
Skewness = 0.09, Kurtosis = 2, MValue = 2
-------------------- Histogram --------------------
[0.067 ns ; 0.084 ns) | @@@@
[0.084 ns ; 0.097 ns) | @@@@@@@@
[0.097 ns ; 0.114 ns) | @@@
---------------------------------------------------

AnyBenchmark.LenghtBigList: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 0.091 ns, StdErr = 0.002 ns (2.54%), N = 15, StdDev = 0.009 ns
Min = 0.074 ns, Q1 = 0.085 ns, Median = 0.092 ns, Q3 = 0.098 ns, Max = 0.109 ns
IQR = 0.013 ns, LowerFence = 0.065 ns, UpperFence = 0.117 ns
ConfidenceInterval = [0.082 ns; 0.101 ns] (CI 99.9%), Margin = 0.010 ns (10.51% of Mean)
Skewness = 0.03, Kurtosis = 2.28, MValue = 2.25
-------------------- Histogram --------------------
[0.069 ns ; 0.079 ns) | @
[0.079 ns ; 0.088 ns) | @@@@@
[0.088 ns ; 0.100 ns) | @@@@@@@@
[0.100 ns ; 0.105 ns) |
[0.105 ns ; 0.114 ns) | @
---------------------------------------------------

AnyBenchmark.LenghtBigHashet: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 0.085 ns, StdErr = 0.003 ns (3.88%), N = 15, StdDev = 0.013 ns
Min = 0.058 ns, Q1 = 0.079 ns, Median = 0.088 ns, Q3 = 0.092 ns, Max = 0.107 ns
IQR = 0.013 ns, LowerFence = 0.059 ns, UpperFence = 0.112 ns
ConfidenceInterval = [0.072 ns; 0.099 ns] (CI 99.9%), Margin = 0.014 ns (16.06% of Mean)
Skewness = -0.43, Kurtosis = 2.45, MValue = 2.75
-------------------- Histogram --------------------
[0.052 ns ; 0.064 ns) | @
[0.064 ns ; 0.080 ns) | @@@
[0.080 ns ; 0.094 ns) | @@@@@@@@
[0.094 ns ; 0.108 ns) | @@@
---------------------------------------------------

AnyBenchmark.LenghtBigEnum: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 1.953 ms, StdErr = 0.005 ms (0.26%), N = 15, StdDev = 0.019 ms
Min = 1.929 ms, Q1 = 1.935 ms, Median = 1.957 ms, Q3 = 1.966 ms, Max = 1.986 ms
IQR = 0.031 ms, LowerFence = 1.888 ms, UpperFence = 2.013 ms
ConfidenceInterval = [1.932 ms; 1.974 ms] (CI 99.9%), Margin = 0.021 ms (1.07% of Mean)
Skewness = 0.19, Kurtosis = 1.61, MValue = 2
-------------------- Histogram --------------------
[1.919 ms ; 1.996 ms) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountSmall: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 37.599 ns, StdErr = 0.048 ns (0.13%), N = 15, StdDev = 0.188 ns
Min = 37.364 ns, Q1 = 37.455 ns, Median = 37.597 ns, Q3 = 37.703 ns, Max = 38.013 ns
IQR = 0.248 ns, LowerFence = 37.083 ns, UpperFence = 38.076 ns
ConfidenceInterval = [37.398 ns; 37.800 ns] (CI 99.9%), Margin = 0.201 ns (0.53% of Mean)
Skewness = 0.64, Kurtosis = 2.32, MValue = 2
-------------------- Histogram --------------------
[37.264 ns ; 38.113 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountMedium: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 37.473 ns, StdErr = 0.037 ns (0.10%), N = 14, StdDev = 0.139 ns
Min = 37.147 ns, Q1 = 37.395 ns, Median = 37.485 ns, Q3 = 37.565 ns, Max = 37.710 ns
IQR = 0.169 ns, LowerFence = 37.142 ns, UpperFence = 37.818 ns
ConfidenceInterval = [37.317 ns; 37.630 ns] (CI 99.9%), Margin = 0.156 ns (0.42% of Mean)
Skewness = -0.59, Kurtosis = 3.05, MValue = 2
-------------------- Histogram --------------------
[37.071 ns ; 37.785 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountBig: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 36.960 ns, StdErr = 0.023 ns (0.06%), N = 14, StdDev = 0.088 ns
Min = 36.809 ns, Q1 = 36.888 ns, Median = 36.966 ns, Q3 = 37.028 ns, Max = 37.103 ns
IQR = 0.140 ns, LowerFence = 36.678 ns, UpperFence = 37.238 ns
ConfidenceInterval = [36.861 ns; 37.059 ns] (CI 99.9%), Margin = 0.099 ns (0.27% of Mean)
Skewness = -0.2, Kurtosis = 1.72, MValue = 2
-------------------- Histogram --------------------
[36.761 ns ; 37.151 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountBigList: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 40.620 ns, StdErr = 0.029 ns (0.07%), N = 14, StdDev = 0.108 ns
Min = 40.436 ns, Q1 = 40.552 ns, Median = 40.635 ns, Q3 = 40.690 ns, Max = 40.804 ns
IQR = 0.138 ns, LowerFence = 40.346 ns, UpperFence = 40.896 ns
ConfidenceInterval = [40.498 ns; 40.742 ns] (CI 99.9%), Margin = 0.122 ns (0.30% of Mean)
Skewness = -0.18, Kurtosis = 1.9, MValue = 2
-------------------- Histogram --------------------
[40.377 ns ; 40.863 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountBigHashet: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 40.813 ns, StdErr = 0.056 ns (0.14%), N = 15, StdDev = 0.216 ns
Min = 40.382 ns, Q1 = 40.651 ns, Median = 40.875 ns, Q3 = 40.977 ns, Max = 41.086 ns
IQR = 0.326 ns, LowerFence = 40.162 ns, UpperFence = 41.467 ns
ConfidenceInterval = [40.582 ns; 41.043 ns] (CI 99.9%), Margin = 0.231 ns (0.57% of Mean)
Skewness = -0.49, Kurtosis = 1.85, MValue = 2
-------------------- Histogram --------------------
[40.267 ns ; 41.201 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountBigEnum: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 38.122 ns, StdErr = 0.051 ns (0.13%), N = 14, StdDev = 0.192 ns
Min = 37.811 ns, Q1 = 37.994 ns, Median = 38.121 ns, Q3 = 38.230 ns, Max = 38.425 ns
IQR = 0.236 ns, LowerFence = 37.641 ns, UpperFence = 38.584 ns
ConfidenceInterval = [37.906 ns; 38.338 ns] (CI 99.9%), Margin = 0.216 ns (0.57% of Mean)
Skewness = 0.08, Kurtosis = 1.92, MValue = 2
-------------------- Histogram --------------------
[37.706 ns ; 38.452 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckSmall: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 8.767 ns, StdErr = 0.023 ns (0.26%), N = 12, StdDev = 0.078 ns
Min = 8.674 ns, Q1 = 8.728 ns, Median = 8.759 ns, Q3 = 8.764 ns, Max = 8.987 ns
IQR = 0.036 ns, LowerFence = 8.673 ns, UpperFence = 8.818 ns
ConfidenceInterval = [8.667 ns; 8.867 ns] (CI 99.9%), Margin = 0.100 ns (1.14% of Mean)
Skewness = 1.68, Kurtosis = 5.4, MValue = 2
-------------------- Histogram --------------------
[8.652 ns ; 9.032 ns) | @@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckMedium: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 8.758 ns, StdErr = 0.007 ns (0.08%), N = 14, StdDev = 0.027 ns
Min = 8.704 ns, Q1 = 8.741 ns, Median = 8.767 ns, Q3 = 8.774 ns, Max = 8.802 ns
IQR = 0.033 ns, LowerFence = 8.691 ns, UpperFence = 8.823 ns
ConfidenceInterval = [8.727 ns; 8.789 ns] (CI 99.9%), Margin = 0.031 ns (0.35% of Mean)
Skewness = -0.46, Kurtosis = 2.05, MValue = 2
-------------------- Histogram --------------------
[8.689 ns ; 8.817 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBig: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 8.515 ns, StdErr = 0.009 ns (0.11%), N = 15, StdDev = 0.036 ns
Min = 8.463 ns, Q1 = 8.496 ns, Median = 8.512 ns, Q3 = 8.537 ns, Max = 8.584 ns
IQR = 0.040 ns, LowerFence = 8.436 ns, UpperFence = 8.597 ns
ConfidenceInterval = [8.476 ns; 8.553 ns] (CI 99.9%), Margin = 0.038 ns (0.45% of Mean)
Skewness = 0.35, Kurtosis = 2.14, MValue = 2
-------------------- Histogram --------------------
[8.444 ns ; 8.604 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBigList: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 10.802 ns, StdErr = 0.016 ns (0.15%), N = 13, StdDev = 0.057 ns
Min = 10.720 ns, Q1 = 10.763 ns, Median = 10.792 ns, Q3 = 10.838 ns, Max = 10.918 ns
IQR = 0.075 ns, LowerFence = 10.651 ns, UpperFence = 10.951 ns
ConfidenceInterval = [10.733 ns; 10.870 ns] (CI 99.9%), Margin = 0.068 ns (0.63% of Mean)
Skewness = 0.47, Kurtosis = 2.12, MValue = 2
-------------------- Histogram --------------------
[10.688 ns ; 10.950 ns) | @@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBigHashset: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 10.877 ns, StdErr = 0.049 ns (0.45%), N = 30, StdDev = 0.271 ns
Min = 10.618 ns, Q1 = 10.745 ns, Median = 10.779 ns, Q3 = 10.865 ns, Max = 11.792 ns
IQR = 0.120 ns, LowerFence = 10.564 ns, UpperFence = 11.046 ns
ConfidenceInterval = [10.696 ns; 11.057 ns] (CI 99.9%), Margin = 0.181 ns (1.66% of Mean)
Skewness = 2.28, Kurtosis = 7.57, MValue = 2
-------------------- Histogram --------------------
[10.503 ns ; 10.890 ns) | @@@@@@@@@@@@@@@@@@@@@@@
[10.890 ns ; 11.370 ns) | @@@@@
[11.370 ns ; 11.633 ns) |
[11.633 ns ; 11.907 ns) | @@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBigEnumerable: Job-HXWWDC(Runtime=.NET Framework 4.8, Toolchain=net48)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 8.998 ns, StdErr = 0.011 ns (0.13%), N = 15, StdDev = 0.044 ns
Min = 8.930 ns, Q1 = 8.965 ns, Median = 8.999 ns, Q3 = 9.028 ns, Max = 9.092 ns
IQR = 0.063 ns, LowerFence = 8.872 ns, UpperFence = 9.122 ns
ConfidenceInterval = [8.951 ns; 9.046 ns] (CI 99.9%), Margin = 0.047 ns (0.52% of Mean)
Skewness = 0.4, Kurtosis = 2.18, MValue = 2
-------------------- Histogram --------------------
[8.907 ns ; 9.115 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnySmall: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 5.240 ns, StdErr = 0.007 ns (0.14%), N = 14, StdDev = 0.028 ns
Min = 5.193 ns, Q1 = 5.216 ns, Median = 5.244 ns, Q3 = 5.264 ns, Max = 5.278 ns
IQR = 0.047 ns, LowerFence = 5.145 ns, UpperFence = 5.334 ns
ConfidenceInterval = [5.208 ns; 5.271 ns] (CI 99.9%), Margin = 0.032 ns (0.60% of Mean)
Skewness = -0.23, Kurtosis = 1.52, MValue = 2
-------------------- Histogram --------------------
[5.178 ns ; 5.293 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyMedium: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 5.940 ns, StdErr = 0.009 ns (0.15%), N = 15, StdDev = 0.035 ns
Min = 5.886 ns, Q1 = 5.922 ns, Median = 5.940 ns, Q3 = 5.969 ns, Max = 5.991 ns
IQR = 0.048 ns, LowerFence = 5.850 ns, UpperFence = 6.041 ns
ConfidenceInterval = [5.903 ns; 5.978 ns] (CI 99.9%), Margin = 0.038 ns (0.63% of Mean)
Skewness = -0.06, Kurtosis = 1.6, MValue = 2
-------------------- Histogram --------------------
[5.879 ns ; 5.998 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyBig: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 5.883 ns, StdErr = 0.007 ns (0.12%), N = 14, StdDev = 0.027 ns
Min = 5.825 ns, Q1 = 5.879 ns, Median = 5.884 ns, Q3 = 5.896 ns, Max = 5.930 ns
IQR = 0.017 ns, LowerFence = 5.854 ns, UpperFence = 5.922 ns
ConfidenceInterval = [5.853 ns; 5.913 ns] (CI 99.9%), Margin = 0.030 ns (0.51% of Mean)
Skewness = -0.38, Kurtosis = 2.71, MValue = 2
-------------------- Histogram --------------------
[5.810 ns ; 5.944 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyBigList: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 3.452 ns, StdErr = 0.005 ns (0.16%), N = 13, StdDev = 0.019 ns
Min = 3.427 ns, Q1 = 3.439 ns, Median = 3.446 ns, Q3 = 3.459 ns, Max = 3.501 ns
IQR = 0.020 ns, LowerFence = 3.408 ns, UpperFence = 3.490 ns
ConfidenceInterval = [3.428 ns; 3.475 ns] (CI 99.9%), Margin = 0.023 ns (0.67% of Mean)
Skewness = 1.01, Kurtosis = 3.44, MValue = 2
-------------------- Histogram --------------------
[3.417 ns ; 3.512 ns) | @@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyBigHashet: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 3.446 ns, StdErr = 0.004 ns (0.12%), N = 15, StdDev = 0.016 ns
Min = 3.421 ns, Q1 = 3.433 ns, Median = 3.448 ns, Q3 = 3.455 ns, Max = 3.485 ns
IQR = 0.022 ns, LowerFence = 3.399 ns, UpperFence = 3.488 ns
ConfidenceInterval = [3.429 ns; 3.464 ns] (CI 99.9%), Margin = 0.017 ns (0.50% of Mean)
Skewness = 0.57, Kurtosis = 2.73, MValue = 2
-------------------- Histogram --------------------
[3.415 ns ; 3.493 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyBigEnum: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 12.968 ns, StdErr = 0.016 ns (0.12%), N = 13, StdDev = 0.058 ns
Min = 12.875 ns, Q1 = 12.923 ns, Median = 12.971 ns, Q3 = 12.996 ns, Max = 13.076 ns
IQR = 0.074 ns, LowerFence = 12.812 ns, UpperFence = 13.107 ns
ConfidenceInterval = [12.898 ns; 13.038 ns] (CI 99.9%), Margin = 0.070 ns (0.54% of Mean)
Skewness = 0.11, Kurtosis = 1.96, MValue = 2
-------------------- Histogram --------------------
[12.855 ns ; 13.109 ns) | @@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.LenghtSmall: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 0.003 ns, StdErr = 0.001 ns (37.42%), N = 15, StdDev = 0.004 ns
Min = 0.000 ns, Q1 = 0.000 ns, Median = 0.001 ns, Q3 = 0.004 ns, Max = 0.014 ns
IQR = 0.004 ns, LowerFence = -0.006 ns, UpperFence = 0.011 ns
ConfidenceInterval = [-0.002 ns; 0.007 ns] (CI 99.9%), Margin = 0.004 ns (154.92% of Mean)
Skewness = 1.43, Kurtosis = 4.17, MValue = 2.4
-------------------- Histogram --------------------
[-0.001 ns ; 0.004 ns) | @@@@@@@@@@@
[ 0.004 ns ; 0.009 ns) | @@@
[ 0.009 ns ; 0.011 ns) |
[ 0.011 ns ; 0.016 ns) | @
---------------------------------------------------

AnyBenchmark.LenghtMedium: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 0.004 ns, StdErr = 0.001 ns (25.29%), N = 14, StdDev = 0.003 ns
Min = 0.000 ns, Q1 = 0.001 ns, Median = 0.002 ns, Q3 = 0.006 ns, Max = 0.010 ns
IQR = 0.004 ns, LowerFence = -0.005 ns, UpperFence = 0.012 ns
ConfidenceInterval = [-0.000 ns; 0.007 ns] (CI 99.9%), Margin = 0.004 ns (106.73% of Mean)
Skewness = 0.69, Kurtosis = 2.06, MValue = 2.44
-------------------- Histogram --------------------
[-0.000 ns ; 0.003 ns) | @@@@@@@@@
[ 0.003 ns ; 0.005 ns) | @
[ 0.005 ns ; 0.009 ns) | @@@
[ 0.009 ns ; 0.012 ns) | @
---------------------------------------------------

AnyBenchmark.LenghtBig: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 0.005 ns, StdErr = 0.001 ns (24.10%), N = 15, StdDev = 0.005 ns
Min = 0.000 ns, Q1 = 0.001 ns, Median = 0.004 ns, Q3 = 0.009 ns, Max = 0.016 ns
IQR = 0.008 ns, LowerFence = -0.011 ns, UpperFence = 0.021 ns
ConfidenceInterval = [0.000 ns; 0.011 ns] (CI 99.9%), Margin = 0.005 ns (99.77% of Mean)
Skewness = 0.56, Kurtosis = 2.03, MValue = 2
-------------------- Histogram --------------------
[-0.001 ns ; 0.005 ns) | @@@@@@@@@
[ 0.005 ns ; 0.012 ns) | @@@@
[ 0.012 ns ; 0.017 ns) | @@
---------------------------------------------------

AnyBenchmark.LenghtBigList: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 0.003 ns, StdErr = 0.001 ns (37.15%), N = 14, StdDev = 0.004 ns
Min = 0.000 ns, Q1 = 0.000 ns, Median = 0.001 ns, Q3 = 0.004 ns, Max = 0.012 ns
IQR = 0.004 ns, LowerFence = -0.006 ns, UpperFence = 0.009 ns
ConfidenceInterval = [-0.001 ns; 0.007 ns] (CI 99.9%), Margin = 0.004 ns (156.81% of Mean)
Skewness = 1.26, Kurtosis = 3.45, MValue = 2.22
-------------------- Histogram --------------------
[-0.000 ns ; 0.004 ns) | @@@@@@@@@@@
[ 0.004 ns ; 0.008 ns) | @
[ 0.008 ns ; 0.012 ns) | @@
---------------------------------------------------

AnyBenchmark.LenghtBigHashet: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 0.002 ns, StdErr = 0.001 ns (39.08%), N = 15, StdDev = 0.003 ns
Min = 0.000 ns, Q1 = 0.000 ns, Median = 0.000 ns, Q3 = 0.003 ns, Max = 0.009 ns
IQR = 0.003 ns, LowerFence = -0.005 ns, UpperFence = 0.008 ns
ConfidenceInterval = [-0.001 ns; 0.005 ns] (CI 99.9%), Margin = 0.003 ns (161.83% of Mean)
Skewness = 1.31, Kurtosis = 3.64, MValue = 2.4
-------------------- Histogram --------------------
[-0.000 ns ; 0.003 ns) | @@@@@@@@@@@
[ 0.003 ns ; 0.006 ns) | @@@
[ 0.006 ns ; 0.008 ns) |
[ 0.008 ns ; 0.011 ns) | @
---------------------------------------------------

AnyBenchmark.LenghtBigEnum: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 1.877 ms, StdErr = 0.003 ms (0.15%), N = 14, StdDev = 0.011 ms
Min = 1.856 ms, Q1 = 1.871 ms, Median = 1.877 ms, Q3 = 1.883 ms, Max = 1.896 ms
IQR = 0.012 ms, LowerFence = 1.852 ms, UpperFence = 1.901 ms
ConfidenceInterval = [1.865 ms; 1.889 ms] (CI 99.9%), Margin = 0.012 ms (0.64% of Mean)
Skewness = -0.18, Kurtosis = 2.28, MValue = 2
-------------------- Histogram --------------------
[1.850 ms ; 1.899 ms) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountSmall: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 24.100 ns, StdErr = 0.129 ns (0.54%), N = 22, StdDev = 0.606 ns
Min = 23.502 ns, Q1 = 23.665 ns, Median = 23.848 ns, Q3 = 24.527 ns, Max = 25.360 ns
IQR = 0.862 ns, LowerFence = 22.372 ns, UpperFence = 25.820 ns
ConfidenceInterval = [23.607 ns; 24.594 ns] (CI 99.9%), Margin = 0.493 ns (2.05% of Mean)
Skewness = 0.86, Kurtosis = 2.14, MValue = 2.13
-------------------- Histogram --------------------
[23.474 ns ; 24.246 ns) | @@@@@@@@@@@@@@@
[24.246 ns ; 24.914 ns) | @@@
[24.914 ns ; 25.481 ns) | @@@@
---------------------------------------------------

AnyBenchmark.TakeCountMedium: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 23.075 ns, StdErr = 0.029 ns (0.13%), N = 13, StdDev = 0.104 ns
Min = 22.900 ns, Q1 = 23.009 ns, Median = 23.091 ns, Q3 = 23.145 ns, Max = 23.226 ns
IQR = 0.137 ns, LowerFence = 22.804 ns, UpperFence = 23.350 ns
ConfidenceInterval = [22.950 ns; 23.200 ns] (CI 99.9%), Margin = 0.125 ns (0.54% of Mean)
Skewness = -0.12, Kurtosis = 1.61, MValue = 2
-------------------- Histogram --------------------
[22.842 ns ; 23.285 ns) | @@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountBig: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 22.930 ns, StdErr = 0.036 ns (0.16%), N = 15, StdDev = 0.138 ns
Min = 22.699 ns, Q1 = 22.828 ns, Median = 22.929 ns, Q3 = 23.022 ns, Max = 23.181 ns
IQR = 0.194 ns, LowerFence = 22.538 ns, UpperFence = 23.313 ns
ConfidenceInterval = [22.782 ns; 23.078 ns] (CI 99.9%), Margin = 0.148 ns (0.64% of Mean)
Skewness = 0.07, Kurtosis = 1.72, MValue = 2
-------------------- Histogram --------------------
[22.626 ns ; 23.254 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountBigList: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 18.040 ns, StdErr = 0.019 ns (0.11%), N = 14, StdDev = 0.071 ns
Min = 17.968 ns, Q1 = 17.985 ns, Median = 18.016 ns, Q3 = 18.069 ns, Max = 18.193 ns
IQR = 0.083 ns, LowerFence = 17.861 ns, UpperFence = 18.193 ns
ConfidenceInterval = [17.960 ns; 18.121 ns] (CI 99.9%), Margin = 0.081 ns (0.45% of Mean)
Skewness = 0.88, Kurtosis = 2.37, MValue = 2
-------------------- Histogram --------------------
[17.929 ns ; 18.232 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountBigHashet: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 27.360 ns, StdErr = 0.022 ns (0.08%), N = 13, StdDev = 0.078 ns
Min = 27.211 ns, Q1 = 27.351 ns, Median = 27.378 ns, Q3 = 27.389 ns, Max = 27.461 ns
IQR = 0.038 ns, LowerFence = 27.294 ns, UpperFence = 27.446 ns
ConfidenceInterval = [27.267 ns; 27.453 ns] (CI 99.9%), Margin = 0.093 ns (0.34% of Mean)
Skewness = -0.78, Kurtosis = 2.45, MValue = 2
-------------------- Histogram --------------------
[27.167 ns ; 27.504 ns) | @@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountBigEnum: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 23.211 ns, StdErr = 0.028 ns (0.12%), N = 15, StdDev = 0.109 ns
Min = 23.076 ns, Q1 = 23.128 ns, Median = 23.205 ns, Q3 = 23.251 ns, Max = 23.403 ns
IQR = 0.124 ns, LowerFence = 22.942 ns, UpperFence = 23.437 ns
ConfidenceInterval = [23.094 ns; 23.328 ns] (CI 99.9%), Margin = 0.117 ns (0.50% of Mean)
Skewness = 0.51, Kurtosis = 1.85, MValue = 2
-------------------- Histogram --------------------
[23.018 ns ; 23.461 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckSmall: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 6.956 ns, StdErr = 0.011 ns (0.15%), N = 14, StdDev = 0.039 ns
Min = 6.863 ns, Q1 = 6.933 ns, Median = 6.950 ns, Q3 = 6.979 ns, Max = 7.016 ns
IQR = 0.045 ns, LowerFence = 6.865 ns, UpperFence = 7.047 ns
ConfidenceInterval = [6.912 ns; 7.001 ns] (CI 99.9%), Margin = 0.044 ns (0.64% of Mean)
Skewness = -0.48, Kurtosis = 2.97, MValue = 2
-------------------- Histogram --------------------
[6.841 ns ; 7.022 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckMedium: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 6.992 ns, StdErr = 0.014 ns (0.20%), N = 15, StdDev = 0.053 ns
Min = 6.933 ns, Q1 = 6.954 ns, Median = 6.975 ns, Q3 = 7.035 ns, Max = 7.099 ns
IQR = 0.081 ns, LowerFence = 6.832 ns, UpperFence = 7.157 ns
ConfidenceInterval = [6.935 ns; 7.049 ns] (CI 99.9%), Margin = 0.057 ns (0.81% of Mean)
Skewness = 0.65, Kurtosis = 1.97, MValue = 2
-------------------- Histogram --------------------
[6.905 ns ; 7.128 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBig: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 6.966 ns, StdErr = 0.012 ns (0.17%), N = 15, StdDev = 0.046 ns
Min = 6.889 ns, Q1 = 6.929 ns, Median = 6.962 ns, Q3 = 6.997 ns, Max = 7.037 ns
IQR = 0.068 ns, LowerFence = 6.828 ns, UpperFence = 7.098 ns
ConfidenceInterval = [6.916 ns; 7.015 ns] (CI 99.9%), Margin = 0.049 ns (0.71% of Mean)
Skewness = 0.1, Kurtosis = 1.71, MValue = 2
-------------------- Histogram --------------------
[6.875 ns ; 7.062 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBigList: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 4.141 ns, StdErr = 0.005 ns (0.12%), N = 14, StdDev = 0.018 ns
Min = 4.117 ns, Q1 = 4.127 ns, Median = 4.139 ns, Q3 = 4.153 ns, Max = 4.174 ns
IQR = 0.026 ns, LowerFence = 4.087 ns, UpperFence = 4.193 ns
ConfidenceInterval = [4.121 ns; 4.161 ns] (CI 99.9%), Margin = 0.020 ns (0.49% of Mean)
Skewness = 0.31, Kurtosis = 1.7, MValue = 2
-------------------- Histogram --------------------
[4.107 ns ; 4.184 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBigHashset: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 4.273 ns, StdErr = 0.018 ns (0.42%), N = 15, StdDev = 0.070 ns
Min = 4.207 ns, Q1 = 4.221 ns, Median = 4.238 ns, Q3 = 4.318 ns, Max = 4.404 ns
IQR = 0.097 ns, LowerFence = 4.076 ns, UpperFence = 4.463 ns
ConfidenceInterval = [4.198 ns; 4.347 ns] (CI 99.9%), Margin = 0.075 ns (1.74% of Mean)
Skewness = 0.79, Kurtosis = 2.01, MValue = 2
-------------------- Histogram --------------------
[4.200 ns ; 4.442 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBigEnumerable: .NET 6.0(Runtime=.NET 6.0)
Runtime = .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 14.245 ns, StdErr = 0.024 ns (0.17%), N = 13, StdDev = 0.086 ns
Min = 14.139 ns, Q1 = 14.177 ns, Median = 14.224 ns, Q3 = 14.299 ns, Max = 14.446 ns
IQR = 0.121 ns, LowerFence = 13.996 ns, UpperFence = 14.481 ns
ConfidenceInterval = [14.142 ns; 14.348 ns] (CI 99.9%), Margin = 0.103 ns (0.72% of Mean)
Skewness = 0.76, Kurtosis = 2.82, MValue = 2
-------------------- Histogram --------------------
[14.091 ns ; 14.495 ns) | @@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnySmall: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 5.193 ns, StdErr = 0.023 ns (0.44%), N = 14, StdDev = 0.085 ns
Min = 5.100 ns, Q1 = 5.142 ns, Median = 5.169 ns, Q3 = 5.214 ns, Max = 5.400 ns
IQR = 0.072 ns, LowerFence = 5.035 ns, UpperFence = 5.321 ns
ConfidenceInterval = [5.097 ns; 5.288 ns] (CI 99.9%), Margin = 0.096 ns (1.85% of Mean)
Skewness = 1.06, Kurtosis = 3.07, MValue = 2
-------------------- Histogram --------------------
[5.090 ns ; 5.211 ns) | @@@@@@@@@@
[5.211 ns ; 5.446 ns) | @@@@
---------------------------------------------------

AnyBenchmark.AnyMedium: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 5.122 ns, StdErr = 0.006 ns (0.12%), N = 14, StdDev = 0.023 ns
Min = 5.072 ns, Q1 = 5.108 ns, Median = 5.121 ns, Q3 = 5.137 ns, Max = 5.170 ns
IQR = 0.029 ns, LowerFence = 5.064 ns, UpperFence = 5.181 ns
ConfidenceInterval = [5.095 ns; 5.148 ns] (CI 99.9%), Margin = 0.026 ns (0.52% of Mean)
Skewness = -0.01, Kurtosis = 2.9, MValue = 2
-------------------- Histogram --------------------
[5.064 ns ; 5.183 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyBig: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 5.122 ns, StdErr = 0.008 ns (0.16%), N = 15, StdDev = 0.032 ns
Min = 5.076 ns, Q1 = 5.099 ns, Median = 5.115 ns, Q3 = 5.137 ns, Max = 5.182 ns
IQR = 0.037 ns, LowerFence = 5.043 ns, UpperFence = 5.193 ns
ConfidenceInterval = [5.088 ns; 5.156 ns] (CI 99.9%), Margin = 0.034 ns (0.67% of Mean)
Skewness = 0.53, Kurtosis = 2, MValue = 2
-------------------- Histogram --------------------
[5.064 ns ; 5.199 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyBigList: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 2.088 ns, StdErr = 0.002 ns (0.12%), N = 15, StdDev = 0.010 ns
Min = 2.072 ns, Q1 = 2.082 ns, Median = 2.085 ns, Q3 = 2.093 ns, Max = 2.107 ns
IQR = 0.011 ns, LowerFence = 2.065 ns, UpperFence = 2.110 ns
ConfidenceInterval = [2.078 ns; 2.098 ns] (CI 99.9%), Margin = 0.010 ns (0.50% of Mean)
Skewness = 0.57, Kurtosis = 2.37, MValue = 2
-------------------- Histogram --------------------
[2.067 ns ; 2.112 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyBigHashet: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 2.096 ns, StdErr = 0.004 ns (0.19%), N = 15, StdDev = 0.015 ns
Min = 2.077 ns, Q1 = 2.084 ns, Median = 2.091 ns, Q3 = 2.105 ns, Max = 2.125 ns
IQR = 0.021 ns, LowerFence = 2.053 ns, UpperFence = 2.137 ns
ConfidenceInterval = [2.079 ns; 2.112 ns] (CI 99.9%), Margin = 0.016 ns (0.78% of Mean)
Skewness = 0.45, Kurtosis = 1.84, MValue = 2
-------------------- Histogram --------------------
[2.069 ns ; 2.133 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyBigEnum: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 9.306 ns, StdErr = 0.019 ns (0.20%), N = 14, StdDev = 0.070 ns
Min = 9.111 ns, Q1 = 9.288 ns, Median = 9.334 ns, Q3 = 9.346 ns, Max = 9.372 ns
IQR = 0.059 ns, LowerFence = 9.200 ns, UpperFence = 9.434 ns
ConfidenceInterval = [9.227 ns; 9.385 ns] (CI 99.9%), Margin = 0.079 ns (0.85% of Mean)
Skewness = -1.5, Kurtosis = 4.56, MValue = 2
-------------------- Histogram --------------------
[9.072 ns ; 9.411 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.LenghtSmall: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 0.008 ns, StdErr = 0.001 ns (11.87%), N = 13, StdDev = 0.003 ns
Min = 0.003 ns, Q1 = 0.006 ns, Median = 0.007 ns, Q3 = 0.009 ns, Max = 0.016 ns
IQR = 0.003 ns, LowerFence = 0.001 ns, UpperFence = 0.013 ns
ConfidenceInterval = [0.004 ns; 0.012 ns] (CI 99.9%), Margin = 0.004 ns (51.23% of Mean)
Skewness = 1.02, Kurtosis = 3.58, MValue = 2
-------------------- Histogram --------------------
[0.002 ns ; 0.005 ns) | @@
[0.005 ns ; 0.009 ns) | @@@@@@@@
[0.009 ns ; 0.013 ns) | @@
[0.013 ns ; 0.014 ns) |
[0.014 ns ; 0.018 ns) | @
---------------------------------------------------

AnyBenchmark.LenghtMedium: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 0.008 ns, StdErr = 0.001 ns (7.05%), N = 14, StdDev = 0.002 ns
Min = 0.004 ns, Q1 = 0.007 ns, Median = 0.008 ns, Q3 = 0.010 ns, Max = 0.012 ns
IQR = 0.003 ns, LowerFence = 0.003 ns, UpperFence = 0.014 ns
ConfidenceInterval = [0.006 ns; 0.011 ns] (CI 99.9%), Margin = 0.002 ns (29.77% of Mean)
Skewness = -0.31, Kurtosis = 2.45, MValue = 2.57
-------------------- Histogram --------------------
[0.003 ns ; 0.006 ns) | @@
[0.006 ns ; 0.007 ns) |
[0.007 ns ; 0.009 ns) | @@@@@@@
[0.009 ns ; 0.011 ns) | @@@@
[0.011 ns ; 0.013 ns) | @
---------------------------------------------------

AnyBenchmark.LenghtBig: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 0.018 ns, StdErr = 0.004 ns (21.17%), N = 15, StdDev = 0.015 ns
Min = 0.002 ns, Q1 = 0.008 ns, Median = 0.011 ns, Q3 = 0.025 ns, Max = 0.046 ns
IQR = 0.017 ns, LowerFence = -0.017 ns, UpperFence = 0.050 ns
ConfidenceInterval = [0.002 ns; 0.033 ns] (CI 99.9%), Margin = 0.016 ns (87.66% of Mean)
Skewness = 0.89, Kurtosis = 2.26, MValue = 2.2
-------------------- Histogram --------------------
[0.001 ns ; 0.017 ns) | @@@@@@@@@@
[0.017 ns ; 0.033 ns) | @@
[0.033 ns ; 0.049 ns) | @@@
---------------------------------------------------

AnyBenchmark.LenghtBigList: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 0.009 ns, StdErr = 0.002 ns (26.13%), N = 15, StdDev = 0.010 ns
Min = 0.000 ns, Q1 = 0.000 ns, Median = 0.007 ns, Q3 = 0.013 ns, Max = 0.027 ns
IQR = 0.012 ns, LowerFence = -0.018 ns, UpperFence = 0.031 ns
ConfidenceInterval = [-0.001 ns; 0.020 ns] (CI 99.9%), Margin = 0.010 ns (108.19% of Mean)
Skewness = 0.67, Kurtosis = 2.03, MValue = 2
-------------------- Histogram --------------------
[-0.001 ns ; 0.010 ns) | @@@@@@@@@
[ 0.010 ns ; 0.020 ns) | @@@
[ 0.020 ns ; 0.030 ns) | @@@
---------------------------------------------------

AnyBenchmark.LenghtBigHashet: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 0.004 ns, StdErr = 0.001 ns (26.95%), N = 15, StdDev = 0.004 ns
Min = 0.000 ns, Q1 = 0.000 ns, Median = 0.003 ns, Q3 = 0.007 ns, Max = 0.016 ns
IQR = 0.007 ns, LowerFence = -0.010 ns, UpperFence = 0.017 ns
ConfidenceInterval = [-0.000 ns; 0.009 ns] (CI 99.9%), Margin = 0.005 ns (111.58% of Mean)
Skewness = 0.91, Kurtosis = 3.11, MValue = 2.22
-------------------- Histogram --------------------
[-0.001 ns ; 0.004 ns) | @@@@@@@@@
[ 0.004 ns ; 0.010 ns) | @@@@@
[ 0.010 ns ; 0.013 ns) |
[ 0.013 ns ; 0.018 ns) | @
---------------------------------------------------

AnyBenchmark.LenghtBigEnum: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 583.897 us, StdErr = 1.288 us (0.22%), N = 15, StdDev = 4.987 us
Min = 575.210 us, Q1 = 580.512 us, Median = 583.399 us, Q3 = 586.560 us, Max = 592.546 us
IQR = 6.048 us, LowerFence = 571.440 us, UpperFence = 595.632 us
ConfidenceInterval = [578.566 us; 589.229 us] (CI 99.9%), Margin = 5.331 us (0.91% of Mean)
Skewness = 0.14, Kurtosis = 1.98, MValue = 2
-------------------- Histogram --------------------
[572.556 us ; 595.200 us) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountSmall: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 15.363 ns, StdErr = 0.040 ns (0.26%), N = 15, StdDev = 0.156 ns
Min = 15.132 ns, Q1 = 15.208 ns, Median = 15.406 ns, Q3 = 15.460 ns, Max = 15.643 ns
IQR = 0.252 ns, LowerFence = 14.831 ns, UpperFence = 15.838 ns
ConfidenceInterval = [15.196 ns; 15.529 ns] (CI 99.9%), Margin = 0.166 ns (1.08% of Mean)
Skewness = -0.03, Kurtosis = 1.72, MValue = 2
-------------------- Histogram --------------------
[15.049 ns ; 15.726 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountMedium: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 15.200 ns, StdErr = 0.035 ns (0.23%), N = 14, StdDev = 0.132 ns
Min = 14.973 ns, Q1 = 15.107 ns, Median = 15.222 ns, Q3 = 15.295 ns, Max = 15.383 ns
IQR = 0.188 ns, LowerFence = 14.824 ns, UpperFence = 15.578 ns
ConfidenceInterval = [15.052 ns; 15.349 ns] (CI 99.9%), Margin = 0.148 ns (0.98% of Mean)
Skewness = -0.3, Kurtosis = 1.67, MValue = 2
-------------------- Histogram --------------------
[14.902 ns ; 15.422 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountBig: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 15.350 ns, StdErr = 0.045 ns (0.29%), N = 15, StdDev = 0.173 ns
Min = 15.077 ns, Q1 = 15.223 ns, Median = 15.366 ns, Q3 = 15.422 ns, Max = 15.673 ns
IQR = 0.199 ns, LowerFence = 14.924 ns, UpperFence = 15.721 ns
ConfidenceInterval = [15.166 ns; 15.535 ns] (CI 99.9%), Margin = 0.184 ns (1.20% of Mean)
Skewness = 0.36, Kurtosis = 1.98, MValue = 2
-------------------- Histogram --------------------
[14.985 ns ; 15.765 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountBigList: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 11.565 ns, StdErr = 0.021 ns (0.18%), N = 14, StdDev = 0.078 ns
Min = 11.332 ns, Q1 = 11.551 ns, Median = 11.572 ns, Q3 = 11.607 ns, Max = 11.657 ns
IQR = 0.057 ns, LowerFence = 11.465 ns, UpperFence = 11.693 ns
ConfidenceInterval = [11.477 ns; 11.653 ns] (CI 99.9%), Margin = 0.088 ns (0.76% of Mean)
Skewness = -1.67, Kurtosis = 5.87, MValue = 2
-------------------- Histogram --------------------
[11.289 ns ; 11.700 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountBigHashet: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 20.108 ns, StdErr = 0.070 ns (0.35%), N = 14, StdDev = 0.262 ns
Min = 19.611 ns, Q1 = 19.998 ns, Median = 20.065 ns, Q3 = 20.115 ns, Max = 20.581 ns
IQR = 0.117 ns, LowerFence = 19.823 ns, UpperFence = 20.289 ns
ConfidenceInterval = [19.812 ns; 20.404 ns] (CI 99.9%), Margin = 0.296 ns (1.47% of Mean)
Skewness = 0.4, Kurtosis = 2.56, MValue = 2
-------------------- Histogram --------------------
[19.469 ns ; 20.215 ns) | @@@@@@@@@@@
[20.215 ns ; 20.724 ns) | @@@
---------------------------------------------------

AnyBenchmark.TakeCountBigEnum: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 18.014 ns, StdErr = 0.029 ns (0.16%), N = 14, StdDev = 0.109 ns
Min = 17.751 ns, Q1 = 17.974 ns, Median = 18.062 ns, Q3 = 18.092 ns, Max = 18.124 ns
IQR = 0.118 ns, LowerFence = 17.796 ns, UpperFence = 18.269 ns
ConfidenceInterval = [17.891 ns; 18.137 ns] (CI 99.9%), Margin = 0.123 ns (0.68% of Mean)
Skewness = -1, Kurtosis = 2.93, MValue = 2
-------------------- Histogram --------------------
[17.692 ns ; 18.184 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckSmall: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 5.138 ns, StdErr = 0.003 ns (0.05%), N = 13, StdDev = 0.010 ns
Min = 5.123 ns, Q1 = 5.132 ns, Median = 5.142 ns, Q3 = 5.144 ns, Max = 5.153 ns
IQR = 0.013 ns, LowerFence = 5.113 ns, UpperFence = 5.163 ns
ConfidenceInterval = [5.127 ns; 5.150 ns] (CI 99.9%), Margin = 0.012 ns (0.23% of Mean)
Skewness = -0.16, Kurtosis = 1.56, MValue = 2
-------------------- Histogram --------------------
[5.118 ns ; 5.158 ns) | @@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckMedium: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 5.123 ns, StdErr = 0.008 ns (0.15%), N = 14, StdDev = 0.029 ns
Min = 5.081 ns, Q1 = 5.106 ns, Median = 5.121 ns, Q3 = 5.139 ns, Max = 5.189 ns
IQR = 0.033 ns, LowerFence = 5.057 ns, UpperFence = 5.188 ns
ConfidenceInterval = [5.091 ns; 5.156 ns] (CI 99.9%), Margin = 0.032 ns (0.63% of Mean)
Skewness = 0.48, Kurtosis = 2.78, MValue = 2
-------------------- Histogram --------------------
[5.079 ns ; 5.205 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBig: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 5.150 ns, StdErr = 0.005 ns (0.09%), N = 15, StdDev = 0.018 ns
Min = 5.117 ns, Q1 = 5.135 ns, Median = 5.153 ns, Q3 = 5.162 ns, Max = 5.176 ns
IQR = 0.027 ns, LowerFence = 5.094 ns, UpperFence = 5.202 ns
ConfidenceInterval = [5.130 ns; 5.169 ns] (CI 99.9%), Margin = 0.019 ns (0.38% of Mean)
Skewness = -0.15, Kurtosis = 1.68, MValue = 2
-------------------- Histogram --------------------
[5.107 ns ; 5.186 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBigList: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 1.740 ns, StdErr = 0.013 ns (0.76%), N = 15, StdDev = 0.051 ns
Min = 1.675 ns, Q1 = 1.705 ns, Median = 1.743 ns, Q3 = 1.767 ns, Max = 1.880 ns
IQR = 0.063 ns, LowerFence = 1.611 ns, UpperFence = 1.861 ns
ConfidenceInterval = [1.686 ns; 1.795 ns] (CI 99.9%), Margin = 0.055 ns (3.14% of Mean)
Skewness = 1.13, Kurtosis = 4.1, MValue = 2
-------------------- Histogram --------------------
[1.666 ns ; 1.720 ns) | @@@@@@@
[1.720 ns ; 1.788 ns) | @@@@@@@
[1.788 ns ; 1.853 ns) |
[1.853 ns ; 1.908 ns) | @
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBigHashset: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 1.916 ns, StdErr = 0.011 ns (0.58%), N = 15, StdDev = 0.043 ns
Min = 1.868 ns, Q1 = 1.888 ns, Median = 1.902 ns, Q3 = 1.945 ns, Max = 2.014 ns
IQR = 0.057 ns, LowerFence = 1.802 ns, UpperFence = 2.031 ns
ConfidenceInterval = [1.870 ns; 1.962 ns] (CI 99.9%), Margin = 0.046 ns (2.40% of Mean)
Skewness = 0.81, Kurtosis = 2.44, MValue = 2
-------------------- Histogram --------------------
[1.865 ns ; 1.910 ns) | @@@@@@@@@@
[1.910 ns ; 1.977 ns) | @@@@
[1.977 ns ; 2.037 ns) | @
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBigEnumerable: .NET 8.0(Runtime=.NET 8.0)
Runtime = .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2; GC = Concurrent Workstation
Mean = 9.522 ns, StdErr = 0.047 ns (0.49%), N = 12, StdDev = 0.161 ns
Min = 9.311 ns, Q1 = 9.390 ns, Median = 9.490 ns, Q3 = 9.664 ns, Max = 9.756 ns
IQR = 0.273 ns, LowerFence = 8.980 ns, UpperFence = 10.074 ns
ConfidenceInterval = [9.315 ns; 9.728 ns] (CI 99.9%), Margin = 0.206 ns (2.17% of Mean)
Skewness = 0.16, Kurtosis = 1.27, MValue = 2
-------------------- Histogram --------------------
[9.218 ns ; 9.520 ns) | @@@@@@@
[9.520 ns ; 9.787 ns) | @@@@@
---------------------------------------------------

AnyBenchmark.AnySmall: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 8.829 ns, StdErr = 0.055 ns (0.62%), N = 37, StdDev = 0.335 ns
Min = 8.360 ns, Q1 = 8.619 ns, Median = 8.704 ns, Q3 = 9.041 ns, Max = 9.608 ns
IQR = 0.423 ns, LowerFence = 7.985 ns, UpperFence = 9.675 ns
ConfidenceInterval = [8.632 ns; 9.027 ns] (CI 99.9%), Margin = 0.198 ns (2.24% of Mean)
Skewness = 0.86, Kurtosis = 2.67, MValue = 2
-------------------- Histogram --------------------
[8.228 ns ; 8.458 ns) | @@
[8.458 ns ; 8.722 ns) | @@@@@@@@@@@@@@@@@@
[8.722 ns ; 9.001 ns) | @@@@@@@
[9.001 ns ; 9.290 ns) | @@@@@@
[9.290 ns ; 9.645 ns) | @@@@
---------------------------------------------------

AnyBenchmark.AnyMedium: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 8.496 ns, StdErr = 0.047 ns (0.55%), N = 19, StdDev = 0.205 ns
Min = 8.310 ns, Q1 = 8.387 ns, Median = 8.416 ns, Q3 = 8.490 ns, Max = 9.118 ns
IQR = 0.102 ns, LowerFence = 8.234 ns, UpperFence = 8.643 ns
ConfidenceInterval = [8.312 ns; 8.681 ns] (CI 99.9%), Margin = 0.185 ns (2.17% of Mean)
Skewness = 1.66, Kurtosis = 4.96, MValue = 2
-------------------- Histogram --------------------
[8.307 ns ; 8.509 ns) | @@@@@@@@@@@@@@@
[8.509 ns ; 8.845 ns) | @@@
[8.845 ns ; 9.219 ns) | @
---------------------------------------------------

AnyBenchmark.AnyBig: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 8.241 ns, StdErr = 0.014 ns (0.17%), N = 15, StdDev = 0.055 ns
Min = 8.134 ns, Q1 = 8.214 ns, Median = 8.239 ns, Q3 = 8.281 ns, Max = 8.316 ns
IQR = 0.067 ns, LowerFence = 8.114 ns, UpperFence = 8.381 ns
ConfidenceInterval = [8.182 ns; 8.300 ns] (CI 99.9%), Margin = 0.059 ns (0.71% of Mean)
Skewness = -0.49, Kurtosis = 2.06, MValue = 2
-------------------- Histogram --------------------
[8.104 ns ; 8.346 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyBigList: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 10.431 ns, StdErr = 0.020 ns (0.20%), N = 14, StdDev = 0.076 ns
Min = 10.323 ns, Q1 = 10.371 ns, Median = 10.431 ns, Q3 = 10.483 ns, Max = 10.566 ns
IQR = 0.112 ns, LowerFence = 10.203 ns, UpperFence = 10.650 ns
ConfidenceInterval = [10.345 ns; 10.517 ns] (CI 99.9%), Margin = 0.086 ns (0.83% of Mean)
Skewness = 0.31, Kurtosis = 1.72, MValue = 2
-------------------- Histogram --------------------
[10.301 ns ; 10.608 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyBigHashet: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 10.855 ns, StdErr = 0.013 ns (0.12%), N = 13, StdDev = 0.046 ns
Min = 10.802 ns, Q1 = 10.821 ns, Median = 10.845 ns, Q3 = 10.887 ns, Max = 10.957 ns
IQR = 0.066 ns, LowerFence = 10.722 ns, UpperFence = 10.985 ns
ConfidenceInterval = [10.799 ns; 10.910 ns] (CI 99.9%), Margin = 0.056 ns (0.51% of Mean)
Skewness = 0.63, Kurtosis = 2.3, MValue = 2
-------------------- Histogram --------------------
[10.776 ns ; 10.983 ns) | @@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.AnyBigEnum: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 9.368 ns, StdErr = 0.011 ns (0.12%), N = 14, StdDev = 0.042 ns
Min = 9.311 ns, Q1 = 9.339 ns, Median = 9.364 ns, Q3 = 9.385 ns, Max = 9.454 ns
IQR = 0.046 ns, LowerFence = 9.270 ns, UpperFence = 9.454 ns
ConfidenceInterval = [9.320 ns; 9.415 ns] (CI 99.9%), Margin = 0.048 ns (0.51% of Mean)
Skewness = 0.55, Kurtosis = 2.38, MValue = 2
-------------------- Histogram --------------------
[9.288 ns ; 9.470 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.LenghtSmall: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 0.001 ns, StdErr = 0.001 ns (81.90%), N = 13, StdDev = 0.002 ns
Min = 0.000 ns, Q1 = 0.000 ns, Median = 0.000 ns, Q3 = 0.000 ns, Max = 0.008 ns
IQR = 0.000 ns, LowerFence = 0.000 ns, UpperFence = 0.000 ns
ConfidenceInterval = [-0.002 ns; 0.004 ns] (CI 99.9%), Margin = 0.003 ns (353.64% of Mean)
Skewness = 2.72, Kurtosis = 9.06, MValue = 2
-------------------- Histogram --------------------
[-0.001 ns ; 0.002 ns) | @@@@@@@@@@@@
[ 0.002 ns ; 0.004 ns) |
[ 0.004 ns ; 0.007 ns) |
[ 0.007 ns ; 0.010 ns) | @
---------------------------------------------------

AnyBenchmark.LenghtMedium: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 0.006 ns, StdErr = 0.001 ns (23.26%), N = 15, StdDev = 0.005 ns
Min = 0.000 ns, Q1 = 0.000 ns, Median = 0.006 ns, Q3 = 0.009 ns, Max = 0.018 ns
IQR = 0.009 ns, LowerFence = -0.013 ns, UpperFence = 0.022 ns
ConfidenceInterval = [0.000 ns; 0.012 ns] (CI 99.9%), Margin = 0.006 ns (96.32% of Mean)
Skewness = 0.47, Kurtosis = 2.32, MValue = 2.5
-------------------- Histogram --------------------
[-0.003 ns ; 0.003 ns) | @@@@@
[ 0.003 ns ; 0.010 ns) | @@@@@@@@
[ 0.010 ns ; 0.012 ns) |
[ 0.012 ns ; 0.018 ns) | @@
---------------------------------------------------

AnyBenchmark.LenghtBig: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 0.002 ns, StdErr = 0.001 ns (37.84%), N = 14, StdDev = 0.004 ns
Min = 0.000 ns, Q1 = 0.000 ns, Median = 0.000 ns, Q3 = 0.006 ns, Max = 0.008 ns
IQR = 0.006 ns, LowerFence = -0.008 ns, UpperFence = 0.014 ns
ConfidenceInterval = [-0.001 ns; 0.006 ns] (CI 99.9%), Margin = 0.004 ns (159.70% of Mean)
Skewness = 0.78, Kurtosis = 1.66, MValue = 2.8
-------------------- Histogram --------------------
[-0.001 ns ; 0.003 ns) | @@@@@@@@@@
[ 0.003 ns ; 0.005 ns) |
[ 0.005 ns ; 0.009 ns) | @@@@
---------------------------------------------------

AnyBenchmark.LenghtBigList: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 0.013 ns, StdErr = 0.005 ns (35.46%), N = 12, StdDev = 0.016 ns
Min = 0.000 ns, Q1 = 0.000 ns, Median = 0.009 ns, Q3 = 0.016 ns, Max = 0.051 ns
IQR = 0.016 ns, LowerFence = -0.024 ns, UpperFence = 0.039 ns
ConfidenceInterval = [-0.008 ns; 0.034 ns] (CI 99.9%), Margin = 0.021 ns (157.35% of Mean)
Skewness = 1.16, Kurtosis = 3.13, MValue = 2
-------------------- Histogram --------------------
[-0.003 ns ; 0.020 ns) | @@@@@@@@@
[ 0.020 ns ; 0.042 ns) | @@
[ 0.042 ns ; 0.061 ns) | @
---------------------------------------------------

AnyBenchmark.LenghtBigHashet: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 0.012 ns, StdErr = 0.004 ns (30.90%), N = 33, StdDev = 0.021 ns
Min = 0.000 ns, Q1 = 0.000 ns, Median = 0.000 ns, Q3 = 0.023 ns, Max = 0.075 ns
IQR = 0.023 ns, LowerFence = -0.034 ns, UpperFence = 0.056 ns
ConfidenceInterval = [-0.001 ns; 0.025 ns] (CI 99.9%), Margin = 0.013 ns (111.92% of Mean)
Skewness = 1.57, Kurtosis = 4.32, MValue = 2.42
-------------------- Histogram --------------------
[-0.005 ns ; 0.012 ns) | @@@@@@@@@@@@@@@@@@@@@@@@
[ 0.012 ns ; 0.022 ns) |
[ 0.022 ns ; 0.039 ns) | @@@@@@
[ 0.039 ns ; 0.049 ns) |
[ 0.049 ns ; 0.066 ns) | @@
[ 0.066 ns ; 0.084 ns) | @
---------------------------------------------------

AnyBenchmark.LenghtBigEnum: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 2.082 ms, StdErr = 0.012 ms (0.58%), N = 79, StdDev = 0.107 ms
Min = 1.923 ms, Q1 = 1.972 ms, Median = 2.110 ms, Q3 = 2.160 ms, Max = 2.365 ms
IQR = 0.188 ms, LowerFence = 1.690 ms, UpperFence = 2.441 ms
ConfidenceInterval = [2.041 ms; 2.124 ms] (CI 99.9%), Margin = 0.041 ms (1.99% of Mean)
Skewness = 0.15, Kurtosis = 2.16, MValue = 3.16
-------------------- Histogram --------------------
[1.890 ms ; 1.939 ms) | @@@@
[1.939 ms ; 2.005 ms) | @@@@@@@@@@@@@@@@@@@@@@@
[2.005 ms ; 2.054 ms) | @@@@@
[2.054 ms ; 2.129 ms) | @@@@@@@@@@
[2.129 ms ; 2.195 ms) | @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
[2.195 ms ; 2.271 ms) | @@@
[2.271 ms ; 2.337 ms) | @@
[2.337 ms ; 2.398 ms) | @
---------------------------------------------------

AnyBenchmark.TakeCountSmall: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 46.951 ns, StdErr = 0.709 ns (1.51%), N = 100, StdDev = 7.095 ns
Min = 35.600 ns, Q1 = 39.960 ns, Median = 49.457 ns, Q3 = 52.809 ns, Max = 61.196 ns
IQR = 12.849 ns, LowerFence = 20.686 ns, UpperFence = 72.083 ns
ConfidenceInterval = [44.544 ns; 49.357 ns] (CI 99.9%), Margin = 2.406 ns (5.13% of Mean)
Skewness = -0.26, Kurtosis = 1.78, MValue = 3.3
-------------------- Histogram --------------------
[35.586 ns ; 39.598 ns) | @@@@@@@@@@@@@@@@@@@@@@@
[39.598 ns ; 42.651 ns) | @@@@@@
[42.651 ns ; 46.664 ns) | @@@@@@@@@@@@@@@@
[46.664 ns ; 50.077 ns) | @@@@@@@@@
[50.077 ns ; 54.090 ns) | @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
[54.090 ns ; 57.976 ns) | @@@@@@
[57.976 ns ; 62.136 ns) | @@@
---------------------------------------------------

AnyBenchmark.TakeCountMedium: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 41.814 ns, StdErr = 0.678 ns (1.62%), N = 100, StdDev = 6.779 ns
Min = 35.756 ns, Q1 = 36.265 ns, Median = 37.647 ns, Q3 = 48.903 ns, Max = 56.226 ns
IQR = 12.638 ns, LowerFence = 17.307 ns, UpperFence = 67.860 ns
ConfidenceInterval = [39.514 ns; 44.113 ns] (CI 99.9%), Margin = 2.299 ns (5.50% of Mean)
Skewness = 0.71, Kurtosis = 1.76, MValue = 2.81
-------------------- Histogram --------------------
[35.699 ns ; 39.533 ns) | @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
[39.533 ns ; 43.612 ns) | @@@@@@@
[43.612 ns ; 44.946 ns) |
[44.946 ns ; 48.779 ns) | @@@@@@@@@
[48.779 ns ; 53.806 ns) | @@@@@@@@@@@@@@@@@@@@@@@@
[53.806 ns ; 58.143 ns) | @
---------------------------------------------------

AnyBenchmark.TakeCountBig: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 35.496 ns, StdErr = 0.162 ns (0.46%), N = 14, StdDev = 0.607 ns
Min = 34.784 ns, Q1 = 35.150 ns, Median = 35.337 ns, Q3 = 35.719 ns, Max = 36.875 ns
IQR = 0.568 ns, LowerFence = 34.297 ns, UpperFence = 36.571 ns
ConfidenceInterval = [34.811 ns; 36.180 ns] (CI 99.9%), Margin = 0.684 ns (1.93% of Mean)
Skewness = 1.03, Kurtosis = 2.89, MValue = 2
-------------------- Histogram --------------------
[34.747 ns ; 36.223 ns) | @@@@@@@@@@@@
[36.223 ns ; 37.094 ns) | @@
---------------------------------------------------

AnyBenchmark.TakeCountBigList: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 38.437 ns, StdErr = 0.140 ns (0.37%), N = 15, StdDev = 0.543 ns
Min = 37.876 ns, Q1 = 38.044 ns, Median = 38.131 ns, Q3 = 38.690 ns, Max = 39.597 ns
IQR = 0.646 ns, LowerFence = 37.074 ns, UpperFence = 39.660 ns
ConfidenceInterval = [37.856 ns; 39.018 ns] (CI 99.9%), Margin = 0.581 ns (1.51% of Mean)
Skewness = 0.81, Kurtosis = 2.2, MValue = 2
-------------------- Histogram --------------------
[37.587 ns ; 38.648 ns) | @@@@@@@@@@@
[38.648 ns ; 39.887 ns) | @@@@
---------------------------------------------------

AnyBenchmark.TakeCountBigHashet: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 38.268 ns, StdErr = 0.066 ns (0.17%), N = 14, StdDev = 0.249 ns
Min = 38.041 ns, Q1 = 38.084 ns, Median = 38.190 ns, Q3 = 38.283 ns, Max = 38.874 ns
IQR = 0.199 ns, LowerFence = 37.786 ns, UpperFence = 38.581 ns
ConfidenceInterval = [37.988 ns; 38.549 ns] (CI 99.9%), Margin = 0.280 ns (0.73% of Mean)
Skewness = 1.18, Kurtosis = 3.16, MValue = 2
-------------------- Histogram --------------------
[37.905 ns ; 39.010 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.TakeCountBigEnum: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 36.392 ns, StdErr = 0.033 ns (0.09%), N = 14, StdDev = 0.122 ns
Min = 36.178 ns, Q1 = 36.295 ns, Median = 36.402 ns, Q3 = 36.469 ns, Max = 36.620 ns
IQR = 0.174 ns, LowerFence = 36.034 ns, UpperFence = 36.730 ns
ConfidenceInterval = [36.255 ns; 36.530 ns] (CI 99.9%), Margin = 0.137 ns (0.38% of Mean)
Skewness = 0.08, Kurtosis = 2.06, MValue = 2
-------------------- Histogram --------------------
[36.112 ns ; 36.686 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckSmall: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 8.167 ns, StdErr = 0.012 ns (0.15%), N = 14, StdDev = 0.046 ns
Min = 8.101 ns, Q1 = 8.132 ns, Median = 8.163 ns, Q3 = 8.189 ns, Max = 8.259 ns
IQR = 0.057 ns, LowerFence = 8.045 ns, UpperFence = 8.275 ns
ConfidenceInterval = [8.115 ns; 8.220 ns] (CI 99.9%), Margin = 0.052 ns (0.64% of Mean)
Skewness = 0.44, Kurtosis = 2.16, MValue = 2
-------------------- Histogram --------------------
[8.090 ns ; 8.285 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckMedium: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 8.126 ns, StdErr = 0.009 ns (0.11%), N = 13, StdDev = 0.031 ns
Min = 8.074 ns, Q1 = 8.101 ns, Median = 8.133 ns, Q3 = 8.145 ns, Max = 8.177 ns
IQR = 0.044 ns, LowerFence = 8.035 ns, UpperFence = 8.211 ns
ConfidenceInterval = [8.089 ns; 8.163 ns] (CI 99.9%), Margin = 0.037 ns (0.45% of Mean)
Skewness = -0.3, Kurtosis = 1.88, MValue = 2
-------------------- Histogram --------------------
[8.057 ns ; 8.194 ns) | @@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBig: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 7.976 ns, StdErr = 0.006 ns (0.08%), N = 15, StdDev = 0.024 ns
Min = 7.939 ns, Q1 = 7.956 ns, Median = 7.982 ns, Q3 = 7.990 ns, Max = 8.022 ns
IQR = 0.034 ns, LowerFence = 7.906 ns, UpperFence = 8.041 ns
ConfidenceInterval = [7.950 ns; 8.002 ns] (CI 99.9%), Margin = 0.026 ns (0.33% of Mean)
Skewness = 0.22, Kurtosis = 1.75, MValue = 2
-------------------- Histogram --------------------
[7.926 ns ; 8.035 ns) | @@@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBigList: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 10.421 ns, StdErr = 0.063 ns (0.61%), N = 37, StdDev = 0.385 ns
Min = 9.996 ns, Q1 = 10.115 ns, Median = 10.266 ns, Q3 = 10.708 ns, Max = 11.334 ns
IQR = 0.593 ns, LowerFence = 9.225 ns, UpperFence = 11.598 ns
ConfidenceInterval = [10.194 ns; 10.648 ns] (CI 99.9%), Margin = 0.227 ns (2.18% of Mean)
Skewness = 0.92, Kurtosis = 2.52, MValue = 2.36
-------------------- Histogram --------------------
[ 9.994 ns ; 10.297 ns) | @@@@@@@@@@@@@@@@@@@@@@
[10.297 ns ; 10.601 ns) | @@@
[10.601 ns ; 10.905 ns) | @@@@@@@
[10.905 ns ; 11.486 ns) | @@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBigHashset: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 10.856 ns, StdErr = 0.032 ns (0.30%), N = 14, StdDev = 0.120 ns
Min = 10.712 ns, Q1 = 10.777 ns, Median = 10.827 ns, Q3 = 10.874 ns, Max = 11.142 ns
IQR = 0.096 ns, LowerFence = 10.633 ns, UpperFence = 11.018 ns
ConfidenceInterval = [10.721 ns; 10.992 ns] (CI 99.9%), Margin = 0.135 ns (1.25% of Mean)
Skewness = 0.96, Kurtosis = 2.96, MValue = 2
-------------------- Histogram --------------------
[10.710 ns ; 11.208 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

AnyBenchmark.EnumeratorCheckBigEnumerable: .NET Framework 4.8(Runtime=.NET Framework 4.8)
Runtime = .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256; GC = Concurrent Workstation
Mean = 9.363 ns, StdErr = 0.016 ns (0.17%), N = 14, StdDev = 0.060 ns
Min = 9.294 ns, Q1 = 9.321 ns, Median = 9.344 ns, Q3 = 9.382 ns, Max = 9.503 ns
IQR = 0.061 ns, LowerFence = 9.230 ns, UpperFence = 9.473 ns
ConfidenceInterval = [9.294 ns; 9.431 ns] (CI 99.9%), Margin = 0.068 ns (0.73% of Mean)
Skewness = 0.89, Kurtosis = 2.75, MValue = 2
-------------------- Histogram --------------------
[9.261 ns ; 9.536 ns) | @@@@@@@@@@@@@@
---------------------------------------------------

// * Summary *

BenchmarkDotNet v0.13.12, Windows 10 (10.0.19045.3930/22H2/2022Update)
12th Gen Intel Core i9-12900H, 1 CPU, 20 logical and 14 physical cores
.NET SDK 8.0.101
  [Host]             : .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2
  Job-FQEJOX         : .NET 7.0.15 (7.0.1523.57226), X64 RyuJIT AVX2
  Job-DISSLE         : .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2
  Job-HXWWDC         : .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256
  .NET 6.0           : .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2
  .NET 8.0           : .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2
  .NET Framework 4.8 : .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256

```

| Method                       | Job                | Runtime            | Mean              | Error          | StdDev          | Median            | Ratio          | RatioSD      | Gen0   | Allocated | Alloc Ratio |
|----------------------------- |------------------- |------------------- |------------------:|---------------:|----------------:|------------------:|---------------:|-------------:|-------:|----------:|------------:|
| AnySmall                     | Job-FQEJOX         | .NET 7.0           |         5.5193 ns |      0.1327 ns |       0.2180 ns |         5.4247 ns |         65.202 |         9.06 |      - |         - |          NA |
| AnyMedium                    | Job-FQEJOX         | .NET 7.0           |         5.1544 ns |      0.0087 ns |       0.0073 ns |         5.1556 ns |         58.754 |         9.03 |      - |         - |          NA |
| AnyBig                       | Job-FQEJOX         | .NET 7.0           |         5.6388 ns |      0.0147 ns |       0.0137 ns |         5.6380 ns |         64.855 |         9.31 |      - |         - |          NA |
| AnyBigList                   | Job-FQEJOX         | .NET 7.0           |         3.4669 ns |      0.0104 ns |       0.0087 ns |         3.4654 ns |         39.521 |         6.10 |      - |         - |          NA |
| AnyBigHashet                 | Job-FQEJOX         | .NET 7.0           |         3.4412 ns |      0.0104 ns |       0.0097 ns |         3.4423 ns |         39.574 |         5.64 |      - |         - |          NA |
| AnyBigEnum                   | Job-FQEJOX         | .NET 7.0           |        11.7390 ns |      0.0935 ns |       0.0829 ns |        11.7125 ns |        134.898 |        20.21 | 0.0032 |      40 B |          NA |
| LenghtSmall                  | Job-FQEJOX         | .NET 7.0           |         0.0000 ns |      0.0000 ns |       0.0000 ns |         0.0000 ns |          0.000 |         0.00 |      - |         - |          NA |
| LenghtMedium                 | Job-FQEJOX         | .NET 7.0           |         0.0005 ns |      0.0020 ns |       0.0016 ns |         0.0000 ns |          0.005 |         0.02 |      - |         - |          NA |
| LenghtBig                    | Job-FQEJOX         | .NET 7.0           |         0.0003 ns |      0.0008 ns |       0.0008 ns |         0.0000 ns |          0.004 |         0.01 |      - |         - |          NA |
| LenghtBigList                | Job-FQEJOX         | .NET 7.0           |         0.0002 ns |      0.0007 ns |       0.0006 ns |         0.0000 ns |          0.002 |         0.01 |      - |         - |          NA |
| LenghtBigHashet              | Job-FQEJOX         | .NET 7.0           |         0.0024 ns |      0.0024 ns |       0.0020 ns |         0.0023 ns |          0.027 |         0.02 |      - |         - |          NA |
| LenghtBigEnum                | Job-FQEJOX         | .NET 7.0           | 2,226,575.1302 ns | 17,281.0371 ns |  16,164.6929 ns | 2,232,578.9062 ns | 25,624,584.160 | 3,797,599.39 |      - |      42 B |          NA |
| TakeCountSmall               | Job-FQEJOX         | .NET 7.0           |        22.8527 ns |      0.2173 ns |       0.1815 ns |        22.8633 ns |        260.402 |        39.29 | 0.0038 |      48 B |          NA |
| TakeCountMedium              | Job-FQEJOX         | .NET 7.0           |        24.4885 ns |      0.2504 ns |       0.2342 ns |        24.5083 ns |        281.775 |        41.39 | 0.0038 |      48 B |          NA |
| TakeCountBig                 | Job-FQEJOX         | .NET 7.0           |        22.7268 ns |      0.2396 ns |       0.2241 ns |        22.8053 ns |        261.403 |        37.73 | 0.0038 |      48 B |          NA |
| TakeCountBigList             | Job-FQEJOX         | .NET 7.0           |        20.2234 ns |      0.1097 ns |       0.0973 ns |        20.2509 ns |        232.443 |        35.16 | 0.0038 |      48 B |          NA |
| TakeCountBigHashet           | Job-FQEJOX         | .NET 7.0           |        30.9384 ns |      0.1732 ns |       0.1620 ns |        31.0010 ns |        355.901 |        51.69 | 0.0076 |      96 B |          NA |
| TakeCountBigEnum             | Job-FQEJOX         | .NET 7.0           |        26.0982 ns |      0.3957 ns |       0.3702 ns |        26.0341 ns |        300.198 |        43.41 | 0.0076 |      96 B |          NA |
| EnumeratorCheckSmall         | Job-FQEJOX         | .NET 7.0           |         5.9595 ns |      0.0359 ns |       0.0336 ns |         5.9563 ns |         68.551 |         9.91 |      - |         - |          NA |
| EnumeratorCheckMedium        | Job-FQEJOX         | .NET 7.0           |         5.9652 ns |      0.0330 ns |       0.0309 ns |         5.9524 ns |         68.601 |         9.78 |      - |         - |          NA |
| EnumeratorCheckBig           | Job-FQEJOX         | .NET 7.0           |         6.4964 ns |      0.0572 ns |       0.0535 ns |         6.4886 ns |         74.742 |        10.93 |      - |         - |          NA |
| EnumeratorCheckBigList       | Job-FQEJOX         | .NET 7.0           |         4.0621 ns |      0.0266 ns |       0.0248 ns |         4.0571 ns |         46.736 |         6.86 |      - |         - |          NA |
| EnumeratorCheckBigHashset    | Job-FQEJOX         | .NET 7.0           |         4.0694 ns |      0.0236 ns |       0.0209 ns |         4.0658 ns |         46.750 |         6.90 |      - |         - |          NA |
| EnumeratorCheckBigEnumerable | Job-FQEJOX         | .NET 7.0           |        14.7422 ns |      0.0895 ns |       0.0837 ns |        14.7575 ns |        169.591 |        24.59 | 0.0032 |      40 B |          NA |
| AnySmall                     | Job-DISSLE         | .NET 8.0           |         5.4162 ns |      0.1301 ns |       0.1278 ns |         5.3537 ns |         62.244 |         8.54 |      - |         - |          NA |
| AnyMedium                    | Job-DISSLE         | .NET 8.0           |         5.2541 ns |      0.0240 ns |       0.0213 ns |         5.2591 ns |         60.364 |         8.95 |      - |         - |          NA |
| AnyBig                       | Job-DISSLE         | .NET 8.0           |         5.2532 ns |      0.0234 ns |       0.0208 ns |         5.2545 ns |         60.346 |         8.89 |      - |         - |          NA |
| AnyBigList                   | Job-DISSLE         | .NET 8.0           |         2.1404 ns |      0.0248 ns |       0.0220 ns |         2.1382 ns |         24.620 |         3.87 |      - |         - |          NA |
| AnyBigHashet                 | Job-DISSLE         | .NET 8.0           |         2.1367 ns |      0.0214 ns |       0.0200 ns |         2.1318 ns |         24.567 |         3.47 |      - |         - |          NA |
| AnyBigEnum                   | Job-DISSLE         | .NET 8.0           |        10.3076 ns |      0.0988 ns |       0.0924 ns |        10.3198 ns |        118.566 |        17.16 | 0.0032 |      40 B |          NA |
| LenghtSmall                  | Job-DISSLE         | .NET 8.0           |         0.0001 ns |      0.0002 ns |       0.0002 ns |         0.0000 ns |          0.001 |         0.00 |      - |         - |          NA |
| LenghtMedium                 | Job-DISSLE         | .NET 8.0           |         0.0001 ns |      0.0004 ns |       0.0004 ns |         0.0000 ns |          0.001 |         0.00 |      - |         - |          NA |
| LenghtBig                    | Job-DISSLE         | .NET 8.0           |         0.0009 ns |      0.0009 ns |       0.0009 ns |         0.0008 ns |          0.011 |         0.01 |      - |         - |          NA |
| LenghtBigList                | Job-DISSLE         | .NET 8.0           |         0.0003 ns |      0.0005 ns |       0.0005 ns |         0.0000 ns |          0.003 |         0.00 |      - |         - |          NA |
| LenghtBigHashet              | Job-DISSLE         | .NET 8.0           |         0.0000 ns |      0.0001 ns |       0.0001 ns |         0.0000 ns |          0.000 |         0.00 |      - |         - |          NA |
| LenghtBigEnum                | Job-DISSLE         | .NET 8.0           |   661,041.0095 ns | 12,414.3956 ns |  12,192.5964 ns |   661,278.4180 ns |  7,595,090.687 | 1,066,456.21 |      - |      40 B |          NA |
| TakeCountSmall               | Job-DISSLE         | .NET 8.0           |        16.5871 ns |      0.3065 ns |       0.2560 ns |        16.5038 ns |        189.334 |        31.13 | 0.0038 |      48 B |          NA |
| TakeCountMedium              | Job-DISSLE         | .NET 8.0           |        16.4673 ns |      0.1167 ns |       0.1091 ns |        16.4832 ns |        189.470 |        27.73 | 0.0038 |      48 B |          NA |
| TakeCountBig                 | Job-DISSLE         | .NET 8.0           |        17.4318 ns |      0.1367 ns |       0.1279 ns |        17.4259 ns |        200.547 |        29.32 | 0.0038 |      48 B |          NA |
| TakeCountBigList             | Job-DISSLE         | .NET 8.0           |        12.6229 ns |      0.1297 ns |       0.1083 ns |        12.5937 ns |        143.948 |        22.60 | 0.0038 |      48 B |          NA |
| TakeCountBigHashet           | Job-DISSLE         | .NET 8.0           |        22.1075 ns |      0.2167 ns |       0.2027 ns |        22.1383 ns |        254.360 |        37.33 | 0.0076 |      96 B |          NA |
| TakeCountBigEnum             | Job-DISSLE         | .NET 8.0           |        19.4930 ns |      0.2367 ns |       0.1976 ns |        19.4981 ns |        222.361 |        35.57 | 0.0076 |      96 B |          NA |
| EnumeratorCheckSmall         | Job-DISSLE         | .NET 8.0           |         5.2012 ns |      0.0294 ns |       0.0275 ns |         5.1994 ns |         59.833 |         8.68 |      - |         - |          NA |
| EnumeratorCheckMedium        | Job-DISSLE         | .NET 8.0           |         5.2079 ns |      0.0542 ns |       0.0507 ns |         5.2125 ns |         59.908 |         8.66 |      - |         - |          NA |
| EnumeratorCheckBig           | Job-DISSLE         | .NET 8.0           |         5.4552 ns |      0.1309 ns |       0.1703 ns |         5.4111 ns |         62.725 |         9.20 |      - |         - |          NA |
| EnumeratorCheckBigList       | Job-DISSLE         | .NET 8.0           |         1.6329 ns |      0.0244 ns |       0.0204 ns |         1.6312 ns |         18.626 |         2.96 |      - |         - |          NA |
| EnumeratorCheckBigHashset    | Job-DISSLE         | .NET 8.0           |         1.9001 ns |      0.0196 ns |       0.0183 ns |         1.9052 ns |         21.867 |         3.22 |      - |         - |          NA |
| EnumeratorCheckBigEnumerable | Job-DISSLE         | .NET 8.0           |         9.4113 ns |      0.1107 ns |       0.1036 ns |         9.4083 ns |        108.290 |        15.99 | 0.0032 |      40 B |          NA |
| AnySmall                     | Job-HXWWDC         | .NET Framework 4.8 |         8.9172 ns |      0.0583 ns |       0.0517 ns |         8.9278 ns |        102.437 |        15.10 | 0.0051 |      32 B |          NA |
| AnyMedium                    | Job-HXWWDC         | .NET Framework 4.8 |         8.9023 ns |      0.0359 ns |       0.0336 ns |         8.9003 ns |        102.409 |        14.84 | 0.0051 |      32 B |          NA |
| AnyBig                       | Job-HXWWDC         | .NET Framework 4.8 |         8.7063 ns |      0.0455 ns |       0.0426 ns |         8.6963 ns |        100.147 |        14.48 | 0.0051 |      32 B |          NA |
| AnyBigList                   | Job-HXWWDC         | .NET Framework 4.8 |        10.8967 ns |      0.0991 ns |       0.0927 ns |        10.9160 ns |        125.333 |        18.08 | 0.0064 |      40 B |          NA |
| AnyBigHashet                 | Job-HXWWDC         | .NET Framework 4.8 |        11.5538 ns |      0.0913 ns |       0.0854 ns |        11.5405 ns |        132.825 |        18.59 | 0.0064 |      40 B |          NA |
| AnyBigEnum                   | Job-HXWWDC         | .NET Framework 4.8 |         9.6281 ns |      0.0294 ns |       0.0261 ns |         9.6330 ns |        110.624 |        16.43 | 0.0064 |      40 B |          NA |
| LenghtSmall                  | Job-HXWWDC         | .NET Framework 4.8 |         0.0869 ns |      0.0110 ns |       0.0103 ns |         0.0877 ns |          0.996 |         0.17 |      - |         - |          NA |
| LenghtMedium                 | Job-HXWWDC         | .NET Framework 4.8 |         0.0802 ns |      0.0130 ns |       0.0122 ns |         0.0814 ns |          0.922 |         0.19 |      - |         - |          NA |
| LenghtBig                    | Job-HXWWDC         | .NET Framework 4.8 |         0.0886 ns |      0.0133 ns |       0.0125 ns |         0.0911 ns |          1.000 |         0.00 |      - |         - |          NA |
| LenghtBigList                | Job-HXWWDC         | .NET Framework 4.8 |         0.0913 ns |      0.0096 ns |       0.0090 ns |         0.0921 ns |          1.051 |         0.19 |      - |         - |          NA |
| LenghtBigHashet              | Job-HXWWDC         | .NET Framework 4.8 |         0.0854 ns |      0.0137 ns |       0.0128 ns |         0.0882 ns |          0.977 |         0.17 |      - |         - |          NA |
| LenghtBigEnum                | Job-HXWWDC         | .NET Framework 4.8 | 1,953,247.9427 ns | 20,814.0771 ns |  19,469.5006 ns | 1,957,055.8594 ns | 22,449,133.005 | 3,105,356.20 |      - |      48 B |          NA |
| TakeCountSmall               | Job-HXWWDC         | .NET Framework 4.8 |        37.5990 ns |      0.2007 ns |       0.1878 ns |        37.5965 ns |        432.480 |        62.44 | 0.0153 |      96 B |          NA |
| TakeCountMedium              | Job-HXWWDC         | .NET Framework 4.8 |        37.4735 ns |      0.1562 ns |       0.1385 ns |        37.4845 ns |        430.604 |        64.31 | 0.0153 |      96 B |          NA |
| TakeCountBig                 | Job-HXWWDC         | .NET Framework 4.8 |        36.9597 ns |      0.0989 ns |       0.0877 ns |        36.9657 ns |        424.670 |        63.21 | 0.0153 |      96 B |          NA |
| TakeCountBigList             | Job-HXWWDC         | .NET Framework 4.8 |        40.6202 ns |      0.1217 ns |       0.1079 ns |        40.6351 ns |        466.783 |        69.91 | 0.0166 |     104 B |          NA |
| TakeCountBigHashet           | Job-HXWWDC         | .NET Framework 4.8 |        40.8127 ns |      0.2308 ns |       0.2159 ns |        40.8755 ns |        469.425 |        67.55 | 0.0166 |     104 B |          NA |
| TakeCountBigEnum             | Job-HXWWDC         | .NET Framework 4.8 |        38.1219 ns |      0.2162 ns |       0.1917 ns |        38.1214 ns |        438.071 |        65.63 | 0.0166 |     104 B |          NA |
| EnumeratorCheckSmall         | Job-HXWWDC         | .NET Framework 4.8 |         8.7670 ns |      0.1002 ns |       0.0783 ns |         8.7591 ns |        101.770 |        14.79 | 0.0051 |      32 B |          NA |
| EnumeratorCheckMedium        | Job-HXWWDC         | .NET Framework 4.8 |         8.7582 ns |      0.0310 ns |       0.0275 ns |         8.7665 ns |        100.631 |        14.97 | 0.0051 |      32 B |          NA |
| EnumeratorCheckBig           | Job-HXWWDC         | .NET Framework 4.8 |         8.5148 ns |      0.0384 ns |       0.0359 ns |         8.5116 ns |         97.958 |        14.25 | 0.0051 |      32 B |          NA |
| EnumeratorCheckBigList       | Job-HXWWDC         | .NET Framework 4.8 |        10.8016 ns |      0.0684 ns |       0.0571 ns |        10.7921 ns |        123.103 |        18.80 | 0.0064 |      40 B |          NA |
| EnumeratorCheckBigHashset    | Job-HXWWDC         | .NET Framework 4.8 |        10.8766 ns |      0.1809 ns |       0.2707 ns |        10.7795 ns |        126.852 |        19.70 | 0.0064 |      40 B |          NA |
| EnumeratorCheckBigEnumerable | Job-HXWWDC         | .NET Framework 4.8 |         8.9985 ns |      0.0472 ns |       0.0441 ns |         8.9986 ns |        103.508 |        14.96 | 0.0064 |      40 B |          NA |
| AnySmall                     | .NET 6.0           | .NET 6.0           |         5.2398 ns |      0.0316 ns |       0.0280 ns |         5.2438 ns |         60.210 |         9.03 |      - |         - |          NA |
| AnyMedium                    | .NET 6.0           | .NET 6.0           |         5.9405 ns |      0.0377 ns |       0.0353 ns |         5.9402 ns |         68.323 |         9.80 |      - |         - |          NA |
| AnyBig                       | .NET 6.0           | .NET 6.0           |         5.8831 ns |      0.0301 ns |       0.0267 ns |         5.8836 ns |         67.594 |        10.04 |      - |         - |          NA |
| AnyBigList                   | .NET 6.0           | .NET 6.0           |         3.4517 ns |      0.0233 ns |       0.0195 ns |         3.4458 ns |         39.345 |         6.04 |      - |         - |          NA |
| AnyBigHashet                 | .NET 6.0           | .NET 6.0           |         3.4463 ns |      0.0174 ns |       0.0162 ns |         3.4477 ns |         39.641 |         5.71 |      - |         - |          NA |
| AnyBigEnum                   | .NET 6.0           | .NET 6.0           |        12.9680 ns |      0.0699 ns |       0.0583 ns |        12.9710 ns |        147.827 |        22.83 | 0.0032 |      40 B |          NA |
| LenghtSmall                  | .NET 6.0           | .NET 6.0           |         0.0027 ns |      0.0042 ns |       0.0040 ns |         0.0006 ns |          0.035 |         0.06 |      - |         - |          NA |
| LenghtMedium                 | .NET 6.0           | .NET 6.0           |         0.0035 ns |      0.0038 ns |       0.0033 ns |         0.0025 ns |          0.041 |         0.04 |      - |         - |          NA |
| LenghtBig                    | .NET 6.0           | .NET 6.0           |         0.0054 ns |      0.0054 ns |       0.0050 ns |         0.0044 ns |          0.064 |         0.06 |      - |         - |          NA |
| LenghtBigList                | .NET 6.0           | .NET 6.0           |         0.0026 ns |      0.0041 ns |       0.0036 ns |         0.0011 ns |          0.030 |         0.04 |      - |         - |          NA |
| LenghtBigHashet              | .NET 6.0           | .NET 6.0           |         0.0019 ns |      0.0031 ns |       0.0029 ns |         0.0000 ns |          0.023 |         0.04 |      - |         - |          NA |
| LenghtBigEnum                | .NET 6.0           | .NET 6.0           | 1,876,763.1836 ns | 12,052.7287 ns |  10,684.4300 ns | 1,877,488.6719 ns | 21,573,726.715 | 3,278,017.59 |      - |      41 B |          NA |
| TakeCountSmall               | .NET 6.0           | .NET 6.0           |        24.1002 ns |      0.4935 ns |       0.6060 ns |        23.8477 ns |        277.947 |        39.09 | 0.0038 |      48 B |          NA |
| TakeCountMedium              | .NET 6.0           | .NET 6.0           |        23.0748 ns |      0.1251 ns |       0.1045 ns |        23.0913 ns |        263.125 |        41.21 | 0.0038 |      48 B |          NA |
| TakeCountBig                 | .NET 6.0           | .NET 6.0           |        22.9302 ns |      0.1478 ns |       0.1382 ns |        22.9288 ns |        263.691 |        37.59 | 0.0038 |      48 B |          NA |
| TakeCountBigList             | .NET 6.0           | .NET 6.0           |        18.0401 ns |      0.0805 ns |       0.0714 ns |        18.0156 ns |        207.305 |        31.02 | 0.0038 |      48 B |          NA |
| TakeCountBigHashet           | .NET 6.0           | .NET 6.0           |        27.3599 ns |      0.0931 ns |       0.0778 ns |        27.3778 ns |        311.893 |        48.13 | 0.0076 |      96 B |          NA |
| TakeCountBigEnum             | .NET 6.0           | .NET 6.0           |        23.2108 ns |      0.1167 ns |       0.1092 ns |        23.2047 ns |        266.990 |        38.64 | 0.0076 |      96 B |          NA |
| EnumeratorCheckSmall         | .NET 6.0           | .NET 6.0           |         6.9561 ns |      0.0445 ns |       0.0394 ns |         6.9500 ns |         79.916 |        11.81 |      - |         - |          NA |
| EnumeratorCheckMedium        | .NET 6.0           | .NET 6.0           |         6.9923 ns |      0.0569 ns |       0.0532 ns |         6.9754 ns |         80.464 |        11.89 |      - |         - |          NA |
| EnumeratorCheckBig           | .NET 6.0           | .NET 6.0           |         6.9656 ns |      0.0494 ns |       0.0462 ns |         6.9621 ns |         80.123 |        11.57 |      - |         - |          NA |
| EnumeratorCheckBigList       | .NET 6.0           | .NET 6.0           |         4.1412 ns |      0.0203 ns |       0.0180 ns |         4.1391 ns |         47.601 |         7.22 |      - |         - |          NA |
| EnumeratorCheckBigHashset    | .NET 6.0           | .NET 6.0           |         4.2728 ns |      0.0745 ns |       0.0697 ns |         4.2376 ns |         49.132 |         7.11 |      - |         - |          NA |
| EnumeratorCheckBigEnumerable | .NET 6.0           | .NET 6.0           |        14.2448 ns |      0.1028 ns |       0.0858 ns |        14.2237 ns |        162.303 |        24.45 | 0.0032 |      40 B |          NA |
| AnySmall                     | .NET 8.0           | .NET 8.0           |         5.1925 ns |      0.0959 ns |       0.0850 ns |         5.1694 ns |         59.639 |         8.73 |      - |         - |          NA |
| AnyMedium                    | .NET 8.0           | .NET 8.0           |         5.1217 ns |      0.0265 ns |       0.0234 ns |         5.1206 ns |         58.865 |         8.89 |      - |         - |          NA |
| AnyBig                       | .NET 8.0           | .NET 8.0           |         5.1222 ns |      0.0343 ns |       0.0321 ns |         5.1150 ns |         58.899 |         8.37 |      - |         - |          NA |
| AnyBigList                   | .NET 8.0           | .NET 8.0           |         2.0881 ns |      0.0103 ns |       0.0097 ns |         2.0854 ns |         24.013 |         3.41 |      - |         - |          NA |
| AnyBigHashet                 | .NET 8.0           | .NET 8.0           |         2.0957 ns |      0.0163 ns |       0.0152 ns |         2.0912 ns |         24.096 |         3.39 |      - |         - |          NA |
| AnyBigEnum                   | .NET 8.0           | .NET 8.0           |         9.3060 ns |      0.0794 ns |       0.0704 ns |         9.3341 ns |        106.981 |        16.35 | 0.0032 |      40 B |          NA |
| LenghtSmall                  | .NET 8.0           | .NET 8.0           |         0.0079 ns |      0.0040 ns |       0.0034 ns |         0.0071 ns |          0.088 |         0.04 |      - |         - |          NA |
| LenghtMedium                 | .NET 8.0           | .NET 8.0           |         0.0084 ns |      0.0025 ns |       0.0022 ns |         0.0081 ns |          0.094 |         0.02 |      - |         - |          NA |
| LenghtBig                    | .NET 8.0           | .NET 8.0           |         0.0177 ns |      0.0155 ns |       0.0145 ns |         0.0106 ns |          0.204 |         0.17 |      - |         - |          NA |
| LenghtBigList                | .NET 8.0           | .NET 8.0           |         0.0095 ns |      0.0103 ns |       0.0096 ns |         0.0075 ns |          0.109 |         0.11 |      - |         - |          NA |
| LenghtBigHashet              | .NET 8.0           | .NET 8.0           |         0.0043 ns |      0.0048 ns |       0.0045 ns |         0.0033 ns |          0.051 |         0.06 |      - |         - |          NA |
| LenghtBigEnum                | .NET 8.0           | .NET 8.0           |   583,897.4870 ns |  5,331.3646 ns |   4,986.9617 ns |   583,399.2188 ns |  6,710,928.141 |   927,062.79 |      - |      40 B |          NA |
| TakeCountSmall               | .NET 8.0           | .NET 8.0           |        15.3626 ns |      0.1663 ns |       0.1555 ns |        15.4056 ns |        176.801 |        26.32 | 0.0038 |      48 B |          NA |
| TakeCountMedium              | .NET 8.0           | .NET 8.0           |        15.2003 ns |      0.1484 ns |       0.1316 ns |        15.2219 ns |        174.734 |        26.66 | 0.0038 |      48 B |          NA |
| TakeCountBig                 | .NET 8.0           | .NET 8.0           |        15.3505 ns |      0.1845 ns |       0.1726 ns |        15.3659 ns |        176.511 |        25.09 | 0.0038 |      48 B |          NA |
| TakeCountBigList             | .NET 8.0           | .NET 8.0           |        11.5651 ns |      0.0883 ns |       0.0783 ns |        11.5721 ns |        132.924 |        20.10 | 0.0038 |      48 B |          NA |
| TakeCountBigHashet           | .NET 8.0           | .NET 8.0           |        20.1079 ns |      0.2957 ns |       0.2621 ns |        20.0648 ns |        231.013 |        34.43 | 0.0076 |      96 B |          NA |
| TakeCountBigEnum             | .NET 8.0           | .NET 8.0           |        18.0136 ns |      0.1230 ns |       0.1090 ns |        18.0617 ns |        206.957 |        30.65 | 0.0076 |      96 B |          NA |
| EnumeratorCheckSmall         | .NET 8.0           | .NET 8.0           |         5.1381 ns |      0.0116 ns |       0.0097 ns |         5.1419 ns |         58.570 |         9.03 |      - |         - |          NA |
| EnumeratorCheckMedium        | .NET 8.0           | .NET 8.0           |         5.1231 ns |      0.0325 ns |       0.0288 ns |         5.1211 ns |         58.861 |         8.73 |      - |         - |          NA |
| EnumeratorCheckBig           | .NET 8.0           | .NET 8.0           |         5.1499 ns |      0.0195 ns |       0.0182 ns |         5.1531 ns |         59.236 |         8.54 |      - |         - |          NA |
| EnumeratorCheckBigList       | .NET 8.0           | .NET 8.0           |         1.7404 ns |      0.0547 ns |       0.0511 ns |         1.7427 ns |         20.000 |         2.83 |      - |         - |          NA |
| EnumeratorCheckBigHashset    | .NET 8.0           | .NET 8.0           |         1.9157 ns |      0.0459 ns |       0.0430 ns |         1.9023 ns |         22.025 |         3.18 |      - |         - |          NA |
| EnumeratorCheckBigEnumerable | .NET 8.0           | .NET 8.0           |         9.5219 ns |      0.2065 ns |       0.1612 ns |         9.4902 ns |        110.520 |        15.98 | 0.0032 |      40 B |          NA |
| AnySmall                     | .NET Framework 4.8 | .NET Framework 4.8 |         8.8293 ns |      0.1976 ns |       0.3355 ns |         8.7041 ns |        102.096 |        14.83 | 0.0051 |      32 B |          NA |
| AnyMedium                    | .NET Framework 4.8 | .NET Framework 4.8 |         8.4963 ns |      0.1847 ns |       0.2053 ns |         8.4157 ns |         97.913 |        13.33 | 0.0051 |      32 B |          NA |
| AnyBig                       | .NET Framework 4.8 | .NET Framework 4.8 |         8.2408 ns |      0.0589 ns |       0.0551 ns |         8.2385 ns |         94.792 |        13.68 | 0.0051 |      32 B |          NA |
| AnyBigList                   | .NET Framework 4.8 | .NET Framework 4.8 |        10.4312 ns |      0.0862 ns |       0.0764 ns |        10.4310 ns |        119.902 |        18.24 | 0.0064 |      40 B |          NA |
| AnyBigHashet                 | .NET Framework 4.8 | .NET Framework 4.8 |        10.8548 ns |      0.0556 ns |       0.0464 ns |        10.8447 ns |        123.719 |        18.92 | 0.0064 |      40 B |          NA |
| AnyBigEnum                   | .NET Framework 4.8 | .NET Framework 4.8 |         9.3675 ns |      0.0479 ns |       0.0425 ns |         9.3642 ns |        107.658 |        16.20 | 0.0064 |      40 B |          NA |
| LenghtSmall                  | .NET Framework 4.8 | .NET Framework 4.8 |         0.0008 ns |      0.0027 ns |       0.0023 ns |         0.0000 ns |          0.010 |         0.03 |      - |         - |          NA |
| LenghtMedium                 | .NET Framework 4.8 | .NET Framework 4.8 |         0.0060 ns |      0.0058 ns |       0.0054 ns |         0.0064 ns |          0.069 |         0.06 |      - |         - |          NA |
| LenghtBig                    | .NET Framework 4.8 | .NET Framework 4.8 |         0.0025 ns |      0.0040 ns |       0.0035 ns |         0.0002 ns |          0.029 |         0.04 |      - |         - |          NA |
| LenghtBigList                | .NET Framework 4.8 | .NET Framework 4.8 |         0.0131 ns |      0.0206 ns |       0.0161 ns |         0.0094 ns |          0.153 |         0.18 |      - |         - |          NA |
| LenghtBigHashet              | .NET Framework 4.8 | .NET Framework 4.8 |         0.0117 ns |      0.0131 ns |       0.0207 ns |         0.0000 ns |          0.170 |         0.30 |      - |         - |          NA |
| LenghtBigEnum                | .NET Framework 4.8 | .NET Framework 4.8 | 2,082,435.3046 ns | 41,349.8025 ns | 107,473.6346 ns | 2,110,085.1562 ns | 23,815,359.344 | 4,121,613.89 |      - |      64 B |          NA |
| TakeCountSmall               | .NET Framework 4.8 | .NET Framework 4.8 |        46.9507 ns |      2.4063 ns |       7.0949 ns |        49.4569 ns |        597.780 |       105.89 | 0.0153 |      96 B |          NA |
| TakeCountMedium              | .NET Framework 4.8 | .NET Framework 4.8 |        41.8135 ns |      2.2990 ns |       6.7788 ns |        37.6470 ns |        518.330 |        81.82 | 0.0153 |      96 B |          NA |
| TakeCountBig                 | .NET Framework 4.8 | .NET Framework 4.8 |        35.4960 ns |      0.6845 ns |       0.6068 ns |        35.3374 ns |        407.777 |        60.11 | 0.0153 |      96 B |          NA |
| TakeCountBigList             | .NET Framework 4.8 | .NET Framework 4.8 |        38.4373 ns |      0.5809 ns |       0.5434 ns |        38.1308 ns |        441.967 |        62.86 | 0.0166 |     104 B |          NA |
| TakeCountBigHashet           | .NET Framework 4.8 | .NET Framework 4.8 |        38.2684 ns |      0.2804 ns |       0.2486 ns |        38.1903 ns |        439.749 |        65.73 | 0.0166 |     104 B |          NA |
| TakeCountBigEnum             | .NET Framework 4.8 | .NET Framework 4.8 |        36.3923 ns |      0.1373 ns |       0.1217 ns |        36.4024 ns |        418.172 |        62.42 | 0.0166 |     104 B |          NA |
| EnumeratorCheckSmall         | .NET Framework 4.8 | .NET Framework 4.8 |         8.1673 ns |      0.0523 ns |       0.0464 ns |         8.1634 ns |         93.856 |        14.09 | 0.0051 |      32 B |          NA |
| EnumeratorCheckMedium        | .NET Framework 4.8 | .NET Framework 4.8 |         8.1260 ns |      0.0368 ns |       0.0308 ns |         8.1328 ns |         92.623 |        14.20 | 0.0051 |      32 B |          NA |
| EnumeratorCheckBig           | .NET Framework 4.8 | .NET Framework 4.8 |         7.9763 ns |      0.0262 ns |       0.0245 ns |         7.9818 ns |         91.736 |        13.14 | 0.0051 |      32 B |          NA |
| EnumeratorCheckBigList       | .NET Framework 4.8 | .NET Framework 4.8 |        10.4208 ns |      0.2269 ns |       0.3852 ns |        10.2657 ns |        119.707 |        18.83 | 0.0064 |      40 B |          NA |
| EnumeratorCheckBigHashset    | .NET Framework 4.8 | .NET Framework 4.8 |        10.8562 ns |      0.1353 ns |       0.1200 ns |        10.8268 ns |        124.759 |        18.83 | 0.0064 |      40 B |          NA |
| EnumeratorCheckBigEnumerable | .NET Framework 4.8 | .NET Framework 4.8 |         9.3627 ns |      0.0682 ns |       0.0605 ns |         9.3436 ns |        107.560 |        15.90 | 0.0064 |      40 B |          NA |
