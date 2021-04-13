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
Note: if you get `Host key verification failed.` you may need to first `ssh w-cylc01.maui.niwa.co.nz` and `ssh w-cylc02.maui.niwa.co.nz` to allow the connection. If you still have an issue try `cylc run --debug hello` and write down the ssh host. Try to ssh to this host, you should be able to connecy to this host without typing your password.

Monitor the suite
```
cylc scan
```



## Example of two jobs followed by a termination step


