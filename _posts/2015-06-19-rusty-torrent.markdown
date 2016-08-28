---
layout: post
title:  "Rusty Torrent"
date:   2015-06-19 18:25:00
categories: recurse-center rust
comments_enabled: true
---

Since last week, [Ken Pratt][1] and I started writing a BitTorrent Client written in [Rust][2]. The idea in this post is to leave some comments about what I learned from it, hoping that maybe it becomes useful to others who would like to create their own BitTorrent Client.

## Experience gained by a failed attempt ##

I tried before to build on my own the BitTorrent Client before, on my own. I naively tried to built everything from scratch, and in a language I was not that confortable. Yes, I failed!

I believe that the experience gained by my failed attempt allowed us to go faster. There were also other factors which allowed us to build this project very quickly: for example, [Ken][1] decided not to build everything from scratch, but to use what is already there as much as possible. This was a very wise decision.

##The implementation ##

We spent 9 days for the current implementation. In this subsection I would like to comment about the milestones of each of the days and leave some comments about parts of the code.

### Day 1 - Tracker Communication working ###

We used a bencoding library which was available in Rust, created the Metainfo file structure and the Info Dictionary info structure. The most important piece this day was to be able to construct the correct Request to the file. It was achieved by the code below:

```Rust
pub fn run(metainfo: Metainfo) {
    let length_string = metainfo.info.length.to_string();
    let encoded_info_hash = percent_encode(&metainfo.info_hash, FORM_URLENCODED_ENCODE_SET);
    let params = vec![("left", length_string.as_ref()),
                      ("info_hash", encoded_info_hash.as_ref()),
                      ("downloaded", "0"),
                      ("uploaded", "0"),
                      ("event", "started"),
                      ("peer_id", "-TZ-0000-00000000000"),
                      ("port", "6881")];
    let url = format!("{}?{}", metainfo.announce, encode_query_params(&params));

    let mut client = Client::new();
    let mut res = client.get(&url).header(Connection::close()).send().unwrap();
}
```

**Why we went this approach?** Simply because the BitTorrent specification does not use the latest URLenconding schemes, so any library that makes requests with URLencoding ended up changing the strings we needed to send.

The end of the day code can be found [here][4].

### Day 2 - Parsing Tracker Response ###

The Tracker Response is a bencoded dictionary, it was pretty easy to extract the information we required. The only tricky part was how peers are encoded in the response. The implementation code:

```Rust
pub struct Peer {
    pub ip: Ipv4Addr,
    pub port: u16,
}

impl Peer {
    fn from_bytes(v: &[u8]) -> Peer {
        let ip = Ipv4Addr::new(v[0], v[1], v[2], v[3]);
        let port = (v[4] as u16) * 256 + (v[5] as u16);
        Peer{ ip: ip, port: port }
    }
}
```

The Peers are represented in 6 bytes. The first four bytes represents the digits of the Ip address, the last two bytes represent the port number. For example: bytes (60, 7E, 68, DB, 1A, E9) refer to address: 96.126.104.219:6889.

The end of the day code can be found [here][5].

### Day 4 - 1 File Downloading, on 1 Peer, In-Order download ###

We made the peer communication for Downloads and made it to download the famous Tom Flag in these 2 days. Some important decisions:

- We had the concept of a Download running. This was supposed to be the control of the Peer Connection. One of the reponsabilities was to handle the peer connections and also to store the file being downloaded.

- The peer connection contained all the information of how to handle the peer communication for Downloads. In particular it knew how to handle each message a Peer can send:

```Rust
enum Message {
    KeepAlive,
    Choke,
    Unchoke,
    Interested,
    NotInterested,
    Have(u32),
    Bitfield(Vec<u8>),
    Request(u32, u32, u32),
    Piece(u32, u32, Vec<u8>),
    Cancel,
    Port,
}

impl Message {
    fn new(id: &u8, body: &[u8]) -> Message {
        match *id {
            0 => Message::Choke,
            1 => Message::Unchoke,
            2 => Message::Interested,
            3 => Message::NotInterested,
            4 => Message::Have(bytes_to_u32(body)),
            5 => Message::Bitfield(body.to_owned()),
            6 => {
                let index = bytes_to_u32(&body[0..4]);
                let offset = bytes_to_u32(&body[4..8]);
                let length = bytes_to_u32(&body[8..12]);
                Message::Request(index, offset, length)
            },
            7 => {
                let index = bytes_to_u32(&body[0..4]);
                let offset = bytes_to_u32(&body[4..8]);
                let data = body[8..].to_owned();
                Message::Piece(index, offset, data)
            },
            8 => Message::Cancel,
            9 => Message::Port,
            _ => panic!("Bad message id: {}", id)
        }
    }
```

- When saying "1 File Downloading, on 1 Peer, In-Order download", I mean that we could only download from a single peer, torrents which contained only one file, and that we requested each of the pieces/blocks in order. For example: if each piece contained 2 blocks, we would have requested piece 1, block 1, piece 1 block 2, then piece 2, block 1, and so on.

The end of the day code can be found [here][6].

### 5th & 6th Day - Multiple peers / Requests Queuing ###

The cool parts begin here. We had to use two concurrent/parallel programming techniques here: mutexes and channels.

We used mutex (Exclusive Lock or Exclusive Access) for the interactions which the *Peer Connections* needed to do on the *Download* entity which were changing the status of the **Downloaded File**.

```Rust
fn connect(peer: &Peer, download_mutex: Arc<Mutex<Download>>) -> Result<PeerConnection, Error> {
        let stream = try!(TcpStream::connect((peer.ip, peer.port)));
        let num_pieces = {
            let download = download_mutex.lock().unwrap();
            download.metainfo.info.num_pieces
        };
```

We used channels for asynchronous calls from the *Download* to the *Peers Connections*. For example, in the case a Peer Connection finished downloading a Block, it should notify all the other Peer connections that the pieces was no longer required, and that in the case they happened to be downloading the same piece, they could cancel it. The code below:

```Rust
fn process_message_from_download(&mut self, message: PeerMessage) -> Result<(), Error> {
        match message {
            PeerMessage::CancelRequest(piece_index, block_index) => {
                match self.requests_in_progress.iter().position(|r| r.matches(piece_index, block_index)) {
                    Some(i) => {
                        let r = self.requests_in_progress.remove(i);
                        self.send_message(Message::Cancel(r.piece_index, r.offset, r.block_length))
                    },
                    None => Ok(())
                }
            }
        }
    }
```

We also do not request one block at a time to other peers, we request ten at a time (defined as MAX_CONCURRENT_REQUESTS):

```rust
fn request_more_blocks(&mut self) -> Result<(), Error> {
    if self.am_i_choked == true {
        return Ok(())
    }
    while self.requests_in_progress.len() < MAX_CONCURRENT_REQUESTS as usize {
        let next_block_to_request = {
            let download = self.download_mutex.lock().unwrap();
            download.next_block_to_request(&self.have)
        };
        match next_block_to_request {
            Some((piece_index, block_index, block_length)) => {
                let offset = block_index * BLOCK_SIZE;
                try!(self.send_message(Message::Request(piece_index, offset, block_length)));
                self.requests_in_progress.push(RequestMetadata {
                    piece_index: piece_index,
                    block_index: block_index,
                    offset: offset,
                    block_length: block_length,
                });
            },
            None => {
                println!("We've downloaded all the pieces we can from this peer.");
                return Ok(())
            }
        }
    }
    Ok(())
}
```

The end of the 6th day code can be found [here][9]

### 7th & 8th Day - Uploading/Seeding working! ###

Unfortunately I could not work with Ken on the 8th Day, but he made possible Seeding/Uploading on his own.

It was added support for sending *BitField* and *Have* messages and to *Upload* the requested blocks by other peers.

An interesting Entity which was created during this day was the **Funnel** inside *Peer Connection*. The idea of the funnel is that there was an entity which was in charge of collecting all the incomming messages from the TCP socket, and also for the outgoing messages on the Channels.

```Rust
let downstream_funnel_thread = {
    let stream = self.stream.try_clone().unwrap();
    let tx = self.incoming_tx.clone();
    thread::spawn(move || DownstreamMessageFunnel::start(stream, tx))
};

// spawn a thread to funnel outgoing messages from the outgoing message channel into the socket
let upstream_funnel_thread = {
    let stream = self.stream.try_clone().unwrap();
    let tx = self.incoming_tx.clone();
    thread::spawn(move || UpstreamMessageFunnel::start(stream, outgoing_rx, tx))
};
```

The end of the 8th day code can be found [here][10]

## Advices to build your own - What I learned from this project ##

The first and most impportant, **do not try to re-invent the wheel everytime**. For example, if you find a ready to use solution for the *bencoding*, use it! If you find wrappers for the TCP connections, use them! It is better the faster you can get feedback about what you are building than to struggle and abandon a project. This does not mean that if you afterwards would like to change one of the modules for one you built form scratch it will be imposible.

The second advice: **work in pairs**: this keeps up the motivation and also I believe it allows a better solution. Many of the discussions we had, lead to the solution which was implemented.

Last advice, but not least, before trying to write a BitTorrent client, read the resouces below:

- [Unnoficial BitTorrent Specification][8]
- [Kirsten blog entries][7]

These resources are really important, and contain a lot of information which is required.

## Conclusions ##

I learned a lot from this project. Ken acted as a mentor for me in this project, he always explained the decisions which were at stake, and he always had patience with my lack of experience with Rust.

This project allowed me us work with Sockets, and Concurrent and Parallel programming.

The latest version of the project code can be found [here][3].

[1]: https://twitter.com/ken_pratt
[2]: http://www.rust-lang.org/
[3]: https://github.com/kenpratt/rusty_torrent
[4]: https://github.com/kenpratt/rusty_torrent/tree/972a8b17699717513a9346805d6829bfd03e693f
[5]: https://github.com/kenpratt/rusty_torrent/tree/fe83707d06d81fe0f37a2b583ad988e9a0acf088
[6]: https://github.com/kenpratt/rusty_torrent/tree/d1f7b43a1e88d1b2e570dc484105b62ef5240662
[7]: http://www.kristenwidman.com/blog/33/how-to-write-a-bittorrent-client-part-1/
[8]: https://wiki.theory.org/BitTorrentSpecification
[9]: https://github.com/kenpratt/rusty_torrent/tree/b19f96e42aecd6b1929757df59fa7f8b8ee6dfd8
[10]: https://github.com/kenpratt/rusty_torrent/tree/5bdf2ddee92906eade0787e5b624a79a3c0e08ee