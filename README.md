## PerfSyn

Perfsyn automatically generates unit-tests that trigger a performance bottleneck
for a Method Under Test (MUT). The approach is described in our [CGO 18 paper](http://mp.binaervarianz.de/cgo2018.pdf):

```
Synthesizing Programs that Expose Performance Bottlenecks
Luca Della Toffola (ETH Zurich, Switzerland)
Michael Pradel (TU Darmstadt, Germany)
Thomas Gross (ETH Zurich, Switzerland)
```

This short tutorial shows how to build and use PerfSyn. 
The code of PerfSyn is actively maintained in another private repository.
Contact **Luca Della Toffola** via [e-mail](luca.dellatoffola@inf.ethz.ch)
to obtain the latest snapshot of the code and/or the version used in the 
evaluation of the paper.

## Dependencies
The program generation and analysis framework are written in Scala and the profiling framework is written in Java,
The result reporting tool that parses and elaborate the generated data is written in Python. 
We provide Docker building and testing scripts (Docker version 1.13.x)
The minimum software requirements to run and compile PerfSyn are:
- Java Virtual Machine (version 7 or greater)
- Python (version 2.7 or greater)
- Gradle (version 3.3 or greater)
- Scala (version 2.10.x or greater)
- Ant (version 1.10.x or greater)

## Build PerfSyn
The repository contains two scripts to build PerfSyn.
The first Gradle script compiles the code and generates PerfSyn Jar files in the
`jars` directory, in addition the script generates are directory `deps` which
contains a copy of all the dependencies. The second directory with the dependencies
is not strictly necessary because the files in `jars` directory are *fat Jars*
which include all the dependencies.

```bash
> gradle uploadArchives copyLibs && ant
```

## Build PerfSyn with Docker
We provide a Docker building and testing infrastructure to build PerfSyn without
installing all the dependencies listed above. To build PerfSyn with Docker you
just need to type command:

```bash
> bash build.sh
```

This will create three Docker images `perfsyn:build`, `perfsyn:ant`, and the
image `perfsyn`. You only need the last image `perfsyn` to run PerfSyn, the
other two images can be manually deleted if necessary.

## Run PerfSyn

To run PerfSyn we provide a script `perfsyn.sh`, the script takes X parameters
and can be run in the following way:
```bash
> bash perfsyn.sh \
      test identifier \
      config directory or config file \
      type of search \
      source code path \
      cache directory \
      output directory \
      jvm heap size in GB
```
The parameters of the script have the following meaning:

Parameter | Description
:----------- | :-----
test identifier | experiment tag used for the output directory name
config (directory or file) | the configuration file or directory from `config`
type of search | the search type, it can take value `a*` or `aco`
source code path | path to source, usually used the be output of command `pwd`
cache directory | cache directory for the parsed sources (it must exists)
output directory | output directory for the results (it must exists)
jvm heap size | the size of the JVM heap in GB (e.g., 32).

To run the experiment that we used for all the MUTs analyzed for the *changes
in relative performance* with ACO:

```bash
> bash perfsyn.sh \
    cgo18-rel \
    configs/cgo18/relative \
    aco \
    `pwd` \
    path/to/cache \
    path/to/output \
    16
```

To elaborate the data that PerfSyn produced in a search run use the script
`python/processing.py` script. The command-line options to the script can be
displayed using the command:

```bash
> python python/processing.py --help

usage: processing.py [-h] [--dir DIR] [--stats] [--plot] [--execute]
                     [--working-dir WD] [--cache-dir CD]

PerfSyn - Finding performance-bugs automatically.

optional arguments:
  -h, --help        show this help message and exit
  --dir DIR         specifies the input directory to scan
  --stats
  --plot
  --execute
  --source-dir WD
  --cache-dir CD
```

To elaborate the data from the previous command used to produce data for
the *changes in relative performance* use:

```bash
> python python/processing.py \
  --dir path/to/output \
  --source-dir `pwd` \
  --cache-dir /path/to/cache \
  --stats
```

This command will create a file `relative.md` inside `path/to/output` which
contains a table with the results and the reported bottlenecks. In the case the
profiled data is for the *unexpected asymptotic complexity* oracle the output
file will be called `complexity.md`.

### Output example
The following example shows the elaborated results that can be found in the
`relative.md`, which are reported in bottom part of the Table X in Section Y.
The column description of the table are self-explanatory.

### Time measurements with [JMH](http://openjdk.java.net/projects/code-tools/jmh/)
To verify that a generated program effectively slows down performance between
two version of the same MUT we provide an option in the script
`python/processing.py` to run the generated program with JMH using the option
`--execute`:

```bash
> python python/processing.py \
  --dir path/to/output \
  --source-dir `pwd` \
  --cache-dir /path/to/cache \
  --stats \
  --execute
```
The output of this command will be again a `relative.md` file but this time
reporting also if the speed-up or slow-down happened running the program using
the current machine setup.

## Run PerfSyn with Docker
We provide an additional script to run PerfSyn in a Docker container. The script
has similar parameters as the previously described one, but the target and
source directories are already specified in the image.
```bash
> bash docker.sh \
      test identifier \
      config directory or config file \
      type of search \
      jvm heap size in GB
```

To run *changes in relative performance* in a Docker container use:
```bash
> bash docker.sh \
    cgo18-rel \
    configs/cgo18/relative \
    aco \
    16
```

Once the Docker container is terminated one can copy the result files from
a volume that is created in the local machine drive using the command:
```bash
> bash docker cp v-perfsyn-cgo18-rel-output:/output /host/path/to/output
```
