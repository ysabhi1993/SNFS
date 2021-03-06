# Simple Network File System (SNFS) with FUSE and GRPC      
CS798-W17 Advanced Distributed Systems    
Instructor: Prof. Samer Al-Kiswany    
Group: Anthony, Zheyu Xu    
[Complete Report](Report/technical_report.pdf)     
## System Design    
Our Simple Network File System (SNFS) leverage FUSE and gRPC projects. FUSE allows us to intercept system calls generated by file-system operations and gRPC enables the client to invoke procedures declared in the server with necessary parameters.    

![Alt text](Report/system_design.png?raw=true "SNFS Design Architecture")    

## Optimization Design    
There are two optimizations implemented in our system (SNFS), i.e. batch-write and server crash-recovery that will be detailed in the following subsections.    
### Batch Write    
The goal of batch-write optimization is to reduce the frequency of disk I/O. Each call to write() system call will not persist the data to disk, instead, it is being placed temporarily in the memory. Only when the commit() is being called, the data placed on temporary memory space will be flushed to persistent storage.    
### Server Crash-Recovery     
Due to batch-write optimization, server-crash can cause data being buffered in the memory lost. Overcoming this issue, we employ a recovery mechanism requiring the client to send start and end offset of the data to be committed in the server. The server then compares these offset and determines if it has the complete set of the data to be written to disk. If the these offsets do not match, then the server will ask for re-transmission from the client.    

## Evaluation    
In order to evaluate SNFS, we measure the performance in terms of three aspects: the latency improvement achieved through batch-write optimization, the extra overhead caused by server crash and rebooting, and the overall I/O overhead compared with Linux NFS distribution.    

![Alt text](Report/batch-normal.png?raw=true "Comparison of Batch Write and (Normal) Write Through")    
As is shown in Figure 2, the latency of writing operations decreases significantly when we use the batch-write optimization. With the growth in the size of a batch, the difference of the two I/O strategies becomes larger, indicating the overhead caused by small I/O operations We attribute this to the fact that batch write requires only one disk seek and one system call. Whereas in write through strategy, the number of both system calls and disk seeks increase proportionally to number of writes.    
      

![Alt text](Report/nfs-comparison.png?raw=true "Overhead of FUSER and gRPC compared with Linux NFS")    
We also compare the performance of SNFS with that of NFS4 Linux. The latency of writing an 10 MB file is measured. Figure 3 shows that the performance of SNFS is extremely poor with regard to the-facto NFS system. The extra time cost is caused by more stages of function calls (including kernel and userspace) in FUSE interface and message passing that require copying data must go through network stack of the both client and server in gRPC implementation. Although SNFS provides the interface that allows users to operate remote files in the same way as they do to local files. The performance of remote file operations are inevitably degraded.    

## Conclusion   
SNFS is a simulation of the Network File System, which allows users to operate remote files in the same way as local files. It leverages FUSE to build the interface of the virtual file system and uses gRPC for communication. SNFS implements batch-write optimization, minimizing the overhead of I/O operations. It also has good fault tolerance with regard to server crashes.   
 
## Build & Run    
### Building the project    
./make clean    
./make    
### Running the server    
mkdir ./server    
./nfsfuse_server    
### Running the client    
mkdir ./client        
./nfsfuse_client ./client      


