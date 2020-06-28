# Approximate with powers

_Source: [robertovitillo.com](https://robertovitillo.com/back-of-the-envelope-estimation-hacks/) (Check out [his book](https://leanpub.com/systemsmanual)!)_ 

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
