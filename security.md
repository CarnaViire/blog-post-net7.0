# Security

In .NET 6.0, we have made two smaller changes worth mentioning in networking security space.

First is delayed client negotiation. This is a (mostly) server-side `SslStream` function. It is used when the server decides that it needs to renegotiate encryption for already established connection. For example, when client accesses a resource that needs a client certificate that hasn't been initially provided.

The new `SslStream` method looks like this:
```C#
public virtual Task NegotiateClientCertificateAsync(CancellationToken cancellationToken = default);
```

The implementation uses two different TLS features depending on the TLS version. For TLS up to 1.2, TLS renegotiation is used ([RFC 5746](https://www.rfc-editor.org/rfc/rfc5746.html)). For TLS 1.3, post handshake authentication extension is used ([RFC 8446](https://datatracker.ietf.org/doc/html/rfc8446#section-4.2.6)). Those two feature are abstracted in SChannel [`AcceptSecurityContext`](https://docs.microsoft.com/en-us/windows/win32/secauthn/acceptsecuritycontext--schannel) function. Thus, delayed client negotiation is fully supported on Windows. Unfortunately, with OpenSSL the story is different and therefore the support is limited to TLS renegotiation, i.e. TLS up to 1.2, on Linux. Moreover, MacOS is not supported at all since its security layer doesn't provide either of those.

Note that neither TLS renegotiation nor post handshake authentication extension are allowed with HTTP/2 ([RFC 8740](https://datatracker.ietf.org/doc/html/rfc8740)) since it multiplexes multiple requests over one connection.


The other security change is proper impersonation implementation. This is Windows only feature where single process can have threads running under different users via [`WindowsIdentity.RunImpersonatedAsync`](https://docs.microsoft.com/en-us/dotnet/api/system.security.principal.windowsidentity.runimpersonatedasync?view=net-6.0). We did not behaved well in two situation which we properly fixed in .NET 6.0. The first one was when doing asynchronous name resolution ([dotnet/runtime#47435](https://github.com/dotnet/runtime/pull/47435)). And the other was when sending HTTP requests where we would honor the impersonated user ([dotnet/runtime#58033](https://github.com/dotnet/runtime/issues/58033)).