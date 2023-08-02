# Flags: parsing the command line

The following example shows how to implement a command line with the following structure:

`<program> <action> [option...]`

It also shows how to check whether a option has been set or not.


```go
// Valid:
//    flag.exe add -a=1 -b=2
//    flag.exe add --a=1 --b=2
//    flag.exe add --a=1 --b=2 --verbose
//    flag.exe multiply -a=2 -b=3
//    flag.exe multiply -a=2
//    flag.exe multiply
// Invalid:
//    flag.exe add -a=1

package main

import (
	"flag"
	"fmt"
	"os"
)

var flagsSet = make(map[string]*flag.Flag)

func flagsReset() {
	flagsSet = make(map[string]*flag.Flag)
}

func listFlags(f *flag.Flag) {
	flagsSet[f.Name] = f
}

// ProcessSum calculates "a+b".
// Please note that "a" and "a" must have been set within the command line.
func ProcessSum() error {
	// Define the possible options that can be found on the command line.
	var a = flag.Int("a", -1, `the value "a"`)
	var b = flag.Int("b", -1, `the value "b"`)
	var v = flag.Bool("verbose", false, `verbose mode`)
	// Parse the command line.
	flag.Parse()
	// Make sure that "a" and "b" are set.
	flagsReset()
	flag.Visit(listFlags)
	if _, ok := flagsSet["a"]; !ok {
		return fmt.Errorf(`option "a" is not set`)
	}
	if _, ok := flagsSet["b"]; !ok {
		return fmt.Errorf(`option "b" is not set`)
	}

	if *v {
		fmt.Printf("calculate: %d + %d\n", *a, *b)
	}
	fmt.Printf("%d\n", *a+*b)
	return nil
}

// ProcessMultiply calculates "a*b".
// Please note that "a" and "a" can be absent from the command line.
func ProcessMultiply() error {
	var a = flag.Int("a", -1, `the value "a"`)
	var b = flag.Int("b", -1, `the value "b"`)
	var v = flag.Bool("verbose", false, `verbose mode`)
	flag.Parse()

	if *v {
		fmt.Printf("calculate: %d * %d\n", *a, *b)
	}
	fmt.Printf("%d\n", *a**b)
	return nil
}

type ActionData struct {
	Description string
	Handler     func() error
}

var Actions = map[string]ActionData{
	"add":      {Description: `"a" + "b"`, Handler: ProcessSum},
	"multiply": {Description: `"a" * "b"`, Handler: ProcessMultiply},
}

func logError(messages []string) {
	for _, message := range messages {
		fmt.Printf("%s\n", message)
	}
	os.Exit(1)
}

func main() {
	// Print the content of the array that contains the command line arguments before we modify it.
	for i, v := range os.Args {
		fmt.Printf("%d: %s\n", i, v)
	}

	// Check the number of arguments in the command line.
	if len(os.Args) < 2 {
		logError([]string{fmt.Sprintf(`invalid number of arguments in command line (%d)`, len(os.Args)-1)})
	}

	// Retrieve the "action" (second argument) from the command line, and extract it from the list of arguments.
	action := os.Args[1]
	fmt.Printf("action: \"%s\"\n", action)
	os.Args = append(os.Args[0:1], os.Args[2:]...)
	for i, v := range os.Args {
		fmt.Printf("%d: %s\n", i, v)
	}

	// Check the action.
	if _, ok := Actions[action]; !ok {
		logError([]string{fmt.Sprintf(`invalid action "%s"`, action)})
	}

	// Process the action.
	if err := Actions[action].Handler(); err != nil {
		logError([]string{err.Error()})
	}
}
```