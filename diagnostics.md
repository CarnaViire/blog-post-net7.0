# Diagnostics

We got many questions, complaints and bug reports about the default behavior of [`HttpClient`](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.httpclient?view=net-6.0) in regards to [`Activity`](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.activity?view=net-6.0) creation ([dotnet/runtime#41072](https://github.com/dotnet/runtime/issues/41072)) and automatic trace header injection ([dotnet/runtime#35337](https://github.com/dotnet/runtime/issues/35337)). These problems were even more pronounced in ASP.NET Core projects where an `Activity` is created automatically, inadvertently turning on `DiagnosticsHandler`, which is part of `HttpClient` handler chain. Moreover, `DiagnosticsHandler` is an internal class without any configuration exposed via `HttpClient` thus forcing users to come up with hacky workarounds to control the behavior ([dotnet/runtime#31862](https://github.com/dotnet/runtime/issues/31862)) or just to turn it completely off ([dotnet/runtime#35337-comment](https://github.com/dotnet/runtime/issues/35337#issuecomment-864293752)).

All of these issues were addressed in .NET 6.0 ([dotnet/runtime#55392](https://github.com/dotnet/runtime/pull/55392)). The header injection can now be controlled with [`DistributedContextPropagator`](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.distributedcontextpropagator?view=net-6.0). It can either be done globally via [`DistributedContextPropagator.Current`](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.distributedcontextpropagator.current?view=net-6.0#System_Diagnostics_DistributedContextPropagator_Current) or per `HttpClient`/`SocketsHttpHandler` with [`SocketsHttpHandler.ActivityHeadersPropagator`](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.socketshttphandler.activityheaderspropagator?view=net-6.0#System_Net_Http_SocketsHttpHandler_ActivityHeadersPropagator). We have also prepared some of the most asked for implementations:
- [`NoOutputPropagator`](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.distributedcontextpropagator.createnooutputpropagator?view=net-6.0#System_Diagnostics_DistributedContextPropagator_CreateNoOutputPropagator) to suppress trace header injection.
- [`PassThroughPropagator`](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.distributedcontextpropagator.createpassthroughpropagator?view=net-6.0) to inject a trace header with the value from the root `Activity`, i.e. act transparently and send the same header value as was received by the application.

And for more granular control over the header injection, a custom `DistributedContextPropagator` might be provided. For example, one for skipping exactly the one layer emitted by `DiagnosticsHandler` (credits to [MihaZupan](https://github.com/MihaZupan)):
```C#
public sealed class SkipHttpClientActivityPropagator : DistributedContextPropagator
{
    private readonly DistributedContextPropagator _originalPropagator = Current;

    public override IReadOnlyCollection<string> Fields => _originalPropagator.Fields;

    public override void Inject(Activity? activity, object? carrier, PropagatorSetterCallback? setter)
    {
        if (activity?.OperationName == "System.Net.Http.HttpRequestOut")
        {
            activity = activity.Parent;
        }

        _originalPropagator.Inject(activity, carrier, setter);
    }

    public override void ExtractTraceIdAndState(object? carrier, PropagatorGetterCallback? getter, out string? traceId, out string? traceState) =>
        _originalPropagator.ExtractTraceIdAndState(carrier, getter, out traceId, out traceState);

    public override IEnumerable<KeyValuePair<string, string?>>? ExtractBaggage(object? carrier, PropagatorGetterCallback? getter) =>
        _originalPropagator.ExtractBaggage(carrier, getter);
}
```

And finally, to pull this all together, set up `ActivityHeadersPropagator`:
```C#
// Set up headers propagator for this client.
var client = new HttpClient(new SocketsHttpHandler() {
    // -> Turns off activity creation as well as header injection
    // ActivityHeadersPropagator = null

    // -> Activity gets created but no trace header is injected
    // ActivityHeadersPropagator = DistributedContextPropagator.CreateNoOutputPropagator()

    // -> Activity gets created, trace header gets injected and contains "root" activity id
    // ActivityHeadersPropagator = DistributedContextPropagator.CreatePassThroughPropagator()

    // -> Activity gets created, trace header gets injected and contains "parent" activity id
    // ActivityHeadersPropagator = new SkipHttpClientActivityPropagator()

    // -> Activity gets created, trace header gets injected and contains "System.Net.Http.HttpRequestOut" activity id
    // Same as not setting ActivityHeadersPropagator at all.
    // ActivityHeadersPropagator = DistributedContextPropagator.CreateDefaultPropagator()
});

// If you want the see the order of activities created, add ActivityListener.
ActivitySource.AddActivityListener(new ActivityListener()
{
    ShouldListenTo = (activitySource) => true,
    ActivityStarted = activity => Console.WriteLine($"Start {activity.DisplayName}{activity.Id}"),
    ActivityStopped = activity => Console.WriteLine($"Stop {activity.DisplayName}{activity.Id}")
});

// Set up activities, at least two layers to show all the differences.
using Activity root = new Activity("root");
// Header format can be overridden, default is W3C, see https://www.w3.org/TR/trace-context/).
// root.SetIdFormat(ActivityIdFormat.Hierarchical);
root.Start();
using Activity parent = new Activity("parent");
// parent.SetIdFormat(ActivityIdFormat.Hierarchical);
parent.Start();

var request = new HttpRequestMessage(HttpMethod.Get, "https://www.microsoft.com");

using var response = await client.SendAsync(request);
Console.WriteLine($"Request: {request}"); // Print the request to see the injected header.
```