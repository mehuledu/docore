https://itnext.io/beginning-net-core-development-with-docker-on-linux-6595a7eebdaa

If you’ve not used docker before this can appear quite daunting however it is relatively straightforward. Each line begins with a command (FROM, COPY, RUN etc) and each command will be executed in order from top to bottom.

    FROM specifies a docker image to use, in line 1 we specify the dotnet 2.1 SDK image which we will use to build our application.
    WORKDIR specifies a working directory inside the image. We will be using /app as our working directory.
    COPY copies files from our local file system into the image. We will copy the csproj file over initially and run restore then copy all the remaining files and run dotnet publish to build our application.
    The runtime portion of the file uses a different docker base image, the aspnetcore-runtime image, it copies over all the files from the build and then defines the application entrypoint.

To use this file to build execute the command:

docker build -t core-api .

This may be quite a lengthy process but each intermediate step shows its status, you should see something like this:

If you now run:

docker image ls

This will list all the images on your system and you should see the newly built core-api:

Some interesting things to note from this step, if you look at the output from docker build you can see lots of ID’s appearing after each step. This is because docker is applying each command to your image incrementally and creating a new layer. These layers are visible using the docker history command:

docker image history core-api:latest

Also, different stages of the process can interact with each other. If you look at the first FROM statement we’ve tagged this as “build”, later in the “runtime” section we are referencing “build” in the COPY statement.

If you want to remove the image, you can use docker rm, this can take the image name or ID:

docker image rm 7c67428fb17a

If you get to the point that you’re seeing hundreds of images in docker you can ask it to automatically remove them:

docker image prune

Or even ask docker to do a full clean of images, containers and networks:

docker system prune

Deploying a Container

Now we have an image, our built application, we need to get it up and running. We do this with the docker “run” command:

docker run core-api:latest

This is the simplest way to start a container, if you then run:

docker container ls

You can see the container is up and running, but in Production mode, not Development as it was when we started it using the dotnet command. Also, if you visit http://localhost/api/Values you should get an error not the JSON response we are expecting. And look at the name docker has given our container, it’s a randomly generated name of docker’s devising, not great if we want to be able to consistently refer to our container in the future.

We can fix all these things with command-line arguments. First of all stop our container from running and remove it:

docker container stop stoic_goldstine
docker container rm stoic_goldstine

(You will need to replace stoic_goldstine with the name of your container returned from docker container ls)

Now, start your container again using this command:

docker run --name core-api --env ASPNETCORE_ENVIRONMENT=Development -p 80:80 core-api:latest

We have added 3 arguments, — name is the name of the container when it’s up and running, — env allows us to pass environment variables to the running container and, perhaps most importantly, -p allows us to map ports on the container to ports on our machine.

When this container starts you can use the ls command and see the name and port mapping we provided:

And if you go to http://localhost/api/Values you should get the JSON response expected.
Job’s done