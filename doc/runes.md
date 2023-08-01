# Runes and UTF-8

```go
package main

import (
	"fmt"
	"os"
	"unicode/utf8"
)

// Convert a string into a list of runes.
func string2runes(text string) (*[]rune, error) {
	var listOfBytes = []byte(text)
	var listOfRunes []rune

	for {
		r, size := utf8.DecodeRune(listOfBytes)
		if r == utf8.RuneError {
			return nil, fmt.Errorf("invalid string")
		}
		listOfRunes = append(listOfRunes, r)
		listOfBytes = listOfBytes[size:]
		if len(listOfBytes) == 0 {
			break
		}
	}
	return &listOfRunes, nil
}

func main() {
	var s string
	var r rune
	var size int
	var listOfBytes []byte

	// This is a rune
	var myRune rune = '\u01B5' // Ƶ

	// This is a string (that contains one rune)
	var myString string = `Ƶ` // Ƶ <=> (U+01B5)
	fmt.Printf("%v (%+q) len:%d\n", myString, myString, len(myString))

	// Convert a string into a list of bytes
	listOfBytes = []byte(myString)
	fmt.Printf("number of bytes:%d (== %d)\n", len(listOfBytes), utf8.RuneLen(myRune))
	for i, v := range listOfBytes {
		fmt.Printf("%3d: %x\n", i, v)
	}

	// Convert a rune into a string
	s = string(myRune)
	fmt.Printf("%v (%+q) len:%d\n", s, s, len(s))

	// Convert a list of runes into a string
	s = string([]rune{'\u01B5', '\u01C2'})
	fmt.Printf("%v (%+q) len:%d\n", s, s, len(s))

	// Convert a string into a rune
	r, size = utf8.DecodeLastRuneInString(`Ƶ`)
	if r == utf8.RuneError {
		fmt.Printf("ERROR: invalid UTF-8 encoded string\n")
		os.Exit(1)
	}
	fmt.Printf("%+q (%d)\n", r, size)

	// Convert a string into a list of runes
	runes, err := string2runes(`ƵϬ`)
	if err != nil {
		fmt.Printf("ERROR: %s\n", err.Error())
		os.Exit(1)
	}
	for i, v := range *runes {
		fmt.Printf("%3d: %s\n", i, string(v))
	}
}
```

> Playground: [link](https://go.dev/play/p/LM2BU4UgSAO)
