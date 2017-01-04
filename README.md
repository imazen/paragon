
*For brevity, let's call unmanaged dependencies udeps.* 

# 2017 update - looking for help

I spent the week after Christmas trying to figure out where NuGet is at, reading its source code, and trying to discover if I'm unblocked yet on anything. https://github.com/imazen/Imazen.NativeDependencyManager still appears blocked by shadow-copy. The docs that do exist contradict one another.

Given that I seem to *always* hit every NuGet corner case, perhaps somone would be interested in picking up a subset this project, and demonstrating how to use NuGet for udep delivery in a way that is seamless for end-users, works with test runners, and can be published via Travis/Appveyor?

Some history:
https://github.com/NuGet/Home/issues/300

And https://github.com/aspnet/dnx/issues/402

### Runtime IDs

.NET Core has documentation on "runtime IDs"
https://docs.microsoft.com/en-us/dotnet/articles/core/rid-catalog
http://www.natemcmaster.com/blog/2016/05/19/nuget3-rid-graph/

**No clear answer on whether this is supported on .NET 4.6.1 or 4.6.2, and to what extent. I can find no mention of changes to udep discovery.**  Last I checked, x64 and x86 subfolders still weren't searched by default. 

### .NET Standard

We now have a way to target both .NET Full and .NET Core at the same time. Yay!
https://github.com/dotnet/standard/blob/master/docs/versions.md

It's a bit leaky, but could be worse :) https://github.com/dotnet/standard/blob/master/docs/netstandard-20/README.md#net-framework-461-supporting-net-standard-20

And targeting specific versions got easier: Targeting different .NET editions is easier: https://docs.nuget.org/ndocs/guides/create-cross-platform-packages


### runtime.{rid} nuget package convention?

Mostly used by Microsoft, but some others have started using it, too. 

https://www.nuget.org/packages?q=runtime.win

**Should we adopt this convention for rid-specific packages? It does namespace away these support packages.**


### Existing packages

We have a mega-package with all udeps together. This won't scale for large binaries.
https://github.com/natemcmaster/libsqlite3-package
See https://github.com/natemcmaster/libsqlite3-package/blob/master/SQLite.nuspec


https://github.com/NuGet/Home/issues/2782

Targeting different .NET editions is easier: https://docs.nuget.org/ndocs/guides/create-cross-platform-packages


# 2015 below here

Skip to end for *specific questions*. 

# Creating a template for .NET/native interop

vNext reduces reliance (and eventually, access) to OS APIs. To replace that functionality (or to be cross-platform), we must turn to open-source C and C++ libraries.

## Challenges

1. Exploits are more common in native code. Low-friction upstream merges, builds, tests, and static analysis are key to maintaining security. Burden of security responsibility always falls on the wrapper maintainer (at least, in the Win ecosystem).
2. Visual Studio does not track or copy udeps.
3. Udeps are not shadow-copied. Resulting issues include (but are not limited to) [frequent deploy failures due to locked files](https://github.com/projectkudu/kudu/issues/1122).
4. Not all udeps are assemblies (unmanaged assemblies often depend on physical .xml or data files which can't be embedded).
5. No API to discover 'original' location. Compilation folder - .CodePoint check. Shadow-copied .Location - check. Directory assemblies were shadow-copied from? We can make educated (but often incorrect) guesses. 
6. NuGet is blind to udeps. Scripted solutions still break in some scenarios.
7. MinGW (the most common cross-platform build environment) is poorly maintained and very underfunded. 
8. Cmake on windows can be painful.
9. No compatibility between VS and GNU build toolchains. 
10. Interop can be impossible with more than one AppDomain per process. (If dll has any static configuration, or lacks its own thread safety). Looking at you, OpenCV.
11. No safe/reliable lifecycle events to hook into and fixup DLL loading/copying and unloading (AFAIK). Mutexes in App_Start, perhaps? Failure would be permanent.
12. Binding generation is still stupidly difficult. CppSharp is very promising; provides the smartest bindings I've seen to date. Unfortunately, it has minimal documentation, and could use assistance from native english speakers.


## Known workarounds

2) Visual Studio ignores native assemblies.

---

Define subfolders in your project for each architecture, with all dependencies in each.

Use LoadLibraryEx to manually load the .dlls depending upon executing architecture. 

**Failures: Challenge 5 makes this imprecise. Makes versioning issues commonplace. Does not work for child dependencies - every project must subsume these CopyFile references to work. Messy**

---

Host copies of udeps on a secure server with HTTPS support. During Application_Start, download the correct dependencies to /bin. **Fails with file locking errors with multiple worker processes - or AppDomains- or recycling.**

---

Embed copies of native dependencies in the managed DLLs, then extract them at runtime. 

**Failures: Very slow. Horrible bloating. Fails with file locking errors with multiple worker processes or AppDomains unless a new copy is created for each AppDomain.**


## End goals

Create a sample cross-platform, multi-architecture NuGet package with udeps - that *just works* in ASP.NET and ASP.NET vNext - even when you switch between 32 and 64-bit test runners. All processes (builds, tests, static analysis, binding generation) must be scripted and AppVeyor compatible.

To cover all bases, it should include indirect udeps and udeps with procwide configuration (like a callback fn).

This includes

* Scripted unmanaged builds from source (with dependencies). Both MinGW (make and cmake) and MSVC approaches should be shown.
* AppVeyor compatible build scripts, unit tests (managed and unmanaged), and static analysis.
* Scripted binding or P/Invoke generation that works well (probably CppSharp).
* A safe & reliable DLL fixup approach that works in all conditions (VS, Test runners, Azure, and IIS)
* Preventing multiple AppDomains per process (optionally).
* Automated nuget publishing (triggered by tags, perhaps?) via AppVeyor

Solid workarounds are fine - but what can't be worked around needs to be fixed in tooling. 


## Prerequisite goals

Define standards for

* Describing udeps for a managed assembly (either through attributes or embedded manifests). In some scenarios this list could be inferred from the P/Invoke metadata, if combined with conventions, and when there are no indirect dependencies.

* Conventions for organization of udeps by architecture and possibly by locale? More than one architecture may need to be deployed or tested, so all variants must be reachable. 

## Real-world cases (Mine are mostly in an ASP.NET context).

I maintain .NET wrappers for libwebp, FreeImage, CAIR, FFmpeg, Ghostscript, and OpenCV. I'm working on a .NET wrapper for LibGD. I've experimented with many approaches to dependency management, but haven't found a reliable AND user-friendly solution. In turn, the ImageResizer plugins that depend on these wrappers are second-class citizens.

## Specific questions 

I know many of these are project-specific, but 'generally best direction' answers are great.

* What is the best approach for maintaining automated windows builds of unmanaged software which in turn has unmanaged dependencies? Example: LibGD depends on zlib, libpng, freetype, libjpeg-turbo, and (optionally) libtiff, libxpm, and fontconfig. So far, to achieve CI, I've had to [version precompiled static versions of these dependencies](https://github.com/imazen/gd-libgd/blob/master/appveyor.yml) for each target platform. 
* CMake or not to CMake? Ensuring cross-platform builds means we have to maintain *nix, msvc, and MinGW makefiles. Is CMake usually up to the task on windows (given enough sweat), or are handmade windows makefiles typically the best answer?
* MinGW or msvc? Is it typically worth the effort porting to MSVC, or is MinGW+gcc the way to go?
* C++/CLI or P/Invoke? Or both, with C++/CLI versions for windows to avoid udep hell?
* Should unmanaged dependencies get their own NuGet package, or be subsumed into the .NET wrapper?
* How should unmanaged dependencies be located? To support AnyCPU, we need multiple platforms side-by-side, which means (at least) subfolders per architecture. Have locale variations ever been known to be a concern?
* How do we handle shadow-copying? Can we shadow-copy unmanaged dependencies ourselves, to prevent the locked file plague?
* Is remote download of dependencies (I.e, during App_Start) feasible? It would seem to depend on being able to shadow-copy from .NET, and on transactional file operations.
* Say nuget package A uses a different version of libpng than nuget package B. Is there a method to allow both to be used side-by-side in .NET, or should we dump all native dependencies in a single architecture-specific folder and let them fight it out with file overwrites?
* What can we ask the awesome AppVeyor folks to implement to make unmanaged builds easier?
* In ASP.NET are there any thread-safe hooks other than App_Start for dll fixup? (I.e, places where a DLL can block all other application threads to do something (during start). 
* Is there any reliable exit phase for cleanup, that is guaranteed to run after all requests have been terminated?

## Recent feedback from a very helpful guy

* Use CMake, it's worth it. Add a wrapper script to fetch dependencies first and set up config.h and compile vars.
* Ideally, compile all things from source, with a git repository for each project, and each project having its own CI setup with dependency fetching.
* MSVC can be very extremely competitive with performance using profile guided optimization. V14 will reduce C99 pain.
* It is possible to move a locked file (on the same drive) with MoveFileEx - use this to get older assemblies out of the way during a deploy. 
* You can schedule deletion of a file via MoveFileEx, using a null destination with (delayuntilreboot) flags.
* An alternative to LoadLibrary - modify the process search path.
* For a thread-safe fixup, use module initializer via [PreApplicationStartMethodAttribute](http://msdn.microsoft.com/en-us/library/system.web.preapplicationstartmethodattribute.aspx), perhaps?
* Don't rely on cleanup.
* Don't try side-by-side native assemblies
* Don't try shadow copying
* 

[SqlServerCompact handles native binaries with an apparently good result](https://github.com/projectkudu/kudu/issues/943). [LibGit2 has struggled with udeps](https://github.com/libgit2/libgit2sharp/pull/700), as has 


There [have been attempts to add native support to NuGet](http://nuget.codeplex.com/discussions/412012), something [about Extension SDKs](https://nuget.codeplex.com/discussions/396720)  and [more than](https://npe.codeplex.com/discussions/462174)  one [issue filed about it](http://nuget.codeplex.com/workitem/1221). CoApp has [a video on making native (uncompiled) NuGet packages](https://www.youtube.com/watch?v=l4MAkR13JPA). 


There's a [NuGet package for copying stuff to the build directory](https://github.com/baseclass/Contrib.Nuget), and [an article about it](http://www.baseclass.ch/blog/Lists/Beitraege/Post.aspx?ID=6&mobile=0). 

The udeps topic [has been brought up in the context of vNext as an issue](https://github.com/aspnet/KRuntime/issues/402) also.


## Libraries and authors who've dealt with the problem

* Microsoft.SqlServer.Compact - last release 2012, maintainer unknown
* LibGit2Sharp - [LinQPad/Testrunner issues](https://github.com/libgit2/libgit2sharp/issues/515), then dealing with [the ASP.NET deloy and Kudu issues](https://github.com/libgit2/libgit2sharp/pull/425), and [switching to .targets based installation](https://github.com/libgit2/libgit2sharp/pull/615) but [multi-platform web is still an open issue](https://github.com/libgit2/libgit2sharp/issues/733). 
..add more here..


https://npe.codeplex.com/discussions/462174
http://stackoverflow.com/questions/19478775/add-native-files-from-nuget-package-to-project-output-directory
http://blog.nuget.org/20141023/package-manifests.html
