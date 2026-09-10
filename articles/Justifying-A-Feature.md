---
title: Justifying a Feature
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Justifying a Feature

Synopsis: As developers, we want to create the tightest, fastest code we can to delight our users. However, we also have to remove our biases and pre-conceived judgements when we want to change code because we think it'll make things better. In this article, I'll talk about a small feature that I added to my mocking package, Rocks, and what I did to determine that the feature was worth it.

## To Cache, or not to Cache

A couple of years ago, I created [Rocks](https://github.com/JasonBock/Rocks), primarily as an experiment to see if it would be easier to do mocks via C# code generation and Compiler API support over using `System.Reflection.Emit`. Implementation-wise, it's been far easier, but I've also been cognizant that performance could be an issue. Thankfully, [benchmarks have shown](https://github.com/ecoAPM/BenchmarkMockNet) that it works well. That said, I'm always thinking about where things could be better. While I haven't sat down yet and done any profiling work (shame on me!), I occasionally will see areas that I think improvements could be made.

One specific case was where I take `Assembly` references and turn them into `MetadataReference` objects. This is needed to create a `CSharpCompilation` object. To do this, I created an extension method called `Transform()`. It's pretty simple in what it does, but one thing I realized is that Rocks will do this every time a new mock is created. Why is this a potential issue? If developers are using a mocking framework like Rocks, they're generating a handful of mocks for things like dependencies specified with interfaces. Now, in Rocks, for each type I need to mock, I get a list of all known dependencies and call `Transform()` on them. My reasoning was that I should only do this once. It's highly unlikely that for a given test assembly, other assemblies will need to be mapped as tests are loaded and executed. Therefore, I thought, I should cache the mapping of an `Assembly` to a `MetadataReference`. This should make subsequent mock generation quicker.

How much faster? My guess was, "faster", but I had no qualitative number in my head. Also, once you add a cache, you run the danger of adding a memory leak of sorts if you're not careful with releasing cached values appropriately. Therefore, rather than just trusting my view that caching should help performance, I really needed to measure the effect. This is the crux of this article: adding any kind of feature should have some kind of beneficial aspect that can be justified. It can be a user adding for a new workflow that will streamline their process, or it can be a developer finding a way to make something work better. In the business case, the measurable difference is that the user finishes a task with a 50% reduction in time. The same is true for code – we should be able to show that a change has a measurable impact.

## Numbers, Please!

That last point in the previous section is crucial. I've seen developers change code because they were convinced their change was going to make the application perform better. I've been that developer who has done that. Sometimes, the change helps, and sometimes it doesn't do anything, or it makes things even worse. Getting benchmark data isn't the hard part these days, but we also have to take care to interpret the results and make sure we're coming to the correct conclusion. Let's see what happens when I add a cache to that method and how I measure the results.

### Adding a Cache

To start, here's what the code looked like before I added the cache:

```c#
internal static IEnumerable<MetadataReference> Transform(
  this IEnumerable<Assembly> @this) =>
    @this.Where(
      _ => !_.IsDynamic && !string.IsNullOrWhiteSpace(_.Location))
      .Select(
        _ => MetadataReference.CreateFromFile(_.Location))
      .Cast<MetadataReference>();
``` 

Adding a cache only requires a couple of changes:

```c#
private static readonly 
  ConcurrentDictionary<Assembly, MetadataReference> transformedAssemblies =
    new ConcurrentDictionary<Assembly, MetadataReference>();

internal static IEnumerable<MetadataReference> Transform(
  this IEnumerable<Assembly> @this) =>
    @this.Where(
      _ => !_.IsDynamic && !string.IsNullOrWhiteSpace(_.Location))
      .Select(
        _ => IEnumerableOfAssemblyExtensions.transformedAssemblies.GetOrAdd(
          _, asm => MetadataReference.CreateFromFile(asm.Location)))
      .Cast<MetadataReference>();
``` 

I decided on using a `ConcurrentDictionary` because it's possible user may run tests in parallel, and I wanted to handle that path correctly. This could end up being a slight area of contention, but my thought was that it would be minimal at best.

Now that I have my new code in place, how do I test it to compare it what I did before?

### First Round of Tests

I decided to use [Benchmark.NET](https://benchmarkdotnet.org/) to gather performance data. It's easily one of my favorite NuGet packages out there. It makes it really easy to run tests to see how well code performs, both from a speed and memory perspective. However, at first glance, I was unsure how I was going to get my data. In my Rocks solution, I have a console application called `Rocks.Sketchpad` that I use to try out different feature ideas for Rocks – it was a good candidate to use for benchmarking. The problem was, I didn't have an easy way to create two versions of Rocks, one with caching and one without, and reference them from the same application.

My original solution was to create a global flag and switch the behavior based on that code:

```c#
internal static bool IsCachingEnabled = false;

internal static IEnumerable<MetadataReference> Transform(
  IEnumerableOfAssemblyExtensions.IsCachingEnabled ?
    this IEnumerable<Assembly> @this) =>
      @this.Where(
        _ => !_.IsDynamic && !string.IsNullOrWhiteSpace(_.Location))
        .Select(
          _ => MetadataReference.CreateFromFile(_.Location))
        .Cast<MetadataReference>() :  
    this IEnumerable<Assembly> @this) =>
      @this.Where(
        _ => !_.IsDynamic && !string.IsNullOrWhiteSpace(_.Location))
        .Select(
          _ => IEnumerableOfAssemblyExtensions.transformedAssemblies.GetOrAdd(
            _, asm => MetadataReference.CreateFromFile(asm.Location)))
        .Cast<MetadataReference>();
``` 

This worked, but ... well, it didn't feel right. I wasn't going to leave this in the code, but even for a testing endeavor, I didn't want a global flag and two code paths available when running performance tests. I decided to remove the flag and run the performance tests twice: once with caching enabled, and once with things done without caching. With this approach, here's the test code I wrote:

```c#
[MemoryDiagnoser]
public class MetadataReferenceCacheBenchmark
{
  private readonly IRock<IA> iaRock;
  private readonly IRock<IB> ibRock;
  private readonly IRock<IC> icRock;
  private readonly IRock<ID> idRock;

  public MetadataReferenceCacheBenchmark()
  {
    var options = new RockOptions();
    this.iaRock = Rock.Create<IA>(options);
    this.iaRock.Handle(_ => _.Foo());
    this.ibRock = Rock.Create<IB>(options);
    this.ibRock.Handle(_ => _.Foo());
    this.icRock = Rock.Create<IC>(options);
    this.icRock.Handle(_ => _.Foo());
    this.idRock = Rock.Create<ID>(options);
    this.idRock.Handle(_ => _.Foo());
  }

  [Benchmark]
  public IA CreateOneMock() =>
    this.iaRock.Make();

  [Benchmark]
  public IA CreateFourMocks()
  {
    this.ibRock.Make();
    this.icRock.Make();
    this.idRock.Make();
    return this.iaRock.Make();
  }
}
``` 

The four interfaces, `IA`, `IB`, `IC` and `ID`, have just one method, `Foo()`.

After running it a couple of times under both conditions, I ended up with the following results (the first table is with caching turned on, the second table has caching disabled):

|     Method             |          Mean    |         Error    |        StdDev    |      Gen 0    |      Gen 1    |     Allocated    |
|------------------------|-----------------:|-----------------:|-----------------:|--------------:|--------------:|-----------------:|
|     CreateOneMock      |      2.609 us    |     0.0518 us    |     0.1271 us    |     0.1501    |     0.0415    |         959 B    |
|     CreateFourMocks    |     11.782 us    |     0.2232 us    |     0.2481 us    |     0.6104    |     0.1689    |        3845 B    |

| Method          |      Mean |     Error |    StdDev |  Gen 0 |  Gen 1 | Allocated |
|-----------------|----------:|----------:|----------:|-------:|-------:|----------:|
| CreateOneMock   |  2.609 us | 0.0518 us | 0.1271 us | 0.1501 | 0.0415 |     959 B |
| CreateFourMocks | 11.782 us | 0.2232 us | 0.2481 us | 0.6104 | 0.1689 |    3845 B |

My first reaction looking at this data was ... what?! This looked like it made no difference at all! The numbers are virtually the same. This was surprising to me. I was convinced that caching should make some kind of measurable impact, but this data was clearly showing that it didn't.

Or was I missing something?

### Digging Into Getting a `MetadataReference`

I stepped away from my tests for a moment and thought about what I seeing. One idea was creating a `MetadataReference` is so fast, it's the same amount of work as caching the value. Or, maybe `MetadataReference` was doing some kind of internal caching on its own. I thought the latter point was dubious at best. I could have looked up the code in GitHub, but I made the assumption that the Compiler API wasn't going to add that caching in. What I decided to look at next was the speed of creating a `MetadataReference` over caching. Basically, eliminate Rocks from the picture entirely and focus on this one aspect of code.

Here's the tests I wrote to do this:

```c#
public class MetadataReferenceBenchmark
{
  private readonly Assembly assembly;
  private readonly 
    ConcurrentDictionary<Assembly, PortableExecutableReference> concurrentMap;
  private readonly 
    Dictionary<Assembly, PortableExecutableReference> map;

  public MetadataReferenceBenchmark()
  {
    this.assembly = typeof(MetadataReferenceBenchmark).Assembly;
    var metadata = MetadataReference.CreateFromFile(this.assembly.Location);
    this.map = new Dictionary<Assembly, PortableExecutableReference> 
      { { this.assembly, metadata } };
    this.concurrentMap = 
      new ConcurrentDictionary<Assembly, PortableExecutableReference>();
    this.concurrentMap.TryAdd(this.assembly, metadata);
  }

  [Benchmark]
  public PortableExecutableReference GetReferenceFromAssembly() => 
    MetadataReference.CreateFromFile(this.assembly.Location);

  [Benchmark]
  public PortableExecutableReference GetReferenceFromMap() =>
    this.map[this.assembly];

  [Benchmark]
  public PortableExecutableReference GetReferenceFromConcurrentMap() =>
    this.concurrentMap[this.assembly];

  [Benchmark]
  public PortableExecutableReference GetReferenceFromConcurrentMapViaGetOrAdd() =>
    this.concurrentMap.GetOrAdd(this.assembly, 
      asm => MetadataReference.CreateFromFile(asm.Location));
}
```

I threw in a test to use a regular old `Dictionary`, just to see how that compared to using `ConcurrectDictionary`. Here are the results:

|     Method                                      |             Mean    |           Error    |          StdDev    |      Gen 0    |     Allocated    |     Allocated    |
|-------------------------------------------------|--------------------:|-------------------:|-------------------:|--------------:|-----------------:|-----------------:|
|     GetReferenceFromAssembly                    |     55,459.57 ns    |     858.7029 ns    |     717.0558 ns    |     1.0376    |        2264 B    |         959 B    |
|     GetReferenceFromMap                         |         22.94 ns    |       0.5245 ns    |       0.4650 ns    |          -    |           0 B    |        3845 B    |
|     GetReferenceFromConcurrentMap               |         28.01 ns    |       0.6476 ns    |       0.7953 ns    |          -    |           0 B    |                  |
|     GetReferenceFromConcurrentMapViaGetOrAdd    |         30.64 ns    |       0.6707 ns    |       0.7724 ns    |          -    |           0 B    |                  |

This clearly shows that there is a lot of overhead in creating the `MetadataReference` over and over again versus pulling it from a cache – it's 3 orders of magnitude slower. This meant that adding caching should be making a difference. But why wasn't I seeing that in my original tests?

### Eliminating the Mock Caching

It dawned on me that I wasn't taking into considering the default behavior of Rocks. When Rocks creates a mock object, it caches that version the first time someone wants to create an instance of that mock. Take a look at this code:

```c#
var rock = Rock.Create<IService>();
rock.Handle(_ => _.Serve());
var chunk = rock.Make();
``` 

Until `Make()` is called, all Rocks does is gather all the expectations that a developer is specifying. At the point `Make()` is invoked, Rocks looks to see if a mock type exists for `IService`. If it doesn't, it goes through all the work of creating the mock type and compiling it. This is when `Transform()` would be invoked as well. The next time the developer wants a mock instance of `IService`, the type is cached, so none of that work happens.

I realized that my built-in mock caching was hiding any of the gains `MetadataReference` caching was providing. But I had an out. Rocks allows you to turn off caching if you want:

```c#
var options = new RockOptions(
  caching: CachingOptions.GenerateNewVersion);
var rock = Rock.Create<IService>(options);
```

I would personally never do this because the implementation will be exactly the same, but there may be reasons why someone wants distinct mock types. Fortunately, by having this capability built-in, I could create performance tests that turn off mock caching and forces Rocks to always create a new mock type.

Now, before I show you the results, I should note that this causes Rocks to perform horribly. I wouldn't recommend turning this option on as default behavior with your own tests that use Rocks for mocking. But my hope was by turning mock caching off, I would now see a difference when I added `MetadataReference` caching. Sure enough, when I ran the tests, here's what I would see (as before, timings with `MetadataReference` caching enabled are in the first table, and the second table has `MetadataReference` caching turned off):

|     Method             |         Mean    |         Error    |       StdDev    |        Gen 0    |        Gen 1    |      Allocated    |
|------------------------|----------------:|-----------------:|----------------:|----------------:|----------------:|------------------:|
|     CreateOneMock      |     10.89 ms    |     0.6359 ms    |     1.875 ms    |     180.1563    |      25.2344    |      628.39 KB    |
|     CreateFourMocks    |     45.07 ms    |     2.9646 ms    |     8.741 ms    |     708.7500    |     104.0625    |     2515.27 KB    |

|     Method             |         Mean    |      Error    |        StdDev    |        Gen 0    |        Gen 1    |     Gen 2    |     Allocated    |
|------------------------|----------------:|--------------:|-----------------:|----------------:|----------------:|-------------:|-----------------:|
|     CreateOneMock      |     23.29 ms    |     2.6 ms    |      7.662 ms    |     262.3438    |     108.3594    |      22.5    |       2.66 MB    |
|     CreateFourMocks    |     60.06 ms    |     4.9 ms    |     14.636 ms    |     951.8750    |     303.1250    |      40.0    |       7.02 MB    |

This convinced me that adding caching for `Transform()` was a good thing to do. Adding a `ConcurrentDictionary` didn't seem to add too much to the overall memory pressure of the process either.

## Is it Really Worth It?

Before I finish this post, I should talk about the actual gains made by adding `MetadataReference` caching. Frankly, it's not a lot. My tests that focus on caching strategies and MetadataReference creation show there's a benefit, but in reality, during a test run, a developer will only make a handful of mock types. Once the first one is made, all of the needed MetadataReferences should be cached. The time it takes to make a mock type dwarfs the gains made by caching `MetadataReference` objects, so this doesn't help a whole lot.

So, did I choose to add it to Rocks? It still makes a difference, and it is measurable, which is why I ended up committing the change, and it's part of the 3.1.0 package. My guess is that users probably won't notice a difference. One might say I spent way too much time on such a small positive gain, but now I know more about setting up the right tests, and I can use this knowledge on any future implementations I do on other projects. What I really should do is create a scenario that create a number of distinct mock types, run that application through a profiler, and find all the hot spots in the code. That's an issue I've made and it'll be interesting to see what the results will be and what I can do to make Rocks even faster.

## Conclusion

Any time code is added to a framework or an application, justification is warranted. If it's not providing value, it's a burden. Take the time to make sure a new feature will bring benefits and add value to users, even if they may not notice it. Be objective with your analysis – question the results you come up with. If you can be diligent, you can have high confidence that your code is lean, fast, and effective. Until next time, happy coding!

> Published: 11.07.2017