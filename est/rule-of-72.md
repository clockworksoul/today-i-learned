# The Rule of 72

_Source: [robertovitillo.com](https://robertovitillo.com/back-of-the-envelope-estimation-hacks/) (Check out [his book](https://leanpub.com/systemsmanual)!)_

[The rule of 72](https://web.stanford.edu/class/ee204/TheRuleof72.html) is a method to estimate how long it will take for a quantity to double if it grows at a certain percentage rate.

```
time = 72 / rate
```
 
For example, let’s say the traffic to your web service is increasing by approximately 10% weekly, how long will it take to double?

```
time = 72 / 10 = 7.2
```

It will take approximately 7 weeks for the traffic to double.

If you combine the rule of 72 with the powers of two, then you can quickly find out how long it will take for a quantity to increase by several orders of magnitudes.
