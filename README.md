# Multithreaded Chat Server
This project implements the client-server model in a multithreaded chat server made in C++.
Multiple clients(users) can join the server and chat with each other. 
To handle sending data to and receiving data from multiple clients, the concept of multithreading is used. Thus, as soon as a client enters a server, handling of the client by the server starts running parallely in a thread.
To ensure proper synchronization of the messages sent by the clients, mutex locks are used. This ensures that the common data between the parallely executing threads is gets modified by only one thread at a time. 

The chat server and client run on command line/terminal. 
The server has to be started using a port number for connections with the clients. 
Once this is done, clients can enter the server using the port number. Clients have to enter a unique username and the entry of each new client is displayed on the server. The clients can then chat with other cliets who have also joined the server. 

The complete log of the chat that occured on the server gets stored and can be viewed at any time. The log of the last run of the server can also be viewed even after the server has been closed down.


## How to Run the Project
To run the project, download the repository, open command line/terminal in WSL(for Windows)/Linux/MacOS, move to the repository directory and run the following commands. Since the client and server are written in C++, make sure g++ is installed before running the following commands.

### Server
First, compile server.cpp using:
`g++ -pthread -o server server.cpp`

Then run it using:
`./server [PORT]` or
`./server`

In place of `[PORT]`, specify a port number Eg: 9999. In case no port number has been specified, 9002 is taken as default port number

### Client
First, compile client.cpp using:
``g++ -pthread -o client client.cpp``

Then run it using:
`./client [PORT]` or
`./client`

In place of `[PORT]`, specify the same port number you used for the server. 9002 is taken as default here also.

### Server Log
To view the server log of the last server run or the server log of the current server run till now, run:

`echo -ne $(cat serverlog.txt | sed  's/$/\\n/' | sed 's/ /\\a /g')`


## Internal Working
Both the server and clients follow TCP protocol and use IPV4 address format. 

In server.cpp, a socket for the server is created and then bind to an address inclusive of the port number. After this, the `listen()` method makes the server socket ready to accept client connections, with a backlog set to 10. After this, the server keeps on looking for client connections in an infinite while loop. 

The server.cpp file contains an array `client_list` which stores the username, address and socket field descriptor of the clients that are connected to the server at a time. As soon as a client socket connects with the server socket, its details, i.e username, address and socket field descriptor values are added to the first empty location in the `clients_list`. This addition is done using the method `add_to list()`. Also another method, `remove_from_list()` is present to remove a client from the `client_list` once the client has disconnected from the server. Both these methods modify a data structure that has to be shared between the various threads and hence contain mutex locks to ensure that only one thread is able to modify `clients_list` at a time. The maximum length of `clients_list` is set as 70 (the variable `MAX_CLIENTS`) and so, the the maximum clients that can chat on the server at a time are 70. Once a client has been added to the list, the server must communicate with it and for this purpose, the `handle_connection()` method has been made. As all clients must be handled parallely, a thread is assigned to each client and this thread runs each client's `handle_connection()` method, thus ensuring concurrent handling of each client by the server.

In each `handle_connection()` method that runs parallely, first, the username is received from the client. It is then checked for in the `clients_list` to make sure the username is unique. In case it is, the client is given permission to start sending and receiving messages from the server. The `handle_connection()` method then keeps on receiving instructions from the client. If it receives a message from the client, it calls the `broadcast()` method to broadcast the message sent to all clients connected to the server. Once the client has left the server, its socket is closed and the client's details are removed from the `clients_list`.

As each clients' handling is taken up by a thread, any message sent by a client is sent to all other clients and any analogously, each client receives any message sent by any other client. Thus, the chat server works perfectly.

Any statement that the server prints also gets stored in the file `serverlog.txt` and can be viewed at any time. The more beneficial use of this file is that once a server has been shut down and the console on which it was run has been closed, the completye log of this latest server session can be viewed on this text file. Since the log in the text file might be difficult to understand due to it containing several ANSI colour escape sequence, the exact log can be viewed on the console using the command shown in **How to Run the Project** section.

In client.cpp, first, a username is asked from the client to make sure it follows a set length guideline (between 2 and 32 characters) and then the client socket is created and is made to connect to the server using server address and port number. After this, the client's username is sent to the server to verify for unique username. Once the client is given permission to start chatting, two threads are created, one for holding the `handle_recv()` function and the other to hold the `handle_send()` function. These two functions handle the client's sending messages to the server and receiving messages from the server. Since these two tasks must occur concurrently as we don't know when a client might want to send a message and when the server will be sending a message to the client, we run these two methods parallely using threads.

The `handle_recv()` method keeps on looking for messages from the server. This is done using the `recv()` method. In case this method receives any message from the server, the message is print on the client's console. The `handle_send()` works in a similar way in that it also keeps on looking for any message that the client has typed in its console. Once it gets a message to be sent to the server, the `write()` method is called upon to perform this task. In case the client types `E` (for exit) in the console, the `handle_send()` method makes sure the client program terminates and the client gets exited from the server. 

In this way, each client can send a message independant of receiving messages from the client and can receive messages from the client independant of seding messages to the client using threads.


## Learnings
The learnings involved in doing this project were:
- I was introduced to socket programming and the workings of computer networks and have developed a base on which to explore socket programming further.
- I worked with threads and mutex locks practically for the first time. I had studied these concepts only in theory before.


## Description of Additional Tasks
- Used socket programming to create connections between a chat server and multiple clients, with data being transmitted both ways
- Used the concept of threads to synchronise the order of messages sent and clients entering and leaving the chat server
- Added extra features to the chat server like unique usernames, colour coding client's names in messages, displaying number of clients already on server to a new client joining the server and making a server log


## Video Demo
https://user-images.githubusercontent.com/74502260/174684917-59188eb4-5a5d-4666-9fca-da28abc38d01.mp4


## Resources Used
The following resources were a great help in making this project:
- [ACM IITR's Repository](https://github.com/acmiitr/Multithreaded-Server)
- [This GeeksforGeeks Article](https://www.geeksforgeeks.org/socket-programming-in-cc-handling-multiple-clients-on-server-without-multi-threading/)
- [Beej's Guide to Network Programming](https://beej.us/guide/bgnet/)

