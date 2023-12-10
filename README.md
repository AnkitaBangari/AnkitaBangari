- 👋 Hi, I’m @AnkitaBangari
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
AnkitaBangari/AnkitaBangari is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
	"sort"
	"sync"
	"time"
)

type Payload struct {
	ToSort [][]int json:"to_sort"
}

type Response struct {
	SortedArrays [][]int json:"sorted_arrays"
	TimeNs       int64   json:"time_ns"
}

func sortSequential(toSort [][]int) (int64, [][]int) {
	start := time.Now()
	var sortedArrays [][]int

	for _, arr := range toSort {
		sorted := make([]int, len(arr))
		copy(sorted, arr)
		sort.Ints(sorted)
		sortedArrays = append(sortedArrays, sorted)
	}

	elapsed := time.Since(start).Nanoseconds()
	return elapsed, sortedArrays
}

func sortConcurrent(toSort [][]int) (int64, [][]int) {
	start := time.Now()
	var sortedArrays [][]int
	var wg sync.WaitGroup
	var mu sync.Mutex

	for _, arr := range toSort {
		wg.Add(1)
		go func(arr []int) {
			defer wg.Done()
			sorted := make([]int, len(arr))
			copy(sorted, arr)
			sort.Ints(sorted)

			mu.Lock()
			sortedArrays = append(sortedArrays, sorted)
			mu.Unlock()
		}(arr)
	}

	wg.Wait()
	elapsed := time.Since(start).Nanoseconds()
	return elapsed, sortedArrays
}

func processSingle(w http.ResponseWriter, r *http.Request) {
	var payload Payload
	err := json.NewDecoder(r.Body).Decode(&payload)
	if err != nil {
		http.Error(w, err.Error(), http.StatusBadRequest)
		return
	}

	timeNs, sortedArrays := sortSequential(payload.ToSort)
	response := Response{
		SortedArrays: sortedArrays,
		TimeNs:       timeNs,
	}

	json.NewEncoder(w).Encode(response)
}

func processConcurrent(w http.ResponseWriter, r *http.Request) {
	var payload Payload
	err := json.NewDecoder(r.Body).Decode(&payload)
	if err != nil {
		http.Error(w, err.Error(), http.StatusBadRequest)
		return
	}

	timeNs, sortedArrays := sortConcurrent(payload.ToSort)
	response := Response{
		SortedArrays: sortedArrays,
		TimeNs:       timeNs,
	}

	json.NewEncoder(w).Encode(response)
}

func main() {
	http.HandleFunc("/process-single", processSingle)
	http.HandleFunc("/process-concurrent", processConcurrent)

	fmt.Println("Server running on port 8000")
	http.ListenAndServe(":8000", nil)
}
--->
