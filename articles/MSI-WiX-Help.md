---
title: MSI/WiX Help
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# MSI/WiX Help

I'm trying to figure out how to get the files within a MSI file to install in a given directory. Here are the contents from a small wxs file I created for an agile presentation I did a week ago:

```xml
<?xml version="1.0" encoding="utf-8"?>
<Wix xmlns="http://schemas.microsoft.com/wix/2003/01/wi">
  <Product Name="Magenic.AgilePresentation.Customer"Id="8CB2FF12-8CEA-4604-AC3C-44E7EBA56FC4" 
    Version="1.0.0.0"Manufacturer="Magenic" Language="1033">
    <Package Id="AC68DC57-3984-496d-87FE-389E89FF3008" Manufacturer="Magenic"
      InstallerVersion="200" Platforms="Intel" Languages="1033" Compressed="yes" 
      SummaryCodepage="1252" />
      <Directory Id="TARGETDIR" Name="SourceDir">
        <Component Id="Component.Customer" Guid="C6AF8AC8-6D52-4660-A1A8-0F478C26B530">
          <File Id="File.Customer" Name="MAGEN~1.DLL"
            LongName="Magenic.AgilePresentation.Customer.dll"
            src="Magenic.AgilePresentation.Customer.dll" Vital="yes" 
            KeyPath="yes" DiskId="1" />
        </Component>
      </Directory>
      <Media Id="1" EmbedCab="yes" Cabinet="_9ADB2A8A34BE47CF9F184E26099C68D0" />
      <Feature Id="DefaultFeature" Level="1" ConfigurableDirectory="TARGETDIR">
        <ComponentRef Id="Component.Customer" />
      </Feature>
  </Product>
</Wix>
```

The problem is when I double-click on the resulting MSI file (created from candle and light). The DLL goes into the C: drive. So (and here's where the newbie mode shines very bright), how can I get that component to install into the current directory, or a specified directory? Any help is appreciated - thanks!

> Published: 08.24.2004 03:34:44 PM CST