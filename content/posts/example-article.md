---
title: "Example Article (tech)"
date: 2020-07-06T09:49:54+12:00
---


A few years ago I [learned Go](https://benhoyt.com/writings/learning-go/) by porting the server for my [Gifty Weddings](https://giftyweddings.com/) side gig from Python to Go. It was a fun way to learn the language, and took me about "two weeks of bus commutes" to learn Go at a basic level and port the code.

Since then, I've really enjoyed working with the language, and have used it extensively at work as well as on side projects like [GoAWK](/writings/goawk/) and [zztgo](/writings/zzt-in-go/). Go usage at [Compass.com](https://www.compass.com/), my current workplace, has grown significantly in the time I've been there -- around half of our 200 plus services are written in Go.

This article describes what I think are some of the great things about Go, gives a very brief overview of the standard library, and then digs into the core language. But if you just want a feel for what real Go code looks like, skip to the [HTTP server examples](#http-server-examples).

<!--more-->



Why Go?
-------

As the following [Google Trends chart](https://trends.google.com/trends/explore?date=2010-01-01%202020-06-09&q=golang&hl=en-US) shows, Go has become very popular over the past few years, partly because of the simplicity of the language, but perhaps more importantly because of the excellent tooling.

![Google Trends data for "golang" from 2010 to 2020](https://benhoyt.com/images/golang-trend.png)

Here are some of the reasons I enjoy programming in Go (and why you might like it too):

* **Small and simple core language.** Go feels similar in size to C, with a very readable [language spec](https://golang.org/ref/spec) that's only about 50 pages long (compared to the [Java spec's](https://docs.oracle.com/javase/specs/jls/se12/jls12.pdf) 770 pages). This makes it easy to learn or teach to others.
* **High quality standard library**, especially for servers and network tasks. More details [below](#the-standard-library).
* **First class concurrency** with *goroutines* (like threads, but lighter) and the `go` keyword to start a goroutine, *channels* for communicating between them, and a runtime whose scheduler coordinates all this.
* **Compiles to native code**, producing easy-to-deploy binaries on all the major platforms.
* **Garbage collection** that doesn't require knob-tweaking ([optimized](https://blog.golang.org/ismmkeynote) for low latency).
* **Statically typed**, but has type inference to avoid a lot of "type stuttering".
* **Great documentation** that is succinct and includes many runnable examples.
* **Excellent tooling.** Just type `go build` to build your project, `go test` to find and run your tests, etc. There's CPU and memory profiling, code coverage, and cross compilation -- all without external tooling.
* **Fast compile times.** The language was designed from day one with fast compile times in mind. In fact, co-creator Rob Pike [jokes](https://www.informit.com/articles/article.aspx?p=1623555) that "Go was conceived while waiting for a big [C++] compilation."
* **Very stable** language and library, with a strict [compatibility promise](https://golang.org/doc/go1compat) that all Go 1 programs will run unchanged on later versions of Go 1.x.
* **Desired.** According to StackOverflow's 2019 survey, it's the [third most wanted](https://insights.stackoverflow.com/survey/2019#most-loved-dreaded-and-wanted) programming language, so it's easy to hire developers who want to use it.
* **Heavily used in cloud tools.** Docker and Kubernetes are written in Go, and Dropbox, Digital Ocean, Cloudflare, and many other companies use it extensively.


The standard library
--------------------

Go's [standard library](https://golang.org/pkg/) is extensive, cross-platform, and well documented. Similar to Python, Go comes with "batteries included", so you can build useful servers and CLI tools right away, without any third party dependencies. Here are some of the highlights (biased towards what I've used):

* Input/output: [OS calls](https://golang.org/pkg/os/), files and directories, [buffered I/O](https://golang.org/pkg/bufio/).
* HTTP: a production-ready [client and server](https://golang.org/pkg/net/http/), TLS, HTTP/2, simple routing, URL and cookie parsing.
* Strings: [all the basics](https://golang.org/pkg/strings/), handling of [raw bytes](https://golang.org/pkg/bytes/), [unicode conversions](https://golang.org/pkg/unicode/).
* Encodings: [JSON](https://golang.org/pkg/encoding/json/), [XML](https://golang.org/pkg/encoding/xml/), [CSV](https://golang.org/pkg/encoding/csv/), [base64](https://golang.org/pkg/encoding/base64/), [hex](https://golang.org/pkg/encoding/hex/), [binary](https://golang.org/pkg/encoding/binary/), more.
* Templating: simple but powerful [text](https://golang.org/pkg/text/template/) and auto-escaped [HTML](https://golang.org/pkg/html/template/) templates.
* Time: simple API but well thought out [date and time](https://golang.org/pkg/time/) functions.
* Regular expressions: a non-backtracking [regexp library](https://golang.org/pkg/regexp/).
* Sorting: generic collection [sorting functions](https://golang.org/pkg/sort/).
* Databases: the [`database/sql`](https://golang.org/pkg/database/sql/) interface, with specific implementations left up to third party libraries.
* Crypto libraries: secure and fast [implementations](https://golang.org/pkg/crypto/) of AES, block ciphers, cryptographic hashes, etc.
* Image: read and write [JPEG](https://golang.org/pkg/image/jpeg/), [PNG](https://golang.org/pkg/image/png/), and [GIF](https://golang.org/pkg/image/gif/), perform basic [compositing](https://golang.org/pkg/image/draw/).
* Big numbers: arbitrary-precision [int and float](https://golang.org/pkg/math/big/).
* Archives and compression: [tar](https://golang.org/pkg/archive/tar/), [zip](https://golang.org/pkg/archive/zip/), [gzip](https://golang.org/pkg/compress/gzip/), [bzip2](https://golang.org/pkg/compress/bzip2/), etc.
* Simple command-line [flag](https://golang.org/pkg/flag/) library.
* Go source code tools: [parser](https://golang.org/pkg/go/parser/), [AST](https://golang.org/pkg/go/ast/), code [formatting](https://golang.org/pkg/go/printer/).
* Reflection: powerful run-time [reflection support](https://golang.org/pkg/reflect/).

In terms of third party packages, typical Go philosophy is almost the opposite of JavaScript's approach of pulling in `npm` packages left, right, and center. Russ Cox (tech lead of the Go team at Google) talks about [our software dependency problem](https://research.swtch.com/deps), and Go co-creator Rob Pike [likes to say](https://go-proverbs.github.io/), "A little copying is better than a little dependency." So it's fair to say that most Gophers are pretty conservative about using third party libraries.

That said, since I originally wrote this talk, the Go team has designed and built [modules](https://blog.golang.org/using-go-modules), the Go team's official answer to how you should manage and version-pin your dependencies. I've found it pleasant to use, and it works with all the normal `go` sub-commands.


Language features
-----------------

So let's dig in to what Go itself looks like, and walk through the language proper.


### Hello world

Go has a C-like syntax, mandatory braces, and no semicolons (except in the formal grammar). Projects are structured via imports and packages -- compilation units that consist of a directory with one or more `.go` files in it. Here's what a "hello world" looks like:

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, world!")
}
```

-------------

[Read the real article here ...](https://benhoyt.com/writings/go-intro/)
