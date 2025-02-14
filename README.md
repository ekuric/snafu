# SNAFU - Situation Normal: All F'ed Up

Most Performance workload tools were written to tell you the performance at a given time under given circumstances.

These scripts are to help enable these legacy tools store their data for long term investigations.

## What workloads do we support?

| Workload                       | Use                    | Status             |
| ------------------------------ | ---------------------- | ------------------ |
| UPerf                          | Network Performance    | Working            |
| fio                            | Storage IO             | Working            |
| YCSB                           | Database Performance   | Working            |
| Pgbench                        | Postgres Performance   | Working            |


## What backend storage do we support?

| Storage        | Status   |
| -------------- | -------- |
| Elasticserach  | Working  |
| Prom           | Planned  |

## how do I develop a snafu extension for my benchmark?

In what follows, your benchmark's name should be substituted for the name "Your_Benchmark".  Use alphanumerics and
underscores only in your benchmark name.

You must supply a "wrapper", which provides these functions:
* build the container image for your benchmark, with all the packages, python modules, etc. that are required to run it.
* runs the benchmark and stores the benchmark-specific results to an elasticsearch server

Your ripsaw benchmark will define several environment variables:
* es - hostname of elasticsearch server
* es_port - port number of elasticsearch server (default 9020)
* es_index - prefix of index name for your results in elasticsearch (default "ripsaw")

It will then invoke your wrapper via the command:

```
python run_snafu.py --tool Your_Benchmark ...
```

Additional parameters are benchmark-specific and are passed to the wrapper to be parsed, with the exception of some
common parameters:

* --tool - which benchmark you want to run
* --samples - how many times you want to run the benchmark (for variance measurement)
* --dir -- where results should be placed

Create a subdirectory for your wrapper with the name Your_Benchmark_wrapper.   The following files must be present in
it:

* Dockerfile - builds the container image in quay.io/cloud-bulldozer which ripsaw will run
* \_\_init\_\_.py - required so you can import the python module
* Your_Benchmark_wrapper.py - run_snafu.py will run this (more later on how)
* trigger_Your_Benchmark.py - run a single sample of the benchmark and generate ES documents from that

In order for run_snafu.py to know about your wrapper, you must add an import statement and a key-value pair for your
benchmark to utils/wrapper_factory.py.

The Dockerfile should *not* git clone snafu - this makes it harder to develop wrappers.   Instead, assume that the image
will be built like this:

```
# docker build -f Your_Benchmark_wrapper/Dockerfile .
```

And use the Dockerfile command:

```
RUN mkdir -pv /opt/snafu
COPY . /opt/snafu/
```

The end result is that your ripsaw benchmark becomes much simpler while you get to save data to a central Elasticsearch
server that is viewable with Kibana and Grafana!

Look at some of the other benchmarks for examples of how this works.

