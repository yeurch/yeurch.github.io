---
layout: post
title:  "Dotnet on Linux Feels First Class Now"
---

A long time ago, in another life it feels, I was a .NET developer, writing internal business
web applications on .NET Framework on Windows and IIS.  Support for .NET on Linux was just
through the mono project, and it always felt a little clunky, and lots of features were missing
compared to the "real" Microsoft .NET Framework.

Over the first few years of .NET Core, support for Linux was improved as more APIs were added
to the .NET Standard definition.  When the MAUI UI framework was released with .NET 6, I was
surprised and disappointed that it did not support Linux, only Windows, MacOS, Android and
iOS.  That said, there are some great third party frameworks for UI out there that do support
Linux, such as [Avalonia](https://avaloniaui.net/).

But when it comes to console apps and background services, I think it's fair to say that recent .NET
Core versions treat Linux as a first class citizen.  I've recently been playing around with Microsoft's
Redis-compatible server [Garnet](https://microsoft.github.io/garnet/docs) on Windows, and wanted
to try it out on my Fedora Linux 39 machine, which doesn't even have .NET on it.  I was expecting
a bunch of fairly complex installation steps, maybe adding in custom package repos, or manually
editing my `PATH` environment variable, that sort of thing.  It really wasn't the case ... Fedora
already has the .NET 8.0 SDK available as a package in the default software repos, so getting
setup with .NET was truly as simple as:

```bash
sudo dnf install dotnet-sdk-8.0
```

Amazing! And definitely easier than Windows.

Then, I followed the instructions for Garnet, and it _just worked, seamlessly_:

```bash
git clone https://github.com/microsoft/garnet
cd garnet
dotnet restore
dotnet build -c Release
cd main/GarnetServer
dotnet run -c Release -f net8.0
```

And that was it ... Garnet was up and running, with identical build and execution steps as on
Windows (save for the forward slash vs backslash when changing directory).

