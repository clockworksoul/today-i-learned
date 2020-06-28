# Ballpark I/O performance figures

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
