- HTTP
    - ~~reworked connection pooling~~
        - https://github.com/dotnet/runtime/issues/44818
        - https://github.com/dotnet/runtime/pull/53851
        - https://github.com/dotnet/runtime/pull/56062
        - https://github.com/dotnet/runtime/pull/56966
    - reworked retry logic
        - https://github.com/dotnet/runtime/issues/44669
        - https://github.com/dotnet/runtime/pull/48758
    - ~~H/2 window update scaling (Anton promised help, at least provide benchmark data)~~
        - https://github.com/dotnet/runtime/issues/43086
        - https://github.com/dotnet/runtime/pull/54755
    - support for SOCKS proxy (community)
        - https://github.com/dotnet/runtime/issues/17740
        - https://github.com/dotnet/runtime/pull/48883
    - WinHTTP
        - H/2 trailing headers (for gRPC)
            - https://github.com/dotnet/runtime/issues/44778
        - H/2 bidirectional streaming
            - https://github.com/dotnet/runtime/issues/44784
        - TCP Keep Alive
            - https://github.com/dotnet/runtime/pull/44889
        - TLS 1.3 on Win 11
            - https://github.com/dotnet/runtime/issues/58587
    - ~~Non validated headers enumeration~~
        - https://github.com/dotnet/runtime/issues/35126
        - https://github.com/dotnet/runtime/pull/53555
    - Get all cookies (avoid ugly hack https://github.com/dotnet/runtime/issues/44094#issuecomment-820262017)
        - https://github.com/dotnet/runtime/issues/44094
    - HTTP/3 and QUIC
        - many PRs and issues
    - ~~ZLib deflate decompression for response content~~
        - https://github.com/dotnet/runtime/issues/38022
        - https://github.com/dotnet/runtime/pull/42717
        - https://github.com/dotnet/runtime/pull/57862
    - ~~Huffman en/decoding optimization~~
        - https://github.com/dotnet/runtime/pull/43603
        - https://github.com/dotnet/runtime/pull/45303
    - Obsoletions
        - https://github.com/dotnet/runtime/issues/33125
- ~~WebSockets~~
    - ~~compression (community)~~
        - https://github.com/dotnet/runtime/issues/31088
        - https://github.com/dotnet/runtime/pull/49304
        - https://devblogs.microsoft.com/dotnet/announcing-net-6-preview-5/#libraries-websocket-compression
- Sockets
    - ~~Task/Span-based async API (Anton promised help with complete list)~~
        - mention community help here
        - https://github.com/dotnet/runtime/issues/42591
        - https://github.com/dotnet/runtime/issues/51452
    - Improvement for port exhaustion problem with SocketsHttpHandler
        - Although strictly speaking this is a socket topic, it’s most useful for HttpClient scalability, since it can help avoiding port exhaustion for advanced users who are willing to tweak their VM-s with PowerShell commands. Blog post is a nice opportunity to document and communicate the trick.
        - https://github.com/dotnet/runtime/issues/48219
    - IPv6 killswitch
        - https://github.com/dotnet/runtime/issues/47583
        - https://github.com/dotnet/runtime/pull/55012
- Security
    - TLS delayed client negotiation
        - https://github.com/dotnet/runtime/issues/49346
        - https://github.com/dotnet/runtime/pull/51905
    - Windows only, send list of trusted CAs in TLS handshake
        - https://github.com/dotnet/runtime/issues/45456
        - https://github.com/dotnet/runtime/pull/55104
    - impersonification fixes
        - https://github.com/dotnet/runtime/pull/47435
        - https://github.com/dotnet/runtime/pull/59155
- Diagnostics
    - OpenTelemetry propagators
        - https://github.com/dotnet/runtime/pull/55392
- URI
    - Don't escape URI
        - https://github.com/dotnet/runtime/issues/58057
        - https://github.com/dotnet/runtime/pull/59274
- ~~DNS~~
    - ~~Cancellable and address specific name resolution~~
        - https://github.com/dotnet/runtime/issues/939
        - https://github.com/dotnet/runtime/pull/33420

https://microsoft.sharepoint.com/teams/netfx/corefx/_layouts/15/Doc.aspx?sourcedoc=%7B0cfbc196-0645-4781-84c6-5dffabd76bee%7D&action=edit&wd=target(Networking.one%7C7f7a68c9-55b6-465a-9c30-bab26b5f7d82%2F6.0%20features%7Cffda5285-ba9c-40e2-9bfe-230a912c5681%2F)&share=IgGWwfsMRQaBR4TGXf-r12vuAatLtaaTfyKdaMWZg7LmBVU

https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-6/#networking