# Go-verview

## Why do I care enough to present this (and maybe why one might care enough to pay attention)

### Relatively Straightforward -> Less time reading code; more time writing code.

### Doctests (similar to Python / Elixir)

```
package main

import (
    "fmt"
    "strings"
)

func Uppercase(s string) string {
    return strings.ToUpper(s)
}

func ExampleUppercase() {
    fmt.Println(Uppercase("hello"))
    // Output: HELLO
}

```
-> Example Function gives Runnable Code (see [https://pkg.go.dev](https://pkg.go.dev))

### Quick, Reproducible Builds

### Potential for Highly Modular Code and Easily Swappable Interfaces

#### Easy Open-Source Use and Contribution

#### Code Reusability

### Composition, not Inheritance -> Built into Language

## What niche does Go actually fill?

- Not Java/C#/Kotlin
- Not C++
- Not Rust

## Weird, but cool error-handling and return types

```
something, somethingErr := doSomething(someParam1, someParam2)
if somethingErr != nil {
	handleErr(somethingErr)
}
something.doOtherThing()
```

## Making CLI Apps: Cobra | Config Management: Viper

https://github.com/Adron/cobra-cli-samples.git

## 4 Different Project Structures

### The Most Common: Dump

![Poop Everywhere](poop_everywhere.jpg)

### Recommended: cmd | pkg/bases | pkg/components

```
C:.
│   .gitignore
│   go.mod
│   go.sum
│   learning.md
│   poop_everywhere.jpg
│   README.md
│
├───cmd
│   ├───app
│   │   │   main.go
│   │   │   README.md
│   │   │
│   │   └───config
│   │           dev.yaml
│   │
│   └───cobra-app
│       │   LICENSE
│       │   main.go
│       │   README.md
│       │
│       └───cmd
│               root.go
│
└───pkg
    ├───bases
    │   └───greet
    │       │   interface.go
    │       │   interface_test.go
    │       │   README.md
    │       │
    │       └───internal
    │               core.go
    │               core_test.go
    │
    └───components
        └───greet
            │   interface.go
            │   interface_test.go
            │   README.md
            │
            └───internal
                    core.go
                    core_test.go
```

### Large-Project Standard: (cmd | pkg)

### Old: (cmd | pkg | vendor)

## CI/CD

#### Sample Dockerfile

```
FROM golang:1.19 as builder

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build ./cmd/appName -o appName .

FROM alpine:latest
WORKDIR /root/
COPY --from=builder /app/appName .
CMD ["./appName"]

```

### GH Actions

#### Basic Build

```
name: Go_Build_Test

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:

  build:
    name: Build
    runs-on: <runner>
    steps:

    - name: Set up Go 1.19
      uses: actions/setup-go@v2
      with:
        go-version: ^1.19

    - name: Check out code into the Go module directory
      uses: actions/checkout@<version tag or hash>

    - name: Test
      run: go test -v <path to test stuff>

    - name: Build
      run: go build -v <path to build stuff>

```

## Future Research

### Bazel/Gazelle

### Gradle/Maven/Mix/Leiningen/NPM Equivalent
