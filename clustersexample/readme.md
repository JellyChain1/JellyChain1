Clusters#

This module runs in the background.
All you have to do is run the zero-downtime.js source code and point your RPC Client/Server and the JellyChain1 server to anyone of the worker ports you will see in your terminal and enjoy. Don't forget to get your keypair first. 
Read the main readme.md for more instructions.

Please refer to the "Clusters" and "What Is Clusters?" Wiki Link for a breakdown of the clusters module and how it is used with JellyChain1.

About Clusters#

Clusters of Node.js processes can be used to run multiple instances of Node.js that can distribute workloads among their application threads. When process isolation is not needed, use the worker_threads module instead, which allows running multiple application threads within a single Node.js instance.

The cluster module allows easy creation of child processes that all share server ports.

How it works#

The worker processes are spawned using the child_process.fork() method, so that they can communicate with the parent via IPC and pass server handles back and forth.

The cluster module supports two methods of distributing incoming connections.

The first one (and the default one on all platforms except Windows) is the round-robin approach, where the primary process listens on a port, accepts new connections and distributes them across the workers in a round-robin fashion, with some built-in smarts to avoid overloading a worker process.

The second approach is where the primary process creates the listen socket and sends it to interested workers. The workers then accept incoming connections directly.

The second approach should, in theory, give the best performance. In practice however, distribution tends to be very unbalanced due to operating system scheduler vagaries. Loads have been observed where over 70% of all connections ended up in just two processes, out of a total of eight.

Because server.listen() hands off most of the work to the primary process, there are three cases where the behavior between a normal Node.js process and a cluster worker differs:

server.listen({fd: 7}) Because the message is passed to the primary, file descriptor 7 in the parent will be listened on, and the handle passed to the worker, rather than listening to the worker's idea of what the number 7 file descriptor references.
server.listen(handle) Listening on handles explicitly will cause the worker to use the supplied handle, rather than talk to the primary process.
server.listen(0) Normally, this will cause servers to listen on a random port. However, in a cluster, each worker will receive the same "random" port each time they do listen(0). In essence, the port is random the first time, but predictable thereafter. To listen on a unique port, generate a port number based on the cluster worker ID.
Node.js does not provide routing logic. It is therefore important to design an application such that it does not rely too heavily on in-memory data objects for things like sessions and login.

Because workers are all separate processes, they can be killed or re-spawned depending on a program's needs, without affecting other workers. As long as there are some workers still alive, the server will continue to accept connections. If no workers are alive, existing connections will be dropped and new connections will be refused. Node.js does not automatically manage the number of workers, however. It is the application's responsibility to manage the worker pool based on its own needs.

Although a primary use case for the node:cluster module is networking, it can also be used for other use cases requiring worker processes.

Class: Worker#

A Worker object contains all public information and method about a worker. In the primary it can be obtained using cluster.workers. In a worker it can be obtained using cluster.worker.

worker.id#

Each new worker is given its own unique id, this id is stored in the id.

While a worker is alive, this is the key that indexes it in cluster.workers.

worker.kill([signal])#

signal <string> Name of the kill signal to send to the worker process. Default: 'SIGTERM'
This function will kill the worker. In the primary worker, it does this by disconnecting the worker.process, and once disconnected, killing with signal. In the worker, it does it by killing the process with signal.

The kill() function kills the worker process without waiting for a graceful disconnect, it has the same behavior as worker.process.kill().

This method is aliased as worker.destroy() for backwards compatibility.

In a worker, process.kill() exists, but it is not this function; it is kill().

worker.process#

<ChildProcess>
All workers are created using child_process.fork(), the returned object from this function is stored as .process. In a worker, the global process is stored.

See: Child Process module.

Workers will call process.exit(0) if the 'disconnect' event occurs on process and .exitedAfterDisconnect is not true. This protects against accidental disconnection.

Event: 'fork'#

worker <cluster.Worker>
When a new worker is forked the cluster module will emit a 'fork' event. This can be used to log worker activity, and create a custom timeout.

Reference nodejs.org cluster