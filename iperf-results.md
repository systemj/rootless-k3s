# iperf3 test results
- From `iperf3 -c iperf3` to over skupper connection.
- 1Gbit LAN

## iperf server machine (old)
- rke2 on bare metal
- Intel(R) Core(TM) i7-3770K CPU @ 3.50GHz

## rootless k3s - slirp4netns - KVM on fast machine
- 12th Gen Intel(R) Core(TM) i7-12700K
```
root@test:/# iperf3 -c iperf3
Connecting to host iperf3, port 5201
[  5] local 10.42.0.55 port 54070 connected to 10.43.195.150 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec   112 MBytes   941 Mbits/sec    2   1.48 MBytes
[  5]   1.00-2.00   sec   105 MBytes   882 Mbits/sec    2   1.48 MBytes
[  5]   2.00-3.00   sec   106 MBytes   886 Mbits/sec    4   1.48 MBytes
[  5]   3.00-4.00   sec   106 MBytes   886 Mbits/sec    4   1.48 MBytes
[  5]   4.00-5.00   sec   105 MBytes   883 Mbits/sec    3   1.48 MBytes
[  5]   5.00-6.00   sec   105 MBytes   883 Mbits/sec    4   1.48 MBytes
[  5]   6.00-7.00   sec   105 MBytes   883 Mbits/sec    4   1.48 MBytes
[  5]   7.00-8.00   sec   105 MBytes   883 Mbits/sec    4   1.48 MBytes
[  5]   8.00-9.00   sec   105 MBytes   883 Mbits/sec    1   1.48 MBytes
[  5]   9.00-10.00  sec   105 MBytes   881 Mbits/sec    4   1.48 MBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  1.04 GBytes   889 Mbits/sec   32             sender
[  5]   0.00-10.00  sec  1.03 GBytes   882 Mbits/sec                  receiver
```

## rootless k3s - slirp4netns - bare metal on slow machine
- Intel(R) Core(TM) i7-3537U CPU @ 2.00GHz
- slirp4netns definitely taking up 75% CPU
```
root@test1:/# iperf3 -c iperf3
Connecting to host iperf3, port 5201
[  5] local 10.42.0.21 port 49496 connected to 10.43.89.158 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec  61.8 MBytes   518 Mbits/sec    5    445 KBytes
[  5]   1.00-2.00   sec  57.1 MBytes   479 Mbits/sec    6    445 KBytes
[  5]   2.00-3.00   sec  57.1 MBytes   479 Mbits/sec    8    445 KBytes
[  5]   3.00-4.00   sec  60.2 MBytes   505 Mbits/sec    0    445 KBytes
[  5]   4.00-5.00   sec  57.1 MBytes   479 Mbits/sec    1    445 KBytes
[  5]   5.00-6.00   sec  58.9 MBytes   494 Mbits/sec    4    445 KBytes
[  5]   6.00-7.00   sec  61.0 MBytes   512 Mbits/sec    1    445 KBytes
[  5]   7.00-8.00   sec  58.6 MBytes   492 Mbits/sec    1    445 KBytes
[  5]   8.00-9.00   sec  57.6 MBytes   483 Mbits/sec    2    445 KBytes
[  5]   9.00-10.00  sec  59.0 MBytes   495 Mbits/sec    2    445 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec   588 MBytes   494 Mbits/sec   30             sender
[  5]   0.00-10.00  sec   586 MBytes   491 Mbits/sec                  receiver

```
