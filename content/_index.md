---
title: "Introduction"
---


# Threat Hunting Labs Introduction

These are a series of labs that cover different types of analysis that can be done on network data when threat hunting. You can do these in any order and you can jump around individual labs to try out the tools or methods that interest you. That being said, here is our suggested order:

1. [Long Connections]({{<relref "long_connections">}})
2. [Beacons]({{<relref "beacons">}})
3. [DNS]({{<relref "dns">}})
4. [Outliers]({{<relref "outliers">}})

You can use the [Basic Tool Usage]({{<relref "basic_usage">}}) guide as a reference for common tasks if a tool is unfamiliar to you.

Each of these labs works off the same packet capture. You have several options for downloading this pcap. Each option below has its snaplen set to a different value in order to reduce the file size. However, this means that large packets will have their payload truncated and your results when going through the labs may vary slightly from what is printed. The overall analysis should still remain unchanged.

Download | MD5 checksum
---------|--------------
{{% button href="https://github.com/activecm/threat-hunting-labs/releases/download/v1.0/sample-1500.pcap" icon="fas fa-download" %}}sample-1500.pcap (1.6 GB){{% /button %}} | `7c1b3b4bd50ea353e96e492e3e359e08`
{{% button href="https://github.com/activecm/threat-hunting-labs/releases/download/v1.0/sample-500.pcap" icon="fas fa-download" %}}sample-500.pcap (832 MB){{% /button %}} | `492f011aa4f6547ca2b52f1e7b2269a0`
{{% button href="https://github.com/activecm/threat-hunting-labs/releases/download/v1.0/sample-200.pcap" icon="fas fa-download" %}}sample-200.pcap (523 MB){{% /button %}} | `6a2a522169a3dc0148c55b987c7f5e66`

{{% notice note %}}
Please note that processing large pcaps in many of these labs will take a couple of minutes to finish.
{{% /notice %}}

----

If you enjoy these labs and are interested in learning more about network threat hunting:

1. Check out our articles on the [Active Countermeasures Blog](https://www.activecountermeasures.com/blog/) (or [Subscribe](https://www.activecountermeasures.com/subscribe/) for email updates)
2. Sign up for our free [Network Threat Hunter Training](https://www.activecountermeasures.com/thunt/) which consists of a series of recorded 1-hour videos

Thank you for participating!

