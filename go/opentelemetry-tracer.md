# OpenTelemetry: Tracer

_Source: [[1]](https://opentelemetry.io/)[[2]](https://github.com/open-telemetry/opentelemetry-go/blob/master/README.md)[[3]](https://docs.google.com/presentation/d/1nVhLIyqn_SiDo78jFHxnMdxYlnT0b7tYOHz3Pu4gzVQ/edit?usp=sharing)_

OpenTelemetry provides a single set of APIs, libraries, agents, and collector services to capture distributed traces and metrics from your application. You can analyze them using Prometheus, Jaeger, and other observability tools.

## Tracer

A Tracer is responsible for tracking the currently active span.

## Quickstart

Below is a brief example of importing OpenTelemetry, initializing a `tracer` and creating some simple spans.

```go
package main

import (
	"context"
	"log"

	"go.opentelemetry.io/otel/api/global"
	"go.opentelemetry.io/otel/exporters/trace/stdout"
	sdktrace "go.opentelemetry.io/otel/sdk/trace"
)

func initTracer() {
	exporter, err := stdout.NewExporter(stdout.Options{PrettyPrint: true})
	if err != nil {
		log.Fatal(err)
    }
    
	tp, err := sdktrace.NewProvider(sdktrace.WithConfig(sdktrace.Config{DefaultSampler: sdktrace.AlwaysSample()}),
		sdktrace.WithSyncer(exporter))
	if err != nil {
		log.Fatal(err)
    }
    
	global.SetTraceProvider(tp)
}

func main() {
    initTracer()
    
	tracer := global.Tracer("ex.com/basic")

	tracer.WithSpan(context.Background(), "foo",
		func(ctx context.Context) error {
			tracer.WithSpan(ctx, "bar",
				func(ctx context.Context) error {
					tracer.WithSpan(ctx, "baz",
						func(ctx context.Context) error {
							return nil
						},
					)
					return nil
				},
			)
			return nil
		},
	)
}
```

## TraceProvider

A global provider can have a TraceProvider registered.
Use the TraceProvider to create a named tracer.

```go
// Register your provider in your init code
tp, err := sdktrace.NewProvider(...)
global.SetTraceProvider(tp)

// Create the named tracer
tracer = global.TraceProvider().Tracer("workshop/main")
```

## OTel API - Tracer methods & when to call

```go
// This method returns a child of the current span, and makes it current.
tracer.Start(ctx, name, options)

// Starts a new span, sets it to be active in the context, executes the wrapped
// body and closes the span before returning the execution result.
tracer.WithSpan(name, func() {...})

// Used to access & add information to the current span
trace.SpanFromContext(ctx)
```
## OTel API - Span methods, & when to call

```go
// Adds structured annotations (e.g. "logs") about what is currently happening.
span.AddEvent(ctx, msg)

// Adds a structured, typed attribute to the current span. This may include a user id, a build id, a user-agent, etc.
span.SetAttributes(core.Key(key).String(value)...)

// Often used with defer, fires when the unit of work is complete and the span can be sent
span.End()
```

## Code examples

### Start/End

```go
func (m Model) persist(ctx context.Context) {
	tr := global.TraceProvider().Tracer("me")
	ctx, span := tr.Start(ctx, "persist")
	defer span.End()

	// Persist the model to the database...
	[...]
}
```

### WithSpan

Takes a context, span name, and the function to be called.

```go
ret, err := tracer.WithSpan(ctx, "operation",
	func(ctx context.Context) (int, error) {
		// Your function here
		[...]
		return 0, nil
	}
)
```

### CurrentSpan & Span

```go
// Get the current span
sp := trace.SpanFromContext(ctx)

// Update the span status
sp.SetStatus(codes.OK)

// Add events
sp.AddEvent(ctx, "foo")

// Add attributes
sp.SetAttributes(
    key.New("ex.com/foo").String("yes"))
```



