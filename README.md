# Go-CVSS

[![reference](https://godoc.org/github.com/pandatix/go-cvss/v5?status.svg=)](https://pkg.go.dev/github.com/pandatix/go-cvss)
[![go report](https://goreportcard.com/badge/github.com/pandatix/go-cvss)](https://goreportcard.com/report/github.com/pandatix/go-cvss)
[![Coverage Status](https://coveralls.io/repos/github/pandatix/go-cvss/badge.svg?branch=master)](https://coveralls.io/github/pandatix/go-cvss?branch=master)
[![CI](https://github.com/pandatix/go-cvss/actions/workflows/ci.yaml/badge.svg)](https://github.com/pandatix/go-cvss/actions?query=workflow%3Aci+)
[![CodeQL](https://github.com/pandatix/go-cvss/actions/workflows/codeql-analysis.yml/badge.svg)](https://github.com/pandatix/go-cvss/actions/workflows/codeql-analysis.yml)

Go-CVSS is a low-allocation Go module made to manipulate Common Vulnerability Scoring System (CVSS)

Specified by [first.org](https://www.first.org/cvss/), the CVSS provides a way to capture the principal characteristics of a vulnerability and produce a numerical score reflecting its severity.

It currently supports :
 - [X] [CVSS 2.0](https://www.first.org/cvss/v2/guide)
 - [X] [CVSS 3.0](https://www.first.org/cvss/v3.0/specification-document)
 - [X] [CVSS 3.1](https://www.first.org/cvss/v3.1/specification-document)
 - [ ] CVSS 4.0 (currently not published)

It won't support CVSS v1.0, as despite it was a good CVSS start, it can't get vectorized, abreviations and enumerations are not strongly specified, so the cohesion and interoperability can't be satisfied.

## Summary

 - [How to use](#how-to-use)
 - [A word on performances](#a-word-on-performances)
   - [CVSS v2.0](#cvss-v20)
   - [CVSS v3.0](#cvss-v30)
   - [CVSS v3.1](#cvss-v31)
 - [Feedbacks](#feedbacks)
   - [CVSS v2.0](#cvss-v20-1)
   - [CVSS v3.0](#cvss-v30-1)
   - [CVSS v3.1](#cvss-v31-1)

## How to use

The following code gives an example on how to use the present Go module.

It parses a CVSS v3.1 vector, then compute its base score and gives the associated rating.
It ends by printing it as the score followed by its rating, as it is often displayed.

```go
package main

import (
	"fmt"
	"log"

	gocvss31 "github.com/pandatix/go-cvss/31"
)

func main() {
	cvss31, err := gocvss31.ParseVector("CVSS:3.1/AV:N/AC:L/PR:L/UI:R/S:C/C:L/I:L/A:N")
	if err != nil {
		log.Fatal(err)
	}
	baseScore := cvss31.BaseScore()
	rat, err := gocvss31.Rating(baseScore)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("%.1f %s\n", baseScore, rat)
	// Prints "5.4 MEDIUM"
}
```

## A word on performances

We are aware that manipulating a CVSS object does not provide the most value to your business needs.
This is why we paid a big attention to performances of this module.

What we made is making this module **0 to 1 allocs/op** for the whole API.
This reduce drastically the pressure on the Garbage Collector using this Go module, without cutting through security (fuzzing ensures the API does not contain obvious security issues). It also reduces the time and bytes per op to a really acceptable level.

The following shows the performances results.
We challenge any other Go implementation to do better :stuck_out_tongue_winking_eye:

### CVSS v2.0

```
goos: linux
goarch: amd64
pkg: github.com/pandatix/go-cvss/20
cpu: Intel(R) Core(TM) i5-2450M CPU @ 2.50GHz
BenchmarkParseVector_Base-4                  1750524           631.0 ns/op       224 B/op          1 allocs/op
BenchmarkParseVector_WithTempAndEnv-4         983698            2216 ns/op       224 B/op          1 allocs/op
BenchmarkCVSS20Vector-4                      6241333           189.2 ns/op        80 B/op          1 allocs/op
BenchmarkCVSS20Get-4                        41707770           38.30 ns/op         0 B/op          0 allocs/op
BenchmarkCVSS20Set-4                        35828242           31.82 ns/op         0 B/op          0 allocs/op
BenchmarkCVSS20BaseScore-4                  15570378           67.29 ns/op         0 B/op          0 allocs/op
BenchmarkCVSS20TemporalScore-4              12213030           92.93 ns/op         0 B/op          0 allocs/op
BenchmarkCVSS20EnvironmentalScore-4         12509287           92.03 ns/op         0 B/op          0 allocs/op
```

### CVSS v3.0

```
goos: linux
goarch: amd64
pkg: github.com/pandatix/go-cvss/30
cpu: Intel(R) Core(TM) i5-2450M CPU @ 2.50GHz
BenchmarkParseVector_Base-4                  1564752           793.1 ns/op       352 B/op          1 allocs/op
BenchmarkParseVector_WithTempAndEnv-4         763330            1588 ns/op       352 B/op          1 allocs/op
BenchmarkCVSS30Vector-4                      4708683           247.6 ns/op        96 B/op          1 allocs/op
BenchmarkCVSS30Get-4                        23825982           43.71 ns/op         0 B/op          0 allocs/op
BenchmarkCVSS30Set-4                        31735555           37.64 ns/op         0 B/op          0 allocs/op
BenchmarkCVSS30BaseScore-4                   7696878           151.7 ns/op         0 B/op          0 allocs/op
BenchmarkCVSS30TemporalScore-4               5761302           200.7 ns/op         0 B/op          0 allocs/op
BenchmarkCVSS30EnvironmentalScore-4          5636908           206.6 ns/op         0 B/op          0 allocs/op
```

### CVSS v3.1

```
goos: linux
goarch: amd64
pkg: github.com/pandatix/go-cvss/31
cpu: Intel(R) Core(TM) i5-2450M CPU @ 2.50GHz
BenchmarkParseVector_Base-4                  1543551           768.7 ns/op       352 B/op          1 allocs/op
BenchmarkParseVector_WithTempAndEnv-4         754154            1614 ns/op       352 B/op          1 allocs/op
BenchmarkCVSS31Vector-4                      4671334           254.7 ns/op        96 B/op          1 allocs/op
BenchmarkCVSS31Get-4                        24025206           43.89 ns/op         0 B/op          0 allocs/op
BenchmarkCVSS31Set-4                        31292902           40.29 ns/op         0 B/op          0 allocs/op
BenchmarkCVSS31BaseScore-4                   7711374           177.9 ns/op         0 B/op          0 allocs/op
BenchmarkCVSS31TemporalScore-4               5245575           234.1 ns/op         0 B/op          0 allocs/op
BenchmarkCVSS31EnvironmentalScore-4          4021914           293.4 ns/op         0 B/op          0 allocs/op
```

## Feedbacks

### CVSS v2.0

 - Section 3.3.1's base vector gives a base score of 7.8, while verbosely documented as 6.4.
 - `round_to_1_decimal` may have been specified, so that it's not guessed and adjusted to fit precomputed scores. It's not even CVSS v3.1 `roundup` specification.

### CVSS v3.0

 - Formulas are pretty, but complex to read as the variables does not refer to the specified abbreviations.
 - There is a lack of examples, as it's achieved by the CVSS v2.0 specification.

### CVSS v3.1

 - There is a lack of examples, as it's achieved by the CVSS v2.0 specification.
