---
date: '2025-03-18T17:20:54Z'
draft: false
title: 'Must pattern in Go and a generics-based generalization'
tags:
- golang
- snippets
---

There is a handy code pattern commonly found in Go codebases, known as Must pattern. To put it simply, for a given `Do` function returning at least an error, the `MustDo` function then omits returning the error, panicking instead. For instance:

```go
// package regexp
func Compile(expr string) (*Regexp, error)
func MustCompile(expr string) *Regexp

// package netip
func ParseAddr(s string) (Addr, error)
func MustParseAddr(s string) Addr
```

In contexts where errors are unlikely to happen, it is a nice way to still handle them while uncluttering the code. Of course this implies the application of the same good practices as when using `panic`. Those functions are often used to create constant-like objects, such as regular expressions:

```go
var reSlug = regexp.MustCompile(`^[a-z0-9-]{3,}$`)

func isSlug(s string) bool { return reSlug.MatchString(s) }
```

They can also be useful for writing quick and dirty code, testing code or any code where there is a high confidence in the result of the execution. To adapt other functions to behave as a `Must...` variant, you can use the code snippet below. For the record, this is basically the generic version of `text/template.Must`.

```go
// must is a helper that wraps a call to a function returning
// (T, error) and panics if the error is non-nil. It is
// intended for use in variable initializations such as:
//
//	var db = must(os.Open("file.go"))
func must[T any](value T, err error) T {
	if err != nil {
		panic(err)
	}
	return value
}
```
