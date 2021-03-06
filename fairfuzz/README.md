## fairfuzz

https://hub.docker.com/r/zjuchenyuan/fairfuzz

Source: https://github.com/carolemieux/afl-rb

```
Current Version: 2.52b
More Versions: Ubuntu 16.04, gcc 5.4, clang 3.8
Last Update: 2017/11
Language: C
Type: Mutation Fuzzer, Compile-time Instrumentation, AFL-based
Tag: targeting rare branches
```

## Guidance

Welcome to the world of fuzzing! 
In this tutorial, we will experience a simple realistic fuzzing towards [MP3Gain](http://mp3gain.sourceforge.net/) 1.6.2.

### Step1: System configuration

```
echo "" | sudo tee /proc/sys/kernel/core_pattern
echo 0 | sudo tee /proc/sys/kernel/core_uses_pid
echo performance | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope
echo 1 | sudo tee /proc/sys/kernel/sched_child_runs_first
echo 0 | sudo tee /proc/sys/kernel/randomize_va_space
```

Error message like `No such file or directory` is fine, and you can just ignore it.

Note: 

Although not all configuration are required by this fuzzer, we provide these command in a uniform manner for consistency between different fuzzers. 

These commands may impair your system security (turning off ASLR), but not a big problem since fuzzing experiments are normally conducted in dedicated machines.

Instead of `echo core > /proc/sys/kernel/core_pattern` given by many fuzzers which still generate a core dump file when crash happens, 
here we disable core dump file generation to reduce I/O pressure during fuzzing. [Ref](http://man7.org/linux/man-pages/man5/core.5.html).

### Step2: Compile target programs

Since fairfuzz is based on AFL, this step is equal to [AFL Guidance](https://hub.docker.com/r/zjuchenyuan/afl).

```
wget https://sourceforge.net/projects/mp3gain/files/mp3gain/1.6.2/mp3gain-1_6_2-src.zip/download -O mp3gain-1_6_2-src.zip
mkdir -p mp3gain1.6.2 && cd mp3gain1.6.2
unzip ../mp3gain-1_6_2-src.zip
# build using afl-gcc
docker run --rm -w /work -it -v `pwd`:/work --privileged zjuchenyuan/afl \
    sh -c "make clean; make"
```

### Step3: Prepare Seed Files

[UNIFUZZ](https://github.com/UNIFUZZ/seeds) provides seed files with various types. Here we provides 10 mp3 seed files, to be downloaded to `seed_mp3` folder.

```
# apt install -y subversion
svn export https://github.com/UNIFUZZ/dockerized_fuzzing_examples/trunk/seed/mp3 seed_mp3
```

### Step4: Start Fuzzing

Here we assume you have built mp3gain in current folder and downloaded mp3 seed files.

```
mkdir -p output/fairfuzz
docker run -it -w /work -v `pwd`:/work --privileged  zjuchenyuan/fairfuzz \
    afl-fuzz -i seed_mp3 -o output/fairfuzz -- ./mp3gain @@
```

Example output can be found here: https://github.com/UNIFUZZ/dockerized_fuzzing_examples/tree/master/output/fairfuzz

### More Usage

See https://github.com/carolemieux/afl-rb

## Paper

ASE 2018: FairFuzz: A Targeted Mutation Strategy for Increasing Greybox Fuzz Testing Coverage [PDF](http://www.carolemieux.com/fairfuzz-ase18.pdf)