---
layout: post
title:  "Build Your Own - Writing a Bittorrent Client"
tags: Bittorrent
---

While exploring Github, I bumped into an interesting repository which is called 
[build-your-own-x](https://github.com/danistefanovic/build-your-own-x). I had free time on quarantine and 
decided to build my own *something*. Specifically, 
[a bittorrent client](https://github.com/danistefanovic/build-your-own-x#build-your-own-bittorrent-client) to 
gain a better understanding of how it works.

**Warning!** This blog post is not a tutorial and is my experience only. If you're looking for a guide, please, refer to
"[Building a BitTorrent client from the ground up in Go](https://blog.jse.li/posts/torrent/)" or 
"[Building a BitTorrent client from scratch in C#](https://www.seanjoflynn.com/research/bittorrent.html)". 
The first one is straight to the point, the second one is, in opposite, more verbose.

_Simplified description of the process_. First of all we request peer list (ip + port) from a bt tracker, 
what is known as announcing. In our case, this is a simple HTTP GET request. 
After our bt client announced itself on the tracker and got peers, 
we can start connecting to peers to figure out what they have. If they have what we need, we request data.

# First Attempt

To keep it simple, I decided to implement very basic functionality which includes **only downloading**.
In addition to that, the app won't support requesting peers via UDP from the torrent tracker. 
The app will be a command-line tool, which accepts only one torrent file. 
The torrent file will consist of only one file in it, which is the Debian network installer.
I suppose, this is the best candidate for a file to download ([link](https://cdimage.debian.org/debian-cd/current/amd64/bt-cd/#indexlist)).

*Detailed description of the process*. The application makes use of ArrayBlockingQueues to implement approach 
"Do not communicate by sharing memory; instead, share memory by communicating", but still somewhere
I had to use concurrently shared memory. Bottom-up, the app has a queue for done pieces. A thread polls that 
queue and writes hash-verified pieces to disk via RandomAccessFile. Several threads are use for 
sending/receiving messages to/from peers. A peer can send a message to request a block of a piece and receive that block(s), 
which together make up the piece. In addition to that, we have a queue for absent torrent pieces, and
a queue for connected (handshake done) peers. A few threads polls these two queues, trying to find a peer 
which has the piece in question. If a match found, the peers starts downloading the piece. When done,
the peer sends the pieces to 'absent pieces' or 'done pieces' queue, depending on hash verification result.
See the code on [Github](https://github.com/eugenesimakin/btclient.kt).

*Issue connecting with peers*. When the work on the project was finished, I needed to run and debug it.
I had no issue announcing my bt client on the torrent tracker, but couldn't connect to any peer I got.
Mostly, the connections didn't establish due to timeouts errors. At first, I thought that this was a NAT issue.
Explanation behind it is pretty simple if you'll account that almost every peer is inside home network with a router.
But, the app written in Go works fine, so I took the peers which receives that app in Go and use them in my app.
Not ideally, but it worked.

When I got some good peers to connect to, I fixed a few minor bugs, and the app works fine.
Still, I couldn't get all 100% of the file. Sometimes, it was 99.43%, sometimes - 99.14%. So, I've decided 
to stop working on it for now, because already spent a lot of time on it.

# Next Steps

The first next step is to get the last percent of the file. This might seem easy, but it's not because
it requires a lot of work. Secondly, the CPU usage is pretty high but unreasonable. Need to get rid of this. 
The third step might be to get seeding work.

# Notes

1) The **ByteBuffers** are quite okay. Their API receives some criticism, what is well-earned. I've got
a few bugs on them. Here are two question from [stackoverflow.com](https://stackoverflow.com/), I'd like to save -
[first](https://stackoverflow.com/questions/23148729/what-is-the-difference-between-limit-and-capacity-in-bytebuffer) and 
[second](https://stackoverflow.com/questions/20982240/is-calling-buffer-flip-twice-in-a-row-problematic).

2) For each peer we have to have a bit map of pieces it has. The **java.util.BitSet** is most suitable for this
purpose. The BitSet has the *[valueOf](http://hg.openjdk.java.net/jdk7/jdk7/jdk/file/00cd9dc3c2b5/src/share/classes/java/util/BitSet.java#l260)* 
method for converting a byte array to a BitSet. If you'll look up its 
implementation, you'll find that internally it sets the buffer to little-endian and uses an array of longs. 
The byte order, which bt protocol utilizes, is big-endian. So, it got me thinking why it does that.
It appears that it does the right thing because in little-endian order it preserves the original byte order 
while converting them to long array.

