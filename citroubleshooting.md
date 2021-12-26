## After submit a tvm PR, developer may experience a CI failure issue like follow!

  ![](https://user-images.githubusercontent.com/2281489/147400370-54634b26-0bed-4b29-94f1-7c5b704fd8ea.png)
  
## Click the [Details], developer can find the issue detail

![](https://user-images.githubusercontent.com/2281489/147400464-222018fa-bfe9-472b-84a2-31e1e98558f3.png)

## To trouble shooting such issue, reproduce it at local is a easy way for try by running the task causing the issue.

![](https://user-images.githubusercontent.com/2281489/147400507-cfa5c664-fd56-418a-b241-1c223740e240.png)

## Run the task may experience some build error for example, complain can not find LLVM

This may caused by user used a absolute path for LLVM,  a easy way for fixing is to delete this config.cmake file in tvm/build then do the cmake and make again

## Run the task should can reproduce issue at local, by using following command user can login the docker and test your command inside the docker

docker/bash.sh -it tlcpack/ci-cpu:v0.79 /bin/bash

## For the said issue that gettid not found issue

The symptom is in local build is ok, but in the said ci , it is not, that is because "gettid" is a wrapper which supported by GLIBC2.30 after, the ci still
use a old glibc which is 2.24.

by using ldd --version can check what glibc version.
  
  


