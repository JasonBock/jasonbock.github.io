---
title: IronPython Visual Studio Integration Woes
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# IronPython Visual Studio Integration Woes

This morning I tried to get IronPython installed on my box. Well, to be specific, I wanted the Visual Studio integration piece working (the actual IronPython install is just a simple xcopy + PATH variable update). I ran into two errors getting the integration piece working - I tried to follow the instructions here, but no dice. Here are the details on the errors I found (I ended up running msbuild from the command-line on the IronPython.sls file to get more details on the errors).

In the LanguageService.csproj compilation section, it tries to run the SettingUpDevenv target: devenv.com /rt "Software\Microsoft\VisualStudio\8.0Exp" /setup. This fails with an exit code of 1 - during this command, the build process chugs and chugs and eventually I get Visual Studio's "I-failed-badly-send-error-information-to-Microsoft" dialog window.

Later on, when building the IronPythonWebSiteCustomWizard.csproj project, the AfterBuild target fails as it tries to copy IronPythonWebSiteCustomwizard.dll to a directory it needs to create: "*Undefined*\PrivateAssemblies". Well, that has illegal characters so that part fails.

I saved the entire build process so now I'm going to dig into the solution/project files in more detail and see if I can tweak/hack/figure-out what's going wrong and fix it. I've also notified Arron, although I'm not sure if he's the right person to contact. If you've tried to build the IronPython integration project and ran into some issues please let me know.

> Published: 09.28.2006 08:24:44 AM CST