# Little’s Law

_Source: [robertovitillo.com](https://robertovitillo.com/back-of-the-envelope-estimation-hacks/) (Check out [his book](https://leanpub.com/systemsmanual)!)_

A queue can be modeled with three parameters:

* the average rate at which new items arrive, λ
* the average time an item spends in the queue, W
* and the average number of items in the queue, L

Lots of things can be modeled as queues. A web service can be seen as a queue, for example. The request rate is the rate at which new items arrive. The time it takes for a request to be processed is the time an item spends in the queue. Finally, the number of concurrently processed requests is the number of items in the queue.

Wouldn’t it be great if you could derive one of the three parameters from the other two? It turns out there is a law that relates these three quantities to one another! It’s called [Little’s Law](https://en.wikipedia.org/wiki/Little%27s_law):

```
L = λ * W
```

What it says is that the average number of items in the queue equals the average rate at which new items arrive, multiplied by the average time an item spends in the queue.

Let’s try it out. Let’s say you have a service that takes on average 100ms to process a request. It’s currently receiving about two million requests per second (rps). How many requests are being processed concurrently?

```
requests = 2e6 requests/s * 1e-1 s = 2e5
```

The service is processing 200K requests concurrently. If each request is CPU heavy and requires a thread, then we will need about 200K threads.

If we are using 8 core machines, then to keep up, we will need about 25K machines.
