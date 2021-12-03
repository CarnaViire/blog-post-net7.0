# `WinHttpHandler` additions

`WinHttpHandler` uses WinHTTP underneath and to add support for a feature, that feature must firstly exist in WinHTTP. In this release, there are two additions that expose or enable WinHttp features for HTTP/2. Both of them are part of a larger theme ([dotnet/core#5713](https://github.com/dotnet/core/issues/5713)) to allow users to use [gRPC .NET](https://github.com/grpc/grpc-dotnet) on .NET Framework.

First is support for trailing headers ([dotnet/runtime#44778](https://github.com/dotnet/runtime/issues/44778)). For .NET Core 3.1 / .NET 5 and higher, the trailing headers are exposed in [`HttpResponseMessage.TrailingHeaders`](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.httpresponsemessage.trailingheaders?view=net-6.0). For .NET Framework, they are exposed in `HttpRequestMessage.Properties["__ResponseTrailers"]` since there is no such property as `TrailingHeaders` on .NET Framework.

The other addition is support for bidirectional streaming ([dotnet/runtime#44784](https://github.com/dotnet/runtime/issues/44784)). This change is completely seamless and `WinHttpHandler` will automatically allow bidirectional streaming when appropriate, i.e. when request content doesn't have a known length and when underlying WinHTTP supports it.

Another feature we exposed, is ability to configure [TCP keep-alive](https://datatracker.ietf.org/doc/html/rfc1122#section-4.2.3.6). TCP keep-alive is used to keep idle connection open and to prevent nodes in the middle, like proxies and firewalls, from dropping the connection sooner than the client expects. In .NET 6.0 we have added 3 new properties to `WinHttpHandler` to configure it:
```C#
public class WinHttpHandler
{
    // Controls whether TCP keep-alive is getting send or not.
    public bool TcpKeepAliveEnabled { get; set; }
    // Delay to the first keep-alive packet during inactivity.
    public TimeSpan TcpKeepAliveTime { get; set; }
    // Interval for subsequent keep-alive packets during inactivity.
    public TimeSpan TcpKeepAliveInterval { get; set; }
}
```
These properties correspond to WinHTTP [`tcp_keepalive`](https://docs.microsoft.com/en-us/windows/win32/winsock/sio-keepalive-vals) structure.

Lastly, we have enabled use of TLS 1.3 with `WinHttpHandler` ([dotnet/runtime#58590](https://github.com/dotnet/runtime/pull/58590)). This feature is also transparent to the user, the only thing needed is Windows support.