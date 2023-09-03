+++
title = "Golang testing using docker services via dockertest"
date = "2023-09-03T05:05:46+04:00"
author = "Danny Hawkins"
authorTwitter = "dannyhawkins"
categories = ["coding"]
tags = ["golang", "docker","dockertest"]
showFullContent = false
readingTime = true
hideComments = false
+++

During my path learning go so far I have come across some amazing libraries and utilites, one of my favourite for integration testing is [dockertest](https://github.com/ory/dockertest).

Whenever I am using a service backed by postgres, mongo, mysql or other services that are not part of my codebase, I generally create a [docker-compose](https://docs.docker.com/compose/) file so that my development environments are isolated. Then when I'm working in a particular project, all I have to do is `docker-compose up -d` to get going, and `docker-compose down` when I'm finished for the day.

For example if I just need a postgres database I would usually have something like

```yaml
services:
  postgres:
    image: postgres:15
    ports:
      - 5432:5432
    volumes:
      - pg-db:/var/lib/postgresql/data
    environment:
      POSTGRES_HOST_AUTH_METHOD: trust
      POSTGRES_DB: example

volumes:
  pg-data:
```

This is great for development, but when running integration tests, if I want a clean database is means I need some additional steps. I can either:

- Manually drop and create the database
- Drop and create the database as part of the integration test
- Many other options

Instead of creating some automation to drop and create databases or tables, dockertest allows us to create a completley clean isolated instance of the service, so we know every time we are starting from nothing. This follow the [cattle not pets](https://geektechstuff.com/2021/06/24/devops-what-does-cattle-not-pets-mean/) idealology from devops.

We are just discussing postgres here, but some other docker services might require a LOT manual steps to get into a good condition for test runs.

## Example

I created an example project [here](https://github.com/danhawkins/go-dockertest-example) to show everything in actions, but I will work through each step I took.

The project I created is very simple and the main work is in the following code:

```golang
package database

import (
	"fmt"
	"log"
	"os"

	"gorm.io/driver/postgres"
	"gorm.io/gorm"
)

type (
	Person struct {
		gorm.Model
		Name string
		Age  uint
	}
)

var db *gorm.DB

func Connect() {
	log.Println("Setting up the database")

	pgUrl := fmt.Sprintf("postgresql://postgres@127.0.0.1:%s/example", os.Getenv("POSTGRES_PORT"))
	log.Printf("Connecting to %s\n", pgUrl)
	var err error

	db, err = gorm.Open(postgres.Open(pgUrl), &gorm.Config{})

	if err != nil {
		panic("failed to connect database")
	}

	// Migrate the schema
	db.AutoMigrate(&Person{})
}

func CreatePerson() {
	log.Println("Creating a new person in the database")
	person := Person{Name: "Danny", Age: 42}
	db.Create(&person)

	log.Println("Trying to write a new person to the database")
}

func CountPeople() int {
	var count int64
	db.Model(&Person{}).Count(&count)
	return int(count)
}
```

So we have a method to connect, a method to create a new person record and a method to count the entries. These are all called from the main entry point

```golang
func main() {
	database.Connect()

	database.CreatePerson()

	count := database.CountPeople()

	log.Printf("Database has %d people", count)
}
```

If I run this from the command line (once docker compose is up) the result increments the count on every run (as it should)

```bash
go-dockertest-example λ git main → go run main.go
2023/09/03 13:41:06 Setting up the database
2023/09/03 13:41:06 Connecting to postgresql://postgres@127.0.0.1:5432/example
2023/09/03 13:41:06 Creating a new person in the database
2023/09/03 13:41:06 Trying to write a new person to the database
2023/09/03 13:41:06 Database has 3 people

go-dockertest-example λ git main → go run main.go
2023/09/03 13:41:07 Setting up the database
2023/09/03 13:41:07 Connecting to postgresql://postgres@127.0.0.1:5432/example
2023/09/03 13:41:08 Creating a new person in the database
2023/09/03 13:41:08 Trying to write a new person to the database
2023/09/03 13:41:08 Database has 4 people
```

## The Test

Now lets look at the test. First there is the setup of dockertest:

```golang
func TestMain(m *testing.M) {
	// Start a new docker pool
	pool, err := dockertest.NewPool("")
	if err != nil {
		log.Fatalf("Could not construct pool: %s", err)
	}

	// Uses pool to try to connect to Docker
	err = pool.Client.Ping()
	if err != nil {
		log.Fatalf("Could not connect to Docker: %s", err)
	}

	pg, err := pool.RunWithOptions(&dockertest.RunOptions{
		Repository: "postgres",
		Tag:        "15",
		Env: []string{
			"POSTGRES_DB=example",
			"POSTGRES_HOST_AUTH_METHOD=trust",
			"listen_addresses = '*'",
		},
	}, func(config *docker.HostConfig) {
		// set AutoRemove to true so that stopped container goes away by itself
		config.AutoRemove = true
		config.RestartPolicy = docker.RestartPolicy{
			Name: "no",
		}
	})

	if err != nil {
		log.Fatalf("Could not start resource: %s", err)
	}

	pg.Expire(10)

	// Wait for the Postgres to be ready
	if err := pool.Retry(func() error {
		_, connErr := gorm.Open(postgres.Open(fmt.Sprintf("postgresql://postgres@localhost:%s/example", postgresPort)), &gorm.Config{})
		if connErr != nil {
			return connErr
		}

		return nil
	}); err != nil {
		panic("Could not connect to postgres: " + err.Error())
	}

	// Set this so our app can use it
	postgresPort := pg.GetPort("5432/tcp")
	os.Setenv("POSTGRES_PORT", postgresPort)

	code := m.Run()

	os.Exit(code)
}
```

So first we create a new docker pool, we make sure the pool responds to a ping then start a postgres instance. When you start a new instance it will grab a random port, so you need a way to pass this to the service, this is why we have the `POSTGRES_PORT` env var. At the very end of the module setup we set that env so that test will use it.

```golang
	// Set this so our app can use it
	postgresPort := pg.GetPort("5432/tcp")
	os.Setenv("POSTGRES_PORT", postgresPort)
```

There is a section for waiting for postgres to be ready, you can use this in different ways, but basically you are using some condition to test the instance is ready. This could be anything from checking a port is addressable, to calling a http healthcheck.

```golang
	// Wait for the Postgres to be ready
	if err := pool.Retry(func() error {
		_, connErr := gorm.Open(postgres.Open(fmt.Sprintf("postgresql://postgres@localhost:%s/example", postgresPort)), &gorm.Config{})
		if connErr != nil {
			return connErr
		}

		return nil
	}); err != nil {
		panic("Could not connect to postgres: " + err.Error())
	}
```

Then there is:

```golang
  // Make sure we expire the instance after 10 seconds
	postgres.Expire(10)
```

this ensures if cleanup does not succeed we will remove the docker instance after 10 seconds regardless.

Finally we have the actual test

```golang
func TestCreatePerson(t *testing.T) {
	// Connect to the database
	database.Connect()

	// Create a person in the database
	database.CreatePerson()

	// Check that the person was created
	count := database.CountPeople()

	if count != 1 {
		t.Errorf("Expected 1 person to be in the database, got %d", count)
	}
}
```

Each time the test runs there will only be one record in the database.
