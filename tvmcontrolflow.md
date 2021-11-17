# TVM Control Flow
### 1. Thread and ThreadLocal

A neural network represent in a compute graph form which contructed by a group of operator like conv2d that also get called by layer in some documents, the neural network execution rely on operator execution for implemention, and a executor or runtime is the manager to order and run these operator for neural network, When a operator get executed in a executor like graph_executor, 
