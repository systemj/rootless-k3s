# iperf3 test results
- 1Gbit LAN

## iperf server machine (old)
- rke2 on bare metal
- Intel(R) Core(TM) i7-3770K CPU @ 3.50GHz

## rootless k3s - slirp4netns - KVM on fast machine
- 12th Gen Intel(R) Core(TM) i7-12700K

### inside k3s over skupper tunnel
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

### inside k3s to bare metal endpoint
```
root@test:/# iperf3 -c 192.168.86.37
Connecting to host 192.168.86.37, port 5201
[  5] local 10.42.0.64 port 56098 connected to 192.168.86.37 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec   114 MBytes   957 Mbits/sec    0   68.8 KBytes
[  5]   1.00-2.00   sec   112 MBytes   941 Mbits/sec    0   68.8 KBytes
[  5]   2.00-3.00   sec   112 MBytes   943 Mbits/sec    0   68.8 KBytes
[  5]   3.00-4.00   sec   112 MBytes   937 Mbits/sec    0   68.8 KBytes
[  5]   4.00-5.00   sec   112 MBytes   942 Mbits/sec    0   68.8 KBytes
[  5]   5.00-6.00   sec   112 MBytes   943 Mbits/sec    0   68.8 KBytes
[  5]   6.00-7.00   sec   113 MBytes   949 Mbits/sec    0   68.8 KBytes
[  5]   7.00-8.00   sec   111 MBytes   932 Mbits/sec    0   68.8 KBytes
[  5]   8.00-9.00   sec   112 MBytes   944 Mbits/sec    0   68.8 KBytes
[  5]   9.00-10.00  sec   113 MBytes   947 Mbits/sec    0   68.8 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  1.10 GBytes   943 Mbits/sec    0             sender
[  5]   0.00-10.01  sec  1.09 GBytes   940 Mbits/sec                  receiver

iperf Done.
```

### outside k3s
```
$ iperf3 -c 192.168.86.37
Connecting to host 192.168.86.37, port 5201
[  5] local 192.168.122.36 port 53906 connected to 192.168.86.37 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec   116 MBytes   971 Mbits/sec   90    783 KBytes
[  5]   1.00-2.00   sec   112 MBytes   942 Mbits/sec   45    650 KBytes
[  5]   2.00-3.00   sec   112 MBytes   943 Mbits/sec    0    771 KBytes
[  5]   3.00-4.00   sec   112 MBytes   940 Mbits/sec   45    626 KBytes
[  5]   4.00-5.00   sec   112 MBytes   941 Mbits/sec    0    752 KBytes
[  5]   5.00-6.00   sec   112 MBytes   935 Mbits/sec   45    602 KBytes
[  5]   6.00-7.00   sec   113 MBytes   948 Mbits/sec    0    731 KBytes
[  5]   7.00-8.00   sec   112 MBytes   935 Mbits/sec    0    843 KBytes
[  5]   8.00-9.00   sec   112 MBytes   943 Mbits/sec   45    717 KBytes
[  5]   9.00-10.00  sec   112 MBytes   941 Mbits/sec    0    829 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  1.10 GBytes   944 Mbits/sec  270             sender
[  5]   0.00-10.00  sec  1.10 GBytes   941 Mbits/sec                  receiver

iperf Done.
```


## rootless k3s - slirp4netns - bare metal on slow machine
- Intel(R) Core(TM) i7-3537U CPU @ 2.00GHz
- slirp4netns definitely taking up 75% CPU

### inside k8s over skupper tunnel
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

iperf Done.
```

### inside k3s to bare metal endpoint
```
root@test:/# iperf3 -c 192.168.86.37
Connecting to host 192.168.86.37, port 5201
[  5] local 10.42.0.29 port 59884 connected to 192.168.86.37 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec  71.5 MBytes   599 Mbits/sec    0   68.8 KBytes
[  5]   1.00-2.00   sec  74.2 MBytes   623 Mbits/sec    0   68.8 KBytes
[  5]   2.00-3.00   sec  67.6 MBytes   567 Mbits/sec    0   68.8 KBytes
[  5]   3.00-4.00   sec  71.4 MBytes   599 Mbits/sec    0   68.8 KBytes
[  5]   4.00-5.00   sec  74.5 MBytes   625 Mbits/sec    0   68.8 KBytes
[  5]   5.00-6.00   sec  74.9 MBytes   628 Mbits/sec    0   68.8 KBytes
[  5]   6.00-7.00   sec  73.9 MBytes   620 Mbits/sec    0   68.8 KBytes
[  5]   7.00-8.00   sec  71.9 MBytes   603 Mbits/sec    0   68.8 KBytes
[  5]   8.00-9.00   sec  73.9 MBytes   620 Mbits/sec    0   68.8 KBytes
[  5]   9.00-10.00  sec  74.5 MBytes   625 Mbits/sec    0   68.8 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec   728 MBytes   611 Mbits/sec    0             sender
[  5]   0.00-10.00  sec   728 MBytes   611 Mbits/sec                  receiver

iperf Done.
```

### outside k3s
```
$ iperf3 -c 192.168.86.37
Connecting to host 192.168.86.37, port 5201
[  5] local 192.168.86.36 port 35354 connected to 192.168.86.37 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec   113 MBytes   948 Mbits/sec    0    314 KBytes
[  5]   1.00-2.00   sec   111 MBytes   928 Mbits/sec    0    346 KBytes
[  5]   2.00-3.00   sec   111 MBytes   931 Mbits/sec    0    346 KBytes
[  5]   3.00-4.00   sec   112 MBytes   940 Mbits/sec    0    365 KBytes
[  5]   4.00-5.00   sec   112 MBytes   943 Mbits/sec    0    365 KBytes
[  5]   5.00-6.00   sec   112 MBytes   942 Mbits/sec    0    365 KBytes
[  5]   6.00-7.00   sec   112 MBytes   938 Mbits/sec    0    383 KBytes
[  5]   7.00-8.00   sec   112 MBytes   941 Mbits/sec    0    383 KBytes
[  5]   8.00-9.00   sec   113 MBytes   947 Mbits/sec    0    383 KBytes
[  5]   9.00-10.00  sec   112 MBytes   940 Mbits/sec    0    383 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  1.09 GBytes   940 Mbits/sec    0            sender
[  5]   0.00-10.00  sec  1.09 GBytes   939 Mbits/sec                  receiver

iperf Done.
```
