# Diagnostics

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