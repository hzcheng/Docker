# Docker Notes

## 1. Dockerfile

### 1.1 Format

```
# Comment
INSTRUCTION arguments
```
A `Dockerfile` must start with a `FROM` instruction. The `FROM` instruction specifies the _Base Image_ from which you are building.

Docker treats lines that begin with # as a comment, unless the line is a valid parser directive. A # marker anywhere else in a line is treated as an argument. 

### 1.2 Parser directives

Parser directives are written as a special type of comment in the form of
```
# directive=value
```
Once a comment, empty line or builder instruction has been processed, Docker no longer looks for parser directives. Therefore, all parser directives must be at the very top of a `Dockerfile`. Line continuation characters are not supported in parser directives.

#### 1.2.1 escape
The `escape` directive sets the character used to escape characters in a `Dockerfile`.
```
# escape=\
```

### 1.3 Environment replacement (ENV)
Environment variables (declared with the `ENV` statement) can also be used in certain instructions as variables to be interpreted by the `Dockerfile`. Environment variables are notated in the `Dockerfile` either with `$variable_name` or `${variable_name}`. 
- `${variable:-word}` indicates that if variable is set then the result will be that value. If variable is not set then word will be the result.
- `${variable:+word}` indicates that if variable is set then word will be the result, otherwise the result is the empty string.

### 1.4 FROM
```
FROM <image> [AS <name>]
or
FROM <image>[:<tag>] [AS <name>]
or
FROM <image>[@<digest>] [AS <name>]
```
The FROM instruction initializes a new build stage and sets the [Base Image](https://docs.docker.com/engine/reference/glossary/#base-image) for subsequent instructions.

- `ARG` is the only instruction that may precede `FROM` in the Dockerfile. 
- `FROM` can appear multiple times within a single Dockerfile to create multiple images or use one build stage as a dependency for another. Simply make a note of the last image ID output by the commit before each new `FROM` instruction. **Each `FROM` instruction clears any state created by previous instructions**.
- Optionally a name can be given to a new build stage by adding _AS name_ to the `FROM` instruction. The _name_ can be used in subsequent `FROM` and `COPY --from=<name|index>` instructions to refer to the image built in this stage.
- The `tag` or `digest` values are optional. If you omit either of them, the builder assumes a latest tag by default. The builder returns an error if it cannot find the `tag` value.
- 
#### 1.4.1 Understand how ARG and FROM interact
`FROM` instructions support variables that are declared by any `ARG` instructions that occur before the first FROM.
```
ARG  CODE_VERSION=latest
FROM base:${CODE_VERSION}
CMD  /code/run-app

FROM extras:${CODE_VERSION}
CMD  /code/run-extras
```
An ARG declared before a `FROM` is outside of a build stage, so it can’t be used in any instruction after a `FROM`. To use the default value of an `ARG` declared before the first `FROM` use an `ARG` instruction without a value inside of a build stage:
```
ARG VERSION=latest
FROM busybox:$VERSION
ARG VERSION
RUN echo $VERSION > image_version
```

### 1.5 RUN
```
RUN <command>
or
RUN ["executable", "param1", "param2"]
```
The `RUN` instruction will execute any commands in a new layer on top of the current image and commit the results. **The resulting committed image will be used for the next step in the `Dockerfile`**.

### 1.6 CMD
```
CMD ["executable","param1","param2"]
or
CMD ["param1","param2"]
or
CMD command param1 param2
```
**There can only be one `CMD` instruction in a `Dockerfile`. If you list more than one `CMD` then only the _last_ `CMD` will take effect. The main purpose of a `CMD` is to provide defaults for an executing container.**

>Note: Don’t confuse RUN with CMD. RUN actually runs a command and commits
>the result; CMD does not execute anything at build time, but specifies the
>intended command for the image.

### 1.7 LABEL
The `LABEL` instruction adds metadata to an image. To specify multiple labels, Docker recommends combining labels into a single `LABEL` instruction where possible. 
```
LABEL multi.label1="value1" \
      multi.label2="value2" \
      other="value3"
```

### 1.8 EXPOSE
```
EXPOSE <port> [<port>...]
```
The EXPOSE instruction informs Docker that the container listens on the specified network ports at runtime. EXPOSE does not make the ports of the container accessible to the host. **To do that, you must use either the -p flag to publish a range of ports or the -P flag to publish all of the exposed ports.** 

### 1.9 ENV
```
ENV <key> <value>
or
ENV <key>=<value> ...
```
For example:
```
ENV myName="John Doe" myDog=Rex\ The\ Dog \
    myCat=fluffy
```
and
```
ENV myName John Doe
ENV myDog Rex The Dog
ENV myCat fluffy
```

### 1.10 ADD
```
ADD <src>... <dest>
or
ADD ["<src>",... "<dest>"]
```
The ADD instruction copies new files, directories or remote file URLs from <src> and adds them to the filesystem of the image at the path <dest>.
The <dest> is an absolute path, or a path relative to `WORKDIR`, into which the source will be copied inside the destination container.

### 1.11 COPY
```
COPY <src>... <dest>
or
COPY ["<src>",... "<dest>"]
```
The `COPY` instruction copies new files or directories from <src> and adds them to the filesystem of the container at the path <dest>.

### 1.12 ENTRYPOINT (??)
```
ENTRYPOINT ["executable", "param1", "param2"]
or
ENTRYPOINT command param1 param2
```
An `ENTRYPOINT` allows you to configure a container that will run as an executable.

### 1.13 VOLUME
```
VOLUME ["/data"]
```
The `VOLUME` instruction creates a mount point with the specified name and marks it as holding externally mounted volumes from native host or other containers.  Refer to [Use volumes](https://docs.docker.com/engine/admin/volumes/volumes/) for more information.

### 1.14 USER
```
USER <user>[:<group>] 
or
USER <UID>[:<GID>]
```
The `USER` instruction sets the user name (or UID) and optionally the user group (or GID) to use when running the image and for any `RUN`, `CMD` and `ENTRYPOINT` instructions that follow it in the `Dockerfile`.

### 1.15 WORKDIR
```
WORKDIR /path/to/workdir
```
The `WORKDIR` instruction sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD` instructions that follow it in the `Dockerfile`. If the `WORKDIR` doesn’t exist, it will be created even if it’s not used in any subsequent `Dockerfile` instruction.



## 2. .dockerignore file
Before the docker CLI sends the context to the docker daemon, it looks for a file named `.dockerignore` in the root directory of the context. If this file exists, the CLI modifies the context to exclude files and directories that match patterns in it. This helps to avoid unnecessarily sending large or sensitive files and directories to the daemon and potentially adding them to images using `ADD` or `COPY`. For example:

```
# comment
*/temp*
*/*/temp*
temp?
```
Lines starting with ! (exclamation mark) can be used to make exceptions to exclusions. 
```
*.md
!README.md
```








































