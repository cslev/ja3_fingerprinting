# ja3_fingerprinting
This repository shows that while JA3(S) fingerprinting is really useful to maintain a fingerprint database of clients in a controlled environment, identifying individual Web services in the wild, however, is not trivial and sometimes provides the same fingerprint for different services.

## Btw., what is JA3(S) fingerprinting?
TL;DR - A JA3 hash represents the fingerprint of an SSL/TLS client application that allows for simple and effective detection of client applications such as Chrome running on OSX (JA3=94c485bca29d5392be53f2b8cf7f4304) or the Dyre malware family running on Windows (JA3=b386946a5a44d1ddcc843bc75336dfce) or Metasploit’s Meterpreter running on Linux (JA3=5d65ea3fb1d4aa7d826733d2f2cbbb1d). JA3 allows us to detect these applications, malware families, and pen testing tools, regardless of their destination, Command and Control (C2) IPs, or SSL certificates.

[More information about JA3(S)](https://engineering.salesforce.com/tls-fingerprinting-with-ja3-and-ja3s-247362855967)


## Requirements for reproducing
The scripts heavily depend on the python library `dpkt`. The easiest way to get it is as follows:
```
$ sudo apt-get install python3-dpkt
```
The rest of the requirements are probably installed by default on your system. If not, please refer to the `import` section of the source file and install all required libraries accordingly.


## Are fingeprints unique enough
Here, we quickly show (in a reproducible way) that JA3S fingerprints can be the same for multiple services.

In the sources you will find a pcap file, called `fingerprint_test.pcap`. 
Using the script `ja3.py`, or more precisely `ja3s.py` for server fingerprinting, and filtering on a specific IP for presentational purposes, we get the following output:
```
$ python3 ja3s.py -j |grep "104.16.248.249" -B 1
        "ja3_digest": "eb1d94daa7e0344597e756a1fb6e7054",
        "source_ip": "104.16.248.249",
--
        "ja3_digest": "eb1d94daa7e0344597e756a1fb6e7054",
        "source_ip": "104.16.248.249",
--
        "ja3_digest": "2b0648ab686ee45e0e7c35fcfb0eea7e",
        "source_ip": "104.16.248.249",
--
        "ja3_digest": "2b0648ab686ee45e0e7c35fcfb0eea7e",
        "source_ip": "104.16.248.249",
--
        "ja3_digest": "2b0648ab686ee45e0e7c35fcfb0eea7e",
        "source_ip": "104.16.248.249",
--
        "ja3_digest": "2b0648ab686ee45e0e7c35fcfb0eea7e",
        "source_ip": "104.16.248.249",
--
        "ja3_digest": "2b0648ab686ee45e0e7c35fcfb0eea7e",
        "source_ip": "104.16.248.249",
--
        "ja3_digest": "2b0648ab686ee45e0e7c35fcfb0eea7e",
        "source_ip": "104.16.248.249",
--
        "ja3_digest": "2b0648ab686ee45e0e7c35fcfb0eea7e",
        "source_ip": "104.16.248.249",
--
        "ja3_digest": "2b0648ab686ee45e0e7c35fcfb0eea7e",
        "source_ip": "104.16.248.249",
--
        "ja3_digest": "2b0648ab686ee45e0e7c35fcfb0eea7e",
        "source_ip": "104.16.248.249",
--
        "ja3_digest": "2b0648ab686ee45e0e7c35fcfb0eea7e",
        "source_ip": "104.16.248.249",
```
As you can see, the Cloudflare's DoH resolver's `ServerHello` (`source_ip: 104.16.248.249`) is present several times in the pcap, however, not always with the same JA3S hash.

Okay, now check whether two different services have the same JA3S fingerprint.
Let's pick the following fingerprint: `eb1d94daa7e0344597e756a1fb6e7054`.
```
$ python3 filters/ja3s.py fingerprint_test.pcap  -j|grep "eb1d94daa7e0344597e756a1fb6e7054" -A 1
        "ja3_digest": "eb1d94daa7e0344597e756a1fb6e7054",
        "source_ip": "172.217.194.103",
--
        "ja3_digest": "eb1d94daa7e0344597e756a1fb6e7054",
        "source_ip": "104.16.248.249",
--
        "ja3_digest": "eb1d94daa7e0344597e756a1fb6e7054",
        "source_ip": "104.16.248.249",
--
        "ja3_digest": "eb1d94daa7e0344597e756a1fb6e7054",
        "source_ip": "74.125.200.95",
--
        "ja3_digest": "eb1d94daa7e0344597e756a1fb6e7054",
        "source_ip": "74.125.200.95",
--
        "ja3_digest": "eb1d94daa7e0344597e756a1fb6e7054",
        "source_ip": "74.125.200.95",
--
        "ja3_digest": "eb1d94daa7e0344597e756a1fb6e7054",
        "source_ip": "74.125.200.95",
```
Here, you can observe that this JA3S hash is "shared" among different services, namely between Cloudflare's DoH resolver (`104.16.248.249`) and one of the Google services (`74.125.200.95`)

## Naive suggestion for improvement
Fingerprint should also contain the source IP then, at least, different services would be easier to identify. Of course, multiple instances behind the same IP and having the same configuration and setup will still be differentiable, but we might not even need that for the original purpose of JA3(S) fingerprinting.
For `ServerHellos`, i.e., for the `ja3s.py`, I have made this change and stored in `ja3s_srcip.py`. 

Running this script, you can observe that none of the different services share the same JA3S fingerprint, however, this does not solve the case of the multiple-instances-behind-the-same-IP "issue". Cloudflare has still produces different hashes for the "same" service.


