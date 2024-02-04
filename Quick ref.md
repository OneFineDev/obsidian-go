# Strings

Formats:
![[Pasted image 20240203110502.png]]

# Pointers

**Reference types:**
- slices
- maps
- functions
- chans
This means their references are passed into functions, so no need to deref, as is the case with structs, strings, etc.

# Interface/Struct Embedding

``` go
type Reader interface {
Read(p []byte) (n int, err error)
}
type Writer interface {
Write(p []byte) (n int, err error)
}

type ReadWriter interface {
Reader
Writer
}
```

``` go
type _Reader struct{} // Implements IReader
type _Writer struct{} // Implements IWriter

type ReadWriter struct {
*_Reader
*_Writer
}

```

So, why would you use composition instead of just adding a struct field? The answer is that when a type is embedded, its exported properties and methods are promoted to the embedding type, allowing them to be directly invoked. For example, the Read method of a bufio.Reader is accessible directly from an instance of bufio.ReadWriter:
``` go
var rw *bufio.ReadWriter = GetReadWriter()
var bytes []byte = make([]byte, 1024)
n, err := rw.Read(bytes) {
// Do something
}
```




# Channels
context.Context for timeouts 
channels for communicating results 
select to catch whichever one acts first