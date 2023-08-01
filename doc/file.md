# File

## Slurp a file that contains text

```go
package main

import (
	"fmt"
	"io"
	"os"
)

const BufferLength = 1024

func logError(messages []string) {
	for _, message := range messages {
		fmt.Printf("- %s\n", message)
	}
	os.Exit(1)
}

func slurpText(path string, bufferLength int) (*string, error) {
	var err error
	var fd *os.File
	var content []byte
	var result string
	var buffer = make([]byte, bufferLength)
	var count int

	if fd, err = os.Open(path); err != nil {
		return nil, err
	}
	for {
		if count, err = fd.Read(buffer); err != nil && err != io.EOF {
			return nil, err
		}
		content = append(content, buffer[:count]...)
		if count < bufferLength {
			break
		}
	}
	if err = fd.Close(); err != nil {
		return nil, err
	}

	result = string(content)
	return &result, nil
}

func main() {
	var text *string
	var err error

	if text, err = slurpText("file.txt", BufferLength); err != nil {
		logError([]string{err.Error()})
	}
	fmt.Printf("Content:\n\n[%s]\n\n", *text)
}
```
