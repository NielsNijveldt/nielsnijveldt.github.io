---
title:  ".NET 8 and Blazor: A brief overview of What's New and Improved"
date:   2023-11-15 20:00:00 +0100
tag: 
    - Blazor
    - ASP.NET
    - Dotnet 8
    - Razor
    - Web Assembly
    - SSR
    - SignalR
---

Today, Microsoft launched .NET 8, the latest version of .NET and ASP.NET.
In this release, Blazor got some significant improvements; in this post, I will provide a brief overview of the update and the new render modes that greatly benefit the creation of Blazor applications.

Download .NET 8 here: [https://dotnet.microsoft.com/en-us/download](https://dotnet.microsoft.com/en-us/download). Don't forget to upgrade your .NET 7 applications, as the support of .NET 7 is until May 14, 2024, while the new version is supported until November 10, 2026.

![picture](/assets/20231115/postvisual.png)
_Visual created by Dall-E_

## Render modes
With the previous version of Blazor you had to choose between two hosting models. You could choose between Blazor Server and Blazor WebAssembly. It was not always easy to choose because both have benefits and downsides. You may have experienced inconveniences for both hosting models while implementing specific scenarios.
After the release of .NET 7 Microsoft started a project called "Blazor United" to provide a better experience creating web apps with .NET.
Later on, they renamed it to "Full stack web UI with Blazor".

With .NET 8, Microsoft offers flexibility with both hosting models. Also, a new solution is provided called Streamrendering.
You can mix and match Blazor Server, Blazor WebAssembly, and Streamrendering to develop a web app that covers all your needs with less or no hacking compared to before.
It's also possible to choose none of these options. Then the static HTML is rendered without the possibility of interactivity.

### StreamRendering
StreamRendering is a solution where a request from the client for a specific page returns an initial response (with placeholders) and allows loading data from a different source. Once the source is retrieved, the placeholders will be updated in this long-running task.
See this simple example:
{% highlight C# %}
@page "/products"
@attribute [StreamRendering(true)]

@using BlazorShop.Client.Components
@using BlazorShop.Model

@inject HttpClient Http
<div>
    <h2>Products</h2>
    @if (result == null)
    {
        <LoadingIndicator />
    }
    else
    {
        <div>
            @foreach (var product in result.Products)
            {
                <ProductDetails Product="@product" />
            }
        </div>
    }
</div>

@code {
    private PagedResult<Model.Product>? result;

    protected override async Task OnInitializedAsync()
    {
        result = await Http.GetFromJsonAsync<PagedResult<Model.Product>>("https://dummyjson.com/products");
    }
}
{% endhighlight %}
This example will show a loading indicator on the initial response. At the end of the OnInitialized the response will be updated and finalized. Now, the client will render an overview of products instead of the loading indicator.
This is achieved without a SignalR connection (Blazor Server) or WebAssembly files.

If you have buttons or interactive components, by default, those will not respond to the user's actions. There are, however, a couple of ways to make the page using StreamRendering interactive. You can use the `Enhance` attribute if there are forms on the page. See [this example](https://github.com/dotnet/aspnetcore/blob/main/src/Components/test/testassets/Components.TestServer/RazorComponents/Pages/Forms/StreamingRenderingForm.razor). Another way to achieve interactivity is to embed a component as child element with either the render mode defined a attribute on the element or by defining a attribute within the component.

A couple of sidenotes:
- The  `AfterRender` and `OnAfterRenderAsync` override will not be called in your page or component since the cycle is done after the `OnInitialized`
- By choosing "Interactive render mode" as None by creating a new project, you will be able to use StreamingRendering without any of the other render modes
- To achieve certain complexity, you might need to use Javascript

### Rendermodes 
Blazor offers three rendermodes: `InteractiveServer`, `InteractiveWebAssembly`, `InteractiveAuto`.
You can either specify these on component level or page level.
When specifying `InteractiveServer` SignalR will be used to have an interactive component/page. With `InteractiveWebAssembly` the webassembly files will be used to render the component/page.
If you choose `InteractiveAuto` the best option will be used to render the component/page. It will download the webassembly files (if not available yet). In the meantime, a websocket will be started for SignalR to provide interactivity. The next time this same component/page will be opened the already downloaded webassembly files will be used instead of the SignalR connection.

{% highlight HTML %}
<div class="something">
    <Component @rendermode="RenderMode.InteractiveServer" />
</div>
{% endhighlight %}


{% highlight C# %}
@page "/counter"
@rendermode InteractiveAuto
{% endhighlight %}

### InteractiveAuto
If you choose `InteractiveAuto`, there are a couple of things to consider.
Because the component's code can run either server or client side, you need to make sure that the resources (eg, API, Database) are publicly available. Otherwise, the component will break.
The best way to use this feature is by choosing the option "Auto (Server and Webassembly)" while creating a new project. This will create two projects in your new solution. One project to provide the server logic and one (client) project to provide the webassembly logic.
Since the server project has a project reference to the client project you only have to define component and pages once. By placing them in the client project, you can use them in both projects and with both render modes.

### Deployment
To deploy the Blazor app, you only need to publish the "server" project. Since it's a server solution, you need an Azure app service to host it (or something similar outside Azure). If you want to host Blazor WebAssembly as a static web app, pick one of the Blazor Web Assembly project templates when creating a new project. However, none of the render modes mentioned earlier will be available since it's a Blazor WebAssembly solution.

The upgrade for Blazor is an excellent enhancement of what we already had. It will make it easier to create modern web applications.
For a complete overview of the .NET 8 update, read both the [.NET 8](https://devblogs.microsoft.com/dotnet/announcing-dotnet-8/) and [ASP.NET Core](https://devblogs.microsoft.com/dotnet/announcing-asp-net-core-in-dotnet-8/) blog posts from Microsoft.
Soon, I will post more content with sample projects, such as Blazor Identity UI, using TailwindCSS within a Blazor .NET 8 solution, and more.