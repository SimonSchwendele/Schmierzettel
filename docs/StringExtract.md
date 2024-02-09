# String Extraction Benchmarkz

The purpose of this tests was to find out the fastest way to extract parts of a string, in this case the keyvalue pairs 
separated by ';' inside of one string.

All Tests are performed with the data:

* "param:dt=123"
* "param:dt=123; bla=blub; stichdatum=01.01.2019
* "param:dt=123; bla=blub; stichdatum=01.01.2019; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub"

# Benchmark 1: Spliting the string
The first test was performedby spliting the string and manually parsing those parts.  
This is the approach seen quite often however is probably the worst one of those as I expect those two `.Split()` to hurt us badly

## Code
```csharp
    public static IEnumerable<ParamField> ExtractParamFieldSplit(string? inputString)
    {
        if (inputString is not { Length: > 6 })
            return Enumerable.Empty<ParamField>();

        var paramindex = inputString.IndexOf("param:",
                                             StringComparison.InvariantCultureIgnoreCase);
        if (paramindex == -1)
            return Enumerable.Empty<ParamField>();

        var splits = inputString[(paramindex + 6)..].Split(';');
        return splits.Select(x =>
        {
            var value = x.Split('=');
            return new ParamField
            {
                Key = value[0].Trim(),
                Value = value[1].Trim()
            };
        });
    }
```

# Benchmark 2: Using Spans and Slices
While it's not that fast using net48 there should be a smiliar performance for this small set of data with a way better performance the huger the set grows while using spans and ranges.  
While there might not be a real performance benefit we'd expect at least some savings due to the reduced allocation.

## Code
```csharp
    public static IEnumerable<ParamField> ExtractParamFieldSpan(string? inputString)
    {
        if (inputString is not { Length: > 6 })
            return Enumerable.Empty<ParamField>();

        var identifierIndex = inputString.IndexOf("param:",
                                                  StringComparison.InvariantCultureIgnoreCase);
        if (identifierIndex == -1)
            return Enumerable.Empty<ParamField>();

        var inputAsSpan = inputString.AsSpan();
        var startIndex = identifierIndex + 6;

        var endOfFirstPair = inputAsSpan.IndexOf(';');
        var pair = endOfFirstPair > 1
            ? inputAsSpan.Slice(startIndex,
                                endOfFirstPair - startIndex)
            : inputAsSpan[startIndex..];

        var separatorIndex = pair.IndexOf('=');
        var results = new List<ParamField>();
        while (separatorIndex != -1)
        {
            results.Add(new ParamField
            {
                Key = pair[..separatorIndex].Trim().ToString(),
                Value = pair[(separatorIndex + 1)..].Trim().ToString()
            });

            pair = pair[(separatorIndex + 1)..];
            separatorIndex = pair.IndexOf('=');
        }

        return results;
    }
```

# Benchmark 3: Using a Regex
While I'd argue that the code itself should be quite readable deciphering a regex sucks really bad.
It's however undeniable that using a regex in those cases most often performs best when used correctly and is less error prone.  
Apart from that the allocation should be at about the same level as using spans

## Code
This regex here however is pretty tiny and just consist of 2 capturing groups, the `=` separator, and a non-capturing group
* `(\w+)`Any Alphanumerics to serve as key
* `=` Separates Key and Value
* `(.*?)` Well anything to be used as value
* until `(?:;|$)` which is not captured and observes if a `;` separates a new key value pair or the end of the input is reached


```csharp
    private static readonly Regex regexRulez = new(@"(\w+)=(.*?)(?:;|$)", RegexOptions.Compiled);
    public static IEnumerable<ParamField> ExtractParamFieldRegex(string? inputString)
    {
        if (inputString is not { Length: > 6 })
            return Enumerable.Empty<ParamField>();

        var matches = regexRulez.Matches(inputString);

        return matches.Cast<Match>()
                      .Select(match => new ParamField
                      {
                          Key = match.Groups[1].Value.Trim(),
                          Value = match.Groups[2].Value.Trim()
                      });
    }

```


# Results
```
BenchmarkDotNet v0.13.12, Windows 10 (10.0.19045.3930/22H2/2022Update)
12th Gen Intel Core i9-12900H, 1 CPU, 20 logical and 14 physical cores
.NET SDK 8.0.101
[Host]             : .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2
.NET 6.0           : .NET 6.0.26 (6.0.2623.60508), X64 RyuJIT AVX2
.NET 8.0           : .NET 8.0.1 (8.0.123.58001), X64 RyuJIT AVX2
.NET Framework 4.8 : .NET Framework 4.8 (4.8.4645.0), X64 RyuJIT VectorSize=256
```


| Method             | Job                | Runtime            |      Mean |    Error |   StdDev |   Gen0 |   Gen1 | Allocated |
|--------------------|--------------------|--------------------|----------:|---------:|---------:|-------:|-------:|----------:|
| SplitExtractSingle | .NET 6.0           | .NET 6.0           |  48.98 ns | 0.568 ns | 0.532 ns | 0.0095 |      - |     120 B |
| SplitExtractThree  | .NET 6.0           | .NET 6.0           |  85.61 ns | 1.679 ns | 3.070 ns | 0.0280 |      - |     352 B |
| SplitExtractTwenty | .NET 6.0           | .NET 6.0           | 246.69 ns | 1.309 ns | 1.161 ns | 0.1197 | 0.0005 |    1504 B |
| SpanExtractSingle  | .NET 6.0           | .NET 6.0           |  64.67 ns | 1.321 ns | 1.852 ns | 0.0147 |      - |     184 B |
| SpanExtractThree   | .NET 6.0           | .NET 6.0           |  62.77 ns | 1.208 ns | 1.342 ns | 0.0147 |      - |     184 B |
| SpanExtractTwenty  | .NET 6.0           | .NET 6.0           |  62.30 ns | 1.237 ns | 1.270 ns | 0.0147 |      - |     184 B |
| RegexExtractSingle | .NET 6.0           | .NET 6.0           |  26.68 ns | 0.518 ns | 0.960 ns | 0.0115 |      - |     144 B |
| RegexExtractThree  | .NET 6.0           | .NET 6.0           |  26.30 ns | 0.534 ns | 0.766 ns | 0.0115 |      - |     144 B |
| RegexExtractTwenty | .NET 6.0           | .NET 6.0           |  26.78 ns | 0.557 ns | 1.100 ns | 0.0115 |      - |     144 B |
|                    |
| SplitExtractSingle | .NET 8.0           | .NET 8.0           |  37.63 ns | 0.759 ns | 0.844 ns | 0.0095 |      - |     120 B |
| SplitExtractThree  | .NET 8.0           | .NET 8.0           |  63.43 ns | 0.232 ns | 0.194 ns | 0.0280 |      - |     352 B |
| SplitExtractTwenty | .NET 8.0           | .NET 8.0           | 185.71 ns | 0.863 ns | 0.721 ns | 0.1197 |      - |    1504 B |
| SpanExtractSingle  | .NET 8.0           | .NET 8.0           |  42.31 ns | 0.201 ns | 0.168 ns | 0.0147 |      - |     184 B |
| SpanExtractThree   | .NET 8.0           | .NET 8.0           |  41.99 ns | 0.437 ns | 0.388 ns | 0.0147 |      - |     184 B |
| SpanExtractTwenty  | .NET 8.0           | .NET 8.0           |  42.70 ns | 0.191 ns | 0.160 ns | 0.0147 |      - |     184 B |
| RegexExtractSingle | .NET 8.0           | .NET 8.0           |  19.96 ns | 0.388 ns | 0.363 ns | 0.0115 |      - |     144 B |
| RegexExtractThree  | .NET 8.0           | .NET 8.0           |  18.94 ns | 0.169 ns | 0.150 ns | 0.0115 |      - |     144 B |
| RegexExtractTwenty | .NET 8.0           | .NET 8.0           |  18.86 ns | 0.138 ns | 0.115 ns | 0.0115 |      - |     144 B |
|                    |
| SplitExtractSingle | .NET Framework 4.8 | .NET Framework 4.8 |  97.04 ns | 0.465 ns | 0.388 ns | 0.0343 |      - |     217 B |
| SplitExtractThree  | .NET Framework 4.8 | .NET Framework 4.8 | 146.96 ns | 0.422 ns | 0.330 ns | 0.0942 |      - |     594 B |
| SplitExtractTwenty | .NET Framework 4.8 | .NET Framework 4.8 | 420.07 ns | 1.209 ns | 1.010 ns | 0.4091 | 0.0038 |    2576 B |
| SpanExtractSingle  | .NET Framework 4.8 | .NET Framework 4.8 | 124.91 ns | 1.910 ns | 1.693 ns | 0.0305 |      - |     193 B |
| SpanExtractThree   | .NET Framework 4.8 | .NET Framework 4.8 | 125.28 ns | 2.491 ns | 2.447 ns | 0.0305 |      - |     193 B |
| SpanExtractTwenty  | .NET Framework 4.8 | .NET Framework 4.8 | 123.78 ns | 0.516 ns | 0.431 ns | 0.0305 |      - |     193 B |
| RegexExtractSingle | .NET Framework 4.8 | .NET Framework 4.8 |  67.39 ns | 0.365 ns | 0.305 ns | 0.0356 |      - |     225 B |
| RegexExtractThree  | .NET Framework 4.8 | .NET Framework 4.8 |  67.28 ns | 0.347 ns | 0.307 ns | 0.0356 |      - |     225 B |
| RegexExtractTwenty | .NET Framework 4.8 | .NET Framework 4.8 |  67.41 ns | 0.669 ns | 0.626 ns | 0.0356 |      - |     225 B |

## Analyzing stuff
This result matches my expectations to a great extent.
Splitting sucks exponentially while becoming slower and slower.

The allocation represents only the allocation during the method calls and not the poco creation.
I still wonder why regex performs worse than rangin on old net48 tho...



# Testcode
<details><summary>Full SourceCode</summary>
```csharp
using System.Text.RegularExpressions;
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Jobs;

namespace Benchmarkz;

[SimpleJob(RuntimeMoniker.Net48)]
[SimpleJob(RuntimeMoniker.Net60)]
[SimpleJob(RuntimeMoniker.Net80)]
[MemoryDiagnoser]
public class RangeBenchmarks
{
private static readonly Regex regexRulez = new(@"(\w+)=(.*?)(?:;|$)",
RegexOptions.Compiled);

    private const string? JustOne = "param:dt=123";
    private const string? Three = "param:dt=123; bla=blub; stichdatum=01.01.2019";
    private const string? Twenty = "param:dt=123; bla=blub; stichdatum=01.01.2019; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub; bla=blub";

    [Benchmark]
    public void SplitExtractSingle() => ExtractParamFieldSplit(JustOne);

    [Benchmark]
    public void SplitExtractThree() => ExtractParamFieldSplit(Three);
    [Benchmark]
    public void SplitExtractTwenty() => ExtractParamFieldSplit(Twenty);

    [Benchmark]
    public void SpanExtractSingle() => ExtractParamFieldSpan(JustOne);

    [Benchmark]
    public void SpanExtractThree() => ExtractParamFieldSpan(Three);
    [Benchmark]
    public void SpanExtractTwenty() => ExtractParamFieldSpan(Twenty);

    [Benchmark]
    public void RegexExtractSingle() => ExtractParamFieldRegex(JustOne);

    [Benchmark]
    public void RegexExtractThree() => ExtractParamFieldRegex(Three);
    
    [Benchmark]
    public void RegexExtractTwenty() => ExtractParamFieldRegex(Twenty);

    public static IEnumerable<ParamField> ExtractParamFieldSplit(string? inputString)
    {
        if (inputString is not { Length: > 6 })
            return Enumerable.Empty<ParamField>();

        var paramindex = inputString.IndexOf("param:",
                                             StringComparison.InvariantCultureIgnoreCase);
        if (paramindex == -1)
            return Enumerable.Empty<ParamField>();

        var splits = inputString[(paramindex + 6)..].Split(';');
        return splits.Select(x =>
        {
            var value = x.Split('=');
            return new ParamField
            {
                Key = value[0].Trim(),
                Value = value[1].Trim()
            };
        });
    }

    public static IEnumerable<ParamField> ExtractParamFieldSpan(string? inputString)
    {
        if (inputString is not { Length: > 6 })
            return Enumerable.Empty<ParamField>();

        var identifierIndex = inputString.IndexOf("param:",
                                                  StringComparison.InvariantCultureIgnoreCase);
        if (identifierIndex == -1)
            return Enumerable.Empty<ParamField>();

        var inputAsSpan = inputString.AsSpan();
        var startIndex = identifierIndex + 6;

        var endOfFirstPair = inputAsSpan.IndexOf(';');
        var pair = endOfFirstPair > 1
            ? inputAsSpan.Slice(startIndex,
                                endOfFirstPair - startIndex)
            : inputAsSpan[startIndex..];

        var separatorIndex = pair.IndexOf('=');
        var results = new List<ParamField>();
        while (separatorIndex != -1)
        {
            results.Add(new ParamField
            {
                Key = pair[..separatorIndex].Trim().ToString(),
                Value = pair[(separatorIndex + 1)..].Trim().ToString()
            });

            pair = pair[(separatorIndex + 1)..];
            separatorIndex = pair.IndexOf('=');
        }

        return results;
    }

    public static IEnumerable<ParamField> ExtractParamFieldRegex(string? inputString)
    {
        if (inputString is not { Length: > 6 })
            return Enumerable.Empty<ParamField>();

        var matches = regexRulez.Matches(inputString);

        return matches.Cast<Match>()
                      .Select(match => new ParamField
                      {
                          Key = match.Groups[1].Value.Trim(),
                          Value = match.Groups[2].Value.Trim()
                      });
    }

    public class ParamField
    {
        public string Key { get; set; }
        public string Value { get; set; }
    }
}

</details>
