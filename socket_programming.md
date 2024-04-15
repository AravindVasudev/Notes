# Socket Programming

Book: https://beej.us/guide/bgnet/

- Sockets are UNIX filers.
- Two types: SOCK_STREAM (TCP/IP), SOCK_DGRAM (UDP).
- Layered Network Model (ISO/OSI):
    - Application
    - Presentation
    - Session
    - Transport
    - Network
    - Data Link
    - Physical
- IPv4 -> 32 bits, ~4 billion addresses.
- IPv6 -> 128 bits, ~79 million billion trillion addresses.
- loopback address: 127.0.0.1 (IPv4), ::1 (IPv6).

## Subnets
- Divides IP into network portion & host portion.
- In old times, there used to be subnet classes (class A, B, C resp).
- Class A being 1 byte, B 2 bytes, C 3 bytes.
- Now, we can fine-tune subnet masks at the bit level.
- The number of mask bits is represented in decimal with a `/`.
    - Eg: 192.0.2.12/30, 2001:db8::/32.

## Port Numbers
- 16 bit numbers used by both TCP & UDP.
- Ports under 1024 are special reserved and OS privileges to access.

## Byte Ordering
- Big Endian: Store big bytes first (i.e., store it in order). Vice-versa is called little endian.
- Intel processors uses little endian.
- Computers store numbers in "host byte order" and this varies based on the processor manufacturer.
- Network Byte Order: Basically Big Endian. Networks ❤️  Big Endian.
- To make the code portable, always assume the host byte order is wrong and convert before transporting.
- Conversion functions:
    - `htons` -> host to network short.
    - `htonl` -> host to network long.
    - `ntohs` -> network to host short.
    - `ntohl` -> network to host long.

## Structs
- `addrinfo` -> Used to prep socket address for subsequent uses.
- `sockaddr` -> holds socket address information for many types of sockets. Have to packed by hand (there are helpers!).
- `sockaddr_in` -> `sockaddr` equivalent to be used with IPv4.
- `sockaddr_in6` -> `sockaddr` for IPv6.

## IP Addresses
- `inet_pton` -> Converts an IP address in numbers-and-dots notation into either a struct in_addr or a struct in6_addr depending on whether you specify AF_INET or AF_INET6.
    - pton stands for “presentation to network”.
- `inet_ntop` -> network to presentation. Converts in_addr to ip address.

## Private Networks
- NAT -> Network Address Translation. Converts private IP to public IP.

## Jumping from IPv4 to IPv6
- tl;dr IPv6 uses a different set of structs and functions. Refer https://beej.us/guide/bgnet/html/split-wide/jumping-from-ipv4-to-ipv6.html.

## getaddrinfo()
- System call that does DNS/Service name lookup and fills out the struct you need!

## socket()

```
int socket(int domain, int type, int protocol);
```
- AF in `AF_INET` stands for "Address Family".
    - Based in the days, there was a idea to have address families that supported multiple "protocol families" (PF in `PF_INET`).
    - That didn't happen. So, eventhough they are interchangable, use `AF_INET` in `sockaddr_in` and `PF_INET` in `socket()`.

## bind()

```
int bind(int sockfd, struct sockaddr *my_addr, int addrlen);
```

- Binds the given socket to a port. Used to listen() to incoming connections.

## connect()

```
int connect(int sockfd, struct sockaddr *serv_addr, int addrlen);
```

- Connect to a given socket.
- Bsically, if you wanna create a client, do `getaddrinfo()` for DNS and packing stuff, followed by `socket()` to create the socket fd, and `connnect()` to it.

## listen()

```
int listen(int sockfd, int backlog);
```

- backlog is the number of connections allowed on the incoming queue.
- We need to `bind()` to a port before we listen.

## accept()

```
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

- accept a queued request.
- returns a brand new file descriptor for that connection. We can `send()` and `recv()` via this socket fd.

## send() and recv()

```
// TCP calls
int send(int sockfd, const void *msg, int len, int flags);
int recv(int sockfd, void *buf, int len, int flags);

// UDP calls
int sendto(int sockfd, const void *msg, int len, unsigned int flags,
           const struct sockaddr *to, socklen_t tolen);
int recvfrom(int sockfd, void *buf, int len, unsigned int flags,
             struct sockaddr *from, int *fromlen);
```

## close() and shutdown()

```
close(sockfd); // close the sokcet fd.
int shutdown(int sockfd, int how);  // close() with more control.
```
- how: `0` Further receives are disallowed, `1` Further sends are disallowed, `2` Further sends and receives are disallowed (like close()).
- shutdown() doesn’t actually close the file descriptor—it just changes its usability. To free a socket descriptor, you need to use close().

## getpeername()

```
int getpeername(int sockfd, struct sockaddr *addr, int *addrlen);
```

- will tell you who is at the other end of a connected stream socket.
