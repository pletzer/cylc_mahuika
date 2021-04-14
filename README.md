# cylc_mahuika
A collection of scripts and configuration files showing how to run jobs in parallel on NeSI's cluster

## Initial setup

Add cylc to your path
```
export PATH=/opt/nesi/share/bin:$PATH
```

## Basic steps

 1. Create a `suite.rc` file describing the dependency of jobs
 2. Register your suite
 3. Run your suite 
 4. Monitor your suite

## Minimal hello world example

We'll start with a simple, "Hello World!" example.
```
mkdir -p mysuites/hello
```

Each cylc suite expects a file `suite.rc`:
```
cat > mysuites/hello/suite.rc << EOF
[meta]
    title = "The cylc Hello World! suite"
[scheduling]
    [[dependencies]]
        graph = "hello"
[runtime]
    [[hello]]
        script = "sleep 60; echo Hello World!"
EOF
```

Register the suite
```
cylc register hello mysuites/hello
```

Run the suite
```
cylc run hello
```
Note: if you get `Host key verification failed.` you may need to first `ssh w-cylc01.maui.niwa.co.nz` and `ssh w-cylc02.maui.niwa.co.nz` to allow the connection. If you still have an issue try `cylc run --debug hello` and write down the ssh host. Try to ssh to this host, you should be able to connect to this host without typing your password.

Monitor the suite
```
cylc scan
```

## Example of two jobs followed by a termination step

The suite.rc now is
```
[meta]
    title = "The cylc Hello World! suite in parallel"
[cylc]
    [[parameters]]
        procid = 0..1
[scheduling]
    [[dependencies]]
        graph = "hello<procid> => final_step"
[runtime]
    [[hello<procid>]]
        script = "sleep 60; echo Greetings from task ${{CYLC_TASK_PARAM_procid}}"
    [[final_step]]
        script = "echo Well done!"
```
Note the procid taking values 0 and 1. The hello processes are now parametrized by `procid`. Secondly, note that task `final_step` depends on the completion of all the `hello<procid>` tasks. Naturally, the number of jobs can be increased to any number by changing line `procid = 0..1` to `procid = 0...9` for instance.  

## Submitting the jobs to the SLURM queue

In the above, the jobs are executed interactively. To submit the jobs to the SLURM queue, edit the above to become:
```
[meta]
    title = "The cylc Hello World! suite in parallel"
[cylc]
    [[parameters]]
        procid = 0..9
[scheduling]
    [[dependencies]]
        graph = "hello<procid> => final_step"
[runtime]
    [[root]]
        [[[job]]]
            batch system = slurm
            execution time limit = 01:00:00
        [[[directives]]]
            --export=NONE
            --tasks=1
            --cpus-per-task=1
    [[hello<procid>]]
        script = "sleep 60; echo Greetings from task ${{CYLC_TASK_PARAM_procid}}"
    [[final_step]]
        script = "echo Well done!"
```
Note the maximum execution time limit of 1 hour, 0 minute, 0 second. Additional SLURM directives are under `[[[directives]]]`. 

## Limiting the number of running jobs at any time

Sometimes it is desirable to liit the number of concurrently executing jobs. The following will limit the number of jobs to 4:
```
[meta]
    title = "The cylc Hello World! suite in parallel"
[cylc]
    [[parameters]]
        procid = 0..9
[scheduling]
    [[[default]]]
        # max number of concurrent jobs
        limit = 4 
    [[dependencies]]
        graph = "hello<procid> => final_step"
[runtime]
    [[root]]
        [[[job]]]
            batch system = slurm
            execution time limit = 01:00:00
        [[[directives]]]
            --export=NONE
            --tasks=1
            --cpus-per-task=1
    [[hello<procid>]]
        script = "sleep 60; echo Greetings from task ${{CYLC_TASK_PARAM_procid}}"
    [[final_step]]
        script = "echo Well done!"
```

## More documentation

https://cylc.github.io/doc/build/7.8.7/html/
