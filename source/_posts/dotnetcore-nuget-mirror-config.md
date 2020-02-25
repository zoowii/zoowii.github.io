---
title: 给dotnet core项目设置nuget国内镜像源
date: 2020-02-25 23:38:23
tags: DotNet, .Net
---

写了个用.Net Core的小工具，部署时发现NuGet被墙，所以本文记录了下修改国内镜像的方法以及可用镜像.

azure在国内有个NuGet的mirror镜像，可以修改NuGet设置来使用.

```XML
    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
        <packageSources>
            <add key="cdn.azure.cn" value="https://nuget.cdn.azure.cn/v3/index.json" protocolVersion="3"/>
            <add key="nuget.org" value="https://api.nuget.org/v3/index.json" protocolVersion="3" />
        </packageSources>
        <packageRestore>
            <add key="enabled" value="True" />
            <add key="automatic" value="True" />
        </packageRestore>
        <bindingRedirects>
            <add key="skip" value="False" />
        </bindingRedirects>
    </configuration>
```

用以上内容修改或者替换Linux中的`~/.nuget/NuGet/NuGet.Config`或者Windows中的`%AppData%\NuGet\NuGet.Config`路径中的文件，如果用docker发布的在Dockerfile中自行修改即可.


本文参考了 [https://www.cnblogs.com/cmt/p/nuget-mirror.html](https://www.cnblogs.com/cmt/p/nuget-mirror.html)