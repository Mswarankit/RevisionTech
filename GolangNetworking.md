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
1. Explain how client server architecture works on the internet?

	Client-Server architecture is based on the interaction between two entities:
	the client and the server. A client is a device or application that intiates requests
	to a server to retrieve resources such as web pages, images, or API data. The server
	is remote device that processes the client's requests and sends back responses.

	Example :
		When a user enters a URL in their browser, the browser acts as the client, sending
		a request to the server. The server receives the request, processes it (eg fetching data
		 from a database), and returns the response to the clients. 

2. What happens "under the hood" when you enter a URL in a browser and press Enter?

	When a user enters a URL in a browser and presses Enter, the following process occurs:

	* __Domain name Resolution(DNS):__ The browser first checks if the IP address for the domain is in local cache. If not, it sends a request to a DNS server, which returns the IP address associated with the domain.

	* __Establishing a connection:__ After obtaining the IP address, the browser initiates a TCP connection with the server through a three way handshake.

	* __Sending an HTTP request:__ The browser constructs an HTTP request containing information about the resource it wants to fetch.

	* __Processing the request on the server:__ The server receives the request, processes it (eg., retrieving data from a database), and constructs an HTTP response.

	* __Receiving and displaying content:__ The browser receives the response and displays the content (eg. and HTML page). if The HTML contains references to other resources such as CSS or images, the browser makes additional requests for those resources.

3. What type of data transfer protocols exist besides HTTP?

	* __FTP (File Transfer Protocol):__ Used to transfer files between a client and a server, such as uploading for downloading files.

	* __SMTP (Simple Mail Transfer Protocol):__ Used to send emails.

	* __IMAP and POP3:__ Used to retrieve emails

	* __WebSocket:__ Provides bidirectional communication between a client and a server, ideal for real time applications like chats.

	* __TLS/SSL:__ Ensures secure data encryption over other protocols, including HTTP and FTP.

	* __DNS:__ Converts domain names into IP addreses.

4. How Does TCP differs from UDP? Provide examples each

	TCP (Transmission Control Protocol) is a reliable protocol that ensures data delivery. It uses acknowledgments to confirm the recepient of data, guarenteeing its integrity, but this leads to higher latency.
	UDP (User Datagram Protocol) does not guarantee data delivery. It is faster and lighter because it does not require acknowledgments or connection establishment.

	UDP (User Datagram Protocol) does not gurantee data delivery. It is faster and lighter because it does not require acknowledgements or connection establishment.

	Examples:
		
	* __TCP__ is used for applications where reliablity is critical, such as websites, email, or file transfers.

	* __UDP__ is suitable for applications requiring speed, such as streaming video, online games, or VoIP.

	
5. What is an IP address? What are the versions of IP?

	An IP address is a unique numerical identifier for a device on a network. It is 
	essential for routing data between devices on the internet.

	There are two versions of IP:

	* __IPv4:__ 32-bit addresses represented as four decimal numbers seperated by dots (eg. 192.168.0.1). The limited number of addresses has led to shortages.

	* __IPv6:__ 128-bit addresses written in hexadecimal format (eg., 2001:0db8:85a3::8a2e:0370:7334). It solves the address shortage issue.

6. What is a CDN (Content Delivery Network), and how does it improve performance?

	A CDN is a network of servers distributed across different geographic locations that store copies of content to deliver it to users from the nearest server. This reduces latency and speeds up loading times.
	For examples, if a user in Asia requests a resource hosted on a server in the US, the CDN redirects the request to the nearest server in Asia that already has a cached copy of the resource. This minimizes the time for data requirements.

7. Describe the layers of the OSI model. At which layer does HTTP operate?
 
	The OSI (Open Systems Interconnections) model divides network interactions into seven layers:
	
	* Physical: Transmission of raw data on physical hardware.
	* Data Link: Handling data exchange between adjacent devices.
	* Network: Routing data using protocols like IP.
	* Transport: Ensuring data delivery using protocols like TCP or UDP.
	* Session: Managing connections between devices.
	* Presentation: Formatting and translating data.
	* Application: Providing intefaces for applications (e.g HTTP)

8. What is NAT (Network Address Translation), why is it used?

	NAT is a process that translates private IP addresses within a local network into one or more public IP addresses for accessing the internet. It is used to conserve public IP addresses and enhance security by hiding the internal network from the outside world.

	For examples, a router with a single public IP address can allow multiple devices within a private network to connect to the internet simaltaneously.

9. How do public and private IP addrsses differ?

	Public IP addresses are visible on the internet and are used to identify devices or servers accessible worldwide. For instance, the IP address of a website is public. 
	Private IP addresses are used within local networks (eg. at home or in an office)
	and are not visible on the internet. They allow devices to communicate with each other with in the network. Examples of private IP ranges include 192.168.x.x and 10.x.x.x.



## HTTP and Browsers

1. Describe the structure of an HTTP request and response?

	An HTTP request consistes of three main parts: the start line, headers and an optional body. The start line specifies the HTTP method (e.g., GET or POST), the resource path (e.g., /home), and the protocol versions (eg. HTTP/1.1). Next come the headers, which are key-value pairs providing additional information such as the content type (Content-Type) or the user agent (User-Agent). The body is included like in methods like POST or PUT and contains the data sent to the server (eg JSON or XML).

	An HTTP response also consistes of three parts: the status line, headers, and body. The status line indicates the protocol version, the status code (eg. 200 for success or 404 for "not found"), and a status message. The headers provide metadata such as the content length (Content-Length) or encoding (Content-Encoding). The body contains the actual content, such as an HTML page or API data.

2. What HTTP methods do you know, and what are they used for ?

	HTTP methods define the action to be performed on the server. For example, the GET method is used to retrieve data from the server. It is safe and idempotent, meaning repeated requests do not change the servers state.

	POST is used to send data to the server, such as creating a new resource. PUT is used to replace an existing resource or create one if it doesn't exists.
	DELETE is used to delete a resource. HEAD is similar to GET but only returns headers, which is useful for checking the existence of a resource.PATCH is used for patially updating a resource.

3. What are HTTP headers? Provide examples of commonly used headers?

	HTTP headers are metadata sent along with requests and responses that provide information about the request or resource. They are used for specifying content, caching settings, authentication, user agents, and other parameters.

	Examples of common headers:
	
	* `Content-Type` specifies the type of data, such as application/json.
	* `Authorization` is used to send tokens or credentials
	* `Cache-Control` manages caching, for example, no-cache.
	* `User-Agent` identifies the client making the request, such as a browser.
	* `Accept` informs the server about the types of responses the client expects.

4. What is the difference between HTTP and HTTPS? How is HTTPs security ensured?

	HTTP transmits data in plain text, making it vulnerable to attacks like "man-in-the-middle." HTTPS, on the other hand, encrypts data using the TLS (Transport Layer Security) protocol, protecting it during transmission.

	HTTPS security is ensured through certificates issued by certification autherties (CAs). When a connection is established, the client verfies the server's certificat. Then, a secure channel is created using symmetric encryption.

5. What is CORS (Cross-Origin Resource Sharing), and how can its issues be resolved?

	CORS is a mechanism that allows servers to control their resources from other domains. By default, browser block cross origin requests to prevent malicious activity.
	To allow such requests, the server must include specific headers like Access-Control-Allow-Origin, which indicates which domains have access to the resources. For example, to allow access for all domains, the server can use Access-Control-Allow-Origin: *

6. How does static and dynamic web pages differ?

	Static web pages fixed content as files on the server. They do not change based on user requests. An example is a simple HTML file.
	
	Dynamic web pages are generated on the server in real time, and their content depends on request parameters. For instance, search result pages are generated based on the user's input. Such pages often use server-side programming languages like Python and Go and connect to databases.

7. What are cookies, localStorage, and sessionStorage? When should each be used?

	Cookies are small piece of data sent by the server to the browser for storage.
	They are automatically sent back with every request to the server. Cookies are usefull for authetication or session tracking.

	localStorage persists data even after the browser is closed.
	sessionStorage deletes data when the session ends (when the tab is closed.)
	
	localstorage and sessionstorage are client side storage methods.
	Cookies are better for data related to the server, while localStorage and sessionStorage deletes data when the session ends (e.g., when the tab is closed.)

8. What is REST, and how does it differ from SOAP?

	REST (Representational State Transfer) is an architectural style that uses  the HTTP protocol to create web services. It is based on CRUD operations and the concepts of resources. REST is lightweight,easy to understand and implement.

	SOAP (Simple Object Access Protocol) is a more complex protocol for web service interaction. It uses XML for data exchange and supports strict security standards and transactions. SOAP is often used in enterprise systems requiring high reliability.

9. What HTTP versions do you know, and what are the key improvements in HTTP/2 and HTTP/3?

	The known HTTP versions are HTTP/1.1, HTTP/2, and HTTP/3. 
	
	HTTP/1.1 was the standard for a long time, but its major limitation was the inability to send multiple requests simultaneously over a single connection.

	HTTP/2 addresses this by enabling multiplexing, allowing multiple requests to be sent at the same time. It also uses a binary format, which speeds up data processing, and supports header compression.

	HTTP/3, in contrast, is based on the QUIC protocol, which uses UDP instead of TCP. This results in faster connection establishment and lower latency, especially in unstable network conditions.



