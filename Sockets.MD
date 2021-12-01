## Socket-level improvements

### Handle port exhaustion by utilizing auto-reuse port range on Windows
When opening concurrent HTTP/1.1 connections on a large scale, you may notice that new connection attempts start failing after some time. On Windows, this happens typically around ~16K concurrent connections, with socket error 10055 (`WSAENOBUFS`) as an internal `SocketException` message.
Normally, the network stack chooses a port that isn’t already bound to another socket, meaning that the maximum number of simultaneusly open connections is limited by the [dynamic port range](https://docs.microsoft.com/en-us/windows/client-management/troubleshoot-tcpip-port-exhaust#default-dynamic-port-range-for-tcpip).
This is a configurable range, typically defaulting to `49152-65535` and a theoretical limit of 2<sup>16</sup>=65536 ports, since a port is a 16 bit number.

To address this problem for the case when remote endpoints differ in IP addresses and/or ports, Windows [introduced](https://support.microsoft.com/en-us/topic/reliability-and-scalability-improvements-in-tcp-ip-for-windows-8-1-and-windows-server-2012-r2-82b226f4-cadc-7676-67db-2195516b7956) a feature called **auto-reuse port range** back in the Windows 8.1 times. .NET framework exposed the related socket option `SO_REUSE_UNICASTPORT` through an opt-in property [`ServicePointManager.ReusePort`](https://docs.microsoft.com/en-us/dotnet/api/system.net.servicepointmanager.reuseport), but this property bacame a no-op API on .NET Core / .NET 5+. Instead, in [dotnet/runtime#48219](https://github.com/dotnet/runtime/issues/54903) we enabled `SO_REUSE_UNICASTPORT` for all outgoing asynchronous `Socket` connections on .NET 6+, allowing ports to be reused between connections as long as:
- The connection’s full 4-tuple of (local port, local address, remote port, remote address) is unique.
- The auto-reuse port range is configured on the machine.

You can set the auto-reuse port range with the following PowerShell cmdlet:

```powershell
Set-NetTCPSetting -SettingName InternetCustom `
                  -AutoReusePortRangeStartPort <start-port> `
                  -AutoReusePortRangeNumberOfPorts <number-of-ports>
```

A reboot is required for the setting to take effect.

From the authors of the Windows feature:

> Due to sticky backward-compatibility problems, the auto-reuse port range must be used exclusively for outbound connections using this special logic. That means if the auto-reuse port range is configured to overlap with a well-known listen port (port 80, for instance), then an attempt to bind a listening socket to that port will fail. Also, if the auto-reuse port range fully covers the regular ephemeral port range, then normal wildcard binds will fail. Normally, choosing an auto-reuse range that is a strict subset of the default ephemeral port range will avoid problems. But the admin must still be careful, because some applications use large port numbers inside the ephemeral port range as "well-known" port numbers.

### An option to globally disable IPv6

Since .NET 5, we are using [DualMode](https://docs.microsoft.com/en-us/dotnet/api/system.net.sockets.socket.dualmode) sockets in `SocketsHttpHandler`.
This allows us to handle IPv4 traffic from an IPv6 socket, and is considered to be a favorable practice by [RFC1933](https://tools.ietf.org/html/rfc1933).
On the other hand, we had several reports from users having trouble connecting through VPN tunnels which do not support IPv6 and/or dual-stack sockets properly.
To mitigate these issues and other potential problems with IPv6,
[dotnet/runtime#55012](https://github.com/dotnet/runtime/pull/55012) implemented a switch to disable IPv6 globally for the entire .NET 6 process.

You can now set the environment variable `DOTNET_SYSTEM_NET_DISABLEIPV6` to `1` or the `System.Net.DisableIPv6` [runtime configuration setting](https://docs.microsoft.com/en-us/dotnet/core/run-time-config/) to `true` if you experience similar problems and decide to address them by disabling IPv6.

### New Span- and Task-based overloads in `System.Net.Sockets`

With the help of the community, we managed to bring `Socket` and related types close to API-complete in terms of `Span`, `Task`, and cancellation support.
The complete API-diff is  way too long to include into this blog post, you can find it [here](https://github.com/dotnet/core/blob/main/release-notes/6.0/api-diff/.Net/6.0.0_System.Net.Sockets.md).
We would like to thank [@gfoidl](https://github.com/gfoidl), [@ovebastiansen](https://github.com/ovebastiansen) and [@PJB3005](https://github.com/PJB3005) for their contributions!
