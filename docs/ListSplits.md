# ListSplit Benchmarkz

# Benchmark 1: ChunkOfSizeT
The first implementation doesnt preallocate a List and thus yields an overhead as the array has to resized way often.

## Code
```csharp
    public static IEnumerable<List<T>> ToChunksOfMaxSizeN<T>(this IEnumerable<T> toChunk, int maxSize)
    {
        if (maxSize <= 0)
            throw new InvalidEnumArgumentException($"{nameof(maxSize)} must be greater than 0");

        var chunk = new List<T>();
        foreach (var o in toChunk)
        {
            chunk.Add(o);
            if (chunk.Count != maxSize)
                continue;

            yield return chunk;
            chunk = [];
        }

        if (chunk.Any())
            yield return chunk;
    }
```

# Benchmark 2: List.Split()
The second implementation is a tad uglier but yields some micro optimizations by buffering the checks and resizing the array that is to be returned on demand

## Code
```csharp
internal static IEnumerable<TSource[]> Split<TSource>(this IEnumerable<TSource> source,
                                                          int size)
    {
        using var enumerator = source.GetEnumerator();

        if (!enumerator.MoveNext())
            yield break;

        var arrayBufferSize = Math.Min(size, 4);

        uint chunkIndex;
        do
        {
            var array = new TSource[arrayBufferSize];
            array[0] = enumerator.Current;
            chunkIndex = 1;

            if (size != array.Length)
            {
                while (chunkIndex < size && enumerator.MoveNext())
                {
                    // Manual resizings still more slim than lists dynamic allocation
                    if (chunkIndex >= array.Length)
                    {
                        arrayBufferSize = Math.Min(size, 2 * array.Length);
                        Array.Resize(ref array, arrayBufferSize);
                    }

                    PushToChunk(array);
                }
            }
            else
            {
                // ReSharper disable once InlineTemporaryVariable -- Suppresses bounding checks ( not required as resized above )
                var cache = array;
                while (chunkIndex < cache.Length && enumerator.MoveNext())
                    PushToChunk(array);
            }

            // Reduces the ( possible ) fragmentation of the last chunky
            if (chunkIndex != array.Length)
                Array.Resize(ref array, (int)chunkIndex);

            yield return array;
        }
        while (chunkIndex >= size && enumerator.MoveNext());
        void PushToChunk(TSource[] array)
        {
            array[chunkIndex] = enumerator.Current;
            chunkIndex += 1;
        }
    }
```

# Results
```
// * Summary *                                                                                                                                                                                                                                                                                                                                                                          BenchmarkDotNet v0.13.12, Windows 10 (10.0.19045.4046/22H2/2022Update)                                                                                                                      12th Gen Intel Core i9-12900H, 1 CPU, 20 logical and 14 physical cores                                                                                                                      .NET SDK 8.0.200                                                                                                                                                                              [Host]             : .NET 8.0.2 (8.0.224.6711), X64 RyuJIT AVX2                                                                                                                             .NET 6.0           : .NET 6.0.27 (6.0.2724.6912), X64 RyuJIT AVX2                                                                                                                           .NET 8.0           : .NET 8.0.2 (8.0.224.6711), X64 RyuJIT AVX2                                                                                                                             .NET Framework 4.8 : .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256
```

| Method   | Job                | Runtime            | ListSize | Mean             | Error          | StdDev           | Median           | Gen0      | Gen1      | Gen2     | Allocated  |
|--------- |------------------- |------------------- |--------- |-----------------:|---------------:|-----------------:|-----------------:|----------:|----------:|---------:|-----------:|
| Split10  | .NET 6.0           | .NET 6.0           | 0        |         34.78 ns |       0.744 ns |         0.796 ns |         35.05 ns |    0.0095 |         - |        - |      120 B |
| Split100 | .NET 6.0           | .NET 6.0           | 0        |         36.83 ns |       0.130 ns |         0.122 ns |         36.79 ns |    0.0095 |         - |        - |      120 B |
| Chunk10  | .NET 6.0           | .NET 6.0           | 0        |         43.80 ns |       0.265 ns |         0.247 ns |         43.85 ns |    0.0108 |         - |        - |      136 B |
| Chunk100 | .NET 6.0           | .NET 6.0           | 0        |         43.20 ns |       0.122 ns |         0.108 ns |         43.19 ns |    0.0108 |         - |        - |      136 B |
| Split10  | .NET 8.0           | .NET 8.0           | 0        |         27.15 ns |       0.180 ns |         0.159 ns |         27.16 ns |    0.0095 |         - |        - |      120 B |
| Split100 | .NET 8.0           | .NET 8.0           | 0        |         27.46 ns |       0.070 ns |         0.062 ns |         27.44 ns |    0.0095 |         - |        - |      120 B |
| Chunk10  | .NET 8.0           | .NET 8.0           | 0        |         28.02 ns |       0.183 ns |         0.171 ns |         27.97 ns |    0.0108 |         - |        - |      136 B |
| Chunk100 | .NET 8.0           | .NET 8.0           | 0        |         28.02 ns |       0.153 ns |         0.136 ns |         28.00 ns |    0.0108 |         - |        - |      136 B |
| Split10  | .NET Framework 4.8 | .NET Framework 4.8 | 0        |         42.41 ns |       0.089 ns |         0.074 ns |         42.42 ns |    0.0229 |         - |        - |      144 B |
| Split100 | .NET Framework 4.8 | .NET Framework 4.8 | 0        |         42.31 ns |       0.056 ns |         0.047 ns |         42.31 ns |    0.0229 |         - |        - |      144 B |
| Chunk10  | .NET Framework 4.8 | .NET Framework 4.8 | 0        |         59.18 ns |       0.095 ns |         0.084 ns |         59.18 ns |    0.0331 |         - |        - |      209 B |
| Chunk100 | .NET Framework 4.8 | .NET Framework 4.8 | 0        |         59.27 ns |       0.105 ns |         0.098 ns |         59.27 ns |    0.0331 |         - |        - |      209 B |
| Split10  | .NET 6.0           | .NET 6.0           | 1        |         72.73 ns |       0.849 ns |         0.753 ns |         72.67 ns |    0.0223 |         - |        - |      280 B |
| Split100 | .NET 6.0           | .NET 6.0           | 1        |         73.64 ns |       0.234 ns |         0.219 ns |         73.65 ns |    0.0223 |         - |        - |      280 B |
| Chunk10  | .NET 6.0           | .NET 6.0           | 1        |         79.46 ns |       0.268 ns |         0.250 ns |         79.49 ns |    0.0210 |         - |        - |      264 B |
| Chunk100 | .NET 6.0           | .NET 6.0           | 1        |         79.57 ns |       0.225 ns |         0.199 ns |         79.57 ns |    0.0210 |         - |        - |      264 B |
| Split10  | .NET 8.0           | .NET 8.0           | 1        |         59.04 ns |       0.286 ns |         0.267 ns |         59.04 ns |    0.0223 |         - |        - |      280 B |
| Split100 | .NET 8.0           | .NET 8.0           | 1        |         58.40 ns |       0.222 ns |         0.185 ns |         58.42 ns |    0.0223 |         - |        - |      280 B |
| Chunk10  | .NET 8.0           | .NET 8.0           | 1        |         61.66 ns |       0.254 ns |         0.237 ns |         61.60 ns |    0.0210 |         - |        - |      264 B |
| Chunk100 | .NET 8.0           | .NET 8.0           | 1        |         61.71 ns |       0.165 ns |         0.154 ns |         61.72 ns |    0.0210 |         - |        - |      264 B |
| Split10  | .NET Framework 4.8 | .NET Framework 4.8 | 1        |         82.35 ns |       0.184 ns |         0.163 ns |         82.31 ns |    0.0446 |         - |        - |      281 B |
| Split100 | .NET Framework 4.8 | .NET Framework 4.8 | 1        |         82.35 ns |       0.232 ns |         0.217 ns |         82.30 ns |    0.0446 |         - |        - |      281 B |
| Chunk10  | .NET Framework 4.8 | .NET Framework 4.8 | 1        |         86.41 ns |       0.195 ns |         0.163 ns |         86.41 ns |    0.0497 |         - |        - |      313 B |
| Chunk100 | .NET Framework 4.8 | .NET Framework 4.8 | 1        |         86.59 ns |       0.361 ns |         0.320 ns |         86.55 ns |    0.0497 |         - |        - |      313 B |
| Split10  | .NET 6.0           | .NET 6.0           | 20       |        168.03 ns |       0.511 ns |         0.478 ns |        168.10 ns |    0.0350 |         - |        - |      440 B |
| Split100 | .NET 6.0           | .NET 6.0           | 20       |        174.85 ns |       0.596 ns |         0.557 ns |        174.64 ns |    0.0515 |         - |        - |      648 B |
| Chunk10  | .NET 6.0           | .NET 6.0           | 20       |        221.65 ns |       0.825 ns |         0.772 ns |        221.69 ns |    0.0527 |         - |        - |      664 B |
| Chunk100 | .NET 6.0           | .NET 6.0           | 20       |        187.68 ns |       0.618 ns |         0.548 ns |        187.51 ns |    0.0446 |         - |        - |      560 B |
| Split10  | .NET 8.0           | .NET 8.0           | 20       |        112.32 ns |       0.479 ns |         0.448 ns |        112.24 ns |    0.0350 |         - |        - |      440 B |
| Split100 | .NET 8.0           | .NET 8.0           | 20       |        127.57 ns |       0.407 ns |         0.380 ns |        127.45 ns |    0.0515 |         - |        - |      648 B |
| Chunk10  | .NET 8.0           | .NET 8.0           | 20       |        159.80 ns |       0.635 ns |         0.594 ns |        159.78 ns |    0.0527 |         - |        - |      664 B |
| Chunk100 | .NET 8.0           | .NET 8.0           | 20       |        136.64 ns |       0.600 ns |         0.532 ns |        136.61 ns |    0.0446 |         - |        - |      560 B |
| Split10  | .NET Framework 4.8 | .NET Framework 4.8 | 20       |        184.13 ns |       0.570 ns |         0.505 ns |        184.26 ns |    0.0701 |         - |        - |      441 B |
| Split100 | .NET Framework 4.8 | .NET Framework 4.8 | 20       |        197.53 ns |       0.915 ns |         0.714 ns |        197.50 ns |    0.1032 |         - |        - |      650 B |
| Chunk10  | .NET Framework 4.8 | .NET Framework 4.8 | 20       |        254.25 ns |       0.682 ns |         0.605 ns |        254.15 ns |    0.1159 |         - |        - |      730 B |
| Chunk100 | .NET Framework 4.8 | .NET Framework 4.8 | 20       |        216.60 ns |       0.579 ns |         0.542 ns |        216.73 ns |    0.0968 |         - |        - |      610 B |
| Split10  | .NET 6.0           | .NET 6.0           | 1000     |      4,659.30 ns |       7.892 ns |         6.996 ns |      4,657.81 ns |    0.6866 |    0.0076 |        - |     8696 B |
| Split100 | .NET 6.0           | .NET 6.0           | 1000     |      3,730.50 ns |      13.624 ns |        12.744 ns |      3,729.67 ns |    0.4196 |    0.0038 |        - |     5312 B |
| Chunk10  | .NET 6.0           | .NET 6.0           | 1000     |      7,879.82 ns |      38.476 ns |        35.991 ns |      7,874.00 ns |    1.8921 |    0.0458 |        - |    23816 B |
| Chunk100 | .NET 6.0           | .NET 6.0           | 1000     |      4,705.97 ns |      21.447 ns |        19.012 ns |      4,705.78 ns |    0.9766 |    0.0153 |        - |    12312 B |
| Split10  | .NET 8.0           | .NET 8.0           | 1000     |      2,938.31 ns |      11.088 ns |        10.372 ns |      2,939.48 ns |    0.6905 |    0.0114 |        - |     8696 B |
| Split100 | .NET 8.0           | .NET 8.0           | 1000     |      2,025.02 ns |       3.930 ns |         3.484 ns |      2,024.77 ns |    0.4196 |    0.0038 |        - |     5312 B |
| Chunk10  | .NET 8.0           | .NET 8.0           | 1000     |      5,583.41 ns |      19.309 ns |        17.117 ns |      5,585.56 ns |    1.8921 |    0.0610 |        - |    23816 B |
| Chunk100 | .NET 8.0           | .NET 8.0           | 1000     |      3,116.25 ns |       7.714 ns |         6.839 ns |      3,116.61 ns |    0.9804 |    0.0114 |        - |    12312 B |
| Split10  | .NET Framework 4.8 | .NET Framework 4.8 | 1000     |      5,353.31 ns |      17.838 ns |        16.686 ns |      5,350.57 ns |    1.5259 |    0.0229 |        - |     9628 B |
| Split100 | .NET Framework 4.8 | .NET Framework 4.8 | 1000     |      3,964.31 ns |      10.418 ns |         9.745 ns |      3,963.76 ns |    0.8545 |    0.0076 |        - |     5392 B |
| Chunk10  | .NET Framework 4.8 | .NET Framework 4.8 | 1000     |      8,958.04 ns |      18.071 ns |        16.019 ns |      8,957.69 ns |    4.0741 |    0.1221 |        - |    25643 B |
| Chunk100 | .NET Framework 4.8 | .NET Framework 4.8 | 1000     |      5,587.55 ns |      10.221 ns |         9.561 ns |      5,587.37 ns |    1.9913 |    0.0305 |        - |    12541 B |
| Split10  | .NET 6.0           | .NET 6.0           | 10000    |     44,285.11 ns |     197.826 ns |       185.047 ns |     44,257.20 ns |    6.4087 |    0.7324 |        - |    80824 B |
| Split100 | .NET 6.0           | .NET 6.0           | 10000    |     34,991.80 ns |      80.992 ns |        75.760 ns |     34,968.03 ns |    3.6011 |    0.3052 |        - |    45216 B |
| Chunk10  | .NET 6.0           | .NET 6.0           | 10000    |     76,120.59 ns |     284.533 ns |       252.231 ns |     76,194.53 ns |   18.4326 |    3.4180 |        - |   232744 B |
| Chunk100 | .NET 6.0           | .NET 6.0           | 10000    |     45,649.65 ns |     301.484 ns |       251.753 ns |     45,585.16 ns |    9.5825 |    1.1597 |        - |   120616 B |
| Split10  | .NET 8.0           | .NET 8.0           | 10000    |     27,637.12 ns |     115.936 ns |       108.446 ns |     27,680.14 ns |    6.4392 |    0.9155 |        - |    80824 B |
| Split100 | .NET 8.0           | .NET 8.0           | 10000    |     17,540.92 ns |      79.912 ns |        70.840 ns |     17,526.46 ns |    3.6011 |    0.3357 |        - |    45216 B |
| Chunk10  | .NET 8.0           | .NET 8.0           | 10000    |     51,725.74 ns |     207.773 ns |       184.186 ns |     51,727.44 ns |   18.4937 |    4.3945 |        - |   232744 B |
| Chunk100 | .NET 8.0           | .NET 8.0           | 10000    |     28,468.41 ns |      78.064 ns |        73.022 ns |     28,478.74 ns |    9.6130 |    1.2207 |        - |   120616 B |
| Split10  | .NET Framework 4.8 | .NET Framework 4.8 | 10000    |     52,063.23 ns |     589.596 ns |       522.662 ns |     51,859.29 ns |   14.0991 |    1.5259 |        - |    89070 B |
| Split100 | .NET Framework 4.8 | .NET Framework 4.8 | 10000    |     37,361.94 ns |     113.097 ns |       100.257 ns |     37,354.12 ns |    7.3242 |    0.5493 |        - |    46258 B |
| Chunk10  | .NET Framework 4.8 | .NET Framework 4.8 | 10000    |     89,433.33 ns |     354.915 ns |       277.095 ns |     89,371.21 ns |   39.5508 |    8.7891 |        - |   249509 B |
| Chunk100 | .NET Framework 4.8 | .NET Framework 4.8 | 10000    |     54,527.73 ns |     106.232 ns |        88.708 ns |     54,521.85 ns |   19.4702 |    2.1973 |        - |   122730 B |
| Split10  | .NET 6.0           | .NET 6.0           | 1000000  |  5,533,921.21 ns |  22,337.859 ns |    19,801.930 ns |  5,538,833.98 ns |  664.0625 |  492.1875 | 492.1875 |  8249699 B |
| Split100 | .NET 6.0           | .NET 6.0           | 1000000  |  3,607,778.98 ns |  10,117.101 ns |     9,463.543 ns |  3,606,839.45 ns |  351.5625 |  160.1563 |        - |  4452475 B |
| Chunk10  | .NET 6.0           | .NET 6.0           | 1000000  | 22,527,238.86 ns | 447,673.764 ns | 1,194,931.902 ns | 22,096,309.38 ns | 2093.7500 | 1281.2500 | 500.0000 | 23449660 B |
| Chunk100 | .NET 6.0           | .NET 6.0           | 1000000  |  5,255,326.38 ns |  26,249.182 ns |    21,919.255 ns |  5,251,902.34 ns |  953.1250 |  468.7500 |        - | 12051877 B |
| Split10  | .NET 8.0           | .NET 8.0           | 1000000  |  4,054,003.67 ns |  58,829.348 ns |    49,125.170 ns |  4,052,357.81 ns |  664.0625 |  492.1875 | 492.1875 |  8249696 B |
| Split100 | .NET 8.0           | .NET 8.0           | 1000000  |  1,792,539.66 ns |  13,426.802 ns |    11,902.511 ns |  1,791,585.25 ns |  353.5156 |  320.3125 |        - |  4452473 B |
| Chunk10  | .NET 8.0           | .NET 8.0           | 1000000  | 24,518,900.84 ns | 565,878.965 ns | 1,668,506.897 ns | 23,946,503.12 ns | 2312.5000 | 2281.2500 | 843.7500 | 23449754 B |
| Chunk100 | .NET 8.0           | .NET 8.0           | 1000000  |  3,338,952.19 ns |  15,577.896 ns |    13,008.249 ns |  3,333,424.61 ns |  957.0313 |  898.4375 |        - | 12051874 B |
| Split10  | .NET Framework 4.8 | .NET Framework 4.8 | 1000000  | 11,028,185.09 ns | 366,288.327 ns | 1,080,009.398 ns | 10,865,642.19 ns | 1437.5000 |  984.3750 | 671.8750 |  9319568 B |
| Split100 | .NET Framework 4.8 | .NET Framework 4.8 | 1000000  |  6,826,392.43 ns | 129,384.711 ns |   127,073.086 ns |  6,811,515.62 ns |  812.5000 |  375.0000 | 125.0000 |  4596194 B |
| Chunk10  | .NET Framework 4.8 | .NET Framework 4.8 | 1000000  | 34,659,248.56 ns | 680,690.093 ns |   931,736.282 ns | 34,303,443.75 ns | 4062.5000 | 1812.5000 | 750.0000 | 25372642 B |
| Chunk100 | .NET Framework 4.8 | .NET Framework 4.8 | 1000000  | 11,817,508.54 ns | 207,145.941 ns |   193,764.442 ns | 11,794,673.44 ns | 2187.5000 | 1000.0000 | 468.7500 | 12310156 B |


## Analyzing stuff
Well its obvious.
Less allocation and faster using the split thingy



# Testcode
<details><summary>Full SourceCode</summary>
```csharp
using System.ComponentModel;
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Jobs;

namespace Benchmarkz;

[SimpleJob(RuntimeMoniker.Net48)]
[SimpleJob(RuntimeMoniker.Net60)]
[SimpleJob(RuntimeMoniker.Net80)]
[MemoryDiagnoser]
public class ChunkBenchmarks
{
    private IEnumerable<uint> dataSet = Enumerable.Empty<uint>();

    [Params(0, 1, 20, 1000, 10000, 1000000)]
    public uint ListSize = 0;

    [GlobalSetup]
    public void Setup() => dataSet = FillArray(ListSize);

    [Benchmark]
    public void Split10() => _ = dataSet.Split(10).ToArray();
    [Benchmark]
    public void Split100() => _ = dataSet.Split(100).ToArray();
 
    [Benchmark]
    public void Chunk10() => _ = dataSet.ToChunksOfMaxSizeN(10).ToArray();
    [Benchmark]
    public void Chunk100() => _ = dataSet.ToChunksOfMaxSizeN(100).ToArray();
 
    private static IEnumerable<uint> FillArray(uint count)
    {
        for (uint i = 0; i < count; i++)
            yield return i;
    }
}

internal static class SplitExtensions
{
    internal static IEnumerable<TSource[]> Split<TSource>(this IEnumerable<TSource> source,
                                                          int size)
    {
        using var enumerator = source.GetEnumerator();

        if (!enumerator.MoveNext())
            yield break;

        var arrayBufferSize = Math.Min(size, 4);

        uint chunkIndex;
        do
        {
            var array = new TSource[arrayBufferSize];
            array[0] = enumerator.Current;
            chunkIndex = 1;

            if (size != array.Length)
            {
                while (chunkIndex < size && enumerator.MoveNext())
                {
                    // Manual resizings still more slim than lists dynamic allocation
                    if (chunkIndex >= array.Length)
                    {
                        arrayBufferSize = Math.Min(size, 2 * array.Length);
                        Array.Resize(ref array, arrayBufferSize);
                    }

                    PushToChunk(array);
                }
            }
            else
            {
                // ReSharper disable once InlineTemporaryVariable -- Suppresses bounding checks ( not required as resized above )
                var cache = array;
                while (chunkIndex < cache.Length && enumerator.MoveNext())
                    PushToChunk(array);
            }

            // Reduces the ( possible ) fragmentation of the last chunky 
            if (chunkIndex != array.Length)
                Array.Resize(ref array, (int)chunkIndex);

            yield return array;
        }
        while (chunkIndex >= size && enumerator.MoveNext());

        void PushToChunk(TSource[] array)
        {
            array[chunkIndex] = enumerator.Current;
            chunkIndex += 1;
        }
    }
    
    public static IEnumerable<List<T>> ToChunksOfMaxSizeN<T>(this IEnumerable<T> toChunk, int maxSize)
    {
        if (maxSize <= 0)
            throw new InvalidEnumArgumentException($"{nameof(maxSize)} must be greater than 0");

        var chunk = new List<T>();
        foreach (var o in toChunk)
        {
            chunk.Add(o);
            if (chunk.Count != maxSize)
                continue;
                
            yield return chunk;
            chunk = [];
        }

        if (chunk.Any())
            yield return chunk;
    }
}

```

</details>
