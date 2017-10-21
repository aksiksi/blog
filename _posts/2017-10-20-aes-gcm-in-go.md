---
layout: post
title: "AES-GCM in Go"
author: {{ site.author.name }}
---

# Introduction

AES-GCM is an AEAD (Authenticated Decryption with Additional Data) scheme that allows you to both encrypt some data and authenticate *another* set of data without encrypting it. 

For example, let's say you would like to encrypt a network packet. The packet has a header, which has to be in plaintext form, and a payload, which has to be encrypted. With AES-GCM, you can encrypt the payload **and** generate a MAC of the header in one step. The receiver would then check the MAC of the header to ensure authenticity, then decrypt the payload if required.

The Go standard library provides an implementation of AES-GCM, specifically under `crypto/cipher`. The idea is to get an instance of an AES cipher and provide it to a GCM instance, resulting in AES-GCM. The `AEAD` Go interface, which is what `GCM` implements, has two important functions: `Seal()` and `Open()`. The former is used for encryption, while the latter is used for decryption.

# Example

Place the code into a file `aesgcm.go` and compile using: `go build aesgcm.go`. Note that we do not use the additional data field of AES-GCM in this example.

```go
package main

import (
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
    "fmt"
    "io"
    "os"
)

func main() {
    // Provide plaintext to encrypt as command line argument
    if len(os.Args) < 2 {
        fmt.Println("Usage: aesgcm <plaintext>")
        os.Exit(0)
    }

    fmt.Println("Plaintext = " + os.Args[1])

    // Convert plaintext from a string to a byte slice
    plaintext := []byte(os.Args[1])

    // Randomly generate a 32 byte (128 bit) AES key
    key := make([]byte, 32)
    if _, err := io.ReadFull(rand.Reader, key); err != nil {
        panic(err.Error())
    }

    fmt.Printf("Key = %x\n", key)

    // Get an AES cipher instance using the key
    block, err := aes.NewCipher(key)
    if err != nil {
        panic(err.Error())
    }

    // Generate a 12 byte nonce for GCM
    nonce := make([]byte, 12)
    if _, err := io.ReadFull(rand.Reader, nonce); err != nil {
        panic(err.Error())
    }

    // Get a GCM instance that uses the AES cipher
    aesgcm, err := cipher.NewGCM(block)
    if err != nil {
        panic(err.Error())
    }

    // Perform AES-GCM encryption with generated nonce
    ciphertext := aesgcm.Seal(nil, nonce, plaintext, nil)
    fmt.Printf("Ciphertext = %x\n", ciphertext)

    // Now perform decryption of the ciphertext
    plaintextNew, err := aesgcm.Open(nil, nonce, ciphertext, nil)
    if err != nil {
        panic(err.Error())
    }

    // This should be equivalent to the original plaintext provided
    fmt.Printf("Decrypted Plaintext = %s\n", plaintextNew)
}
```
