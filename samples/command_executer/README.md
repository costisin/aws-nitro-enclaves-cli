# command-executer

An application that can run either as a server or a client. When running as a
server, it listens on a port for commands and executes them afterwards, sending
the output back to the client. When running as a client, it connects to the
above mentioned port, sends commands and waits for the reply (unless --no-wait
is used). The command-executer is useful for executing shell commands inside
the enclave and sending/receiving files to/from the enclave.

The command executer sample should be used for testing and debugging only,
and not in a production environment.

## Building

```
	$ cargo build --release
```

## Running

1. Build the project (see above).

2. Use the Dockerfile in resources/ either as an example or as is
and build an EIF.

```
	$ export NITRO_CLI_BLOBS=$(realpath ../../blobs/x86_64)
	$ nitro-cli build-enclave --docker-dir "./resources" --docker-uri mytag --output-file command-executer.eif
```
---
**NOTES**

* In order to build an EIF as a non-root user, that user must be able to manage
the docker daemon. Please see
[the official Docker documentation](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user)
to learn how to do that.
* These steps can either be done on your local machine or on the EC2 instance
where you are going to launch the enclave.

---

3. Copy __both__ the EIF __and__ the `command-executer` binary to the EC2
instance you are about to run an enclave on.

4. Launch an enclave with the EIF containing command-executer.

```
	$ ./nitro-cli run-enclave --cpu-count 4 --memory 2048 --eif-path command_executer.eif --enclave-cid 16
	Start allocating memory...
	Started enclave with enclave-cid: 16, memory: 2048 MiB, cpu-ids: [1, 5, 2, 6]
	{
	  "EnclaveID": "i-abc12345def67890a-enc9876abcd543210ef12",
	  "ProcessID": 12345,
	  "EnclaveCID": 16,
	  "NumberOfCPUs": 4,
	  "CPUIDs": [
	    1,
	    5,
	    2,
	    6
	  ],
	  "MemoryMiB": 2048
	}
```

5. Use the command-executer to send shell commands to the enclave

```
	$ ./command-executer run --cid 16 --port 5005 --command "whoami"
```

6. Use the command-executer to send files to the enclave (e.g. binaries you built in the instance)

```
	$ ./command-executer send-file --cid 16 --localpath "./stress-ng" --port 5005 --remotepath "/usr/bin/stress-ng"
```

## Help
Further information about the semantics of each command can be obtained by running
```
	$ ./command-executer help [<command>]
```
