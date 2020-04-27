# Checking Whether a File Exists

I didn't just learn this, but for some reason I always have to look this idiom up.

## Example

The following Go code will check if the specified file exists or not:

```go
package main

import "fmt"
import "os"

func main() {

  if _, err := os.Stat("file-exists.go"); err == nil {
    fmt.Printf("File exists\n");  
  } else {
    fmt.Printf("File does not exist\n");  
  }
}
```

## Error Checking

If you want to check if a file exists before continuing the program:

```go
package main

import "fmt"
import "os"

func main() {

  if _, err := os.Stat("file-exists2.file"); os.IsNotExist(err) {
    fmt.Printf("File does not exist\n");  
  }

  // continue program
}
```

_(from https://golangr.com/file-exists/)_