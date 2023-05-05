Download Link: https://assignmentchef.com/product/solved-csee-4119-program-2-reliable-transport-over-udp
<br>
In this assignment, you will build your own reliable transport protocol, over UDP. Your protocol must provide in-order, reliable delivery of UDP datagrams, and must do so in the presence of packet loss, delay, corruption, duplication, and re-ordering.

To reduce your workload, we will provide you with the receiver and you will implement the sender side. Since you probably cannot modify the OS on your machine, we have included a simulated channel in the receiver side that drops, delays, and corrupts packets, and effectively simulates an unreliable channel. We will also provide several useful functions that you can use in your implementation.

<h1>1. Protocol Description</h1>

Our simple protocol has four packet types: <em>start</em>, <em>end</em>, <em>data</em>, and <em>ack</em>. <strong>All packets</strong> follow the general format below.

<strong>Packet specification</strong>:




<table width="0">

 <tbody>

  <tr>

   <td width="134">checksum</td>

   <td width="130">seqnum</td>

   <td width="122">flag</td>

   <td width="115">optional</td>

   <td width="123">data</td>

  </tr>

 </tbody>

</table>




<ul>

 <li>checksum: (4 bytes): 32-bit checksum</li>

 <li>seqnum: (4 bytes): sequence number</li>

 <li>flag (1 byte): start packet: 0, data packet: 1, end packet: 2, special packet: 1, ack packet: 4</li>

 <li>optional (1 byte): optional field if needed for performance improvement</li>

 <li>data: at most (MAX_SIZE – 10) bytes</li>

</ul>




To initiate a connection, send a <em>start</em> packet. The receiver will use the sequence number provided in the start packet as the initial sequence number for all packets in that connection. After sending the start packet, send additional packets in the same connection using the data packet type, adjusting the sequence number appropriately. The last data in a connection should be transmitted with the end packet type. If the file to transmit only requires one packet, that packet is sent with a special packet flag.




A single packet will not exceed MAX_SIZE bytes. Because of the constraints on the total size of packets in UDP and layers below, MAX_SIZE cannot be more than ~1400 bytes. You should set MAX_SIZE to 500 bytes in this assignment.




<strong>Receiver Description: </strong>

<strong> </strong>

We will provide you the receiver Receiver.py for you. The receiver responds to data packets with <em>cumulative acknowledgements</em>. Upon receiving a packet of type start, data, special, or end, the receiver generates an ack packet with the sequence number it expects to receive next, which is the lowest sequence number not yet received in order.




Receiver has a maximum window size WND_SIZE which is set to 10 packets in this assignment. This means if the next expected packet is N, the receiver will not accept out-oforder packets with sequence number greater than N+ WND_SIZE.




The receiver has a default timeout of 5 seconds, this means it will automatically close any connections for which it does not receive packets for that duration.




To simulate an unreliable channel, we have used a function channel(packet), defined in utils.py, that receives the received packet, and two constants, packet drop probability PROB_LOSS and packet corruption probability PROB_CORR, and drops or corrupts the packet according to those probabilities. It further delays all the packets by some random time in DELAY_RANGE currently set to interval (80, 120) msecs. The output of channel function is either a packet (corrupted or uncorrupted) or <em>None</em> (in the case of packet drop). The output of this function has been considered as what is actually received by the receiver to emulate an unreliable channel.




The receiver does not send back any ack if the packet is dropped (output of channel is <em>None</em>), otherwise it sends an ack packet.




The receiver is invoked with the following command:




python Receiver.py &lt;output file&gt; &lt;receiver address&gt; &lt;receiver port&gt;




Output file is the name of the file where the receiver writes data, and receiver address could be simply localhost and receiver port is the UDP port number used by the receiver.




<strong>Sender specification </strong>

<strong> </strong>

Your sender should read an input file and transmit it to a specified receiver using UDP sockets. It should split the input file into appropriately sized chunks of data, specify an initial sequence number for the connection, and append a checksum to each packet.




Function for generating packet checksums will be provided for you (see utils.py), also functions for making packet, and extracting fields from a packet. In utils.py, we have also provided you with read_file function that divides the file into chunks of size chunk_size. For example you can use f = read_file(filename, chunk_size), and f.next() to return the next chunk.




Your sender must implement a reliable transport algorithm using a sliding window mechanism. Your sender must be able to accept ack packets from the receiver. Any ack packets with an invalid checksum should be ignored.




Your sender should provide reliable delivery of file under the following network conditions:




<ul>

 <li>Loss: arbitrary levels</li>

 <li>Corruption: arbitrary types and frequency.</li>

 <li>Re-ordering: packets may receive in any order, and</li>

 <li>Delay: packets may be delayed (here randomly in DELAY_RANGE).</li>

</ul>




Your sender should be invoked with the following command:




python Sender.py &lt;input file&gt; &lt;receiver address&gt; &lt;receiver port&gt;




<ul>

 <li>The sender should implement a 500ms timeout interval to automatically retransmit packets that were never acknowledged (potentially due to ack packets being lost). We do not expect you to use an adaptive timeout (though this is one of the bonus options). Further the sender should perform fast retransmission, i.e., upon the receipt of three duplicate acks, it should transmit the corresponding packet without waiting for timeout.</li>

 <li>Sender should only retransmit the oldest unacknowledged packet (rather than naively retransmitting the last N packets).</li>

 <li>Your sender should support a window size W of at most rwnd packets (we do not expect adaptive window size but this is one of the bonus options).</li>

 <li>Your sender should roughly meet or exceed the performance (in both time and number of packets required to complete the file transfer) of a properly implemented sender.</li>

 <li>Your sender should be able to handle arbitrary files (i.e., it should be able to send an image file just as easily as a text file).</li>

 <li>Any ack packets received with an invalid checksum should be ignored.</li>

 <li>Your sender MUST NOT produce console output during normal execution. Python exception messages are ok, but try to avoid having your program crash in the first place.</li>

 <li>We will evaluate your sender and receiver on correctness, time of completion for a file transfer, and number of packets sent (and re-sent). Transfer completion time and number of packets used in a transfer will be measured against our own reference implementation of a sliding-window based sender. Note that a “Stop-And-Go” sender (i.e., without pipelining) will not have adequate performance.</li>

</ul>




<ol start="2">

 <li>Bonus Points:</li>

</ol>

<em>Congestion Control (10 points): </em>When packet loss, corruption, delay, and reordering are minimal, a large window size will obtain higher performance. A large window on a lossy channel, however, will lead to a large amount of overhead due to retransmissions. Modify your sender to dynamically adjust its window size based on a variant of TCP congestion control mechanism.

<em> </em>

<em>Accounting for variable round-trip times (5 points)</em>: Modify your sender to determine the round trip time between the sender and the receiver, and adjust the timeout interval appropriately through a similar mechanism to TCP.




<em>Bidirectional transfer (5 points)</em>: While our protocol as defined only supports unidirectional data transfer, it could be modified to allow bi-directional transfer (i.e., both ends of the connection could send and receive simultaneously). Implement this functionality by modifying both the sender and receiver as necessary.




Be sure to provide a description of your updated protocol in your README.txt file.




<ol start="3">

 <li>Downloads</li>

</ol>

The utils.py and Receiver.py are on the course webpage.