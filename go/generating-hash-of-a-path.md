# Generating a Hash From Files in a Path

_Source: https://github.com/sger/go-hashdir/blob/master/hashdir.go (Heavily modified)_

The `Walk` method from `path/filepath` walks the file tree rooted at root, calling `walkFn` for each file or directory in the tree, including root. All errors that arise visiting files and directories are filtered by `walkFn`. The files are walked in lexical order, which makes the output deterministic but means that for very large directories Walk can be inefficient. Walk does not follow symbolic links.

```go
package main

import (
    "crypto/sha256"
    "encoding/hex"
    "io"
    "os"
    "path/filepath"
)

// Create hash value with local path and a hash algorithm
func Create(path string, hashAlgorithm string) (string, error) {
    hash := sha256.New()

    walkFn := func(path string, info os.FileInfo, err error) error {
        if err != nil {
            return nil
        }

        // Ignore directories
        if info.IsDir() {
            return nil
        }

        // Open the file for reading
        file, err := os.Open(path)
        if err != nil {
            return err
        }

        // Make sure the file is closed when the function returns
        defer file.Close()

        // Copy the file in the hash interface and check for any error
        if _, err := io.Copy(hash, file); err != nil {
            return "", err
        }

        return nil
    }

    // Walk through the path, and generate a hash from each file.
    err = filepath.Walk(path, walkFn)

    if err != nil {
        return "", nil
    }

    bytes := hash.Sum(nil)
    encoded := hex.EncodeToString(hashInBytes)
}

```
