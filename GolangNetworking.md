#### This learning will be more about client and server design from scratch.

All my files is tested in `main.go`

__Simple HTTP__ 
---
In our simple HTTP we will have both client side and server side.

**Client Side**

```
package main

import (
	"fmt"
	"io"
	"net/http"
)

func main() {
	resp, err := http.Get("http://localhost:8080")
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	defer resp.Body.Close()

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	fmt.Println("Response:", string(body))
}

```

**Server Side**  \
I have server side which default will print welcome and counter will how many time we hit the server.

```
package main

import (
	"fmt"
	"net/http"
	"sync"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		Counter()
		fmt.Fprintf(w, "Welcome to my server. Now tell me what to do!!!")
	})
	http.HandleFunc("/counter", func(w http.ResponseWriter, r *http.Request) {
		Counter()
		fmt.Fprintf(w, "Counter of each time we hit main or counter handler: %d", counter)
	})
	if err := http.ListenAndServe(":8080", nil); err != nil {
		fmt.Println(err)
	}
}

var (
	counter int
	mu      sync.Mutex
)

func Counter() {
	mu.Lock()
	counter++
	mu.Unlock()
}

```

__Advanced HTTP__
---

**Client Side**

```
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"net/http"
)

type RequestBody struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}

type ResponseBody struct {
	Message string `json:"message"`
}

func main() {
	requestBody := RequestBody{Name: "Jack Daniels", Age: 80}
	jsonBody, err := json.Marshal(requestBody)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	resp, err := http.Post("http://localhost:8080/api", "application/json", bytes.NewBuffer(jsonBody))
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	defer resp.Body.Close()

	var responseBody ResponseBody
	err = json.NewDecoder(resp.Body).Decode(&responseBody)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	fmt.Println("Response:", responseBody.Message)
}

```

**Server Side**

```
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
)

type RequestBody struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}

type ResponseBody struct {
	Message string `json:"message"`
}

// Post request and response
func ApiHandler(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodPost {
		http.Error(w, "Invalid request method", http.StatusMethodNotAllowed)
	}
	var requestBody RequestBody
	err := json.NewDecoder(r.Body).Decode(&requestBody)
	if err != nil {
		http.Error(w, "Bad Request", http.StatusBadRequest)
		return
	}

	responseBody := ResponseBody{
		Message: fmt.Sprintf("Hello, %s! You are %d years old.", requestBody.Name, requestBody.Age),
	}

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(responseBody)

}

func main() {
	http.HandleFunc("/api", ApiHandler)
	if err := http.ListenAndServe(":8080", nil); err != nil {
		fmt.Println(err)
	}
}

```