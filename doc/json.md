# JSON

## Deserialization

We deserialize a "_serialized input text_" into a "_target structure_".

### Basic example

As you can see with this example:

* If a key is absent from the serialized input text, then the value of the corresponding key in the target structure is not affected by the deserialization process. The value of the corresponding key in the target structure is whatever was set before the call to the function `json.Unmarshal`.
* If the serialized input text represents a structure with a key that does not belong to the target structure, then this (extra) key is just ignored during the deserialization process.

Simply put, the function `json.Unmarshal` maps _what it finds_ from the given serialized input text to the target structure. Extra keys, _whether from the serialized input text, or within the target structure_, are just ignored.

```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
)

type Data struct {
	Dict    map[string]int `json:"dict"`
	String  string         `json:"string"`
	Integer int            `json:"integer"`
	Array   []int          `json:"array"`
}

func (d *Data) Reset() {
	d.Dict = make(map[string]int)
	d.String = ""
	d.Integer = 0
	d.Array = make([]int, 0)
}

func (d *Data) Dump() {

	fmt.Printf("string:    <%s>\n", d.String)
	fmt.Printf("integer:   %d\n", d.Integer)
	for i, value := range d.Array {
		fmt.Printf("array[%d]:  %d\n", i, value)
	}
	for key, value := range d.Dict {
		fmt.Printf("dict[%s]:   %d\n", key, value)
	}
	fmt.Print("\n")
}

func main() {
	var err error
	var json_text string
	var data Data

	// All fields are defined
	//
	// {
	//   "dict": {
	//     "b": 2,
	//     "a": 1
	//   },
	//   "string": "this is a string",
	//   "nul": null,
	//   "integer": 10,
	//   "array": [
	//     10,
	//     20,
	//     30
	//   ]
	// }

	json_text = `{"dict":{"b":2,"a":1},"string":"this is a string","nul":null,"integer":10,"array":[10,20,30]}`
	data.Reset()
	if err = json.Unmarshal([]byte(json_text), &data); nil != err {
		fmt.Print("Cannot deserialize the input text")
		os.Exit(1)
	}
	data.Dump()

	// One field is not defined: "string"
	//
	// {
	//   "dict": {
	//     "b": 2,
	//     "a": 1
	//   },
	//   "nul": null,
	//   "integer": 10,
	//   "array": [
	//     10,
	//     20,
	//     30
	//   ]
	// }
	//
	// In this case, after the JSON deserialization, the value of the undefined field is whatever was set 
	// *before* the deserialization. In other words, if a field is not set in the input, then its value is 
	// not affected by the execution of the function "json.Unmarshal".

	json_text = `{"dict":{"b":2,"a":1},"nul":null,"integer":10,"array":[10,20,30]}`
	data.Reset()
	if err = json.Unmarshal([]byte(json_text), &data); nil != err {
		fmt.Print("Cannot deserialize the input text")
		os.Exit(1)
	}
	data.Dump()
}
```

Result:

```
string:    <this is a string>
integer:   10
array[0]:  10
array[1]:  20
array[2]:  30
dict[b]:   2
dict[a]:   1

string:    <>
integer:   10
array[0]:  10
array[1]:  20
array[2]:  30
dict[b]:   2
dict[a]:   1
```

> Playground: [link](https://go.dev/play/p/suyHBmeX2pQ)

### Nul values

Sometimes, you want a key (within the target structure) to be assigned the value `nil`, if no corresponding key was found within the "_serialized input text_".

This way, you can make the difference between the absence and the presence of a key within the "_serialized input text_" (_even though, `nil` is a value_).

In order to achieve this goal, we declare the value of the key as being a pointer which value is (pre)set to `nil` (prior to calling `json.Unmarshal`). The function `json.Unmarshal` properly handles the fact that the value is a pointer.

```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
)

type Data struct {
	Dict    map[string]int `json:"dict"`
	String  *string        `json:"string"`
	Integer int            `json:"integer"`
	Array   *[]int         `json:"array"`
}

func (d *Data) Reset() {
	d.Dict = make(map[string]int)
	d.String = nil
	d.Integer = 0
	d.Array = nil
}

func (d *Data) Dump() {
	if nil != d.String {
		fmt.Printf("string:    <%s>\n", *d.String)
	} else {
		fmt.Printf("string:    nul\n")
	}

	fmt.Printf("integer:   %d\n", d.Integer)

	if nil != d.Array {
		for i, value := range *d.Array {
			fmt.Printf("array[%d]:  %d\n", i, value)
		}
	} else {
		fmt.Printf("array:     nul\n")
	}

	for key, value := range d.Dict {
		fmt.Printf("dict[%s]:   %d\n", key, value)
	}
	fmt.Print("\n")
}

func main() {
	var err error
	var json_text string
	var data Data

	// All fields are defined
	json_text = `{"dict":{"b":2,"a":1},"string":"this is a string","nul":null,"integer":10,"array":[10,20,30]}`
	data.Reset()
	if err = json.Unmarshal([]byte(json_text), &data); nil != err {
		fmt.Print("Cannot deserialize the input text")
		os.Exit(1)
	}
	data.Dump()

	// One field is not defined: "string"
	// In this case, after the JSON deserialization, the value of the undefined field is whatever was set before the deserialization.
	// In other words, if a field is not set in the input, then its value is not affected by the execution of the function "json.Unmarshal".
	json_text = `{"dict":{"b":2,"a":1},"nul":null,"integer":10}`
	data.Reset()
	if err = json.Unmarshal([]byte(json_text), &data); nil != err {
		fmt.Print("Cannot deserialize the input text")
		os.Exit(1)
	}
	data.Dump()

}
```

Result:

```
string:    <this is a string>
integer:   10
array[0]:  10
array[1]:  20
array[2]:  30
dict[a]:   1
dict[b]:   2

string:    nul
integer:   10
array:     nul
dict[b]:   2
dict[a]:   1
```

> Playground: [link](https://go.dev/play/p/NXqsF_l7Gt7)


## Serialization

We serialize a "structure" into "_serialized output text_".

### Using "nul"

If you want to safely differentiate between the absence and the presence of a key's value, then you can declare the key's value as being a pointer. If no value is assigned for the key in the structure, then the corresponding value within the "serialized output text" will be `nul`. Otherwise, a (typed) value is assigned.

```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
)

type Data struct {
	Dict    *map[string]int `json:"dict"`
	String  *string         `json:"string"`
	Integer *int            `json:"integer"`
	Array   *[]int          `json:"array"`
}

func string2ref(s string) *string { return &s }

func int2ref(i int) *int { return &i }

func dict2ref(d map[string]int) *map[string]int { return &d }

func array2ref(a []int) *[]int { return &a }

func main() {
	var err error
	var json_text []byte
	var data Data

	data = Data{
		Dict:    nil,
		String:  nil,
		Integer: nil,
		Array:   nil,
	}

	if json_text, err = json.Marshal(data); nil != err {
		fmt.Print("Cannot deserialize the input text")
		os.Exit(1)
	}
	fmt.Printf("%s\n\n", string(json_text))

	data = Data{
		Dict:    dict2ref(map[string]int{"a": 1}),
		String:  string2ref("abcd"),
		Integer: int2ref(100),
		Array:   array2ref([]int{1, 2}),
	}

	if json_text, err = json.Marshal(data); nil != err {
		fmt.Print("Cannot deserialize the input text")
		os.Exit(1)
	}
	fmt.Printf("%s\n\n", string(json_text))
}
```

Result:

```
{"dict":null,"string":null,"integer":null,"array":null}

{"dict":{"a":1},"string":"abcd","integer":100,"array":[1,2]}
```

> Playground: [link](https://go.dev/play/p/87Ds3joOjWv)

### Do not dump a field if it has a "zero value"

"Zero values" differ depending on the type of the variables. For a string, it is "". For an integer, it is 0...

```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
)

type Data struct {
	Dict    map[string]int `json:"dict,omitempty"`
	String  string         `json:"string,omitempty"`
	Integer int            `json:"integer,omitempty"`
	Array   []int          `json:"array,omitempty"`
}

func (d *Data) Dump() {
	fmt.Printf("string:    <%s>\n", d.String)
	fmt.Printf("integer:   %d\n", d.Integer)
	for i, value := range d.Array {
		fmt.Printf("array[%d]:  %d\n", i, value)
	}
	for key, value := range d.Dict {
		fmt.Printf("dict[%s]:   %d\n", key, value)
	}
	fmt.Print("\n")
}

func main() {
	var err error
	var json_text []byte
	var data Data

	data = Data{
		Dict:    make(map[string]int),
		String:  "",
		Integer: 0,
		Array:   make([]int, 0),
	}
	data.Dump()

	if json_text, err = json.Marshal(data); nil != err {
		fmt.Print("Cannot deserialize the input text")
		os.Exit(1)
	}
	fmt.Printf("%s\n\n", string(json_text))

	data = Data{
		Dict:    make(map[string]int),
		String:  "1",
		Integer: 1,
		Array:   []int{1, 2},
	}
	data.Dump()

	if json_text, err = json.Marshal(data); nil != err {
		fmt.Print("Cannot deserialize the input text")
		os.Exit(1)
	}
	fmt.Printf("%s\n\n", string(json_text))
}
```

Result:

```
string:    <>
integer:   0

{}

string:    <1>
integer:   1
array[0]:  1
array[1]:  2

{"string":"1","integer":1,"array":[1,2]}
```

> Playground: [link](https://go.dev/play/p/XHF959PCj-B)
