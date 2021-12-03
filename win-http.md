# `WinHttpHandler` additions

- WinHTTP
        - H/2 trailing headers (for gRPC)
            - https://github.com/dotnet/runtime/issues/44778
        - H/2 bidirectional streaming
            - https://github.com/dotnet/runtime/issues/44784
        - TCP Keep Alive
            - https://github.com/dotnet/runtime/pull/44889
        - TLS 1.3 on Win 11
            - https://github.com/dotnet/runtime/issues/58587