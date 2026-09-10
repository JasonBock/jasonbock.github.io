---
title: Modernizing .NET Applications
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Modernizing .NET Applications

Overview: .NET is moving towards a unified platform in Nov. 2020. What does this mean for you? In this article, I'll discuss how things are changing in .NET and what that means to you as a .NET developer.

## Assess the State of Affairs

.NET is a framework that has been around in production since 2002. It's the runtime behind numerous applications across a wide variety of platforms, such as Windows, Mac, and Linux. In recent history, .NET has moved to an open-source model as well, which allows anyone to contribute and collaborate with the .NET team members from Microsoft. However, with such a long history, it's inevitable that some forms of divergence have occurred over time. Specifically, there are three main versions of .NET:

* .NET Framework –This is the Windows-only version of .NET that has been the mainstay since day 1.
* Mono – This is an open-source version of .NET that was maintained for many years outside of Microsoft, and was eventually acquired by Microsoft through their purchase of Xamarin.
* .NET Core – This is Microsoft's open-source, cross-platform version of .NET that was first released in 2016 and is currently on version 3.1.

As one could imagine, trying to maintain all three versions over time has its disadvantages. Furthermore, it's clear that having a lean, cross-platform runtime is the choice for the future. Therefore, Microsoft announced that .NET would become one platform, designated as .NET 5. You can read all the details here, but one key point that is evident about .NET 5 is that the .NET Framework is "done" as of version 4.8. Microsoft will provide security and bug fixes as long as Windows is supported (.NET Framework is shipped with Windows as a core component) but no new features will be added, and the .NET Framework will always be Windows-only.

What does this mean for .NET developers who are maintaining applications that target the .NET Framework? Before we address potential modernization and migration concerns, let's discuss general application aspects that should be considered.

### Stability of the System

Occasionally I'll have conversations with developers who want to move the latest version of whatever language, component, or package that they're currently using, or change them wholesale. My question for them is, is there a driving reason from those who use your applications to do this? Do they care if you move from language A to language B? To be clear, there can be viable explanations why these changes should be done. Maybe a new version of a package contains performance improvements and bug fixes. But having a stable system is a desirable aspect of an application.

If you're able to reliably deploy applications targeting the .NET Framework and react to user requests in a timely fashion, you may want to stick with what you have. It's not cross-platform, it may not be deployed into Kubernetes, but it delivers features to your users. Remember, there are applications written in COBOL that still run critical systems in 2020.

### Strength of the Design
How healthy is your application? Are there distinct domains defined such that it's easy to find where things are in code? What's the status of testing – do you have numerous, fast, targeted unit tests? Are you using analyzers to help find issues in code?

Too many times I've talked to clients who want to "migrate" to whatever the "new" state of the art is, but the current implement was done so poorly it made little sense to use what was there. Subsequentially, that increased the amount of time it took to travel that migration path. If you have a well-designed system, it makes it easier to evolve it over time. This must be considered if you want to move from .NET Framework to .NET 5.

If your existing application does not have a good architecture with separation of concerns, you may be better off maintaining it as-is in .NET Framework, because a rewrite might be required to modernize the code. On the other hand, if your application is well-architected, the cost of migration is often quite reasonable, and you may want to modernize to gain immediate benefits from .NET Core, as well as moving to the platform on which Microsoft is investing for the future.

### Cross-Platform Support
Do you have a business need to support multiple operating systems or device types? This can be a major driver toward the adoption of .NET Core and ultimately .NET 5. Modern .NET runs on all major (and some minor) operating systems, extending your reach as a .NET developer to provide business value.

### Cloud-Native Development
The .NET Framework can be used to build cloud solutions, but only when running on the Windows operating system. Most modern cloud development is focused on the use of containers, which generally rely on the Linux operating system.

Increasingly there are Windows containers, but they continue to lag behind Linux containers in popularity, and generally require more server resources. As a result, most modern cloud-native development efforts assume the use of Linux containers and related technologies such as Kubernetes.

Even if you did focus on Windows containers, .NET Core requires substantially less memory and provides much higher performance compared to the older .NET Framework. To gain the best performance and highest server density the best bet is to use .NET Core.

## Migration Paths to Consider

Having considered those aspects of your current application, let's say that you've decided to migrate the application from .NET Framework to .NET Core. Even if you have a solid CI/CD pipeline in place and your code is architected and designed well, you still need to keep in mind potential issues that you might run into. The goal would be to move to .NET Core 3.1 right now, as .NET 5 is still in its early days and is only at "preview 3" status as of the time this article was written (May 2020). Let's look at specific areas you should investigate.

### Application Types

Building applications targeting the .NET Framework means you probably used frameworks to build either web sites or native Windows applications. Successfully porting these code bases to .NET depends on the frameworks in play. Here's a synopsis of the major frameworks from Microsoft and what you might run into.

#### ASP.NET

ASP.NET has a long history in .NET, and there are numerous ways to build web applications with this framework, such as Web Forms, MVC, and WebApi. With .NET Core, .NET developers can build REST APIs, web sites with MVC or Razor Pages, and client native applications using Blazor. In some cases, porting web APIs to .NET Core will mostly require changes to different types and methods now being used.

If applications were built with Web Forms, there's no migration path available. Therefore, Web Forms applications may require a substantial amount of time to move to .NET Core. The only path forward is to rewrite the application into a modern, supported UI technology such as Razor Pages or Blazor.

#### WCF

WCF was heavily used for .NET Framework applications to build and host services. In .NET Core, there is no support for WCF. A new alternative is to start using gRPC, which is supported on .NET as of .NET Core 3.0. However, there is no way to migrate WCF services to gRPC, though there are third-party tools available to help with this migration. Furthermore, this site provides guidance on moving from WCF to gRPC.

Another alternative is CoreWCF. This is an open source community-driven project to support moving WCF services to .NET Core with minimal friction. However, as of the writing of this article, the code was not considered "production ready", nor does it come from Microsoft so it does not have the same production support models as .NET does.

#### Windows Forms and WPF

With .NET Core 3.0, support was added that allows Windows Forms and WPF applications to target the modern .NET runtime. This doesn't mean that these applications are now cross-platform as these frameworks are Windows-specific. It does mean that these applications can now target a runtime that is more performant and easier to deploy. Retargeting these applications is generally straightforward (read these articles on WinForms and WPF for more details), though, as always, testing is a must to ensure applications still work as expected.

### Package Versions

NuGet is the standard package system in .NET, with hundreds of thousands of packages available for developers to use in a wide variety of scenarios encountered in software development. At this point, many packages have been updated to support .NET Core and/or .NET Standard (a bridging technology to help migration from .NET Framework to modern .NET). It's important to review all the packages used in an application and determine if any of them do not have support for at least .NET Core 3.1 and/or .NET Standard 2.0. If they don't, decisions will have to made to either find another package with similar functionality or update the repository via a fork to support .NET Core.

### Deployment

With the .NET Framework, Windows is the only available platform to use. With .NET Core, code can be deployed to a wide variety of operating systems. This has the potential of lowering cost and improving performance. However, simply upgrading an application to .NET Core does not mean it can be hosted in Kubernetes or cloud-native environments running Linux. Depending on the specifics of the application structure, it may be detrimental to deploy to these newer targets. If your application has been designed using well-known principles of cloud-native, distributed, service architecture, you are in a better position to not only move off Windows, but to target different environments and operating systems.

### Language Choices

There's one other area to consider. There are three language choices from Microsoft that target .NET: C#, F#, and VB. It's interesting to note that in Microsoft's 2017 language strategy blog post, it was stated that:

> We will keep Visual Basic straightforward and approachable. We will do everything necessary to keep it a first class citizen of the .NET ecosystem: When API shapes evolve as a result of new C# features, for instance, consuming those APIs should feel natural in VB. We will keep a focus on the cross-language tooling experience, recognizing that many VB developers also use C#. We will focus innovation on the core scenarios and domains where VB is popular.

Fast-forward to 2020, and the story has arguably changed. According to this post:

> We do not plan to evolve Visual Basic as a language. This supports language stability and maintains compatibility between the .NET Core and .NET Framework versions of Visual Basic. Future features of .NET Core that require language changes may not be supported in Visual Basic. Due to differences in the platform, there will be some differences between Visual Basic on .NET Framework and .NET Core.

There are several ways to read between the lines, but it's clear that VB is "done" as a language. This means that any future changes to the CLR may be exposed to C# and F#, but VB will not support it.

Does this mean you should abandon VB? For current applications written with VB, they may work targeting .NET Core 3.1, but over time, they may stop working with newer versions of .NET. It is probably wise to start thinking about how to eventually move your applications from VB to C# or F#.

For new applications, my recommendation would be to use C# or F#. They have a future of innovation and evolution.

## Conclusion

Microsoft's vision of "One .NET" will become a reality near the end of 2020. This does not require applications targeting the .NET Framework to update to this version, but there are numerous advantages in doing so. The effort required will vary from minimal to substantial depending on what the application does and how it was implemented. Magenic has vast experience using .NET and we have already helped clients migrate to .NET Core. Contact us to discuss your current state of affairs, and how we can assist your migration and transformation of .NET applications!

> Published: 05.06.2020