# Hexagonal Architecture

_Sources: [1](https://web.archive.org/web/20180822100852/http://alistair.cockburn.us/Hexagonal+architecture), [2](https://eskavision.com/hexagonal-architecture/), [3](https://jmgarridopaz.github.io/content/hexagonalarchitecture.html), [4](https://en.wikipedia.org/wiki/Hexagonal_architecture_(software)), [5](https://beyondxscratch.com/2017/08/19/decoupling-your-technical-code-from-your-business-logic-with-the-hexagonal-architecture-hexarch/)_

Hexagonal architecture is a software architectural pattern that uses the principles of _loose coupling_ to allow inputs and outputs to be easily connected.

It could be said that hexagonal archiecture is simply [_inversion of control_](https://en.wikipedia.org/wiki/Inversion_of_control) taken to its logical conclusion.

I'll explain.

Hexagonal architecture can be described as an architecture built around the principle of inversion of control, in which a program only knows about a component as an abstraction, passing the control of the functionality to the implentation.

## The Name

The name "hexagonal architecture" doesn't actually mean anything. It's a silly name that has nothing to do with the implementation. This actually confused me for a while because I kept trying to find the connection between the name and the implementation that didn't exist.

Seriously, the [original author of hexagonal architecture](https://web.archive.org/web/20180822100852/http://alistair.cockburn.us/Hexagonal+architecture) chose this shape simply because it allowed enough space to illustrate all the components.

## The Architecture

The guiding principle of hexagonal architecture is that functionalities should be represented by abstractions into which specific implementations can be [injected](https://en.wikipedia.org/wiki/Dependency_injection).

Using this principle, an application might be structured so that it could be run by different kinds of clients (humans, tests cases, other applications, etc.), and it could be tested in isolation from external devices of the real world that the application depends on (databases, servers, other applications, etc.).

This can be illustrated by breaking the architecture down as follows:

* The **hexagon** represents the application itself. It contains the business logic, but no _direct_ references to any technology, framework or real world device.
* The **drivers** can be anything that triggers an interaction with the application, such as a user.
* The **driven actors** can be anything the application triggers an interaction with, such as a database.

![Figure 1](https://eskavision.com/wp-content/uploads/2020/04/figure1.png)

### The Hexagon

Inside the hexagon we just have the things that are important for the business problem that the application is trying to solve. It can be whatever you want.

### The Actors

Outside the hexagon we have the "actors": any real world thing that the application interacts with, including humans, devices, or other applications.

As mentioned above, there are two kinds of actors:

* _Driver actors_ can be anything that triggers an interaction with the application. Drivers are the users (either humans or devices) of the application.
* _Driven actors_ can be anything the application triggers an interaction with. A driven actor provides some functionality needed by the application for implementing the business logic.

### Ports and Adaptors

_Ports_ are the places where different kinds of actors can "plug in".

_Adapters_ are the software components -- implementations -- that can "plug into" a particular kind of ports.

For example, your application might have a "logging port", into which a "logging adapter" might plug. One logging adapter might write to a local file, while another might stream to a cloud-based data store.

### Architecture Summary

As we have seen, the elements of the architecture are:

* The Hexagon ==> the application
  * Driver Ports ==> API offered by the application
  * Driven Ports ==> SPI required by the application

* Actors ==> environment devices that interact with the application
  * Drivers ==> application users (either humans or hardware/software devices)
  * Driven Actors ==> provide services required by the application

* Adapters ==> adapt specific technology to the application
  * Driver Adapters ==> use the drivers ports
  * Driven Adapters ==> implement the driven ports

## Hexagon Implementation

![Figure 2](https://eskavision.com/wp-content/uploads/2020/04/hexagonal-architecture.png)

### Domain

```go
type Book struct {
    Id   int
    Name string
}
```

### Driven Port

In this example, we have a port for interacting with our data repository. In this example, it specifies a single method that accepts a 

```go
type BookRepository interface {
    FindById(id int) (Book, error)
}
```

### Database Driven Port Adapter

A driven adapter implements a driven port interface, converting the technology agnostic methods of the port into specific technology methods.

This example implicitly implements `BookRepository` by satisfying its `FindById` contract.

```go
type DatabaseBookRepository struct {
	configs data.DatabaseConfigs
	db      *sql.DB
}

func (a *DatabaseBookRepository) FindById(id) (Book, error) {
	db, err := a.connect("books")
	defer db.Close()
	if err != nil {
		return Book{}, err
	}

	query := "SELECT name FROM books WHERE id=$1"
	book := Book{Id:id}
	err = db.QueryRow(query, id).Scan(&book.Name)
	if err != nil {
		return book, err
	}

    return book, nil
}
```
