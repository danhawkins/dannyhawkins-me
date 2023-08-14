+++
title = "Embed Vite React in Golang binary, with live reload!"
date = "2023-07-14T11:05:46+04:00"
author = "Danny Hawkins"
authorTwitter = "dannyhawkins"
categories = ["coding"]
featuredImage = "posts/go-react-vite.png"
tags = ["go", "react", "vite", "fullstack"]
keywords = ["go", "react", "vite", "embed", "live reload"]
showFullContent = false
readingTime = true
hideComments = false
+++

I recently followed this [excellent article](https://dev.to/pacholoamit/one-of-the-coolest-features-of-go-embed-reactjs-into-a-go-binary-41e9) from [Pacholo Amit](https://dev.to/pacholoamit), he documents a method where you can embed the static assets built from Vite using the Echo framework in go, into a single binary.

Whilst his solution was great, it was missing something still for me, I like to be able to make changes in code / css and have it instantly updated in the browser, being able to see the frontend change with live reload is a must have.

I created an example project of my own that starts of using his method but then adds a live reloading method in the first Pull Request https://github.com/danhawkins/go-vite-react-example

## The Solution

I solved it in [this pr](https://github.com/danhawkins/go-vite-react-example/pull/1) where we run the Vite dev server alongside a proxy mode of the echo server in Golang, meaning that we proxy all requests that are relevant directly to the vite server, but we only do this in dev mode.

In our case dev mode is defined by the environment variable ENV being "dev".

The first step is to create a .env file

```bash
ENV=dev
```

Then we update the frontend.go file likes so
```go
package frontend

import (
	"embed"
	"log"
	"net/url"
	"os"

	_ "github.com/joho/godotenv/autoload"
	"github.com/labstack/echo/v4"
	"github.com/labstack/echo/v4/middleware"
)

var (
	//go:embed dist/*
	dist embed.FS

	//go:embed dist/index.html
	indexHTML embed.FS

	distDirFS     = echo.MustSubFS(dist, "dist")
	distIndexHTML = echo.MustSubFS(indexHTML, "dist")
)

func RegisterHandlers(e *echo.Echo) {
	if os.Getenv("ENV") == "dev" {
		log.Println("Running in dev mode")
		setupDevProxy(e)
		return
	}
	// Use the static assets from the dist directory
	e.FileFS("/", "index.html", distIndexHTML)
	e.StaticFS("/", distDirFS)
}

func setupDevProxy(e *echo.Echo) {
	url, err := url.Parse("http://localhost:5173")
	if err != nil {
		log.Fatal(err)
	}
	// Setep a proxy to the vite dev server on localhost:5173
	balancer := middleware.NewRoundRobinBalancer([]*middleware.ProxyTarget{
		{
			URL: url,
		},
	})

	e.Use(middleware.ProxyWithConfig(middleware.ProxyConfig{
		Balancer: balancer,
		Skipper: func(c echo.Context) bool {
			// Skip the proxy if the prefix is /api
			return len(c.Path()) >= 4 && c.Path()[:4] == "/api"
		},
	}))
}

```

We have a new condition in RegiserHandlers that checks if the env is dev and if so we call setupDevProxy.

All this does is create a balancer group with one member (the vite dev server) and use a skipper function if the path is prefixed with /api. This keeps our api routes isolated.

I found this to work really well so far, and the developer experience is the best I have had so far when working on fullstack web applications.