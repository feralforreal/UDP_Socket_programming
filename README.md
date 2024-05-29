# UDP_Socket_programming
UDP (Use Datagram Protocol) is a connectionless transmission model with a minimum of protocol mechanism. It has no handshaking dialogues and no mechanism of acknowledgements (ACKs). Therefore, every packet delivered by using UDP protocol is not guaranteed to be received, but UDP avoids unnecessary delay, thanks to its simplicity.

Approach 
The client:
1. The client takes two command line arguments: an IP address of the machine on which the server application is running, and the port the server application is using. (The IP address can be obtained using hostname -i) For example:
$ gcc uftp_client.c -o client
$ ./client 192.168.1.101 5001
The server:
1. The server takes one command line argument: a port number for the server to use. example:
$ gcc uftp_server.c -o server # Compile your c-program
$ ./server 5001 # Running your server with port number 5001
And then it waits for a UDP connection after binding to port 5001
And accordingly client could send get, put, delete, ls, exit [file_name]”

A brief explanation of some of the functions used in the code is provided here. However, for in depth understanding of the functions, please read the manpages of the functions.
o socket() : The input parameters of the function lets you determine which type of socket you want in your application. It may be a TCP or UDP socket. The function returns a socket descriptor which can prove helpful later system calls or -1 on error. A quick look at the function:
o sockfd = socket(PF_INET,SOCK_DGRAM,0); o sockfd is a UDP socket.
 o bind() : Once we have our socket ready, we have to associate it with a port number on the local machine. It will return -1 on error. When calling bind function, it should be noted that ports below 1024 are reserved. Any port number above 1024 and less than 65535 can be used. A quick reference:
o bind(sockfd, (struct sockaddr *)&my_addr, sizeof my_addr);
o sendto(): This function is used to send the data. Since it is used in connectionless datagrams, the input parameter of this function includes the destination address. It returns the number of bytes sent on success and -1 on error.
 o ssize_t sendto( int sockfd, void *buff, size_t nbytes, int flags, const struct sockaddr* to, socklen_t addrlen);
o “buff” is the address of the data (nbytes long).
o “to” is the address of a sockaddr containing the destination address.
o recvfrom() : This function is used to receive the data from an unconnected datagram socket. The input paramters contain the address of the originating machine. It returns the number of bytes received on success and -1 on error
 o ssize_t recvfrom( int sockfd, void *buff, size_t nbytes, int flags, struct sockaddr* from, socklen_t *fromaddrlen);
 


--------------
Code Structure
--------------

Functionality common to both the client and server lives in the "common" directory under the "src" directory. There are
three files:

* utils.h: Some very basic primitive operations, such as logging, configurable global variables, and some functions for
           processing socket addresses, time values, and strings.
* proto.h: The meat of the of protocol implementation. This contains the relevant structs for messages and connections,
           functions for creating and tearing them down, and for sending and receiving messages. A large portion of
           this file is also the code needed to serialize and deserialize the protocol messages into and from byte
           arrays for sending and receiving.
* processorUtils.h: Some support code for running "processors" in the client and server. Processors are functions
                    invoked when messages are received to advance the state of the connection.

The server lives under "src/server" as "udp_server.c", and the client lives under "src/client" as "udp_client.c". Some
test code lives in the "test" directory. Both the client and server use the poll(2) syscall to either wait for
incoming datagrams or wait a specified amount of time until another event (like a resend or connection timeout) would
occur. Both are designed using an "event driven" model hinging around non-blocking sockets and poll(2); the server
is designed to be able to support many simultaneous clients.

===============
Protocol Design
===============

The following describes how the protocol messages are laid out in a byte-wise fashion. All values are assumed to be
in network byte order (big endian). Messages have the following structure:

* Byte 0: The protocol magic value 0xF1. This is used as quick check to determine if a datagram is a valid message or
          not.
* Byte 1: A flags field. The following are valid flags:
       * 0x1 (MYFTP_F_MOREDATA): There are further messages coming with additional data for a "list", "get" or "put"
                                 connection.
       * 0x2 (MYFTP_F_RETRY): This is not the first time the remote has sent this message.
       * 0x4 (MYFTP_F_FINAL): This is the final message the remote is sending, and the remote expects no further
                              responses.
* Bytes 2-3: A 16-bit opcode:
       *  1 (MYFTP_OP_COMMAND): A command message sent from a client.
       *  2 (MYFTP_OP_START): A start message sent from a server.
       *  3 (MYFTP_OP_OK): An OK acknowledgement from either the client or server.
       *  4 (MYFTP_OP_ERROR): An error message from either the client or server.
       *  5 (MYFTP_OP_DATA): A data message from either the client or server during a "get" or "put" (where appropriate)
       *  6 (MYFTP_OP_LIST): A list data message from the server to the client for an "ls" command.
       *  7 (MYFTP_OP_FIN): A fin message, sent by both ends at the end of a connection.
* Bytes 4-5: The 16-bit unsigned connection ID.
* Bytes 6-9: The 32-bit unsigned sequence number.

The remainder of the message is opcode dependent.
For MYFTP_OP_COMMAND:
   * bytes 10-13: A 32-bit unsigned number of milliseconds for the requested max timeout window.
   * bytes 14-15: The 16-bit unsigned length of the command string (denoted as N).
   * bytes 16-(16+N): The command string (where N is determined by the preceeding value).
   * bytes (16+N+1)-(16+N+3): The 16-bit unsigned length of the arg string (denoted as M).
   * bytes (16+N+4)-(16+N+M+4): The argument string.

For MYFTP_OP_START:
   * bytes 10-13: The 32-bit unsigned sequence number this message is in response to.
   * bytes 14-17: The 32-bit unsigned value for the server's final adjusted maximum timeout window,
                  which will be used by both the client and server.

For MYFTP_OP_OK:
   * bytes 10-13: The 32-bit unsigned sequence number this message is in response to.

For MyFTP_OP_ERROR:
   * bytes 10-13: The 32-bit unsigned sequence number this message is in response to.
   * bytes 14-15: The 16-bit unsigned error code generated at the remote.
   * bytes 16-17: The 16-bit unsigned length of the error message (denoted as N).
   * bytes 18-(18+N): The error message string.

For MYFTP_OP_DATA:
   * bytes 10-13: The 32-bit unsigned block number for the current data message.
   * bytes 14-17: The 32-bit unsigned (approximate) number of total blocks.
   * bytes 18-19: The 16-bit unsigned length of the data block (denoted as N).
   * bytes 20-(20+N): The block data payload for this message.

For MYFTP_OP_LIST:
   * bytes 10-13: The 32-bit unsigned page number for the current list message.
   * bytes 14-15: The 16-bit unsigned number of file names packed in this response.
   * bytes 16-17: The 16-bit unsigned length length of the packed name string (denoted as N).
   * bytes 18-(18+N): The file names packed as a array. Each filename is null terminated.

For MYFTP_OP_FIN:
   * bytes 10-13: The 32-bit unsigned sequence number this message is in response to.
   * bytes 14-15: The 16-bit unsigned connection ID for the remote connection this is in response to.


======================
How I Have Tested This
======================

I have written a few libcheck tests (under test) to check some of the primitive functions I have written in the
"common" directory. Additionally, the programs are built with essentially all of the GCC warning flags turned on
(and set trigger errors in all but a couple of trivial cases). I have run the programs with valgrind using both
error checking and leak checking, exhaustively testing many permutations of commands to verify (with high probability)
that neither the client nor server leak memory.

The code features a number of logging statements (that can be optionally turned on) for ease of debugging, and has a
couple of convenience methods to dump messages to stderr, or dump the contents of a byte buffer in hex.

I have also tested the performance against a known good TFTP server and client using the same systems, and get roughly
the same performance on file transfers as TFTP (when the window size option is off), which is expected (and a good
sign) given the structure and underpinnings of the protocols are very similar.

I additionally tested the client and server using the built in traffic control tooling. The client and server have been
tested to reliably transfer files correctly even with the following significant impediments:

* Up to 20% packet loss (would probably work with even more).
* Large packet delays and jitters (tested up to 750 ms).
* A significant number (30%+) of reordered and duplicate IP packets (and thus duplicated datagrams).

including all of the above at the same time.

====================
Building and Running
====================

**NOTE**: I have only built and developed this against Debian Linux. The server and client may build and run on Mac
          OS X or a BSD. They WILL NOT BUILD on Windows (without something like cygwin or WSL) because Windows lacks
          support for poll(2) and generally has a very poor libc implementation (msvc).

**NOTE**: This may not build against compilers other than GCC. It's likely that it will build with clang/llvm.

To build the client and server, (a relatively recent copy) of gcc and make are required. To build both at the same time:

   make clean && make all

The client and server binaries are placed in the "bin" directory. The server may be started as

   ${PROJECT_PATH}/bin/server ${YOUR_PORT}

The client may be started as:

   ${PROJECT_PATH}/bin/client ${SERVER_ADDRESS} ${SERVER_PORT}

There are a handful of environment variables that alter the behavior of the client and server. To enable more
aggressive logging, set DEBUG to either 1 (info logs) or 2 (info and debug logs). An easy way to do is this is the
following:

   DEBUG=1 ${PROJECT_PATH}/bin/server ${YOUR_PORT}
   DEBUG=1 ${PROJECT_PATH}/bin/client ${SERVER_ADDRESS} ${SERVER_PORT}

The commands for the client should work as specified in the programming assignment document.
