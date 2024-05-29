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
 
