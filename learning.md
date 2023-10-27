# Go-verview

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
