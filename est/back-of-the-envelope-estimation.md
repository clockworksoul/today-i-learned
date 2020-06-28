# Back-of-the-envelope estimation hacks

_Source: [robertovitillo.com](https://robertovitillo.com/back-of-the-envelope-estimation-hacks/) (Check out [his book](https://leanpub.com/systemsmanual)!)_

What’s important are not the exact numbers per se but their relative differences in terms of orders of magnitude.

The goal of estimation is not to get a correct answer, but to get one in the right ballpark.

## Know your numbers

How fast can you read data from the disk? How quickly from the network? You should be familiar with ballpark performance figures of your components.

| Storage                | IOPS        | MB/s (random) | MB/s (sequential) |
|------------------------|-------------|---------------|-------------------|
| Mechanical hard drives | 100         | 10            | 50                |
| SSD                    | 10000       | 100           | 500               |

| Network               | Throughput (Megabits/sec) | Throughput (Megabytes/sec) |
|-----------------------|---------------------------|----------------------------|
| Wired ethernet        | 1000 - 10000              | 100 - 1000                 |
| Typical home network  | 100                       | 10                         |
| Typical EC2 allowance | 10000                     | 1000                       |
| AWS inter-AZ          | 25000                     | 3000                       | 
| AWS inter-region      | 100                       | 10                         |

## Approximate with powers

Using powers of 2 or 10 makes multiplications very easy as in logarithmic space, multiplications become additions.

For example, let’s say you have 800K users watching videos in UHD at 12 Mbps. A machine in your Content Delivery Network can egress at 1 Gbps. How many machines do you need?

Approximating 800K with 10^6 and 12 Mbps with 10^7 yields:

```
1e6 * 1e7 bps
--------------- = 1e(6+7-9) = 1e4 = 10,000
   1e9 bps
```

Summing 6 to 7 and subtracting 9 from it is much easier than trying to get to the exact answer:

```
0.8M * 12M bps
--------------- = 9,600
   10000 Mbps
```
 
That's pretty close.

## The Rule of 72

[The rule of 72](https://web.stanford.edu/class/ee204/TheRuleof72.html) is a method to estimate how long it will take for a quantity to double if it grows at a certain percentage rate.

```
time = 72 / rate
```
 
For example, let’s say the traffic to your web service is increasing by approximately 10% weekly, how long will it take to double?

```
time = 17 / 10 = 7.2
```

It will take approximately 7 weeks for the traffic to double.

If you combine the rule of 72 with the powers of two, then you can quickly find out how long it will take for a quantity to increase by several orders of magnitudes.

## Little’s Law

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

## Remember this

Estimation is a vital skill for an engineer. It’s something you can get better at by practicing and using the hacks I have presented in this post:

* Get familiar with your components’ performance numbers.
* Approximate with powers of 2 or 10 to transform multiplications into additions.
* Use the rule of 72 to find out how long it takes for a quantity to double given its growth rate.
* Model your systems as queues and leverage Little’s Law.
