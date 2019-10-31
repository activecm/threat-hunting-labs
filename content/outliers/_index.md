---
title: "Outliers"
weight: 40
---

## Goal

Identify systems with suspiciously high or low metrics in different areas. These outliers can then be used to dig into further with other types of analysis.

## Tools Used

* Zeek

## Hunt

All of the scenarios here use Zeek logs. So be sure to [analyze your pcap using Zeek]({{<relref "basic_usage#process-a-pcap">}}) before starting.

### Number of Connections

More advanced techniques look at how long connections are held open or how regular connections are being made between two IP addresses. Sometimes it is useful to simply see how many total connections were made. A high number of connections made indicates that systems were communicating quite a bit and this type of analysis can be used to determine where to dig further.

```bash
cat conn.log | zeek-cut id.orig_h id.resp_h id.resp_p proto | awk 'BEGIN{ FS="\t" } { arr[$1 FS $2 FS $3 FS $4] += 1 } END{ for (key in arr) printf "%s%s%s\n", key, FS, arr[key] }' | sort -nrk 5 | head
```

__Output__
```
192.168.88.2	165.227.88.15	53	udp	108856
10.55.200.10	172.16.200.11	53	udp	64285
10.55.100.111	165.227.216.194	443	tcp	20054
10.55.182.100	10.233.233.5	80	tcp	4190
10.55.200.10	216.239.34.10	53	udp	3856
10.55.200.11	193.108.88.128	53	udp	3660
10.55.200.11	88.221.81.192	53	udp	2742
10.55.200.11	205.251.195.166	53	udp	2289
10.55.200.11	216.239.34.10	53	udp	2265
10.55.200.10	205.251.195.166	53	udp	1931
```

* `cat conn.log | zeek-cut id.orig_h id.resp_h id.resp_p proto` - We're taking Zeek's `conn.log` and only keeping the source IP, destination IP, destination port, and destination protocol.
* `awk` - The following explains the pieces of the `awk` script.
  * `BEGIN{ FS="\t" }` - Set the `FS` (field separator) variable to a tab character. This is what is separating columns in our Zeek logs as well as what we want to use in our output. `BEGIN` means this instruction is only executed one time, before any data is processed.
  * `{ arr[$1 FS $2 FS $3 FS $4] += 1 }` - Creates an array (named `arr`) with the number of connections. The important part here is that we are using the concatenation of the first four fields (`$1` through `$4`) as our array key. Which means that as long as the source and destination IPs, destination port, and protocol remain the same it counts the connections under the same key. `awk` executes this instruction repeatedly for every line of data.
  * `END{ for (key in arr) printf "%s%s%s\n", key, FS, arr[key] }` - Here we are looping through all the elements in the array and printing out the results. `END` signifies that `awk` only executes this instruction one time, after processing all the data.
* `sort -nrk 5 | head` - The number of connections is the 5th and final column printed in the output. Here we are sorting on this column in descending order and keeping the top results.

From the results we can see that `192.168.88.2` was communicating with `165.227.88.15` a large number of times. The dataset used is taken over a period of 24 hours so we can see that 108,856 connections divided by 86,400 seconds in a day means over 1 connection was sent every second on average. Furthermore, we know that these connections were made over UDP port 53, which is normally DNS. Typically, DNS results are cached in a number of places including:

* operating system's local cache
* local or remote DNS recursive resolver

It is suspicious that DNS requests were being made so frequently. This could be a result of a misconfiguration or misbehaving software. Or it could indicate malicious software. Assuming we are familiar with the network configuration, we should be able to quickly tell that `165.227.88.15` is not an IP of a DNS server we recognize, which makes this traffic even more suspicious.

### Total Data Transferred

Large data transfers out of a network can indicate data exfiltration. Databases containing sensitive information, intellectual property in the form of images, videos, PDFs, or other binary formats are often targets for attackers. Each of these can consist of large amounts of information that can then be detected when transferred out of your network.

The following command will display the total number of bytes sent from the IP address in column 1 to the IP address in column 2. When the IP address in column 1 is an internal address and column 2 an external address, it means that this data was exfiltrated out of your network.

```bash
cat conn.log | zeek-cut id.orig_h id.resp_h orig_bytes | awk 'BEGIN{ FS="\t" } { arr[$1 FS $2] += $3 } END{ for (key in arr) printf "%s%s%s\n", key, FS, arr[key] }' | sort -nrk 3 | head
```

__Output__
```
192.168.88.2	165.227.88.15		6723739
10.55.100.111	23.38.115.36		981527
10.55.100.111	34.233.92.30		958540
10.55.100.111	24.220.113.56		778452
10.55.100.111	24.220.113.58		775648
10.55.100.111	23.52.163.40		734881
10.55.100.100	23.38.115.36		705408
10.55.100.103	134.170.58.189		637453
10.55.100.111	23.63.220.157		618329
10.55.100.105	23.38.115.36		615374
```

* `cat conn.log | zeek-cut id.orig_h id.resp_h orig_bytes` - We're taking Zeek's `conn.log` and only keeping the source IP, destination IP, bytes sent by the source IP.
* `awk` - The following explains the pieces of the `awk` script.
  * `BEGIN{ FS="\t" }` - Set the `FS` (field separator) variable to a tab character. This is what is separating columns in our Zeek logs as well as what we want to use in our output. `BEGIN` means this instruction is only executed one time, before any data is processed.
  * `{ arr[$1 FS $2] += $3 }` - Creates an array (named `arr`). The important part here is that we are using the concatenation of the source and destination IPs as our array key. Which means that this command will add up the bytes for each pair of IPs. `awk` executes this instruction repeatedly for every line of data.
  * `END{ for (key in arr) printf "%s%s%s\n", key, FS, arr[key] }` - Here we are looping through all the elements in the array and printing out the results. `END` signifies that `awk` only executes this instruction one time, after processing all the data.
* `sort -nrk 3 | head` - The number of connections is the 3rd and final column printed in the output. Here we are sorting on this column in descending order and keeping the top results.

You can also modify the command to get the total amount of data sent in both directions.

```bash
cat conn.log | zeek-cut id.orig_h id.resp_h orig_bytes resp_bytes | awk 'BEGIN{ FS="\t" } { arr[$1 FS $2] += $3+$4 } END{ for (key in arr) printf "%s%s%s\n", key, FS, arr[key] }' | sort -nrk 3 | head
```

__Output__
```
10.55.100.111	162.252.74.5	2027554933
10.55.100.103	13.107.4.50		91904287
10.55.100.111	24.220.113.59	29008351
10.55.100.110	72.21.81.240	23388664
10.55.100.106	40.77.228.30	21117996
10.55.100.111	23.38.115.36	19471375
10.55.100.107	13.107.4.50		17989779
10.55.100.110	23.38.115.36	17960372
10.55.100.106	13.107.4.50		17653174
10.55.100.100	13.107.4.50		17528299
```

### Uncommon User Agent Strings

Attacker tools which send HTTP traffic will often include a User Agent header (UA) in the HTTP request. Many tools (e.g. nikto) will have a custom UA that identifies the tool. If an attacker forgets to set a custom value it should be a red flag to have such UA appear on your network. Even tools which have their default value set to a common web browser will sometimes make typos, such as including an extra space, that likewise will appear as anomalies. One final reason to look at UA's is that if your network has consistent patching then you should have relatively few unique UA strings as each system will have identical browsers and versions. A unique value in this case could indicate: a system with missing patches, a user installing unauthorized software, or an attacker attempting to blend in by choosing a common UA but it doesn't match what's on your network.

This command uses Zeek's `http.log` file to find all UA strings, and count how many of each appear in connections.

```bash
cat http.log | zeek-cut user_agent | sort | uniq -c | sort -n | head
```

__Output__
```
      1 client connection
      1 Windows-Update-Agent/7.9.9600.18756 Client-Protocol/1.21
      2 Microsoft-CryptoAPI/6.3
      9 Windows-Update-Agent/10.0.10011.16384 Client-Protocol/1.40
     12 OfficeClickToRun
     25 Microsoft BITS/7.8
     40 MICROSOFT_DEVICE_METADATA_RETRIEVAL_CLIENT
     44 Mozilla/5.0 (Windows NT 10.0; Win64; x64; Trident/7.0; rv:11.0) like Gecko
     48 Mozilla/4.0 (compatible; FCT 5.6.0; Windows NT 5.1)
    374 Microsoft-Delivery-Optimization/10.0
```

Next, we can investigate the unique UA strings to find out who was making the request and where the request was going. In both of the below cases, we are pulling out the source and destination IPs from the Zeek log, along with the HTTP `Host` header and the requested `URI`. Finally, we include the UA string so that we can filter out only the UAs we found above.

```bash
cat http.log | zeek-cut id.orig_h id.resp_h host uri user_agent | grep 'client connection'
```

__Output__
```
10.55.200.10	191.239.52.100	tele.trafficmanager.net	/{AA35F099-DF2E-4104-8F15-FCB887FB32F3}	client connection
```

This appears to be a tracking server of some kind. Some quick reconnaissance on the domain shows that it belongs to Microsoft. We can tentatively assume that this is benign metrics tracking.

```bash
cat http.log | zeek-cut id.orig_h id.resp_h host uri user_agent | grep 'Windows-Update-Agent/7.9.9600.18756 Client-Protocol/1.21'
```

__Output__
```
10.55.200.10	52.183.118.171	statsfe2.update.microsoft.com	/ReportingWebService/ReportingWebService.asmx	Windows-Update-Agent/7.9.9600.18756 Client-Protocol/1.21
```

In this case, the domain is also pretty clearly a Microsoft property, and we can verify that Microsoft also owns the destination IP. This appears to be part of the normal update process, but since it is a unique UA string the next step might be to investigate the system `10.55.200.10` to see why it is not running the same version of software as all the other systems.

Note that the `Host` header can be spoofed in HTTP requests so you should not rely on this alone. You should verify in the DNS logs that the domain listed there actually resolved to the IP address in the HTTP log. Furthermore, the destination IP address may change and no longer be associated with that domain in this age of cloud computing. It is quite common for services to rotate or recycle IP addresses for a given domain name.

