# Bitmessage whitepaper note
## reference
> This note contents are almost from [whitepaper](https://bitmessage.org/bitmessage.pdf) of Bitmessage with some modifications as well as :exclamation: , :question: and :white_check_mark: for the use of discussing and recognizing how Bitmessage works.

## abstract
> - trustless  decentralized  peer‐to‐peer  protocol
- Users  need  not 
exchange  any  data  beyond  a  relatively  short  (around  **36  character**) 
[address](https://bitmessage.org/wiki/Address)  to  **ensure  security**  and  they  **need  not  have  any  concept  of 
public or private keys to use the system**
- mask non‐content data, like the sender  and receiver of messages, from 
those **not involved in the communication**

## introduction
> - **masks 
the sender and receiver** of messages from others
- guarantees that the **sender** of a message **cannot be 
spoofed** :white_check_mark: ```藉由數位簽章的幫助``` , without relying on trust and without burdening the user with the details of key management

## Authentication
> - users  **exchange  a  hash  of  a  [public  key](https://bitmessage.org/wiki/Public_key_to_bitmessage_address)  that  also  functions  as  the  user’s 
address**.:white_check_mark: ```public key應該含encrytion和signing的public part```  If  the  public  key  can  be  obtained  by  the  underlying  protocol,  then  it  can  easily  be  hashed  to 
verify that it belongs to the intended recipient
- The data exchanged by the user can also include **a version 
number for forwards capability, a [stream](https://bitmessage.org/wiki/Stream) number** , and a 
**checksum** :exclamation:
- Encoded with **[base58](https://zh.wikipedia.org/wiki/Base58) and prepended with recognizable characters (like BM for Bitmessage)** :exclamation:

## Message Transfer
> - a message transfer mechanism similar to Bitcoin’s transaction and block transfer system **but :exclamation: with  a  [proof‐of‐work](https://bitmessage.org/wiki/Proof_of_work)  for  each  message**
- in order to send a message through the 
network, a proof‐of‐work must be completed in the form of a **partial hash collision** :white_check_mark: ```兩個初始輸入hash後部分相同```
- The difficulty of the 
proof‐of‐work should be **proportional to the size of the message** :exclamation: and should be set such that an average 
computer must expend an average of **four minutes** of work in order to send a typical message. With the 
release  of  new  software,  the  difficulty  of  the  proof‐of‐work  can  be  **adjusted**
- Each  message  must  also 
**include the time** :exclamation:in order to **prevent the network from being flooded by a malicious user rebroadcasting 
old messages**. If the time in a message is too old, peers will not relay it
- Just like Bitcoin transactions and blocks, **all users would receive all messages**. They would be responsible 
for  attempting  to  **decode  each  message  with  each  of  their  private  keys  to  see  whether  the  message  is 
bound for them** :exclamation:

## Scalability
> - If all nodes receive all messages, it is natural to be concerned about the system’s scalability
-  after the number of messages being sent through the Bitmessage network reaches a 
certain **threshold**, nodes begin to self‐segregate into **large clusters or streams**.
- Users would **start out using 
only stream 1. The stream number is encoded into each address. Streams are arranged in a hierarchy**. :exclamation:
- Once it 
starts exceeding comfortable thresholds, **new addresses should be created in child streams** and the nodes 
creating those addresses should consider themselves to be members of that stream and behave as such. :question: ```建立新位址的nodes是在parent stream的nodes嗎?一個節點一個位址?根據wiki描述-A stream is a collection of nodes, connected together and having addresses associated with the specific stream number.```
- From  then  on,  **if  the  node  has  no  active  addresses  in  the  parent  stream**,  they  need  only  maintain 
connections  with  peers  which  are  also  members  of  this  child  stream :question: ```node可以在多個stream(parent & child)?可以有多位址?node和stream和address的關係?```
- With  the  exception  of  nodes  in 
stream 1, the root stream, nodes should occasionally connect to peers in their parent stream in order to 
**advertise their existence**. :exclamation:
- **Each node should maintain a list of peers in their stream and in the two child 
streams**. **Additionally,  nodes  should  each  maintain  a  short  list  of  peers  in  stream  1.** :exclamation:
- In  order  to  send  a 
message, a node must first connect to the stream encoded in the Bitmessage address :question: ```node連到stream是指node和stream中的nodes連接?stream和address的關係?```. If it is not aware of 
any peers in the destination stream, it connects to the closest parent stream for which it is aware of peers 
and then downloads lists of peers which are themselves in the two child streams :question: ```承上```. It can now connect to 
the child stream and continues this process until it arrives at the destination stream. :question: ```承上```
- After sending the 
message  and  listening  for  an  acknowledgement,  it  can  disconnect  from  the  peers  of  that  stream. If  the 
user  replies  to  the  message,  their  Bitmessage  client  repeats  the  same  process  to  connect  to  the  original 
sender’s  stream. After  this  process  has  been  carried  out  once,  connecting  to  the  destination  stream  a 
second time would be trivial **as the sending node would now already have a list of nodes that are in the 
destination stream saved** :question: ```知道對方的nodes的意思?```

## [Broadcasts](https://bitmessage.org/wiki/Broadcast) 
> - After 
entering  the  **broadcaster’s  Bitmessage  address  into  a  [‘Subscription’](https://bitmessage.org/wiki/Subscriptions)  section  of  their  Bitmessage  client**, 
messages from the broadcaster appear in the user’s inbox.
- anonymously publish content using an **authenticated identity** :question: ```匿名和真實性如何實作?``` to everyone who wishes to listen.

## Behavior when the receiver is offline
> - An  **object**  is  **a  public  key  request,  a  public  key,  a  person‐to‐person  message,  or  a  broadcast  message**. :exclamation: Objects are broadcast throughout a Bitmessage stream.
- **nodes store all objects** for **two days**  and  then  delete them.
- Nodes  joining  the  network  **request  a  list  of  objects  from  their  peer  and 
download the objects that they do not have**. :exclamation:
- If a  node  is  offline  for  more than  two  days,  the  sending  node  will 
notice that it **never received an acknowledgement and rebroadcasts the message after an additional two 
days**. It will continue to rebroadcast the message, with **[exponential backoff](https://en.wikipedia.org/wiki/Exponential_backoff)** :white_check_mark: ```若未收到，傳送天數會指數成長，待了解```, forever.
- In the worst case, if a 
user is offline for n days, he must go back online and stay connected for n days (or connect once every 
two days for n days) in order to receive all of his messages. :white_check_mark: ```同上，待了解```

## Passive operating mode :question: ```不明瞭```
> A particularly paranoid person who wishes to receive messages may operate in an entirely passive mode 
by  specifying,  **in  flags  attached  to  his  public  key,  that  he  will  not  send  acknowledgements**. :exclamation:
>
> It  would, 
perhaps,  be  wiser  for  him  to  instead  send  acknowledgements  but  to  recruit  another  possibly  random 
node  to  send  the  acknowledgement  for  him.  The  other  node  need  not  even  be  aware  that he  has  been 
recruited for this purpose. :question:
>
> :question:

## Spam
> The existing **proof‐of‐work requirement** may be sufficient to make spamming users uneconomic. If it is 
not then there are several courses of action that can be taken:
>
> - **Increase the difficulty** of the proof‐of‐work
- Have each client distribute x public keys for each public key that they actually use. **Acknowledge 
messages bound for those public keys but never show the user the messages** :white_check_mark: ```建立多個假帳號，其訊息皆被丟棄，僅一或少數為真實帳號```. Spammers would 
need x times as much computing power to spam the same number of users.
- include **extra bits in Bitmessage addresses** and require that those bits be included **in a message**, 
thus proving that the sender has the Bitmessage address. :white_check_mark: ```互相信任的彼此約定好的bits``` **Bots who crawl the web looking for Bitmessage 
addresses would thwart this option** :white_check_mark: ```可能在網頁上留下bits的相關資訊```

## Conclusion
> **Paired with the 
BitTorrent protocol, individuals could distribute content of any size** :exclamation:
