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

__CRUD + PATCH Using Movies__
---

**Client Side**

```
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io"
	"net/http"
)

type Movie struct {
	ID          int     `json:"id"`
	Title       string  `json:"title"`
	Director    string  `json:"director"`
	ReleaseYear int     `json:"release_year"`
	Rating      float64 `json:"rating"`
}

func createMovie(movie Movie) {
	jsonData, _ := json.Marshal(movie)
	resp, err := http.Post("http://localhost:8080/movies", "application/json", bytes.NewBuffer(jsonData))
	if err != nil {
		fmt.Printf("Error Creating movie: %v\n", err)
		return
	}
	defer resp.Body.Close()
	body, _ := io.ReadAll(resp.Body)
	fmt.Printf("Created movie: %s\n", string(body))
}

func getMovie(id int) {
	resp, err := http.Get(fmt.Sprintf("http://localhost:8080/movies/%d", id))
	if err != nil {
		fmt.Printf("Error getting movie: %v\n", err)
		return
	}
	defer resp.Body.Close()

	body, _ := io.ReadAll(resp.Body)
	fmt.Printf("Movie details: %s\n", string(body))
}

func updateMovie(id int, movie Movie) {
	jsonData, _ := json.Marshal(movie)
	req, _ := http.NewRequest(http.MethodPut, fmt.Sprintf("http://localhost:8080/movies/%d", id), bytes.NewBuffer(jsonData))
	req.Header.Set("Content-Type", "application/json")
	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil {
		fmt.Printf("Error updating movie: %v\n", err)
		return
	}
	defer resp.Body.Close()

	body, _ := io.ReadAll(resp.Body)
	fmt.Printf("Updated movie: %s\n", string(body))
}

func patchMovie(id int, updates map[string]interface{}) {
	jsonData, _ := json.Marshal(updates)
	req, _ := http.NewRequest(http.MethodPatch, fmt.Sprintf("http://localhost:8080/movies/%d", id), bytes.NewBuffer(jsonData))
	req.Header.Set("Content-Type", "application/json")
	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil {
		fmt.Printf("Error patching movie: %v\n", err)
		return
	}
	defer resp.Body.Close()

	body, _ := io.ReadAll(resp.Body)
	fmt.Printf("Updated movie: %s\n", string(body))
}

func deleteMovie(id int) {
	req, _ := http.NewRequest(http.MethodDelete,
		fmt.Sprintf("http://localhost:8080/movies/%d", id),
		nil)

	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil {
		fmt.Printf("Error deleting movie: %v\n", err)
		return
	}
	defer resp.Body.Close()

	fmt.Printf("Movie deleted. Status: %s\n", resp.Status)
}

func main() {
	movie := Movie{
		Title:       "3 Idiots",
		Director:    "Rajkumar Hirani",
		ReleaseYear: 2009,
		Rating:      8.0,
	}
	createMovie(movie)
	getMovie(1)
	movie.Rating = 8.2
	updateMovie(1, movie)

	updates := map[string]interface{}{
		"rating": 8.4,
	}
	patchMovie(1, updates)

	deleteMovie(1)
}

```

**Server Side**

```
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
	"strconv"
	"strings"
	"sync"
)

type Movie struct {
	ID          int     `json:"id"`
	Title       string  `json:"title"`
	Director    string  `json:"director"`
	ReleaseYear int     `json:"release_year"`
	Rating      float64 `json:"rating"`
}

var (
	movies = make(map[int]Movie)
	mu     sync.RWMutex
	nextID = 1
)

func createMovieHandler(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodPost {
		http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
		return
	}

	var movie Movie
	if err := json.NewDecoder(r.Body).Decode(&movie); err != nil {
		http.Error(w, err.Error(), http.StatusBadRequest)
		return
	}

	mu.Lock()
	movie.ID = nextID
	movies[nextID] = movie
	nextID++
	mu.Unlock()

	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusCreated)
	json.NewEncoder(w).Encode(movie)
}

func getMovieHandler(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodGet {
		http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
		return
	}
	idStr := strings.TrimPrefix(r.URL.Path, "/movies/")
	id, err := strconv.Atoi(idStr)
	if err != nil {
		http.Error(w, "Invalid movie ID", http.StatusBadRequest)
		return
	}
	mu.RLock()
	movie, exists := movies[id]
	mu.RUnlock()

	if !exists {
		http.Error(w, "Movie not found", http.StatusBadRequest)
		return
	}
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(movie)

}

func updateMovieHandler(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodPut {
		http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
		return
	}

	idStr := strings.TrimPrefix(r.URL.Path, "/movies/")
	id, err := strconv.Atoi(idStr)
	if err != nil {
		http.Error(w, "Invalid movie ID", http.StatusBadRequest)
		return
	}
	var movie Movie

	if err := json.NewDecoder(r.Body).Decode(&movie); err != nil {
		http.Error(w, err.Error(), http.StatusBadRequest)
		return
	}
	mu.Lock()
	movie.ID = id
	movies[id] = movie
	mu.Unlock()
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(movie)
}

func patchMovieHandler(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodPatch {
		http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
		return
	}

	idStr := strings.TrimPrefix(r.URL.Path, "/movies/")
	id, err := strconv.Atoi(idStr)
	if err != nil {
		http.Error(w, "Invalid movie ID", http.StatusBadRequest)
		return
	}

	mu.Lock()
	movie, exists := movies[id]
	if !exists {
		mu.Unlock()
		http.Error(w, "Movie not found", http.StatusNotFound)
		return
	}

	var updates map[string]interface{}
	if err := json.NewDecoder(r.Body).Decode(&updates); err != nil {
		mu.Unlock()
		http.Error(w, err.Error(), http.StatusBadRequest)
		return
	}

	for key, value := range updates {
		switch key {
		case "title":
			movie.Title = value.(string)
		case "director":
			movie.Director = value.(string)
		case "release_year":
			movie.ReleaseYear = int(value.(float64))
		case "rating":
			movie.Rating = value.(float64)
		}
	}
	movies[id] = movie
	mu.Unlock()

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(movie)
}

func deleteMovieHandler(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodDelete {
		http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
		return
	}
	idStr := strings.TrimPrefix(r.URL.Path, "/movies/")
	id, err := strconv.Atoi(idStr)
	if err != nil {
		http.Error(w, "Invalid movie ID", http.StatusBadRequest)
		return
	}

	mu.Lock()
	delete(movies, id)
	mu.Unlock()
	w.WriteHeader(http.StatusNoContent)
}

func main() {
	http.HandleFunc("/movies", createMovieHandler)
	http.HandleFunc("/movies/", func(w http.ResponseWriter, r *http.Request) {
		switch r.Method {
		case http.MethodGet:
			getMovieHandler(w, r)
		case http.MethodPut:
			updateMovieHandler(w, r)
		case http.MethodPatch:
			patchMovieHandler(w, r)
		case http.MethodDelete:
			deleteMovieHandler(w, r)
		default:
			http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
		}
	})
	if err := http.ListenAndServe(":8080", nil); err != nil {
		fmt.Println(err)
	}
}

```

