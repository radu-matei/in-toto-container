# In-Toto verifications in a container

This is a project that enables running [In-Toto][in-toto] verifications inside a Linux container.

> This project assumes familiarity with the [In-Toto Specification][in-toto-spec].

> Verification is the process by which data and metadata included in the final product is used to ensure its correctness. Verification is performed by the client by checking the supply chain layout and links for correctness, as well as by performing the inspection steps.

## Why run verifications in a container?

The main reason for providing a containerized environment for In-Toto verifications is related to dependencies - at a minimum, running an In-Toto verification [requires Python, OpenSSL, git, and pip][deps]. On top of this, individual layouts might require additional dependencies in order to execute the verification (for example, the official demo needs to execute `tar` locally). Running in a container ensures the desired dependencies and their versions are available for clients running the verifications.

Additionally, using this method, verifications can be executed on any operating system

## Requirements

We introduce the concept of a _verification image_ - a container image where verifications are performed. At a minimum, the image needs to satisfy the requirements for running the `in-toto-verify` binary, ane the existence of the `/in-toto` directory at the root of the file system. 

Such a container image can be built using the following Dockerfile:

```
FROM python:latest

RUN mkdir /in-toto
RUN pip install in-toto
```

## How does this work?

The root layout, keys, target files, and links directory are passed as arguments to this utility, they are copied inside the verification image in the `/in-toto` directory, and then the `in-toto-verify` utility is run in the container.

> Note that currently, the utility doesn't currently support passing all flags to `in-toto-verify`, and currently a single key is passed.

```
$ make e2e
$ itc verify --image radumatei/in-toto-container:v1 --layout testdata/intoto/demo.layout.template --layout-key testdata/intoto/alice.pub --links testdata/intoto/ --target testdata/intoto/foo.tar.gz
validating layout structure and signatures...
running in-toto verifications in container based on image radumatei/in-toto-container:v1...
copying file /in-toto/key.pub in container for verification...
copying file in-toto/package.2f89b927.link in container for verification...
copying file in-toto/write-code.776a00e2.link in container for verification...
copying file in-toto/foo.tar.gz in container for verification...
copying file /in-toto/layout.template in container for verification...
Loading layout...
Loading layout key(s)...
Verifying layout signatures...
Verifying layout expiration...
Reading link metadata files...
Verifying link metadata signatures...
Verifying sublayouts...
Verifying alignment of reported commands...
Verifying command alignment for 'write-code.776a00e2.link'...
Verifying command alignment for 'package.2f89b927.link'...
Verifying threshold constraints...
Skipping threshold verification for step 'write-code' with threshold '1'...
Skipping threshold verification for step 'package' with threshold '1'...
Verifying Step rules...
Verifying material rules for 'write-code'...
Verifying product rules for 'write-code'...
Verifying 'ALLOW foo.py'...
Verifying material rules for 'package'...
Verifying 'MATCH foo.py WITH PRODUCTS FROM write-code'...
Verifying 'DISALLOW *'...
Verifying product rules for 'package'...
Verifying 'ALLOW foo.tar.gz'...
Verifying 'ALLOW foo.py'...
Executing Inspection commands...
Executing command for inspection 'untar'...
Running 'untar'...
Recording materials '.'...
Running command 'tar xfz foo.tar.gz'...
Recording products '.'...
Creating link metadata...
Verifying Inspection rules...
Verifying material rules for 'untar'...
Verifying 'MATCH foo.tar.gz WITH PRODUCTS FROM package'...
Verifying 'DISALLOW foo.tar.gz'...
Verifying product rules for 'untar'...
Verifying 'MATCH foo.py WITH PRODUCTS FROM write-code'...
Verifying 'DISALLOW foo.py'...
The software product passed all verification.
```

[in-toto]: https://github.com/in-toto/in-toto
[in-toto-spec]: https://github.com/in-toto/docs/blob/master/in-toto-spec.md
[deps]: https://github.com/in-toto/in-toto#install-
