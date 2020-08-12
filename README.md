# tack_paper_doc
This repo is to share some slides and videos for the SIGCOMM2020 paper “[TACK: Improving Wireless Transport Performance by Taming Acknowledgments](http://conferences.sigcomm.org/sigcomm/2020/)”.

## Abstract 

The shared nature of the wireless medium induces contention between data transport and backward signaling, such as acknowledgement. The current way of TCP acknowledgment induces control overhead which is counter-productive for TCP performance especially in wireless local area network (WLAN) scenarios. In this paper, we present a new acknowledgement called TACK ("Tame ACK"), as well as its TCP implementation TCP-TACK. TCP-TACK works on top of commodity WLAN, delivering high wireless transport goodput with minimal control overhead in the form of ACKs, without any hardware modification. To minimize ACK frequency, TACK abandons the legacy received-packet-driven ACK. Instead, it balances byte-counting ACK and periodic ACK so as to achieve a controlled ACK frequency. Evaluation results show that TCP-TACK achieves significant advantages over legacy TCP in WLAN scenarios due to less contention between data packets and ACKs. Specifically, TCP-TACK reduces over 90% of ACKs and also obtains an improvement of ∼28% on goodput. We further find it performs equally well as high-speed TCP variants in wide area network (WAN) scenarios, this is attributed to the advancements of the TACK-based protocol design in loss recovery, round-trip timing, and send rate control.

## FAQ

### Q: Why TCP-TACK works better than TCP BBR in the Pantheon?

A: This is because Stanford Pantheon sets the default sender/receiver buffer to 16MB in the end-hosts. TCP BBR is a kernel-space implementation whose performance is limited by the sender/receiver buffer in the case of high-bandwidth transport. This TCP-TACK version is implemented in the user space, we have already set a large enough buffer to assure it is not the bottleneck. Our other experiment results with fair comparison show TCP-TACK has very similar behavior as TCP BBR.
	
### Q:How well does this generalize to other environments (cellular, datacenters)?

A:TACK and its corresponding advancements can also be adopted in scenarios where the acknowledgement overhead is non-negligible. When the cellular adopts the Frequency Division Duplex, the contension between the data packets and ACKs is less than that in the WLAN scenario. In datacenters, reducing ACK frequency might reduce the CPU overhead on both endpoints and intermedium nodes.

### Q:How much does this benefit beyond what hardware aggregation offloads can offer?

A: Hardware aggregation offloads such as LRO and GRO is the effective way to reducing the acknowledgement overhead of stack, but it is not minimized. TACK aims to minimize the acknowledgement overhead without any dedicate hardware support.


