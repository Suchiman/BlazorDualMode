Demo on how to dynamically switch from client to server side mode by appending `?mode=server` to the url.

# Howto
This [commit](https://github.com/Suchiman/BlazorDualMode/commit/6e850cbe020812f85bf9c158c48635d942e02f2a) summarizes the necessary changes.

1. Create a `Blazor (ASP.NET Core hosted)` Project, then change the `Startup` class of the `.Server` Project to enable server side features.
This doesn't have adverse effects on Client Side Blazor but enables the Server Side services.
For that you need to have `services.AddServerSideBlazor();` in `ConfigureServices` and `endpoints.MapBlazorHub<Client.App>("app");` in `Configure`
2. We can now serve Client Side and Server Side apps but we need to polyfil the `HttpClient` that is provided in DI in Client Side by default. Server Side doesn't register it by default so we detect this and then register an `HttpClient` in DI that behaves similiar for compatibility
	```csharp
	// Server Side Blazor doesn't register HttpClient by default
	if (!services.Any(x => x.ServiceType == typeof(HttpClient)))
	{
		// Setup HttpClient for server side in a client side compatible fashion
		services.AddScoped<HttpClient>(s =>
		{
			// Creating the URI helper needs to wait until the JS Runtime is initialized, so defer it.
			var uriHelper = s.GetRequiredService<IUriHelper>();
			return new HttpClient
			{
				BaseAddress = new Uri(uriHelper.GetBaseUri())
			};
		});
	}
	```
3. At this point, the only difference is which blazor JS file we load in the browser. This can be either achieved by serving a different `index.html` (for which i couldn't see an easy way) or using a small piece of JS to decide which file to load
	```js
	<script id="blazorMode"></script>
	<script>
		document.getElementById("blazorMode").src = window.location.search.includes("mode=server") ? "_framework/blazor.server.js" : "_framework/blazor.webassembly.js";
	</script>
	```