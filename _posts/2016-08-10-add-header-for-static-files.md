---
layout: post
title:  "Add Headers for Static Files with ASP.NET Core"
date:   2016-08-10 22:42:35 +0800
categories: dotnet
---

Sometimes we need to add some headers for static file requests. 

For example, if we want to allow third-party website to access images or audios 
through XHR (it's usefull when you want to do some custom decoding with these files). 
You need to add `Access-Control-Allow-Origin: *` (relpace `*` with your domain if you don't realy want to allow any domain to access).

If you use `Microsoft.AspNetCore.StaticFiles` to handle static file requests, you should find `app.UseStaticFiles()` in startup.cs.

Then what we need to do is add an Option like this:

```cs
app.UseStaticFiles(new StaticFileOptions()
{
    OnPrepareResponse = context =>
    {
        context.Context.Response.Headers["Access-Control-Allow-Origin"] = "*";
    }
});
```
Done!

`StaticFileOptions` has some other properties to customize the StaticFileMiddleware behavior.

```cs
//
//     Used to map files to content-types.
public IContentTypeProvider ContentTypeProvider { get; set; }
//
//     The default content type for a request if the ContentTypeProvider cannot determine
//     one. None is provided by default, so the client must determine the format themselves.
//     http://www.w3.org/Protocols/rfc2616/rfc2616-sec7.html#sec7
public string DefaultContentType { get; set; }
//
//     Called after the status code and headers have been set, but before the body has
//     been written. This can be used to add or change the response headers.
public Action<StaticFileResponseContext> OnPrepareResponse { get; set; }
//
//     If the file is not a recognized content-type should it be served? Default: false.
public bool ServeUnknownFileTypes { get; set; }
```

Have fun!

