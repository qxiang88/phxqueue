# PhxQueue

[简体中文README](README.zh_cn.md)

PhxQueue is a high-availability, high-throughput and highly reliable distributed queue based on the Paxos protocol. It guarantees At-Least-Once Delivery. It is widely used in WeChat for WeChat micro-messaging, WeChat Pay, WeChat Media Platform, and many other important business.

## Features

*	Absolutely No data lose and strictly reconciliation mechanism

*	Server batch enqueue

*	Dequeue in order strictly

*	Multiple Subscribers

*	Dequeue speed limit

*	Dequeue replay

*	Consumer loadbalance

*	All module are scalable

*	A group of Store or Lock deploy on multiple region is supported

## Build from Source

### Download PhxQueue source

Download the [phxqueue_0.1.0.tar.gz]() and un-tar it to `$PHXQUEUE_DIR`.

### Install Dependence

*	Prepare the `$DEP_PREFIX` diectory for dependence installation. For example:

	```sh
	export $DEP_PREFIX='/usr/local'
	```

*	Protocol Buffers and glog

	Build [Protocol Buffers](https://github.com/google/protobuf/releases) and [glog](https://github.com/google/glog/releases) with `./configure CXXFLAGS=-fPIC --prefix=$DEP_PREFIX`. Then make some links:

	```sh
	rm -r $PHXQUEUE_DIR/third_party/protobuf/
	rm -r $PHXQUEUE_DIR/third_party/glog/
	ln -s $DEP_PREFIX $PHXQUEUE_DIR/third_party/protobuf
	ln -s $DEP_PREFIX $PHXQUEUE_DIR/third_party/glog
	```

*	Boost

	Build [Boost](http://www.boost.org/) (only version 1.56 is supported) with `./bootstrap.sh --prefix=$DEP_PREFIX`. Then make some links:

	```sh
	rm -r $PHXQUEUE_DIR/third_party/boost/
	ln -s $DEP_PREFIX $PHXQUEUE_DIR/third_party/boost
	```

*	LevelDB

	Build [LevelDB](https://github.com/google/leveldb/releases) in `$PHXQUEUE_DIR/third_party/leveldb/` and `ln -s out-static lib`.

*	PhxPaxos and PhxRPC

	Build [PhxPaxos](https://github.com/Tencent/phxpaxos/releases) in `$PHXQUEUE_DIR/third_party/phxpaxos/`. Build [PhxRPC](https://github.com/Tencent/phxrpc/releases) in `$PHXQUEUE_DIR/third_party/phxrpc/`.

*	libco

	Git clone [libco](https://github.com/Tencent/libco) to `$PHXQUEUE_DIR/third_party/colib/`.

### Build PhxQueue

```sh
cd $PHXQUEUE_DIR/
make
```

## The PhxQueue Distribution

PhxQueue is structured like this:

```
phxqueue/ ................. The PhxQueue root directory
├── bin/ .................. Generated Binary files
├── etc/ .................. Example Configure files
├── lib/ .................. Generated Library files
├── phxqueue/ ............. PhxQueue source files
├── phxqueue_phxrpc/ ...... PhxQueue with PhxRPC implementation
└── ...
```

the output files are located in `bin/` and `lib/`, while the sample configure files are located in `etc/`.

## Start a simple PhxQueue

The built PhxQueue is ready to run simple demos.

### Preparation

PhxQueue open multiple files on the same time. Make sure to set enough (> 4000) open files limit with `ulimit -Sn` or `ulimit -n`.

### Start Store

Now Start 3 Store node (add `-d` if run as daemon):

```sh
bin/store_main -c etc/store_server.0.conf
bin/store_main -c etc/store_server.1.conf
bin/store_main -c etc/store_server.2.conf
```

You can find informations and errors in log files:

```sh
ps -ef | grep store_main
tail -f log/store.0/store_main.INFO
tail -f log/store.1/store_main.INFO
tail -f log/store.2/store_main.INFO
```

### Start Lock

Now Start 3 Lock node (add `-d` if run as daemon):

```sh
bin/lock_main -c etc/lock_server.0.conf
bin/lock_main -c etc/lock_server.1.conf
bin/lock_main -c etc/lock_server.2.conf
```

You can find informations and errors in log files:

```sh
ps -ef | grep lock_main
tail -f log/lock.0/lock_main.INFO
tail -f log/lock.1/lock_main.INFO
tail -f log/lock.2/lock_main.INFO
```

### Start Scheduler

Now Start 3 Scheduler node (add `-d` if run as daemon):

```sh
bin/scheduler_main -c etc/scheduler_server.0.conf
bin/scheduler_main -c etc/scheduler_server.1.conf
bin/scheduler_main -c etc/scheduler_server.2.conf
```

You can find informations and errors in log files:

```sh
ps -ef | grep scheduler_main
tail -f log/scheduler.0/scheduler_main.INFO
tail -f log/scheduler.1/scheduler_main.INFO
tail -f log/scheduler.2/scheduler_main.INFO
```

### Start Consumer

Now Start 3 Consumer node (add `-d` if run as daemon, not for this test):

```sh
bin/consumer_main -c etc/consumer_server.0.conf
bin/consumer_main -c etc/consumer_server.1.conf
bin/consumer_main -c etc/consumer_server.2.conf
```

You can find informations and errors in log files:

```sh
ps -ef | grep consumer_main
tail -f log/consumer.0/consumer_main.INFO
tail -f log/consumer.1/consumer_main.INFO
tail -f log/consumer.2/consumer_main.INFO
```

### Send Single Test Requests

Now the deploy of simple PhxQueue is finished! Use the benchmark tool to send some test request:

```sh
bin/test_producer_echo_main
```

You will get the output from test Producer:

```sh
produce echo succeeded!
```

Now let's see the output of Consumer (only 1 of 3 consumers):

```sh
consume echo succeeed! ...
```

### Run Benchmark

```sh
bin/producer_benchmark_main 10 5 5 10
```

Watch the Consumer log files again:

```sh
tail -f log/consumer.0/consumer_main.INFO
tail -f log/consumer.1/consumer_main.INFO
tail -f log/consumer.2/consumer_main.INFO
```

Now you can see the Consumer dequeue log like this:

```
INFO: Dequeue ret 0 topic 1000 sub_id 1 store_id 1 queue_id 44 size 1 prev_cursor_id 9106 next_cursor_id 9109
```

### Clear Test Logs or Data

While running PhxQueue, losts of logs and data are generated. Run `log/clear_log.sh` to clear logs. Run `data/clear_data.sh` to delete data. Please make sure the data is useless before delete.

## Deploy Distributed PhxQueue

Normally, each node should be deployed on one machine. Change the `etc/*.conf` for each node.

Files located in directory `etc/`:

```
globalconfig.conf .................Global config
topicconfig.conf ................. Topic config
storeconfig.conf ................. Store config
consumerconfig.conf ...............Consumer config
schedulerconfig.conf ..............Scheduler config
lockconfig.conf ...................Lock config
```

Deloy and modify these files on all target machine.

## Contribution

Please follow [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html) in PRs.
