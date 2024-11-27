#### This Revision for my overall learning about golang.


This is starting way to intilize any golang projects.
```
go mod init projectName(github.com/Username/projectName)
```

There are basically two types of modules

`go.mod`
---
 when we initlize any project what are the packages we need are stored in go.mod. All the modules which are needed to be used in project are maintained in go.mod file.

`go.sum`
 ---
 it maintains the checksum so when yu run the latest at that moment. it will create a sum 

 `go mod tidy`
  ---
  This command is used to install or reinstall latest modules which are requires in the projects.

 `go run main.go`
  ---
  This command will run the main command from the main command it will run the whole projects.

  #### Sample code for golang 

  ```
  package main

  import "fmt"

  func main() {
    // This is the comment we used single line comment
    fmt.Println("Hello, Gophers")
    
  }
  ```

  /*
  This is multiline comment we used in golang.
  */