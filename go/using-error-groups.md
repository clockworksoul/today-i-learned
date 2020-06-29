# Using error groups in Go

_Source: [[1]](https://godoc.org/golang.org/x/sync/errgroup#WithContext)[[2]](https://www.oreilly.com/content/run-strikingly-fast-parallel-file-searches-in-go-with-sync-errgroup/)[[3]](https://bionic.fullstory.com/why-you-should-be-using-errgroup-withcontext-in-golang-server-handlers/)_

The `golang.org/x/sync/errgroup` package provides synchronization, error propagation, and Context cancelation for groups of goroutines working on subtasks of a common task.

Most interestingly, it provides the `Group` type, which represents an _error group_: a collection of goroutines working on subtasks that are part of the same overall task.

## `errgroup.Group`

The `Group` type represents a collection of goroutines working on subtasks that are part of the same overall task.

It has two attached methods: `Go()` and `Wait()`.

### Go()

```go
func (g *Group) Go(f func() error)
```

`Go()` calls the given function in a new goroutine.

The first call to return a non-nil error cancels the group; its error will be returned by `Wait()`.

### Wait()

```go
func (g *Group) Wait() error
```

`Wait()` blocks until all function calls from the `Go()` method have returned, then returns the first non-nil error (if any) from them.

## Creating an `errgroup.Group`

An error group can be created in one of two ways: with or without a `Context`.

### With context

A `Group` can be created using the `WithContext()` function, which returns a new `Group` and an associated `Context` derived from its `ctx` parameter.

The derived `Context` is canceled the first time a function passed to `Go()` returns a non-nil error or the first time `Wait()` returns, whichever occurs first.

```go
func WithContext(ctx context.Context) (*Group, context.Context)
```

### A zero Group

A zero `Group` is valid and does not cancel on error.

```go
g := new(errgroup.Group)
```

or simply

```go
g := &errgroup.Group{}
```

## Using `errgroup.Group`

You can use a `Group` as follows:

1. Create the `Group`, ideally using a `Context`.
2. Pass the `Group` one or more functions of type `func() error` to the `Go()` method to execute concurrently.
3. Call `Wait()` method, which blocks until one of the functions has an error or all functions complete.

### Generation 1: Serial Service Calls

```go
func main() {
    err := SendRequestToService()
    if err != nil {
        log.Fatal(err)
    }
}
```

### Generation 2: Using an error group

```go
func main() {
    g, ctx := errgroup.WithContext(context.Background())

    g.Go(func() error {
        return SendRequestToService()
    })

    err := g.Wait()
    if err != nil {
        // Note that ctx is automatically canceled!
        log.Fatal(err)
    }
}
```
