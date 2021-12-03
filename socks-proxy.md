 - support for SOCKS proxy (community)
        - https://github.com/dotnet/runtime/issues/17740
        - https://github.com/dotnet/runtime/pull/48883

# SOCKS Proxy support

We have already written about this addition in [Preview 5 blog post](https://devblogs.microsoft.com/dotnet/announcing-net-6-preview-5/#libraries-socks-proxy-support). This was a long standing issue ([dotnet/runtime#17740](https://github.com/dotnet/runtime/issues/17740)) that got eventually implemented by a community contributor [Huo Yaoyuan](https://github.com/huoyaoyuan). This change added support for SOCKS4, SOCKS4a and SOCKS5 proxies.

SOCKS proxy is very versatile tool. For example, it can provide similar functionality to VPN. Most notably SOCKS proxy is used to access [Tor](https://www.torproject.org/) network.

To configure `HttpClient` to use SOCKS proxy, you only need to use proper `socks` schema when defining the proxy[^socks]:
```C#
var client = new HttpClient(new HttpClientHandler()
{
    // Specify the whole Uri (schema, host and port) as one string or use Uri directly.
    Proxy = new WebProxy("socks5://127.0.0.1:9050")
});

var response = await client.GetAsync("https://check.torproject.org/");
Console.WriteLine(response);
var content = await response.Content.ReadAsStringAsync();
Console.WriteLine(content);
```
This example assumes you're running `tor` instance on your computer. If the request succeeds you should be able to find "**Congratulations. This browser is configured to use Tor.**" in the response content.

[^socks]: In the original blog post, we made a mistake and used wrong overload of [`WebProxy` constructor](https://docs.microsoft.com/en-us/dotnet/api/system.net.webproxy.-ctor?view=net-6.0#System_Net_WebProxy__ctor_System_String_System_Int32_). It expects only host name in the first argument and cannot be used with any other proxy type than HTTP.