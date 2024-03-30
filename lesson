## CMD AND ENTRYPOINT
Before we proceed to understanding the difference between CMD and ENTRYPOINT there are some concepts you must first understand. We will look into the difference between the shell form and exec form of passing commands in a Dockerfile and Docker default Command.

### SHELL FORM and EXEC FORM.
Instructions in a dockerfile can be passed either in a shell form or exec form. These forms influence how these instructions are executed in a Dockerfile.

When instructions are passed in the shell form, during execution Docker  internally calls a shell and normal shell processing occurs. But when instructions are passed in the exec form, during execution Docker directly executes the executable without invoking the shell, thus normal shell processing does not occur. 

Hence the name;
 - Shell form: The shell is invoked, normal shell processing takes place
 - Exec Form: The executable is executed without the shell being invoked, shell processing does not take place.



To further butress what is stated above we will use sample dockerfiles to illustrate the difference between:

#### SHELL FORM:

```
<instruction><command>

FROM ubuntu
RUN apt-get update -y
ENV name master shell
ENTRYPOINT echo "Hello, $name"
```
![shell form](./images/shell%20form%20image.png)

#### EXEC form
```
<instruction>["executable","param1","param2",.......]

FROM ubuntu
RUN apt-get update -y
ENV name entrypoint
ENTRYPOINT ["/bin/echo", "Hello, $name"]
```

- BUILD THE SHELL form dockerfile and name the image `test_shell`.
`docker build -t test_shell .`
![build test shell](./images/build%20test%20shell.png)

`docker run -it --name shell test_shell`
![test shell result](./images/test_shell%20result.png)

- BUILD THE EXEC FORM dockerfile and name the image `test_exec`.
`docker build -t test_exec .`
![build test exec](./images/build%20test%20exec.png)

`docker run -it --name exec test_exec`
![test exec result](./images/test_exec%20result.png)



Observe that when we run a container with the `test_shell` image it prints: `Hello, master shell` 

HOWEVER, 

when we run a container with the `test_exec` image it prints: `Hello, $name`

WHY IS THIS? 

When instruction is executed in shell form it calls the /bin/sh -c <command> under the hood and normal shell processing happens, the name variable is interpreted and value is outputed.

But in the exec form no shell is invoked and zero processing happens, don't forget shell stands for **Command-line interpreter** and the way it fucntions is that it processes commands and outputs results. So in this case with no intepreter to process the variable in the Dockerfile it just returns `$name`.


To solve this, we edit the exec form dockerfile to use `/bin/bash` as the executable, bash will be the interpreter
```
FROM ubuntu
RUN apt-get update -y
ENV name exec shell
ENTRYPOINT ["/bin/bash", "-c", "echo Hello, $name"]
```
Build an image called `test_exec2` and run a container from it. It prints: `Hello, exec shell`
![1-test_exec2 result](./images/1-test_exec2%20result.png)
![2-test_exec2 result](./images/2-test_exec2%20result.png)

Now we are familiar with shell and exec form we will move on to another concept we need to undertand before delving into CMD and ENTRYPOINT


**Container Default commands** 

Have you ever wondered why sometimes you run a container and it immediately enters an exited state? Examples the containers we ran above, run `docker ps -a`to check the state of all running containers and you will observe that they are in an exited state.

![exited state1](./images/exited%20state1.png)

To further illustrate this, if you run an ubuntu container `docker run ubuntu` it runs an instance of ubuntu container and exits immediately, if you use the command `docker ps` to see all running containers you won't find it, only when you run `$docker ps -a` to list all containers including those that have stopped you will see that the container is in an exited state. Why does this happen?

`docker run ubuntu`
![run ubuntu image1](./images/run%20ubuntu%20image1.png)

`docker ps`
![run ubuntu image2](./images/run%20ubuntu%20image2.png)

`docker ps -a`
![run ubuntu image3](./images/run%20ubuntu%20image3.png)

This happens because containers are not the same as virtual machines, containers unlike VMs do not host an OS, they share the underlying system OS. Containers are only meant to host a specific task or process

unlike VMs, containers are not meant to host an OS, they are only meant to host a specific task or process e.g host a webserver, a DB server etc. and exits when the task is completed. The container only runs as long as the process inside is still alive, if the process stops or it crashes then the containers exits.

![docker and vm difference](./images/docker%20and%20vm%20difference.png)

That means when we ran the ubuntu container, a process started and stopped hence the container exiting.

The next logical question is;
WHAT and WHO defines this process that runs by default inside the container? How does the container know to run this process by deafault when we ran the container from the ubuntu image? This is where CMD and ENTRYPOINT comes in.

To answer the question we will look at the Dockerfile of  Nginx, MySQL and ubuntu docker images on Dockerhub.

![mysql dockerfile](./images/mysql%20dockerfile.png)

![nginx dockerfile](./images/nginx%20dockerfile.png)

![ubuntu dockerfile](./images/ubuntu%20dockerfile.png)

Observe the CMD instruction, this is what defines the default task that  runs within the container when it's spun.

The dockerfile of nginx, MySQL and ubuntu reveals CMD["nginx"] CMD["mysqld_safe"] CMD["bash"] for nginx, mysql and ubuntu respectively. But this doesn't explain why the ubuntu container exited, why did it exit? We will see that below


The `nginx` and `mysqld_safe` command starts the nginx and mysql service respectively inside the container so the container doesn't exit so long as the service is still running within it but, with the ubuntu image `bash` is the default command. You see `bash` is not really a process, it is simply a shell that listens for input from the terminal and when it doesn't find a terminal it exits. 

Thus when we run the command `docker run ubuntu`to spin up a container from the ubuntu image, docker creates the container and launches the bash program and by default docker does not attach a terminal to a container that it spins up, thus the bash program fails to find a terminal and so it exits. 
When the bash program exits the container also exits since the program within it is no longer alive.

That is why the ubuntu container exited on creation. 

Now we know what the CMD instruction does, it defines the default task/process that runs when a docker container spins up.

#### Override the Default Command.

It is possible to specify a different command to run when the container runs, e.g if in the scenarion above we want to spin up an ubuntu contaienr but we don't want the default process to be `bash`, we want to specify a different command to run as the default process.

One way we can achieve this is by appending a command to the docker run command, when this is done it overrides the default command defined in the i.e the CMD instruction in the image. 

We will illustrate this below:

**Build a Docker image that has the the ping command installed**

```
FROM ubuntu
RUN apt-get update -y && apt-get install -y iputils-ping
CMD [ "bash" ] 
```
##### Build the image with the name ping_google
`docker build -t ping_google`

![build ping_google image](./images/build%20ping_google%20image.png)

##### Run a container with name `ping_google_test1` and check it's state
`docker run -d --name ping_google_test1`
![run ping google_test1](./images/run%20ping%20google_test1.png)

As expected the container exited as the `bash` could not find a terminal.

Now we will if the container still exits if we append a command to override the default bash command.

#### Run a container with the name `ping_google_test2` that pings google.com and check it's status after a few minutes


**`docker run -d --name <name of container>  <name of image>  <name of command>`**

`docker run -d --name ping_google_test2 ping_google ping google.com` 

The command above: 
- **-d**:  stands for detached mode, this sends the program to the background
- **ping ping_google_test2**: This is the name of the container we want to spin up, you can choose anything
- **ping_google**: This is the name of image to be used to spin up the container
- **ping google.com**: This is the appended command that overrides the default CMD 

![run ping google_test2](./images/run%20ping%20google_test2.png)

Observe that the container did not exit, this is because the command appended i.e `ping google.com` is still running in the background. You can bring the command to the foreground by removing the -d flag when running the command.

![bring to foreground](./images/bring%20to%20foreground.png)

We have now a method to override the default command of a container, but this method isn't always ideal as we would have to specify our desired command i.e `ping google.com` on the command line everytime we want to spin up a container, one way to workaround this is to write a new dockerfile and make our desired command the new default CMD.

We will illusrate that below with a new dockerfile, with these changes a container spun from the image created from this dockerfile will ping  `google.com`. 

```
FROM ubuntu
RUN apt-get update -y && apt-get install -y iputils-ping
CMD [ "ping", "google.com" ] 
```

Build the image
`docker build -t new_ping .`

![build new_ping](./images/build%20new_ping.png)

Run the container
`docker run --name auto_google new_ping`

![run new_ping](./images/run%20new_ping.png)


BUT, what if we want to ping a website other than `google.com`? we already hardcoded `google.com` on the dockerfile so if we wish to ping a different website we would need to run the docker run command with a new default command; `docker run --name ping_yahoo ping_google ping yahoo.com` and as we said earlier this is not ideal. 

Since we are pinging webistes, it would be better if the only thing we are passing on the commandline is the websites we want to ping. This is where the ENTRYPOINT INSTRUCTION becomes useful. We will illustrate thsi below.

### ENTRYPOINT INSTRUCTION.

The ENTRYPOINT INSTRUCTION is like the CMD instruction in that you can specify the default program that will run when the container starts, where it differs is that whatever you append on the commandline when you run the docker run command will be appended defined ENTRYPOINT command. Let's illustrate this:

We have a dockerfile below that uses the ENTRYPOINT instruction to define a default command

```
FROM ubuntu
RUN apt-get update -y && apt-get install -y iputils-ping
ENTRYPOINT [ "ping" ] 
```
`Ping` is the default program that will run within the container when it starts, and anything specified on the commandline will be appended to the ENTRYPOINT. So if decide to append `bing.com` on the commandline, the command at start up when the container is spun up will be:

```
ping goal.com
``` 
Let's illustrate it, build an image with the dockerfile above and name it ping.

`docker build -t ping`

![docker build ping](./images/docker%20build%20ping.png)

Run a container and append the website `bing.com`

`docker run --name p_g ping bing.com`

![run ping bing](./images/run%20ping%20bing.png)

You can see that with the ENTRYPOINT instruction, anything you append on the command line to the docker run command gets appended to the ENTRYPOINT command and does not override it as is the case with the CMD instruction.

Another scenario to consider is will happen if we run the docker run command as we did above but without appending a website? Once again we will illustrate this.

Spin up a container with the ping image without attaching a website

`docker run --name trial ping`

![destinatio address required](./images/destination%20address%20required.png)

We get the error `destination address required`, this is because the command at startup when the container is spun up is: ping.

So how do we rectify this error? How do we set a default value for the ping command if one was not specified on the commandline? We can use both the ENTRYPOINT and the CMD instruction. The dockerfile below illustrates how we will solve this:

```
FROM ubuntu
RUN apt-get update -y && apt-get install -y iputils-ping
ENTRYPOINT [ "ping" ] 
CMD ["yahoo.com"]
```
Build the image and name it flexible

`docker built -t flexible .`

![build flexible](./images/build%20flexible.png)
The commanbd at start-up within a container is spun from the image built with the above dockerfile, will be:

`ping yahoo.com`

Let's try this:

`docker run --name auto_flexible flexible`

![test auto flexible](./images/test%20auto_flexible.png)


That's not all, thanks to the  usage of both ENTRYPOINT and CMD means can easily override yahoo.com and ping any website we desire by simply appending the website on the commandline when we run the docker run command. e.g `docker run --name google_flexible google.com `

![test google_flexible](./images/test%20google_flexible.png)


In a netshell, both CMD and ENTRYPOINT are used to define the dafault task that runs within a container at start up.

But while the CMD command can be overriden from the commandline, it is not possible to do so with ENTRYPOINT, with ENTRYPOINT any additional param specified on the commandline is appended to the entrypoint command.

Both CMD and ENTRYPOINT are used together to make things more seemless.

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>..

