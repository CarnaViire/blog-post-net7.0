# Disable Canonicalization in URI

`HttpClient` uses `System.Uri` which does validation and canonicalization per [RFC 3986](https://datatracker.ietf.org/doc/html/rfc3986) and modifies some of the URIs in a way that might break their end-customers. For example, larger services or SDKs might need to pass a URI transparently from their source (e.g. Kestrel) to `HttpClient`, which was impossible in .NET 5.0 (see [dotnet/runtime#52628](https://github.com/dotnet/runtime/issues/52628), [dotnet/runtime#58057](https://github.com/dotnet/runtime/issues/58057)).

.NET 6.0 is introducing a new API flag [`UriCreationOptions.DangerousDisablePathAndQueryCanonicalization`](https://docs.microsoft.com/en-US/dotnet/api/system.uricreationoptions.dangerousdisablepathandquerycanonicalization?view=net-6.0) (see [dotnet/runtime#59274](https://github.com/dotnet/runtime/pull/59274)) which will allow the user to disable any canonicalization on URI and take it "as is".

Setting `DangerousDisablePathAndQueryCanonicalization` means no validation and no transformation of the input will take place past the authority. As a side effect, `Uri` instances created with this option do not support `Uri.Fragment`s -- it will always be empty. Moreover, `Uri.GetComponents(UriComponents, UriFormat)` may not be used for `UriComponents.Path` or `UriComponents.Query` and will throw `InvalidOperationException`.

Be aware that disabling canonicalization also means that reserved characters will not be escaped (e.g. space characters will not be changed to `%20`), which may corrupt the HTTP request and makes the application subject to request smuggling. Only set this option if you have ensured that the URI string is already sanitized.

```c#
var uriString = "http://localhost/path%4A?query%4A#/foo";

var options = new UriCreationOptions { DangerousDisablePathAndQueryCanonicalization = true };
var uri = new Uri(uriString, options);
Console.WriteLine(uri); // outputs "http://localhost/path%4A?query%4A#/foo"
Console.WriteLine(uri.AbsolutePath); // outputs "/path%4A"
Console.WriteLine(uri.Query); // outputs "?query%4A#/foo"
Console.WriteLine(uri.PathAndQuery); // outputs "/path%4A?query%4A#/foo"
Console.WriteLine(uri.Fragment); // outputs an empty string

var canonicalUri = new Uri(uriString);
Console.WriteLine(canonicalUri.PathAndQuery); // outputs "/pathJ?queryJ"
Console.WriteLine(canonicalUri.Fragment); // outputs "#/foo"
```

Note: The API is part of a larger API surface we designed for 7.0 (see [dotnet/runtime#59099](https://github.com/dotnet/runtime/issues/59099)).