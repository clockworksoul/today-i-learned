# Amazon EBS Volume Types

*Source: [Amazon EBS volume types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html)*

Amazon EBS provides the following volume types, which differ in performance characteristics and price, so that you can tailor your storage performance and cost to the needs of your applications. The volumes types fall into these categories:

* Solid state drives (SSD) — Optimized for transactional workloads involving frequent read/write operations with small I/O size, where the dominant performance attribute is IOPS.

* Hard disk drives (HDD) — Optimized for large streaming workloads where the dominant performance attribute is throughput.

* Previous generation — Hard disk drives that can be used for workloads with small datasets where data is accessed infrequently and performance is not of primary importance. We recommend that you consider a current generation volume type instead.

There are several factors that can affect the performance of EBS volumes, such as instance configuration, I/O characteristics, and workload demand. To fully use the IOPS provisioned on an EBS volume, use EBS-optimized instances. For more information about getting the most out of your EBS volumes, see Amazon EBS volume performance on Linux instances.

## Solid state drives (SSD)

The SSD-backed volumes provided by Amazon EBS fall into these categories:

* General Purpose SSD — Provides a balance of price and performance. We recommend these volumes for most workloads.

* Provisioned IOPS SSD — Provides high performance for mission-critical, low-latency, or high-throughput workloads.

The following is a summary of the use cases and characteristics of SSD-backed volumes. For information about the maximum IOPS and throughput per instance, see [Amazon EBS–optimized instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-optimized.html).

### General Purpose SSD

#### `gp2`

* **Durability:** 99.8% - 99.9%
* **Use cases:** low-latency interactive apps; development and test environments
* **Volume size:** 1 GiB - 16 TiB
* **IOPS per volume (16 KiB I/O):** 3 IOPS per GiB of volume size (100 IOPS at 33.33 GiB and below to 16,000 IOPS at 5,334 GiB and above)
* **Throughput per volume:** 250 MiB/s
* **Amazon EBS Multi-attach supported?** No
* **Boot volume supported?** Yes

#### `gp3`

* **Durability:** 99.8% - 99.9%
* **Use cases:** low-latency interactive apps; development and test environments
* **Volume size:** 1 GiB - 16 TiB
* **IOPS per volume (16 KiB I/O):** 3,000 IOPS base; can be increased up to 16,000 IOPS (see below)
* **Throughput per volume:** 125 MiB/s base; can be increased up to 1,000 MiB/s (see below)
* **Amazon EBS Multi-attach supported?** No
* **Boot volume supported?** Yes

The maximum ratio of provisioned IOPS to provisioned volume size is 500 IOPS per GiB. The maximum ratio of provisioned throughput to provisioned IOPS is .25 MiB/s per IOPS. The following volume configurations support provisioning either maximum IOPS or maximum throughput:

* 32 GiB or larger: 500 IOPS/GiB x 32 GiB = 16,000 IOPS
* 8 GiB or larger and 4,000 IOPS or higher: 4,000 IOPS x 0.25 MiB/s/IOPS = 1,000 MiB/s


### Provisioned IOPS SSD

#### `io1`

* **Durability:** 99.8% - 99.9% durability
* **Use cases:** Workloads that require sustained IOPS performance or more than 16,000 IOPS; I/O-intensive database workloads
* **Volume size:** 4 GiB - 16 TiB
* **IOPS per volume (16 KiB I/O):** 64,000 (see note 2)
* **Throughput per volume:** 1,000 MiB/s
* **Amazon EBS Multi-attach supported?** Yes
* **Boot volume supported?** Yes

Note 2: Maximum IOPS and throughput are guaranteed only on Instances built on the Nitro System provisioned with more than 32,000 IOPS. Other instances guarantee up to 32,000 IOPS and 500 MiB/s. io1 volumes that were created before December 6, 2017 and that have not been modified since creation might not reach full performance unless you modify the volume.

#### `io2`

* **Durability:** 99.999% durability
* **Use cases:** Workloads that require sustained IOPS performance or more than 16,000 IOPS; I/O-intensive database workloads
* **Volume size:** 4 GiB - 16 TiB
* **IOPS per volume (16 KiB I/O):** 64,000 (see note 3)
* **Throughput per volume:** 1,000 MiB/s
* **Amazon EBS Multi-attach supported?** Yes
* **Boot volume supported?** Yes

Note 3: Maximum IOPS and throughput are guaranteed only on Instances built on the Nitro System provisioned with more than 32,000 IOPS. Other instances guarantee up to 32,000 IOPS and 500 MiB/s. io1 volumes that were created before December 6, 2017 and that have not been modified since creation might not reach full performance unless you modify the volume.

#### `io2 Block Express` (in opt-in preview only as of March 2021)

* **Durability:** 99.999% durability
* **Use cases:** Workloads that require sub-millisecond latency, and sustained IOPS performance or more than 64,000 IOPS or 1,000 MiB/s of throughput
* **Volume size:** 4 GiB - 64 TiB
* **IOPS per volume (16 KiB I/O):** 256,000
* **Throughput per volume:** 4,000 MiB/s
* **Amazon EBS Multi-attach supported?** No
* **Boot volume supported?** Yes
