# TVM Control Flow
### 1. Thread and ThreadLocal
#### Background
In tvm, a neural network represented in a compute graph form which have ANF or GNF format to deal with compute scope etc problem, such compute graph constructed by a group of operator like conv2d named with layer in some documents, after compilation these operator get compiled into backend specify code, the neural network execution rely on backend specify operator code to do execution, and a executor or runtime is the manager to order and run these operator for neural network.

#### Executor/Runtime
Runtime get involved by Execution steps, after comilation finished the executable binary to execute every neural network operator will get managed by a executor to run for example graph_executor, the executor for example graph executor basiclly need to laugch a task to execute the operator. to run the task, executor need to figure out couple requirement, the first is is support **multiple runtime** at same time, the second is how to **distribute** the execution on different thread for the **data parallelism**, the third is how  many **cpu thread** are asked for such task.

#### Multiple Runtime Support for Execution at Same time.
Current tvm in architecture  support multiple different running in same time(partially), this would ask tvm to provide different threads pool for different runtime operator execution, tvm use a solution that declare the thread pool management class as a thread local variable to deal with such problem, such solution with a assumtion that as the runtime execution stay on different thread, then they have the neccessary to execute the runtime operator in different threads pool in parallel.

#### thread_local
To cross platform support per thread variable, tvm use c++ keyword thread_local to implement the thread variable(tvm use dmlc ThreadLocalStorage feature that rely on thread_local). There is a variable in runtime thread_pool.cc  which response to maintain a thread pool, by using dmlc::ThreadLocalStorage, such threadpool variable create as a thread variable, then any caller as they stay on different thread would get a different threadpool variable with different threads group.

#### fused_xxx and TVMBackendParallelLaunch
An after compile NN layer is single function that fused multiple operator, for example fused_conv2d_multiply_add fused 3 operator, the said function will call TVMBackendParallelLaunch to launch a task, in this TVMxxx function the said ThreadLocal varaible would get used to maintain a thread pool, let's say we have 2 fused
function, as the first fused function called, the said thread varaible ThreadLocal not exit then it would get create with multiple threads as worker, as second fused
function running, the thread varaible already there hence no new threads get created and a existing ThreadLocal variable woulld get returned.

### Data parallelism on worker thread
Let's start at single executor/runtime case, after ThreadLocal class initialized finished, tvm have M threads, each thread is running a function called RunWorker, the question is how the parallel compute get associate with particular thread? for a normal case a worker thread function logic should already defined before thread launch, but for a already laugnched thread, how to make it to execute a variant logc? here TVM use a ***task queue*** and a ***lambda function structure*** to resolved how to do distribute the arbitrary function in existing theads problem.

### Lambda function
```
(gdb) ptype FTVMParallelLambda
type = int (*)(int, TVMParallelGroupEnv *, void *)
```
The said is the lambda function used in TVM thread pool for parallelism, it is a **function pointer** with 3 parameters, first is task number second is env third is the
data need to be processing and may also have the storage information.

### Task queue
When **TVMBackendParallelLaunch** get called, that run a task, for example to compute a GEMM in parallel in 4 thread, first the all matrix data involved by the said
GEMM would get provided in a **single variable** data, there also have a parameter to tell how may it need the parallel be, this parameter is num_task , here the value is 4, then 4 "task structure" are created and push into a task queue, now the task queue have 4 task, the 4 worker thread get notify and go to poll from the queue, by using such way the 4 task get fairly distribute into 4 thread.

### OpenMP
Once OpenMP supported ,the said thread local logic not aplly annymore, this part is a TODO part.



