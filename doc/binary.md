# Integers to binary and vice versa

## uint64

```go
package main

import (
	"bytes"
	"encoding/binary"
	"fmt"
	"log"
)

func printArray(array []byte) {
	for i, v := range array {
		fmt.Printf("%2d: 0x%02X\n", i, v)
	}
}

func main() {
	var buffer *bytes.Buffer
	var value1 uint64
	var value2 uint64
	var values = make([]uint64, 2)

	// Convert an unsigned value into a sequence of bytes and vice versa.

	fmt.Print("Convert an unsigned value into a sequence of bytes and vice versa\n\n")
	value1 = 0x0807060504030201
	buffer = new(bytes.Buffer)
	_ = binary.Write(buffer, binary.LittleEndian, value1)
	fmt.Printf("value1: 0x%016X (%d)\n", value1, value1)
	printArray(buffer.Bytes())

	_ = binary.Read(bytes.NewReader(buffer.Bytes()), binary.LittleEndian, &value2)
	fmt.Printf("value2: 0x%016X (%d)\n", value2, value2)

	if value1 != value2 {
		log.Panic("unexpected values!")
	}

	// Convert a sequence of unsigned values into a sequence of bytes and vice versa.

	fmt.Print("\nConvert a sequence of unsigned values into a sequence of bytes and vice versa\n\n")
	buffer = new(bytes.Buffer)
	_ = binary.Write(buffer, binary.LittleEndian, []uint64{0x01, 0x02})
	printArray(buffer.Bytes())

	_ = binary.Read(bytes.NewReader(buffer.Bytes()), binary.LittleEndian, &values)
	fmt.Printf("value1: 0x%016X (%d)\n", values[0], values[0])
	fmt.Printf("value2: 0x%016X (%d)\n", values[1], values[1])
}
```

> Playground: [link](https://goplay.tools/snippet/XNS40oO2QP4)

## uint8, uint16... and integers smaller than 64 bytes?

OK, but what I want to convert integers smaller than 64 bytes (that is `uint64`) ?

No problem:
* the `Write()` function only writes has many bytes as necessary.
* the `Read()` function only reads as many bytes as required by the size of the target type.



```go
package main

import (
	"bytes"
	"encoding/binary"
	"fmt"
	"log"
)

func printArray(array []byte) {
	for i, v := range array {
		fmt.Printf("%2d: 0x%02X\n", i, v)
	}
}

func main() {
	var buffer *bytes.Buffer
	var value1 uint32
	var value2 uint32
	var values = make([]uint32, 2)
	var err error

	// Convert an unsigned value into a sequence of bytes and vice versa.

	fmt.Print("Convert an unsigned value into a sequence of bytes and vice versa\n\n")
	value1 = 0x04030201
	buffer = new(bytes.Buffer)
	err = binary.Write(buffer, binary.LittleEndian, value1)
	if err != nil {
		panic("unexpected error!")
	}
	fmt.Printf("value1: 0x%08X (%d)\n", value1, value1)
	printArray(buffer.Bytes())

	err = binary.Read(bytes.NewReader(buffer.Bytes()), binary.LittleEndian, &value2)
	if err != nil {
		panic("unexpected error!")
	}
	fmt.Printf("value2: 0x%08X (%d)\n", value2, value2)

	if value1 != value2 {
		log.Panic("unexpected error!")
	}

	// Convert a sequence of unsigned values into a sequence of bytes and vice versa.

	fmt.Print("\nConvert a sequence of unsigned values into a sequence of bytes and vice versa\n\n")
	buffer = new(bytes.Buffer)
	err = binary.Write(buffer, binary.LittleEndian, []uint32{0x01, 0x02})
	if err != nil {
		panic("unexpected error!")
	}
	printArray(buffer.Bytes())

	err = binary.Read(bytes.NewReader(buffer.Bytes()), binary.LittleEndian, &values)
	if err != nil {
		panic("unexpected error!")
	}
	fmt.Printf("value1: 0x%016X (%d)\n", values[0], values[0])
	fmt.Printf("value2: 0x%016X (%d)\n", values[1], values[1])
}
```