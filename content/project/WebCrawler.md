+++
title = "Building a Website Crawler"
date = "2017-04-21T13:50:46+02:00"
tags = ["projects"]
description = "Building a Web Crawler from scratch using Golang"
keywords = ["Programming","Go","Golang","Web","Crawler","web-crawler","recursive"]
+++

[1]: https://medium.com/@a4word/webl-a-simple-web-crawler-written-in-go-c1ce50b4f687
[2]: https://schier.co/blog/2015/04/26/a-simple-web-scraper-in-go.html
[3]: https://github.com/richmondgoh8/web-crawler
[4]: https://github.com/jackdanger/collectlinks
Today, we are going to explore building a web scraper to mark a milestone in our progression towards learning GO. This exercise will be emphasizing on the skills that you should already be familiar with. It will cover skills such as:<!--more-->

*Scenario* - Crawl a page and deeper into the relative urls and ensure that there are no dead links in any of your website.

**Sneak Preview**
<img src="/images/snippet.png" class="img-responsive center-block" />
```
go run crawl_less.go http://example.com
```
+ Usage of reading a command line argument
+ Scanning a webpage for links
+ Going a level deeper and scanning for more links

```
//crawl.go
func main() {
	flag.Parse()

	args := flag.Args()
	fmt.Println(args)
	if len(args) < 1 {
		fmt.Println("Please specify start page")
		os.Exit(1)
	}
}
```

In the current main file, this will be the code that will be responsible for taking in command arguments and checking to ensure that there is 1 url in the argument after running
```
go run crawl.go http://rlc4u.com
```
Now, to spice things up further, we create a function called retrieve which will proceed to collect the full html elements of the indicated link and print it out.
```
func retrieve(uri string) {

	resp, err := http.Get(uri)
	if err != nil {
		panic(err)
	}
	defer resp.Body.Close()

	body, _ := ioutil.ReadAll(resp.Body)
	fmt.Println(string(body))
	}
}
```
We will now try to get the terminal to print that all links in the page which include relative links
```
links := collectlinks.All(resp.Body) // Here we use the collectlinks package
for _, link := range links {
  fmt.Println(link) // an iterator in many other languages get all links including relative
}
```
These statements will proceed to run a number of iterations based on the numbers of links that the library has collected for us which includes relative link.

Now that we have figured out what exactly, resp.body contains, we no longer need to see them anymore. The idea behind the next step would consist of extracting all elements containing the tag "a href". This time, we will be using a library of [Jack Danger][4] that will do those extractions for us.

Currently, we have a bunch of urls with no base url. We will now try to convert all relative urls to an absolute url.
```
func retrieve(uri string) {
...
    ...
	for _, link := range links {
		absolute := fixUrl(link, uri)
		fmt.Println(absolute)
	}
}

func fixUrl(href, base string) string {
	uri, err := url.Parse(href)
	if err != nil {
		return ""
	}
	baseUrl, err := url.Parse(base)
	if err != nil {
		return ""
	}
	uri = baseUrl.ResolveReference(uri)
	return uri.String()
}
```
Now, once we print our urls, we will see thats all our paths that was relative is now converted to absolute urls

Going deeper, we want to ensure that the links to want to crawl deeper are accessible and we proceed to add in the following code.
```
// retrieve.go
...
  ...
for _, link := range links {
  //fmt.Println(link) // an iterator in many other languages get all links including relative
  absolute := fixUrl(link, uri)
  //fmt.Println(absolute)
  resp, _ := http.Get(absolute)

  switch resp.StatusCode {
  case 200:
    fmt.Println("[Up] \t\t", absolute)
  default:
    fmt.Println("[Down] \t\t", absolute)
  }
}
```

We make use of maps in order to prevent the crawler from digging deeper into the same url which would prevent an endless loop. A Sample of what it looks like is:
```
visitedPage   = make(map[string]bool)

if strings.Contains(absolute, baseURL) && !visitedPage[absolute] {
			checkWebStatus(absolute, uri)
		}
```

We proceed to upgrade our retrieve.go in such a way that we throw the url into a queue and allows our crawler to go digging into the urls that it has found The method we are using is recursive.
```
//replacement of retrieve.go
func enqueue(uri string) {
	//fmt.Println("fetching", uri)
	visitedPage[uri] = true
	resp, err := http.Get(uri)
	if err != nil {
		return
	}
	defer resp.Body.Close()
	links := collectlinks.All(resp.Body)
	for _, link := range links {
		absolute := fixUrl(link, uri)
		if !strings.Contains(absolute, baseURL) && !visitedLinks[absolute] {
			visitedLinks[absolute] = true
			checkWebStatus(absolute, uri)
		}
		if strings.Contains(absolute, baseURL) && !visitedPage[absolute] {
			UrlCrawlCount++
			//fmt.Println(absolute)
			checkWebStatus(absolute, uri)
			enqueue(absolute)
		}
	}
}
```
If you read the code, you will realize that we have applied the same theory to links so as to prevent checking the web status of the same link twice. With that, you have completed a fully-functional web crawler that is capable of digging into your website ensuring that the links in them are kept up to date.

```
package main

//Recurisve Web Crawling
import (
	"flag"
	"fmt"
	"github.com/briandowns/spinner"
	"github.com/jackdanger/collectlinks"
	"net/http"
	"net/url"
	"os"
	"strings"
	"time"
)

var (
	visitedPage   = make(map[string]bool)
	visitedLinks  = make(map[string]bool)
	BrokenPage    = make(map[string]string)
	UrlCrawlCount = 0
	Linkcount     = 0
	brokenLinks   = 0
	baseURL       = ""
)

func main() {
	flag.Parse()
	args := flag.Args()
	//fmt.Println(args)
	if len(args) < 1 {
		fmt.Println("Please specify start page")
		os.Exit(1)
	}
	baseURL = args[0]

	s := spinner.New(spinner.CharSets[9], 100*time.Millisecond)
	s.Prefix = fmt.Sprintf("crawling %s, please wait ", baseURL)
	s.Start()
	enqueue(baseURL)

	tmpCount := 0
	s.Stop()
	fmt.Println("================================================================================================")
	fmt.Println("================================================================================================")
	fmt.Println("Broken Links:", brokenLinks, "Ok Links:", Linkcount, "Web Pages Crawled:", UrlCrawlCount)
	for key, value := range BrokenPage {
		tmpCount++
		fmt.Println(fmt.Sprintf("[%v] \n broken  : %s \n source: %s", tmpCount, key, value))
	}
}

// fixUrl converts all relative links to absolute links
func fixUrl(href, base string) string {
	uri, err := url.Parse(href)
	if err != nil {
		return ""
	}
	baseUrl, err := url.Parse(base)
	if err != nil {
		return ""
	}
	uri = baseUrl.ResolveReference(uri)
	return uri.String()
}

func enqueue(uri string) {
	//fmt.Println("fetching", uri)
	visitedPage[uri] = true
	resp, err := http.Get(uri)
	if err != nil {
		return
	}
	defer resp.Body.Close()
	links := collectlinks.All(resp.Body)
	for _, link := range links {
		absolute := fixUrl(link, uri)
		if !strings.Contains(absolute, baseURL) && !visitedLinks[absolute] {
			visitedLinks[absolute] = true
			checkWebStatus(absolute, uri)
		}
		if strings.Contains(absolute, baseURL) && !visitedPage[absolute] {
			UrlCrawlCount++
			//fmt.Println(absolute)
			checkWebStatus(absolute, uri)
			enqueue(absolute)
		}
	}
}

// checkWebStatus checks all given links if they are invalid
func checkWebStatus(urlParams string, baseline string) {
	resp, _ := http.Get(urlParams)
	if resp != nil && resp.StatusCode == 200 {
		Linkcount++
	} else {
		brokenLinks++
		BrokenPage[urlParams] = baseline
	}
}
```

Get the full source code from [My Github Repo][3].
