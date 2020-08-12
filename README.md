# tack_paper_doc
This repo is to share some slides and videos for the SIGCOMM2020 paper “[TACK: Improving Wireless Transport Performance by Taming Acknowledgments](http://conferences.sigcomm.org/sigcomm/2020/)”.

## Abstract 

The shared nature of the wireless medium induces contention between data transport and backward signaling, such as acknowledgement. The current way of TCP acknowledgment induces control overhead which is counter-productive for TCP performance especially in wireless local area network (WLAN) scenarios. In this paper, we present a new acknowledgement called TACK ("Tame ACK"), as well as its TCP implementation TCP-TACK. TCP-TACK works on top of commodity WLAN, delivering high wireless transport goodput with minimal control overhead in the form of ACKs, without any hardware modification. To minimize ACK frequency, TACK abandons the legacy received-packet-driven ACK. Instead, it balances byte-counting ACK and periodic ACK so as to achieve a controlled ACK frequency. Evaluation results show that TCP-TACK achieves significant advantages over legacy TCP in WLAN scenarios due to less contention between data packets and ACKs. Specifically, TCP-TACK reduces over 90% of ACKs and also obtains an improvement of ∼28% on goodput. We further find it performs equally well as high-speed TCP variants in wide area network (WAN) scenarios, this is attributed to the advancements of the TACK-based protocol design in loss recovery, round-trip timing, and send rate control.

## FAQ

### Q: Wifi is only used in the last mile. I wonder if something like a split TCP will be a much better approach than changing the entire ack semantic to account for WiFi idiosyncrasies. This is especially because changing the ack semantics is extremely complex.

A: We agree that TCP splitting is a way to reduce the complexity of the TACK-based solution, this is based on the fact that the last-mile WiFi network usually has a smaller delay and converges fast. However, TCP splitting uses a proxy access node that divides the end-to-end TCP connection, which needs further modification on the access point (router). Another well-known problem of TCP splitting is that the split TCP connection is no longer reliable or secure, and a server failure may cause the client to believe that data has been successfully received when it has not. The cost performance of TACK with/without TCP splitting is worth being further studied. 


### Q: Adding TACK to BBR requires a lot more changes. Because BBR is tightly coupled with acks, every state of BBR requires changes to accommodate TACKs. This is rather heavy handed and I am not sure if it is worth the trouble.

A: The TACK-based protocol is a dual-side modification solution. It requires the receiver to cooperate with the sender on the congestion control. We argue that BBR requires much less changes than other congestion controllers. Take CUBIC and BBR as an example, we list the reasons as follows.

(1) The TACK-based protocol requires applying pacing at the sender. BBR is a rate-based congestion controller and it adopts pacing by default. However, we have to add an extra pacing logic and convert the window size to the pacing rate if we adopt CUBIC.

(2) Fine-grained monitoring does not equal to fine-grained control. BBR’s RTT and bandwidth estimations are all coupled with frequent ACKs, which is called fine-grained monitoring. However, this can be implemented with small amount of work by moving the monitoring logic from sender to receiver. This receiver-based paradigm also fits the TACK well. On the other hand, BBR does not rely on fine-grained control. For example, one round of pacing rate control can be as large as 8 RTTs. In this case, BBR can works well with TACK, and the modification efforts are bounded.

### Q: Is there interaction between congestion controllers and your proposed ideas? Have you missed something by making this choice?

A: TACK impacts the implementation of congestion controllers. For example, to work with TACK, it is required to change the sender-based control to receiver-based control. In our paper, we have given an example for the design and implementation of a TACK-based congestion controller co-designing BBR in a receiver-based way. Our evaluation result in Figure 27 shows that it has a similar behavior with the standard BBR. In other words, as an ACK mechanism, our current experience shows that TACK does not impact much on the performance of congestion controllers. However, we believe more substantial investigations are needed in the future, to answer the question of how TACK work with more congestion controllers such as CUBIC, Reno and Vegas. We leave this as an open issue to the readers. 

### Q: How would one determine appropriate values of L and $\beta$?

A: TACK aims to minimize the number of ACKs but assures enough number of ACKs for feedback robustness. In practice, it is suggested L=2 and $\beta$=4. Our real product deployment under both WAN and WLAN scenarios serves as a strong validation of its practicability. If not for special needs, it is not necessary to change the values of L and $\beta$.

### Q: Why TCP-TACK works better than TCP BBR in the Pantheon in WAN scenarios?

A: This is because Stanford Pantheon sets the default send/receive buffer to 16MB in the end-hosts. TCP BBR is a kernel-space implementation whose performance is limited by the send/receive buffer in long-fat network with high loss rate (e.g., 2%). This TCP-TACK version is implemented in the user space, we have already set a large enough buffer to assure it is not the bottleneck. Our other experiment results with fair comparison (i.e., enough send/receive buffer) show TCP-TACK has very similar behavior as TCP BBR.
	
### Q:How well does this generalize to other environments (cellular, datacenters)?

A:TACK and its corresponding advancements can also be adopted in scenarios where the acknowledgement overhead is non-negligible. When the cellular adopts the Frequency Division Duplex, the contension between the data packets and ACKs is less than that in the WLAN scenario. However, the uplink in cellular is usually more constrained and ACKs mostly go that path and thus reduction in ACKs could improve cellular transport performance as well. In datacenters, reducing ACK frequency might reduce the CPU overhead on both endpoints and intermedium nodes.

### Q:How much does this benefit beyond what hardware aggregation offloads can offer?

A: Hardware aggregation offloads such as LRO and GRO is the effective way to reducing the acknowledgement overhead of stack, but it is not minimized. TACK aims to minimize the acknowledgement overhead without any dedicate hardware support. WLAN also adopts the hardware aggregations such as AMPDU and AMSDU. This technologies have issues because they are very loosely coupled from transport and thus may induce delays if aggressive aggregation is sought. Our solution combines in the transport layer and thus have more contextual information to make the right decisions about ACK frequency.


