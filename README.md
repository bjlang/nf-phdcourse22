# Nextflow training for CRG PhD course 2022

Project repository for the Nextflow introductory training taking place online on the 25th of October 2022 in the framework of the CRG PhD introductory course 2022.

## Table of Contents

* [Prerequisites](#Prerequisites)
* [Nextflow in a nutshell](#Nextflow-in-a-nutshell)
  * [How?](#How)
  * [Technical requirements](#Technical-requirements)
* [Installation](#Installation)
* [Nextflow hands-on](#Nextflow-hands-on)
  * [1 - Define the pipeline parameters](#Step-1---define-the-pipeline-parameters)
  * [2 - Create transcriptome index file](#Step-2---Create-transcriptome-index-file)
  * [3 - Collect read files by pairs](#Step-3---Collect-read-files-by-pairs)
  * [4 - Perform expression quantification](#Step-4---Perform-expression-quantification)
  * [5 - Quality control](#Step-5---Quality-control)
  * [6 - MultiQC report](#Step-6---MultiQC-report)
  * [7 - Handle completion event](#Step-7---Handle-completion-event)
  * [8 - Custom scripts](#Step-8---Custom-scripts)
  * [9 - Modularization with DSL2](#Step-9---Modularization-with-DSL2)
  * [10 - Executors](#Step-10---Executors)
  * [11 - Use configuration profiles](#Step-11---Use-configuration-profiles)
  * [12 - Run a pipeline from a GitHub repository](#Step-12---Run-a-pipeline-from-a-GitHub-repository)
  * [13 - Conda/Bioconda packages](#Step-13---Condabioconda-packages)
  * [14 - Metrics and reports](#Step-14---Metrics-and-reports)
  * [15 - Run in the cloud using AWS Batch](#Step-10---Run-in-the-cloud-using-AWS-Batch)
* [Docker hands-on](#Docker-hands-on)
  * [1 - Run a container](#Step-1---Run-a-container)
  * [2 - Pull a container](#Step-2---Pull-a-container)
  * [3 - Run a container in interactive mode](#Step-3---Run-a-container-in-interactive-mode)
  * [4 - Your first Dockerfile](#Step-4---Your-first-Dockerfile)
  * [5 - Build the image](#Step-5---Build-the-image)
  * [6 - Add a software package to the image](#Step-6---Add-a-software-package-to-the-image)
  * [7 - Run Salmon in the container](#Step-7---Run-Salmon-in-the-container)
  * [8 - File system mounts](#Step-8---File-system-mounts)
  * [9 - Upload the container in the Docker Hub (bonus)](#DockerHub)
* [Where to get help](#Where-to-get-help)
* [More resources](#More-resources)

## Prerequisites

To correctly follow this course attendees should be familiar with:

* UNIX-based environments (i.e. basic UNIX commands)
* CLI (command-line interface)
* Scripting languages
* Basic knowledge of bioinformatics tools

**If you don't meet all the prerequisites listed above, we would like to warn you that it is possible that you could not follow the pace of the course since it was designed for the "advanced" group. Of course, we will happy if you still want to join us just following our explanations and revisit the course in the future since it will be openly available on github.**

## Nextflow in a nutshell

A workflow engine for data analysis pipelines with a strong focus on enabling:

* Portability
* Reproducibility
* Scalability
* Usability

### <a name="How"></a> How? 

* Functional/reactive programming model
* Decoupling and isolating tasks
* Concise domain specific language for recurrent task operations
* Pragmatic, allowing quick prototyping and iterations
* Hide complexity
* Coexists with errors (!)

### Technical requirements

* Unix-like OS (Linux, macOS, etc.)
* [Java](https://jdk.java.net/) 11 (or later, up to 18)
* [Singularity](https://github.com/sylabs/singularity) 2.5.x (or later)
* [Docker](https://www.docker.com/) engine 1.10.x (or later, optional)
* [Conda](https://conda.io/) 4.5 (or later, optional)
* [Graphviz](http://www.graphviz.org/) (optional)
* [AWS Batch](https://aws.amazon.com/batch/) computing environment properly configured (optional)

## Installation

Load Java 11

```
module use /software/as/el7.2/EasyBuild/CRG/modules/all
module load Java/11.0.2
```

Clone this repository with the following command:

```
git clone https://github.com/cbcrg/nf-phdcourse22.git && cd nf-phdcourse22
```

Then, install Nextflow by using the following command:

```
curl -s https://get.nextflow.io | bash
```

The above snippet creates the `nextflow` launcher in the current directory.

Then, please load version 3.8.3 of Singularity on your environment using the command below:

```
module load Singularity/3.8.3
```

Finally, copy the Singularity image using the command below:

```
mkdir singularity && cp /users/cn/jespinosa/phd_course_22/singularity/quay.io-nextflow-rnaseq-nf-latest.img ./singularity/
```

<!-- Finally pull the following Docker container:

```
docker pull nextflow/rnaseq-nf
``` -->

> If you are running the tutorial in local with docker then you could use this command instead `docker pull nextflow/rnaseq-nf`

## Nextflow hands-on

During this tutorial you will implement a proof of concept of a RNA-seq pipeline which:

1. Indexes a transcriptome file
2. Performs quality controls
3. Performs quantification
4. Creates a MultiqQC report

### Step 1 - define the pipeline parameters

The script `script1.nf` defines the pipeline input parameters. Run it by using the
following command:

```
./nextflow run script1.nf
```

Try to specify a different input parameter, for example:

```
./nextflow run script1.nf --reads this/and/that
```

#### Exercise 1.1

Modify the `script1.nf` by adding a fourth parameter named `outdir` and set it to a default path
that will be used as the pipeline output directory.

#### <a name="ex-1.2"></a>  Exercise 1.2

Modify the `script1.nf` to print all the pipeline parameters using `log.info` instead
of the `println` command and a [multiline string](https://www.nextflow.io/docs/latest/script.html#multi-line-strings)
statement.  

Tip: see an example [here](https://github.com/nextflow-io/rnaseq-nf/blob/42974a2/main.nf#L34-L40).

#### Recap

In this step you have learned:

1. How to define parameters in your pipeline script
2. How to pass parameters by using the command line
3. The use of `$var` and `${var}` variable placeholders
4. How to use multiline strings
5. How to use `log.info` to report values


### Step 2 - Create transcriptome index file

Nextflow allows the execution of any command or user script by using a `process` definition.

A process is defined by providing three main declarations:
the process [inputs](https://www.nextflow.io/docs/latest/process.html#inputs),
the process [outputs](https://www.nextflow.io/docs/latest/process.html#outputs)
and finally the command [script](https://www.nextflow.io/docs/latest/process.html#script).

The second example `script2.nf` adds the `index` process. Open it to see how the process is defined.

It takes the transcriptome file as input and creates the transcriptome index by using the `salmon` tool.

Note how the input declaration defines a `transcriptome` variable in the process context
that is used in the command script to reference that file in the Salmon command line.

Finally, to trigger the execution of the process the workflow declaration is needed. The `workflow` keyword
enables the composition of several processes and operators in a sub-workflow.

Try to run it by using the command:

```
./nextflow run script2.nf
```

The execution will fail because Salmon is not installed in your environment.

Add the command line option `-with-singularity` to launch the execution through a Singularity container
as shown below:

```
./nextflow run script2.nf -with-singularity
```

This time it works because it uses the Singularity container `quay.io/nextflow/rnaseq-nf:latest` defined in the
`nextflow.config` file.

In order to avoid to add the option `-with-singularity` add the following line in the `nextflow.config` file:

```
singularity.enabled = true
```

#### Exercise 2.1

Enable the Singularity execution by default adding the above setting in the `nextflow.config` file.

#### Exercise 2.2

Print the output of the `index_ch` channel by using the [view](https://www.nextflow.io/docs/latest/operator.html#view) operator.

#### Exercise 2.3

Use the command `tree work` to see how Nextflow organises the process work directory.
 
#### Recap

In this step you have learned:

1. How to define a process executing a custom command
2. How process inputs are declared
3. How process outputs are declared
4. How to access the number of available CPUs
5. How to print the content of a channel


### Step 3 - Collect read files by pairs

This step shows how to match *read* files into pairs, so they can be mapped by *Salmon*.

Edit the script `script3.nf` and add the following statement as the last line:

```
read_pairs_ch.view()
```

Save it and execute it with the following command:

```
./nextflow run script3.nf
```

It will print an output similar to the one shown below:

```
[gut, [/.../data/ggal/gut_1.fq, /.../data/ggal/gut_2.fq]]
```

The above example shows how the `read_pairs_ch` channel emits tuples composed by
two elements, where the first is the read pair prefix and the second is a list
representing the actual files.

Try it again specifying different read files by using a glob pattern:

```
./nextflow run script3.nf --reads 'data/ggal/*_{1,2}.fq'
```

#### Exercise 3.1

Use the [set](https://www.nextflow.io/docs/latest/operator.html#set) operator in place
of `=` assignment to define the `read_pairs_ch` channel.

#### Exercise 3.2

Use the `checkIfExists` for the [fromFilePairs](https://www.nextflow.io/docs/latest/channel.html#fromfilepairs) method to make sure it returns some file pairs.


#### Recap

In this step you have learned:

1. How to use `fromFilePairs` to handle read pair files.
2. How to use the `set` operator to define a new channel variable.
3. How to use the `checkIfExists` option.


### Step 4 - Perform expression quantification

The script `script4.nf` adds the `quantification` process.

Note that the workflow concatenates both processes by channeling the `index` output into the `quantification` process.  

Also note that the second input of the `quantification` process is declared as a `tuple` composed by two elements:
the `pair_id` and the `reads` in order to match the structure of the items emitted
by the `read_pairs_ch` channel.

Execute it by using the following command:

```
./nextflow run script4.nf -resume
```

You will see the execution of the `quantification` process.

The `-resume` option causes the execution of any step that has been already processed to be skipped.

Try to execute it with more read files as shown below:

```
./nextflow run script4.nf -resume --reads 'data/ggal/*_{1,2}.fq'
```

You will notice that the `quantification` process is executed more than
one time.

Nextflow parallelizes the execution of your pipeline simply by providing multiple input data
to your script.


#### Exercise 4.1

Add a [publishDir](https://www.nextflow.io/docs/latest/process.html#publishdir) directive
to the `quantification` process to store the process results in a directory of your choice.

#### Recap

In this step you have learned:
 
1. How to connect two processes
2. How to resume the script execution skipping already computed steps
3. How to use the `publishDir` to store process results in a path of your choice


### Step 5 - Quality control 

This step implements a quality control of your input reads. The inputs are the same
read pairs which are provided to the `quantification` steps

You can run it by using the following command:

```
./nextflow run script5.nf -resume 
```

<!-- The script will report the following error message:

```
Channel `read_pairs_ch` has been used twice as an input by process `fastqc` and process `quantification`
``` -->

#### Exercise 5.1

Use the [tag](https://www.nextflow.io/docs/latest/process.html?highlight=tag#tag) directive to declare a custom label that will be shown in the different logs.
#### Recap

In this step you have learned:

1. How to use the `tag` directive to associate each process execution with a custom label.

### Step 6 - MultiQC report

This step collects the outputs from the `quantification` and `fastqc` steps to create
a final report by using the [MultiQC](http://multiqc.info/) tool.
 

Execute the script with the following command:

```
./nextflow run script6.nf -resume --reads 'data/ggal/*_{1,2}.fq' 
```

It creates the final report in the `results` folder in the current work directory.

In this script note the use of the [mix](https://www.nextflow.io/docs/latest/operator.html#mix)
and [collect](https://www.nextflow.io/docs/latest/operator.html#collect) operators chained
together to get all the outputs of the `quantification` and `fastqc` process as a single
input.


#### Recap

In this step you have learned:

1. How to collect many outputs to a single input with the `collect` operator
2. How to `mix` two channels in a single channel
3. How to chain two or more operators togethers


### Step 7 - Handle completion event

This step shows how to execute an action when the pipeline completes the execution.

Note that Nextflow processes define the execution of *asynchronous* tasks i.e. they are not
executed one after another as they are written in the pipeline script as it would happen in a
common *imperative* programming language.

The script uses the `workflow.onComplete` event handler to print a confirmation message
when the script completes.

Try to run it by using the following command:

```
./nextflow run script7.nf -resume --reads 'data/ggal/*_{1,2}.fq'
```

#### Bonus!

Send a notification email when the workflow execution completes using the `-N <email address>`
command line option. Note: this requires the configuration of a SMTP server in nextflow config
file. See [mail documentation](https://www.nextflow.io/docs/latest/mail.html#mail-configuration)
for details.


### Step 8 - Custom scripts

Real world pipelines use a lot of custom user scripts (BASH, R, Python, etc). Nextflow
allows you to use and manage all these scripts in consistent manner. Simply put them
in a directory named `bin` in the pipeline project root. They will be automatically added
to the pipeline execution `PATH`.

For example, create a file named `fastqc.sh` with the following content:

```
#!/bin/bash
set -e
set -u

sample_id=${1}
reads=${2}

mkdir fastqc_${sample_id}_logs
fastqc -o fastqc_${sample_id}_logs -f fastq -q ${reads}
```

Save it, give execute permission and move it in the `bin` directory as shown below:

```
chmod +x fastqc.sh
mkdir -p bin 
mv fastqc.sh bin
```

Then, open the `script7.nf` file and replace the `fastqc` process' script with 
the following code:

```
  script:
    """
    fastqc.sh "$sample_id" "$reads"
    """
```

Run it as before:

```
./nextflow run script7.nf -resume --reads 'data/ggal/*_{1,2}.fq'
```

#### Recap

In this step you have learned:

1. How to write or use existing custom scripts in your Nextflow pipeline.
2. How to avoid the use of absolute paths by having your scripts in the `bin/` project folder.

### Step 9 - Modularization with DSL2

Nextflow allows the definition of modules of tasks and sub-workflows. The resulting modules and sub-workflows can be then imported into another Nextflow script using the `include` declaration.

The folder `modularised_pipeline` defines the same workflow we used in the previous examples but
structuring modules and sub-workflows in different files.

Modules are declared in the `modules` directory in several files and define the same processes we used in the previous examples. Note, that one file can include more than one process (module) declaration, as in this [example](https://github.com/cbcrg/nf-phdcourse22/blob/2540e3155105792c6386394712174ad13ddb4415/modularised_pipeline/modules/salmon_modules.nf#L1-L37).

The two `Salmon` processes are then orchestrated in a sub-workflow named `salmon_index_quantification.nf` that can be found in the workflows directory.

Both modules and sub-workflows are included in the script `rnatoy.nf` which declares the workflow logic
to be executed.

Also, note that parameters have been now defined in the configuration file

You can run this example with the following commands:

```
cd modularised_pipeline
./nextflow run rnatoy.nf
```

#### Exercise 9.1

Modify the `index` process in `modules/salmon_modules.nf` to emit a named output. Then modify the workflow declaration in the `workflows/salmon_index_quantification.nf` accordingly to reference the output channel. See Nextflow [documentation](https://www.nextflow.io/docs/latest/dsl2.html#process-named-output) for details.

#### Exercise 9.2

Think how `rnatoy.nf` could be modified to be included as a sub-workflow in another script.

Note: Implicit defined workflows consist of a workflow definition that does not declare any name. 
When a workflow is meant to be both included as a sub-workflow or run as an application script it should
include both a named workflow and a implicit declaration, see [here](https://www.nextflow.io/docs/latest/dsl2.html#workflow-entrypoint).

#### Recap

In this step you have learned:

1. How to structure a Nextflow pipeline in a modularised manner. 
2. How to import modules and sub-workflows in a script.
3. How to declare named process outputs.
4. How to write a workflow script that can be used either as library module or as an application script.

### Step 10 - Executors

Real world genomic application can spawn the execution of thousands of jobs. In this
scenario a batch scheduler is commonly used to deploy a pipeline in a computing cluster,
allowing the execution of many jobs in parallel across many computing nodes.

Nextflow has built-in support for the most common used batch schedulers such as Univa Grid Engine
and SLURM and [others](https://www.nextflow.io/docs/latest/executor.html).

To run your pipeline with a batch scheduler modify the `nextflow.config` file specifying
the target executor and the required computing resources if needed. For example:

```
process.executor = 'slurm'
process.queue = 'short'
process.memory = '10 GB'
process.time = '30 min'
process.cpus = 4
```

The above configuration specifies the use of the SLURM batch scheduler to run the
jobs spawned by your pipeline script. Then it specifies to use the `short` queue (partition),
10 gigabyte of memory and 4 CPUs per job, and each job can run for no more than 30 minutes.

Note: the pipeline must be executed in a shared file system accessible to all the computing
nodes.

To run your pipeline on the CRG cluster remember that the CRG has its own executor, hence you will need something like the snippet below:

```
process.executor = 'crg'
process.queue = 'cn-el7'
```

#### Exercise 10.1

Print the head of the `.command.run` script generated by Nextflow in the task work directory
and verify it contains the SLURM `#SBATCH` directives for the requested resources.

#### Exercise 10.2

Modify the configuration file to specify different resources requested for
the `quantification` process.

Tip: see the [process](https://www.nextflow.io/docs/latest/config.html#scope-process) documentation for an example.

#### Recap

In this step you have learned:

1. How to deploy a pipeline in a computing cluster.
2. How to specify different computing resources for different pipeline processes.

### Step 11 - Use configuration profiles

The Nextflow configuration file can be organised in different profiles
to allow the specification of separate settings depending on the target execution environment.

For the sake of this tutorial modify the `nextflow.config` as shown below:

```
profiles {
  standard {
    process.container = 'quay.io/nextflow/rnaseq-nf:latest'
    singularity.autoMounts = true
    singularity.enabled = true
  }
  
  cluster {
    process.executor = 'slurm'
    process.queue = 'short'
    process.memory = '10 GB' 
    process.time = '30 min'
    process.cpus = 8
  }

  batch {
    process.container = 'quay.io/nextflow/rnaseq-nf:latest' 
    process.executor = 'awsbatch'
    process.queue = 'nextflow-ci'
    workDir = 's3://nextflow-ci/work'
    aws.region = 'eu-west-1'
    aws.batch.cliPath = '/home/ec2-user/miniconda/bin/aws'
  }
}
```

The above configuration defines three profiles: `standard`, `cluster` and `batch`. The name of the
profile to use can be specified when running the pipeline script by using the `-profile` option. For example:

```
./nextflow run script7.nf -profile cluster
```

The profile `standard` is used by default if no other profile is specified by the user. Also, note that two profiles can be used simultaneously by providing them separated by a comma `-profile standard,cluster`.


#### Recap

In this step you have learned:

1. How to organise your pipeline configuration in separate profiles


### Step 12 - Run a pipeline from a GitHub repository

Nextflow allows the execution of a pipeline project directly from a GitHub
repository (or similar services eg. BitBucket and GitLab).

This simplifies the sharing and the deployment of complex projects and tracking changes in
a consistent manner.

The following GitHub repository hosts a complete version of the workflow introduced in this
tutorial:

https://github.com/nextflow-io/rnaseq-nf

You can run it by specifying the project name as shown below:

```
./nextflow run nextflow-io/rnaseq-nf -with-singularity
```

It automatically downloads it and store in the `$HOME/.nextflow` folder.

Use the command `info` to show the project information, e.g.:

```
./nextflow info nextflow-io/rnaseq-nf
```

Nextflow allows the execution of a specific *revision* of your project by using the `-r`
command line option. For Example:

```
./nextflow run nextflow-io/rnaseq-nf -r dsl2
```

Revisions are defined by using Git tags or branches defined in the project repository.

This allows a precise control of the changes in your project files and dependencies over time.


### Step 13 - Conda/Bioconda packages

Conda is popular package and environment manager. The built-in support for Conda
allows Nextflow pipelines to automatically create and activate the Conda
environment(s) given the dependencies specified by each process.

To use a Conda environment with Nextflow specify it as a command line option
as shown below:

```
./nextflow run script7.nf -with-conda env.yml
```

The use of a Conda environment can also be provided in the configuration file by
adding the following setting in the `nextflow.config` file:

```
process.conda = "env.yml"
```

See the [Conda](https://www.nextflow.io/docs/latest/conda.html) section
in the Nextflow documentation for details.

### Step 14 - Metrics and reports

Nextflow is able to produce multiple reports and charts providing several runtime metrics
and execution information.

Run the [rnaseq-nf](https://github.com/nextflow-io/rnaseq-nf) pipeline
previously introduced as shown below:

```
./nextflow run rnaseq-nf -with-singularity -with-report -with-trace -with-timeline -with-dag dag.png
```

The `-with-report` option enables the creation of the workflow execution report. Open
the file `report.html` with a browser to see the report created with the above command.

The `-with-trace` option enables the creation of a tab separated file containing runtime
information for each executed task. Check the content of the file `trace.txt` for an example.

The `-with-timeline` option enables the creation of the workflow timeline report showing
how processes where executed along time. This may be useful to identify the most time consuming
tasks and bottlenecks. See an example at [this link](https://www.nextflow.io/docs/latest/tracing.html#timeline-report).

Finally the `-with-dag` option enables to rendering of the workflow execution direct acyclic graph
representation. Note: this feature requires the installation of [Graphviz](http://www.graphviz.org/) in your computer.
See [here](https://www.nextflow.io/docs/latest/tracing.html#dag-visualisation) for details.

Note: runtime metrics may be incomplete for short running tasks as in the case of this tutorial.

### Step 15 - Run in the cloud using AWS Batch

The built-in support for [AWS Batch](https://aws.amazon.com/batch/) allows the execution of your workflow scripts
by only changing a few settings in the `nextflow.config` file. For example:

```
    process.container = 'quay.io/nextflow/rnaseq-nf:latest'
    process.executor = 'awsbatch'
    process.queue = 'nextflow-ci'
    workDir = 's3://nextflow-ci/work'
    aws.region = 'eu-west-1'
    aws.batch.cliPath = '/home/ec2-user/miniconda/bin/aws'
```

A S3 bucket must be provided by using the `workDir` configuration setting. Also the name of a queue
previously created in the AWS Batch environment needs to be specified using the `process.queue` setting.

See the [AWS Batch documentation](https://www.nextflow.io/docs/latest/awscloud.html#id2) for details.

## Docker hands-on

Get practice with basic Docker commands to pull, run and build your own containers.
 
A container is a ready-to-run Linux environment which can be executed in an isolated
manner from the hosting system. It has own copy of the file system, processes space,
memory management, etc.
 
Containers are a Linux feature known as *Control Groups* or [Ccgroups](https://en.wikipedia.org/wiki/Cgroups)
introduced with kernel 2.6.

Docker adds to this concept an handy management tool to build, run and share container images.

These images can be uploaded and published in a centralised repository know as
[Docker Hub](https://hub.docker.com), or hosted by other parties like for example [Quay](https://quay.io).


### Step 1 - Run a container

Run a container is easy as using the following command:

```
docker run <container-name>
```

For example:

```
docker run hello-world
```

### Step 2 - Pull a container

The pull command allows you to download a Docker image without running it. For example:

```
docker pull debian:jessie-slim
```

The above command download a Debian Linux image.


### Step 3 - Run a container in interactive mode

Launching a BASH shell in the container allows you to operate in an interactive mode
in the containerised operating system. For example:

```
docker run -it debian:jessie-slim bash
```

Once launched the container you will noticed that's running as root (!).
Use the usual commands to navigate in the file system.

To exit from the container, stop the BASH session with the exit command.

### Step 4 - Your first Dockerfile

Docker images are created by using a so called `Dockerfile` i.e. a simple text file
containing a list of commands to be executed to assemble and configure the image
with the software packages required.

In this step you will create a Docker image containing the Salmon tool.


Warning: the Docker build process automatically copies all files that are located in the
current directory to the Docker daemon in order to create the image. This can take
a lot of time when big/many files exist. For this reason it's important to *always* work in
a directory containing only the files you really need to include in your Docker image.
Alternatively you can use the `.dockerignore` file to select the path to exclude from the build.

Then use your favourite editor eg. `vim` to create a file named `Dockerfile` and copy the
following content:

```
FROM debian:jessie-slim

LABEL maintainer="Your name <youremail@mailprovider.com>"
LABEL description="Your description"
LABEL label1="your label"

RUN apt-get update && apt-get install -y curl cowsay 

ENV PATH=$PATH:/usr/games/
   
```

When done save the file.


### Step 5 - Build the image  

Build the Docker image by using the following command:

```
docker build -t my-image .
```

Note: don't miss the dot in the above command. When it completes, verify that the image
has been created listing all available images:

```
docker images
```

You can try your new container by running this command:

```
docker run my-image cowsay Hello Docker!
```

### Step 6 - Add a software package to the image

Add the Salmon package to the Docker image by adding to the `Dockerfile` the following snippet:

```
RUN curl -sSL https://github.com/COMBINE-lab/salmon/releases/download/v0.8.2/Salmon-0.8.2_linux_x86_64.tar.gz | tar xz \
 && mv /Salmon-*/bin/* /usr/bin/ \
 && mv /Salmon-*/lib/* /usr/lib/
```

Save the file and build again the image with the same command as before:

```
docker build -t my-image .
```

You will notice that it creates a new Docker image with the same name *but* with a
different image ID.

### Step 7 - Run Salmon in the container

Check that everything is fine running Salmon in the container as shown below:

```
docker run my-image salmon --version
```

You can even launch a container in an interactive mode by using the following command:

```
docker run -it my-image bash
```

Use the `exit` command to terminate the interactive session.


### Step 8 - File system mounts

Create an genome index file by running Salmon in the container.

Try to run Salmon in the container with the following command:

```
docker run my-image \
  salmon index -t $PWD/data/ggal/transcriptome.fa -i index
```

The above command fails because Salmon cannot access the input file.

This happens because the container runs in a complete separate file system and
it cannot access the hosting file system by default.

You will need to use the `--volume` command line option to mount the input file(s) eg.

```
docker run --volume $PWD/data/ggal/transcriptome.fa:/transcriptome.fa my-image \
  salmon index -t /transcriptome.fa -i index
```

An easier way is to mount a parent directory to an identical one in the container,
this allows you to use the same path when running it in the container eg.

```
docker run --volume $HOME:$HOME --workdir $PWD my-image \
  salmon index -t $PWD/data/ggal/transcriptome.fa -i index
```

### <a name="DockerHub"></a> Step 9 - Upload the container in the Docker Hub (bonus)

Publish your container in the Docker Hub to share it with other people.

Create an account in the https://hub.docker.com web site. Then from your shell terminal run
the following command, entering the user name and password you specified registering in the Hub:

```
docker login
```

Tag the image with your Docker user name account:

```
docker tag my-image <user-name>/my-image
```

Finally push it to the Docker Hub:

```
docker push <user-name>/my-image
```

After that anyone will be able to download it by using the command:

```
docker pull <user-name>/my-image
```

Note how after a pull and push operation, Docker prints the container digest number e.g.

```
Digest: sha256:0ac11ff903d39ad7db18e63c8958fb11864192840b3d9ece823007a54f3703e0
Status: Downloaded newer image for nextflow/rnaseq-nf:latest
```

This is a unique and immutable identifier that can be used to reference container image
in a univocally manner. For example:

```
docker pull nextflow/rnaseq-nf@sha256:0ac11ff903d39ad7db18e63c8958fb11864192840b3d9ece823007a54f3703e0
```

## Where to get help

* [Nextflow documentation](http://docs.nextflow.io) - The Nextflow docs home.
* [Nextflow patterns](https://github.com/nextflow-io/patterns) - A collection of Nextflow implementation patterns.
* [Nextflow gitter](https://gitter.im/nextflow-io/nextflow) - Nextflow channel on Gitter where you can post your questions.

## More resources

* [CalliNGS-NF](https://github.com/CRG-CNAG/CalliNGS-NF) - An Variant calling pipeline implementing GATK best practices.
* [nf-core](http://nf-co.re/) - A community collection of production ready omics pipelines.
