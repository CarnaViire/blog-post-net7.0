# HTTP/3 and QUIC

In .NET 5.0, we released an experimental implementation of [QUIC and HTTP/3](https://devblogs.microsoft.com/dotnet/net-5-new-networking-improvements/#http-3). It was limited only to Insider build of Windows and there was quite a bit of ceremony to get it working.

In .NET 6.0, we have significantly simplified the setup. On Windows, we ship [MsQuic](https://github.com/microsoft/msquic) library as part of the runtime, so there's no need to download or reference anything external. The only limitation is that Windows 11 or Windows Server 2022 is required. This is due to TLS 1.3 support for QUIC in SChannel that is not available in earlier Windows versions. On Linux, we publish MsQuic as a standard Linux package `libmsquic` (deb and rpm) in [Microsoft Package Repository](https://docs.microsoft.com/en-us/windows-server/administration/linux-package-repository-for-microsoft-software). The reason for not bundling MsQuic with runtime on Linux is that we ship `libmsquic` with its own [QUIC TLS OpenSSL](https://github.com/quictls/openssl) version providing necessary APIs. And since we bundle OpenSSL with MsQuic, we need to be able to do security patches outside of the normal .NET release schedule.

We have also greatly improved stability and implemented a lot of missing features with around [90 issues closed](https://github.com/dotnet/runtime/issues?q=is%3Aissue+project%3Adotnet%2Fruntime%2F2+is%3Aclosed+milestone%3A6.0.0+) in .NET 6.0 milestone. Unfortunately, we weren't able to finalize API surface for `System.Net.Quic` library. This library represents .NET implementation of QUIC protocol (transport layer used by HTTP/3). In our case, it's a managed layer built on top of MsQuic. We didn't have enough confidence that the current shape of API would stand the test of time, so we decided to keep it private in this release. As a result, .NET 6.0 contains QUIC protocol implementation, but doesn't expose it. It's only used internally for HTTP/3 in `HttpClient` and in Kestrel server.

Despite putting a lot of effort in bug squishing in this release, we still don't think the quality is fully production ready. And since any HTTP request could inadvertently be upgraded to HTTP/3 via [`Alt-Svc` header](https://www.ietf.org/archive/id/draft-ietf-quic-http-34.html#name-http-alternative-services) and start failing, we've opted to keep HTTP/3 capability disabled by default in this release. In `HttpClient`, it's hidden behind `System.Net.SocketsHttpHandler.Http3Support` `AppContext` switch.


All the details how to set everything up have already been described in our previous articles: [`HttpClient`](https://devblogs.microsoft.com/dotnet/http-3-support-in-dotnet-6/) and [Kestrel](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/http3?view=aspnetcore-6.0). On Linux, acquire `libmsquic` package, on Windows, make sure the OS version is at least 10.0.20145.1000. Then, you only need to enable HTTP/3 support and set the `HttpClient` to use HTTP/3:
```C#
using System.Net;

// Set this switch programmatically or in csproj:
// <RuntimeHostConfigurationOption Include="System.Net.SocketsHttpHandler.Http3Support" Value="true" />
AppContext.SetSwitch("System.Net.SocketsHttpHandler.Http3Support", true);

// Set up the client to request HTTP/3.
var client = new HttpClient()
{
    DefaultRequestVersion = HttpVersion.Version30,
    DefaultVersionPolicy = HttpVersionPolicy.RequestVersionOrHigher,
};
var resp = await client.GetAsync("https://<http3 endpoint>");

// Print the response version.
Console.WriteLine($"status: {resp.StatusCode}, version: {resp.Version}");
```

We encourage you to try HTTP/3 out! And in case you encounter any problem, please file an issue in [dotnet/runtime](https://github.com/dotnet/runtime/issues).