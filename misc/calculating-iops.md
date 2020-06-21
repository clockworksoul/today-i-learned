# Calculating IOPS from Latency

IOPS stands for input/output operations per second. It represents how quickly a given storage device or medium can read and write commands in every second.

It is calulated as:

```IOPS = 1000 milliseconds / (Average Seek Time + Average Latency)```

Generally a HDD will have an IOPS range of 55-180, while a SSD will have an IOPS from 3,000 – 40,000.

## HDD

For HDD with spinning disks, this is dependent rotational speed, in which average latency is the time it takes the disk platter to spin halfway around.

It is calculated by dividing 60 seconds by the Rotational Speed of the disk, then dividing that result by 2 and multiplying everything by 1,000 milliseconds:

```
IOPS = ((60 / RPM)/2) * 1000
```

## SSD

There's no spinning disk, so as a rule of thumb you can just plug in 0.1 ms as the average latency.
