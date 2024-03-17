# Enum HasFlag() Benchmark

# Benchmark 1: Native Dotnet Method

In C# theres the possibility to just call `Enum.HasFlag(enumMember)`.
This method [has been rewritten](https://github.com/dotnet/coreclr/pull/13748) during the Release of dotnet 5.
Thus the Benchmarks for net48 will suck really really bad.
The reason behind this mess is due to the fact that it had to perform some checks and is not jit optimized 

# Benchmark 2: GenericContainsFlag

Flagged enums are actually kinda simple.
They're only binary values.
Flag A = 0x0001
Flag B = 0x0010
-> Flag A AND Flag B = 0x0011

So its sufficient to perform a logical OR operation:  
```
  0x0011
& 0x0100
= 0x0000 // Does not contain this EnumMember

  0x0011
& 0x0010
= 0x0010 // Contains this EnumMember
```

This generic approach has to deal with the same problem as the native implementation.
FlagChecks can only work on enum classes which contan the `[Flags]` attribute.

I'll go for a cast here and I expect this to peform even worse than dotnets native method

```csharp
    public static bool ContainsFlagGeneric<T>(this T @enum,
                                       T flagToTest)
        where T: Enum
        => (Convert.ToInt32(@enum) & Convert.ToInt32(flagToTest)) != 0;
```

# Benchmark 3: ExplicitContainsFlag

When the EnumType is known ( and thus its ensured that it contains the attribute ) I can directly perform the logical AND .
While this implementation will outperform both other Methods by large using net48 it might perform worse on more recent targetFrameworks.

```csharp
    public static bool ContainsFlag(this TestEnum testEnum,
                                    TestEnum flag)
        => (testEnum & flag) != 0;
```

# Results
Well everything as expected

| Method               | Job                | Runtime            | TestEnum             |       Mean |     Error |    StdDev |     Median |   Gen0 | Allocated |
|----------------------|--------------------|--------------------|----------------------|-----------:|----------:|----------:|-----------:|-------:|----------:|
| DotnetsHasFlag       | .NET 6.0           | .NET 6.0           | X1                   |  0.0024 ns | 0.0041 ns | 0.0039 ns |  0.0000 ns |      - |         - |
| ContainsFlagGeneric  | .NET 6.0           | .NET 6.0           | X1                   | 30.0068 ns | 0.1473 ns | 0.1306 ns | 30.0199 ns | 0.0076 |      96 B |
| ContainsFlagExplicit | .NET 6.0           | .NET 6.0           | X1                   |  0.0077 ns | 0.0045 ns | 0.0042 ns |  0.0064 ns |      - |         - |
| DotnetsHasFlag       | .NET 8.0           | .NET 8.0           | X1                   |  0.0003 ns | 0.0005 ns | 0.0005 ns |  0.0000 ns |      - |         - |
| ContainsFlagGeneric  | .NET 8.0           | .NET 8.0           | X1                   | 18.3309 ns | 0.0862 ns | 0.0720 ns | 18.3499 ns | 0.0076 |      96 B |
| ContainsFlagExplicit | .NET 8.0           | .NET 8.0           | X1                   |  0.0002 ns | 0.0004 ns | 0.0003 ns |  0.0000 ns |      - |         - |
| DotnetsHasFlag       | .NET Framework 4.8 | .NET Framework 4.8 | X1                   |  9.5784 ns | 0.0195 ns | 0.0173 ns |  9.5734 ns | 0.0076 |      48 B |
| ContainsFlagGeneric  | .NET Framework 4.8 | .NET Framework 4.8 | X1                   | 48.1679 ns | 0.1372 ns | 0.1216 ns | 48.1179 ns | 0.0153 |      96 B |
| ContainsFlagExplicit | .NET Framework 4.8 | .NET Framework 4.8 | X1                   |  0.0102 ns | 0.0056 ns | 0.0053 ns |  0.0111 ns |      - |         - |
| DotnetsHasFlag       | .NET 6.0           | .NET 6.0           | X1, X2, X4           |  0.0080 ns | 0.0052 ns | 0.0049 ns |  0.0087 ns |      - |         - |
| ContainsFlagGeneric  | .NET 6.0           | .NET 6.0           | X1, X2, X4           | 29.6568 ns | 0.0660 ns | 0.0617 ns | 29.6794 ns | 0.0076 |      96 B |
| ContainsFlagExplicit | .NET 6.0           | .NET 6.0           | X1, X2, X4           |  0.0006 ns | 0.0024 ns | 0.0022 ns |  0.0000 ns |      - |         - |
| DotnetsHasFlag       | .NET 8.0           | .NET 8.0           | X1, X2, X4           |  0.0011 ns | 0.0014 ns | 0.0013 ns |  0.0005 ns |      - |         - |
| ContainsFlagGeneric  | .NET 8.0           | .NET 8.0           | X1, X2, X4           | 20.8817 ns | 0.0855 ns | 0.0714 ns | 20.8671 ns | 0.0076 |      96 B |
| ContainsFlagExplicit | .NET 8.0           | .NET 8.0           | X1, X2, X4           |  0.0003 ns | 0.0006 ns | 0.0006 ns |  0.0000 ns |      - |         - |
| DotnetsHasFlag       | .NET Framework 4.8 | .NET Framework 4.8 | X1, X2, X4           | 10.0027 ns | 0.0244 ns | 0.0216 ns | 10.0089 ns | 0.0076 |      48 B |
| ContainsFlagGeneric  | .NET Framework 4.8 | .NET Framework 4.8 | X1, X2, X4           | 48.1968 ns | 0.1167 ns | 0.1035 ns | 48.1573 ns | 0.0153 |      96 B |
| ContainsFlagExplicit | .NET Framework 4.8 | .NET Framework 4.8 | X1, X2, X4           |  0.0076 ns | 0.0029 ns | 0.0027 ns |  0.0076 ns |      - |         - |
| DotnetsHasFlag       | .NET 6.0           | .NET 6.0           | X1, X(...) X512 [47] |  0.0144 ns | 0.0063 ns | 0.0056 ns |  0.0145 ns |      - |         - |
| ContainsFlagGeneric  | .NET 6.0           | .NET 6.0           | X1, X(...) X512 [47] | 29.5644 ns | 0.0550 ns | 0.0488 ns | 29.5493 ns | 0.0076 |      96 B |
| ContainsFlagExplicit | .NET 6.0           | .NET 6.0           | X1, X(...) X512 [47] |  0.0012 ns | 0.0021 ns | 0.0020 ns |  0.0000 ns |      - |         - |
| DotnetsHasFlag       | .NET 8.0           | .NET 8.0           | X1, X(...) X512 [47] |  0.0000 ns | 0.0001 ns | 0.0001 ns |  0.0000 ns |      - |         - |
| ContainsFlagGeneric  | .NET 8.0           | .NET 8.0           | X1, X(...) X512 [47] | 20.8049 ns | 0.0664 ns | 0.0588 ns | 20.8043 ns | 0.0076 |      96 B |
| ContainsFlagExplicit | .NET 8.0           | .NET 8.0           | X1, X(...) X512 [47] |  0.0006 ns | 0.0010 ns | 0.0008 ns |  0.0000 ns |      - |         - |
| DotnetsHasFlag       | .NET Framework 4.8 | .NET Framework 4.8 | X1, X(...) X512 [47] |  9.9909 ns | 0.0198 ns | 0.0165 ns |  9.9934 ns | 0.0076 |      48 B |
| ContainsFlagGeneric  | .NET Framework 4.8 | .NET Framework 4.8 | X1, X(...) X512 [47] | 48.0942 ns | 0.0702 ns | 0.0622 ns | 48.0951 ns | 0.0153 |      96 B |
| ContainsFlagExplicit | .NET Framework 4.8 | .NET Framework 4.8 | X1, X(...) X512 [47] |  0.0060 ns | 0.0041 ns | 0.0038 ns |  0.0067 ns |      - |         - |
| DotnetsHasFlag       | .NET 6.0           | .NET 6.0           | X1, X(...)X1024 [54] |  0.0051 ns | 0.0049 ns | 0.0046 ns |  0.0045 ns |      - |         - |
| ContainsFlagGeneric  | .NET 6.0           | .NET 6.0           | X1, X(...)X1024 [54] | 27.9598 ns | 0.0830 ns | 0.0777 ns | 27.9430 ns | 0.0076 |      96 B |
| ContainsFlagExplicit | .NET 6.0           | .NET 6.0           | X1, X(...)X1024 [54] |  0.0034 ns | 0.0047 ns | 0.0044 ns |  0.0011 ns |      - |         - |
| DotnetsHasFlag       | .NET 8.0           | .NET 8.0           | X1, X(...)X1024 [54] |  0.0000 ns | 0.0001 ns | 0.0001 ns |  0.0000 ns |      - |         - |
| ContainsFlagGeneric  | .NET 8.0           | .NET 8.0           | X1, X(...)X1024 [54] | 20.5399 ns | 0.1075 ns | 0.1006 ns | 20.5378 ns | 0.0076 |      96 B |
| ContainsFlagExplicit | .NET 8.0           | .NET 8.0           | X1, X(...)X1024 [54] |  0.0004 ns | 0.0009 ns | 0.0008 ns |  0.0000 ns |      - |         - |
| DotnetsHasFlag       | .NET Framework 4.8 | .NET Framework 4.8 | X1, X(...)X1024 [54] |  9.9675 ns | 0.0257 ns | 0.0228 ns |  9.9658 ns | 0.0076 |      48 B |
| ContainsFlagGeneric  | .NET Framework 4.8 | .NET Framework 4.8 | X1, X(...)X1024 [54] | 47.9990 ns | 0.1296 ns | 0.1149 ns | 47.9698 ns | 0.0153 |      96 B |
| ContainsFlagExplicit | .NET Framework 4.8 | .NET Framework 4.8 | X1, X(...)X1024 [54] |  0.0055 ns | 0.0037 ns | 0.0034 ns |  0.0054 ns |      - |         - |


# Testcode
<details><summary>Full SourceCode</summary>

```csharp
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Jobs;

namespace Benchmarkz;

[SimpleJob(RuntimeMoniker.Net48)]
[SimpleJob(RuntimeMoniker.Net60)]
[SimpleJob(RuntimeMoniker.Net80)]
[MemoryDiagnoser(displayGenColumns: true)]
public class EnumFlags
{
[Params(0x0001,
        0x0007,
        0x03FF,
        0x07FF)]
public TestEnum TestEnum;

    [Benchmark]
    public void DotnetsHasFlag() => _ = TestEnum.HasFlag(TestEnum.X4);
    
    [Benchmark]
    public void ContainsFlagGeneric() => _ = TestEnum.ContainsFlagGeneric(TestEnum.X4);
    
    [Benchmark]
    public void ContainsFlagExplicit() => _ = TestEnum.ContainsFlag(TestEnum.X4);

}

public static class EnumExtensions
{
    public static bool ContainsFlagGeneric<T>(this T @enum,
                                              T flagToTest) where T: Enum
        => (Convert.ToInt32(@enum) & Convert.ToInt32(flagToTest)) != 0;

    public static bool ContainsFlag(this TestEnum testEnum,
                                    TestEnum flag) 
        => (testEnum & flag) != 0;
}


[Flags]
public enum TestEnum
{
X0 = 0,
X1 = 1,
X2 = 1 << 1,
X4 = 1 << 2,
X8 = 1 << 3,
X16 = 1 << 4,
X32 = 1 << 5,
X64 = 1 << 6,
X128 = 1 << 7,
X265 = 1 << 8,
X512 = 1 << 9,
X1024 = 1 << 10,
}

```
</details>
