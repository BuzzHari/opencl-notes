[[#]] OpenCL 1.2

Notes on youtube video lecture by AJ Guilon.


## High-Level Overview

* OpenCL Models:
    * Device Model
    * Execution Model
    * Memory Model
    * Host API

* Some Use Cases:
    * Fast permutations: If the deveice can move memory faster than the host.
    * Data translation: Change from on data format to another one.
    * Numerical software: If devices can execute instructions fast enough.

* OpenCL Specification Parts:
    * Core Specification(FULL_PROFILE): What every implementation must support.
    * Embedded Profile(EMBEDDED_PROFILE): Subset of core specification, Relaxation of FULL_PROFILE.
    * Extensions: Additions to the specification, Might become part fo the core later.

* OpenCL Components:
    * C Host API: What you call from the host, Directs devices.
    * OpenCL C: Used to program the device, Based on C99, Many built-in functions
    * Models: Memory, execution ...

### OpenCL Models

* Device Model:
    * Device(like a GPU), has multiple compute units(CU).
    * Each CU, has several processing elements(PE), each of them having their own private memory.
    * Processing element can be thought of as a very simple processor. All instructions are executed on the processing element. 
    * The device, has several memory regions.
    * Global Memory:
        * Shared with all processing elements.
        * Host can access this memory too, can map it, copy data to/from this memory.
        * Is OpenCL persistent storage, the memory will remain across subsequent executions. Other memory regions are scratch space.
    * Constant Memory:
        * Shared with all processing elements.
        * Read-only memory.
        * Very efficient way to share data with all devices PEs.
        * Not persistent.
    * Local Memory (in CUs):
        * Shared with all PEs in a CU.
        * Very efficient way to share data with all CU PEs.
        * Cannot be accessed by other compute units.
        * Not persistent.
    * Private Memory (Attached to a PE):
        * Accessible by a single processing element.
        * No other PE can access this memory.
        * Not persistent.
    
* Execution Model:
    * OpenCL executes kernel functions on the device. A kernel is just a special name for functions in this context, has nothing to do with the OS kernel.
    * Ordinay functions with a special signature.
    * Kernel calls have two parts:
        * Ordinary function argument list.
        * External execution parameters that control parallelsim.
    * Role of the host in kernel execution:
        * Coordinatres execution.
        * Provides arguments to the kernel.
        * Provides execution parameters to launch the kernel.
    * **NDRange**: Execution Strategy:
        * The same kernel function will be invoked many times. The argument list is identical for all invocations.
        * Basically we call the same funciton over and over. The execution parameters dictate the number of times it'll run. 
        * Hosts sets extra execution parameters prior to launch.
        * Identifying the call: How do kernel functions know what to work on?
            * Execution parameters provide an index space. Each function invocation can access its index.
            * For example, if a function is called 10 times, each function will know it's index, for instance function 3 will know it's *call 3 of 10*.
            * The index space is n-Dimensional.
            * Global work size, global work offset.
        * Index Space:
            * Think of it as set of indices, elemnts are n-element tuples.
            * The set is populated before kernel execution.
            * Kernels pull an index out and run.
            * It stops when the index set is empty.
        * Work-item: invocation of the kernel for a particular index.
        * Gloabl ID: globally unique id for a work-item (from index space). 
        * Global Work Size: the number of work-items (per dimension).  
        * Work Dimension: dimension of the index space.
        * Mapping NDRange to Devices:
            * PR runs instructions, so work-items should run on PEs.
            * Assign multiple work-items to each PE, handle global work size > number of PE.
        * Not a good execution model if the local memory isn't used.
        * Work-groups:
            * Partition the global work into smaller pieces.
            * Each partition is called a work-group.
            * Work-groups execute on compute units. Work-item in the work-group is mapped to CU PEs.
            * All work-items in the work-group share local memory.
            * Work-group size has a physical meaning, i.e. the size depends on the device, it is device specfic.
            * Work-items can find out what work-group they are in.
            * The Work-item perspective:
                * Has private memory attached to it.
                * Constant memory attached to it.
                * Has access to global memory.a
                * Work-items in a work-group also have local memory attached to them.
            * Work-Group Size:
                * Maximum work-group size is a device charactueristic. This can be queried from the device.
                * Maximum work-group size is an integer. work-group could be sized n-dimensional.
                * How to determine the best work-group size? -- TODO.
            * n-Dimensional Work-Groups:
                * You can have multiple dimensions.
                * The device maximum work-group size is scalar.
                * Work-Group Size = (w_1, w_2,...w_k), `w_1 * w_2 *...*w_k <= max work-group size`.
    * Final Kernel Call Points:
        * Host will provide execution dimensions to the device, this creates an index space.
        * Parameters can be values or global memory objects.
        * Global memory is persistent between calls:
            * Constant, Local, Private memory is just scratch space.
            * They are going to be reset per kernel call.
        * OpenCL implementation has considerable flexibility.

* Host API:
    * Platform:
        * Is an implementation of OpenCL.
        * They are like drivers for particular devices.
    * The Context:
        * You create a context for a particular platform. You cannot have multiple platforms in a context.
        * A context is a container. Contains devices and memory.
    * Programs: 
        * Programs are just collections of kernels. You extract kernels from your program to call them.
        * OpenCL applications have to load kernels.
        * Programs are device specific.
    * Asynchronous Calls:
        * The host manages devices asynchronously.
        * Host issues commands to the device.
        * Commands tell the device to do something.
        * Devices take commands and do as they say.
        * Host waits for commands to complete. It's not busy wait.
        * Commands can be dependent on other commands.
        * OpenCL commands are issued by `clEnqueue*` calls.
            * A `cl_event` object returned by `clEnquere*` calls is used for dependencies.
    * OpenCL has command-queues.
    * A command-queue is attached to a single device.
    * You can create as many command-queues as you want.
    * `clEnqueue*` commands have a command-queue parameter.
    * So, when the Host calls a command, the command is generated and is placed into the command-queue.
    * The command will run and control the device, when it reaches the front of the queue.
    * The Host can find out if the command has been completed or not.
    * Host API controls the device.
    * Asynchronous execution model.
