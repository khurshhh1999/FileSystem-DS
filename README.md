# **Distributed File System**
The main objective of the File System is to offer a platform for data storage, that's [**distributed**](#system-components) on multiple machines, [**highly available**](#data-replication-mechanism), [**reliable**](#data-replication-mechanism) and [**fault tolerant**](#fault-tolerance). 

## **Data Replication Mechanism**
File System does Data Replication to increase its reliability and availability. The system aims for a 3/n replication ratio, meaning that every file that's uploaded to the file system is replicated 3 times across the system. The replication also enables the system to provide a multi-source download to users that speeds up the download process and stops the server capacity from being a bottleneck during download.

The Replication is a periodic routine that initiated by the Tracker Node once every 2 mins (parameter, can be changed). The Replication routine goes as follows:
- The Tracker Node refers to its database and determines the files that need replication.
- The Tracker Node chooses a suitable source Data Keeper Node, and a suitable destination Data Keeper Node for replication.
- The Tracker Node sends a replication request to the source Data Keeper Node and provides the needed information about the destination Data Keeper Node.
- The source Data Keeper Node establishes communication with the destination Node and the file replication starts.
- Once the replication is done, the source Data Keeper Node sends a completion confirmation to the Tracker Node so it can update its database.

The Replication algorithm handles the following network errors:
- If a file has been partially replicated (to 2 machines), and any of the two sources went offline, it chooses the other.
- If all possible destinations are offline, no replicas are made.
- If a replication was interrupted by network failure from either source or destination Data Keeper Nodes, the system recovers after a timeout and the replication is restarted.

## **Types of Requests**
The system supports 3 types of requests:
- An Upload request.
- A download request.
- A display request (analogous to the `ls` command in UNIX).


## **Request Handling**
The system handles the 3 formerly mentioned types of requests in the following fashion:
- ### **Upload Request Handler**

    The upload request handler works as follows:
    - The authenticated user sends an upload request to the Tracker Node.
    - The Tracker Node refers to its Database and selects a Data Keeper Node for the file transfer.
    - The Tracker Node sends back the IP:Port combination to the client software in order to establish communication with the user.
    - The users re-sends the request to the designated Data Keeper Node and established connection.
    - The Data Keeper Node starts receiving data from the user 1 chunk (1 MB) at a time, and stores the data into the user's directory.
    - When the file transfer is finished, the Data Keeper Node sends a completion confirmation to the Tracker Node.
    - The Tracker Node updates its files database and sends a completion confirmation back to the user.

## **Fault Tolerance**
The system is designed with fault tolerance in mind. The system is able to identify and handle the following types of faults:
- User - Tracker Node connection drop.
- User - Data Keeper Node connection drop.
- Data Keeper Node - Data Keeper Node connection drop.
- Tracker - Data Keeper Node connection drop.

The system identifies these types of faults by running blocking functions (send/receive) on different threads and uses channels to notify the main thread of completion. The main thread makes sure that any thread that doesn't terminate before a preset timeout is gracefully handled and forcibly terminated.

The system is also able to identify other types of errors such as:
- Wrong file names in upload requests.
- Unauthorized access of files.

## **System Testing**
The system has been tested in a distributed configuration (1 Tracker Node and 3-4 Data Keeper Nodes) under all types of supported requests.
