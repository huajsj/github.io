# TVM Control Flow
### 1. Thread and ThreadLocal
#### Background
In tvm, a neural network represented in a compute graph form which have ANF or GNF format to deal with compute scope etc problem, such compute graph contructed by a group of operator like conv2d named with layer in some documents, after compilation these operator get compiled into backend specify code, the neural network execution rely on backend specify operator code execution, and a executor or runtime is the manager to order and run these operator for neural network.

#### Runtime
Runtime get involved by Execution steps, after comilation finished the executable binary to execute every neural network operator will get managed by a executor to run for example graph_executor, the executor for example graph executor basiclly need to laugch a task to execute the operator. to run the task, executor need to figure out couple requirement, the first is how to many **cpu thread** are asked for such task, secondly how **distribute** the execution on different thread for the **data parallelism**, the third is support **multiple runtime** at same time.

#### Multiple Runtime Support for Execution at Same time.
Current tvm in architecture partially support multiple different running in same time, this ask tvm to provide different threads pool for runtime operator execution, tvm use a solution that declare the thread pool management class as a thread local variable to deal with such problem, such solution with a assumtion that as the runtime execution being on different thread, then they have the neccessary to execute the runtime operator in different threads pool in parallel.

#### thread_local
To cross platform support per thread variable, tvm use c++ keyword thread_local to implement the thread variable(tvm use dmlc ThreadLocalStorage feature that rely on thread_local), 

