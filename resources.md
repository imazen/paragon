
## Resources


### Native interop/loading projects to investigate


* [Nuclei.Fusion is a sophisticated AssemblyResolver handler](https://github.com/pvandervelde/Nuclei/blob/master/src/nuclei.fusion/FusionHelper.cs) [nuget](http://www.nuget.org/packages/Nuclei.Fusion/)
* [ILBundle is a primitive AssemblyResolve event handler injector](https://github.com/BenPhegan/ILBundle/blob/master/Content/ILBundle.cs.pp) [nuget](http://www.nuget.org/packages/ILBundle/)
* [Dynamic Libraries permits runtime assembly loading and string-based name invocation](https://github.com/Boyko-Karadzhov/Dynamic-Libraries). Currently uses WinAPI (LoadLibrary, GetProcAddress,FreeLibrary), but [could be ported to mono](http://dimitry-i.blogspot.com/2013/01/mononet-how-to-dynamically-load-native.html).
* [Dynamic Interop is similar to Dynamic Libraries, but with better x-plat support](https://github.com/jmp75/dynamic-interop-dll) [nuget](http://www.nuget.org/packages/DynamicInterop/)
* [Lost.Native is a thin LoadLibrary wrapper](http://www.nuget.org/packages/Lost.Native/) [view source](https://bitbucket.org/lost/native/src/8ad3850680dd1094baa1c61268c14e9b88d8532f/DynamicLibrary.cs?at=default)
* [DependencySharp helps extract embedded COM assmeblies](http://www.nuget.org/packages/DependencySharp/). Does not seem to have any architecture awareness.



### NuGet packages for post-processing

* [Fody.Costura](https://github.com/Fody/Costura) enables dependency embedding as a post-process.
* [Fody.ModuleInit](https://github.com/fody/moduleinit) provides module initializer support.
* [InjectModuleInitializer (gh)](https://github.com/einaregilsson/InjectModuleInitializer) [nuget](http://www.nuget.org/packages/InjectModuleInitializer/) post-processes assemblies and allows code to be [executed before any other module code](http://einaregilsson.com/module-initializers-in-csharp/).
* http://www.nuget.org/packages/Afterthought/


### Misc

* [Export C interface from C#](* http://www.nuget.org/packages/UnmanagedExports/)
