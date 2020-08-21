
# DotNet Core - Web Application using Kestrel and Nancy from Scratch

The below is a step by step for making an embedded web app using Kestrel and Nancy. It's lightweight and cross platform so will run place .Net Core can. Used for making a quick web API in an existing service, a full app, etc...

Current at least from .net core 2.2

## Project Start

Use an existing project or create a console project.

## Packages

These packages should be added via Nuget.

- Microsoft.AspNetCore.Hosting

- Microsoft.AspNetCore.Owin

- Microsoft.AspNetCore.Server.Kestrel

- Microsoft.AspNetCore.StaticFiles

- Microsoft.Extensions.FileProviders.Embedded

- Microsoft.Extensions.FileProviders.Physical

- Nancy

- Nancy.Authentication.Basic

## Folder Layout

![][image_ref_h6mdfc8d]


## WebServer Class

The static class "Config" should be reworked according to however you need, its like this for easy customization.

```c#
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.FileProviders;
using Nancy;
using Nancy.Authentication.Basic;
using Nancy.Bootstrapper;
using Nancy.Owin;
using Nancy.TinyIoc;
using System;
using System.IO;
using System.Linq;
using System.Reflection;
using System.Security.Claims;
using System.Security.Principal;

namespace WebCore
{
    public static class Config
    {
        public static string USERNAME = null;
        public static string PASSWORD = null;
        public static string WEBINTERFACE_URI = "http://localhost:6958";
        public static bool SERVESTATIC = false;
    }

    public class UserMapper : IUserValidator
    {
        public ClaimsPrincipal Validate(string username, string password)
        {
            if (username == Config.USERNAME && password == Config.PASSWORD)
            {
                return new ClaimsPrincipal(new GenericIdentity(username));
            }
            return null;
        }
    }

    public class ApplicationBootstrapper : DefaultNancyBootstrapper
    {
        protected override void ApplicationStartup(TinyIoCContainer container, IPipelines pipelines)
        {
            base.ApplicationStartup(container, pipelines);

            if (!String.IsNullOrEmpty(Config.USERNAME) && !String.IsNullOrEmpty(Config.PASSWORD))
            {
                pipelines.EnableBasicAuthentication(
                    new BasicAuthenticationConfiguration(
                        container.Resolve<IUserValidator>(),
                        "realm"
                    )
                );
            }
        }

        public ApplicationBootstrapper() { }
    }

    public class Startup
    {
        private readonly IConfiguration config;

        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder();
            config = builder.Build();
        }

        public void Configure(IApplicationBuilder app)
        {
            if (Config.SERVESTATIC)
            {
                if (!Directory.Exists(Path.Combine(Directory.GetCurrentDirectory(), "static")))
                {
                    throw new Exception("Did you copy the static folder, or copy after build enabled?");
                }
                app.UseStaticFiles(new StaticFileOptions()
                {
                    FileProvider = new PhysicalFileProvider(Path.Combine(Directory.GetCurrentDirectory(), "static")),
                    RequestPath = new PathString("/static")
                });

                app.UseStaticFiles(new StaticFileOptions()
                {
                    FileProvider = new EmbeddedFileProvider(typeof(Startup).Assembly, typeof(Startup).Namespace + ".assets"),
                    RequestPath = new PathString("/assets")
                });
            }
            app.UseOwin(x => x.UseNancy(opt => opt.Bootstrapper = new ApplicationBootstrapper()));
        }
    }

    public class WebServer
    {
        IWebHost host;

        public static string ReadManifestData<TSource>(string embeddedFileName) where TSource : class
        {
            var assembly = typeof(TSource).GetTypeInfo().Assembly;
            var resourceName = assembly.GetManifestResourceNames().First(s => s.EndsWith(embeddedFileName, StringComparison.CurrentCultureIgnoreCase));

            using (var stream = assembly.GetManifestResourceStream(resourceName))
            {
                if (stream == null)
                {
                    throw new InvalidOperationException("Could not load manifest resource stream.");
                }
                using (var reader = new StreamReader(stream))
                {
                    return reader.ReadToEnd();
                }
            }
        }

        public WebServer(string webServerListenURI, bool serveStatic = false, string basicAuthUsername = null, string basicAuthPassword = null)
        {
            Config.WEBINTERFACE_URI = webServerListenURI;
            Config.USERNAME = basicAuthUsername;
            Config.PASSWORD = basicAuthPassword;
            Config.SERVESTATIC = serveStatic;

            this.host = new WebHostBuilder()
                .SuppressStatusMessages(true)
                .UseUrls(Config.WEBINTERFACE_URI)
                .ConfigureLogging((ctx, log) => {
                    //log.Services.Clear();
                })
                .UseKestrel((options) => {
                    //options.ListenAnyIP(int.Parse(configuration.WebInterfacePort));
                })
                .UseStartup<Startup>()
                .Build();
        }

        public void Start()
        {
            this.host.RunAsync();
        }

        public void StartBlocking()
        {
            this.host.Run();
        }

        public void OpenBrowser(string path = "")
        {
            var browser = new System.Diagnostics.Process()
            {
                StartInfo = new System.Diagnostics.ProcessStartInfo(Config.WEBINTERFACE_URI + (String.IsNullOrEmpty(path) ? "" : "/" + path)) { UseShellExecute = true }
            };
            browser.Start();
        }
    }
}

```

## Starting the Web Server

WebServer exposes two methods: Start and StartBlocking. Start runs async, StartBlocking blocks thread.

```c#
var webServer = new WebServer("http://localhost:6000/");
//webServer.OpenBrowser();
webServer.StartBlocking();
```

### WebServer Constructor Options

basicAuthUsername, basicAuthPassword:

- Passing username and password as parameters to WebServer constructor will turn on basic authentication. 

- Default is NO AUTHENTICATION (BASIC).

"serveStatic":

- allows serving a "static" folder located at the base of the project at "BASE/static". The files in this folder should have a "Build Action" property of "content" and the property for all the files here should "Copy To Output Directory" for the files to work properly. Serving through here would be slow and take managed resources, opt to serve static files through a web server optimized for this.

- allows serving an "assets" folder located at the base of the project at "BASE/assets". The are embedded resource so are bundled along with the resulting dll or exe. The files in this folder should have a "Build Action" property of "Embedded resource" for the files to work properly.

- Default is FALSE.

### WebServer Utils

WebServer has a static method named "ReadManifestData" that will load an embedded resource as text. To use create a file, add to project, right click the file and change the content in properties to "Embedded Resource" (in visual studio). This is a quick hack to get around having to copy over static files. The paths are specifed with "." instead of "/" like normal. See the example routes below with path of "/test" for an example. I would probably just use the assets folder for this though.

## Constructing Routes

Use normal Nancy routes.

```c#
using Nancy;
using Nancy.Security;
using System;
using System.Collections.Generic;
using System.Text;

namespace WebCore
{
    public class Routes : NancyModule
    {
        public Routes()
        {
            //this.RequiresAuthentication();

            Get("/test", (args) =>
            {
                return Response.AsText(
                    WebServer.ReadManifestData<Routes>($"resources.index.html"),
                    "text/html"
                );
            });
        }
    }
}
```

## Views

https://github.com/NancyFx/Nancy/wiki/View-engines

[image_ref_h6mdfc8d]: data:image/jpeg;base64,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AKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAv6ZZW92t5LdXEsENrAJmMUQkY5dEAALKOr569q1l8KC+8NXGuaRdS3MNtIyTRzwCJwFUMWGHYEAH26GotK0fU3stRijsJXa6sUKYKjCmZHBIJB5EbYHU8HGDmmaHpV/BezTvZzeSNNuJTKqlkCvbybSWHAycj6gjqCKTTSuaw5JXj1MSOMyNtUqDgn5mCjj3NNq5pfk/bv3/l+X5cn+sxjOxsde+cVoyzWb28sGy0CLZROrIihzL8mfm6565FHUzSuYywSPBJOq5jjYKzZHBOcfyNR11Mj2SxyRzPZCza7hKLAV3GIbs7tvPQ85560khsWusXEFlHABgsk8LljuG3HlgYHrxnbnuBSuPlObigkmWVo1yIk3vyOBkDP5kU0RuY2kCMUUgMwHAJ6ZP4Gte0cWf22aSS0eSW3bailSobzF4wOO2QORj2q1NcxNp15HbmzUyJBKyBYxzsO/bnuD2HIycd6LhY5ypDBILdbgr+6ZygbI6gAkfqK3ry9tUa/MEOn5inUW+IUOVO7d/vDgdc4zxiq2rtai2kitnjKC+lZVRgcKVXGPb/Ci4WMaiiiqJCiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiitzw54dl1q43vmOzjPzv3P+yPf+VJuw0m3ZGHRVvVIUt9WvIYl2xxzuij0AYgVXhj86eOIuke9gu9z8q5PU+1F9LhbWw+1s7q9lMVpbTXEgG4pEhY49cCpbrS9QsYxJd2F1boTtDSwsgJ9MkV32q+BNevJrqK2m0iCxeUtHHHEI22ZO0MVjycDHUnkVyd5plzo+l6rYXagTRXtsCVOQw2TEEexFctPExqfC16HXUwrgrtMrW3iPV7MYgvXT9ysGdoJ2LnaMkdskA9QOBwBVG5u57sxmeQv5UaxRjoFRegA7evuSSeSas6fa291HMr73uR/qollEe4YOTkgg4444J7Vo3FnYzXsW2B4bdLSOSVjMBjKrg8Rk5yeeDknPFdbk9Eciirtrqc/RXTNYWlu1vaxiZZhqTQ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
