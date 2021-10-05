# Cloud Docker Intro
  <ul>
  <h3>docker run hello-world</h3>
  <p>Unable to find image 'hello-world:latest' locally<br>
latest: Pulling from library/hello-world<br>
9db2ca6ccae0: Pull complete<br>
Digest: sha256:4b8ff392a12ed9ea17784bd3c9a8b1fa3299cac44aca35a85c90c5e3c7afacdc<br>
Status: Downloaded newer image for hello-world:latest<br>
Hello from Docker!<br>
This message shows that your installation appears to be working correctly.<br>
...
  </p>
  <i>This simple container returns Hello from Docker! to your screen. While the command is simple, notice in the output the number of steps it performed. The docker daemon searched for the hello-world image, didn't find the image locally, pulled the image from a public registry called Docker Hub, created a container from that image, and ran the container for you.</i><br>
  <br>
  <i> second time you run this, the docker daemon finds the image in your local registry and runs the container from that image. It doesn't have to pull the image from Docker Hub.</i>
  <br><br>
  
  <h3>docker images</h3>
  <p>This is the image pulled from the Docker Hub public registry. The Image ID is in SHA256 hash format—this field specifies the Docker image that's been provisioned. When the docker daemon can't find an image locally, it will by default search the public registry for the image.</p>
  
  <h3>docker ps</h3>
  <p>look at the running containers by running the following command</p>
  
  <h3>docker ps -a</h3>
  <p> In order to see all containers, including ones that have finished executing, run docker ps -a</p>
  
  <p>The container Names are also randomly generated but can be specified with docker run --name [container-name] hello-world</p>
  </ul>
  
# BUILD 
  <ul>
  <h3>mkdir test && cd test</h3>
  <p>Let's build a Docker image that's based on a simple node application.</p>
  <h3>Create a Dockerfile:</h3>
  <p>cat > Dockerfile <<EOF<br>
  # Use an official Node runtime as the parent image<br>
  FROM node:6<br>
  # Set the working directory in the container to /app<br>
  WORKDIR /app<br>
  # Copy the current directory contents into the container at /app<br>
  ADD . /app<br>
  # Make the container's port 80 available to the outside world<br>
  EXPOSE 80<br>
  # Run app.js using node when the container launches<br>
  CMD ["node", "app.js"]<br>
  EOF</p><br>
  
  
  <h3>To create the node application. A simple HTTP server that listens on port 80 and returns "Hello World".</h3>
  <p>cat > app.js <<EOF<br>
  const http = require('http');<br>
  const hostname = '0.0.0.0';<br>
  const port = 80;<br>
  const server = http.createServer((req, res) => {<br>
      res.statusCode = 200;<br>
        res.setHeader('Content-Type', 'text/plain');<br>
          res.end('Hello World\n');<br>
  });<br>
  server.listen(port, hostname, () => {<br>
      console.log('Server running at http://%s:%s/', hostname, port);<br>
  });<br>
  process.on('SIGINT', function() {<br>
      console.log('Caught interrupt signal and will exit');<br>
      process.exit();<br>
  });<br>
  EOF</p>

  <h3>build the image.</h3>
  <p>docker build -t node-app:0.1 .</p>
  <i>The -t is to name and tag an image with the name:tag syntax. The name of the image is node-app and the tag is 0.1. The tag is highly recommended when building Docker images. If you don't specify a tag, the tag will default to latest and it becomes more difficult to distinguish newer images from older ones. Also notice how each line in the Dockerfile above results in intermediate container layers as the image is built.</i>
  </ul>
  
# Run
  <ul>
  <b>c=Code to run containers based on the image you built</b>
  <p>docker run -p 4000:80 --name my-app node-app:0.1</p>
  <i>The --name flag allows you to name the container if you like. The -p instructs Docker to map the host's port 4000 to the container's port 80. Now you can reach the server at http://localhost:4000. Without port mapping, you would not be able to reach the container at localhost.</i>
  
  <b> SEE RESULT in a new terminal</b>
  <p>curl http://localhost:4000</p>
  <i>The container will run as long as the initial terminal is running. If you want the container to run in the background (not tied to the terminal's session), you need to specify the -d flag.</i>
  
  <b>Run the following command to stop and remove the container</b>
  <p>docker stop my-app && docker rm my-app</p>
  
  <b>to start the container in the background</b>
  <p>docker run -p 4000:80 --name my-app -d node-app:0.1</p>
  
  <b>Look at the logs by executing</b>
  <p>docker logs [Container ID]</p>
  
  <b>If you want to follow the log's output as the container is running, use the -f option</b>
  <p>docker logs -f [Contianer_id]</p>
  </ul>
  
# Debug
  <ul>
  <b>Start an interactive Bash session inside the running container.</b>
  <p>docker exec -it [Container id] id</p>
  <i>The -it flags let you interact with a container by allocating a pseudo-tty and keeping stdin open. Notice bash ran in the WORKDIR directory (/app) specified in the Dockerfile. From here, you have an interactive shell session inside the container to debug.</i>
  <b>You can examine a container's metadata in Docker by using Docker inspect</b>
  <p>docker inspect [container_id]</p>
  <b>Use --format to inspect specific fields from the returned JSON</b>
  <p>docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAdress}}{{end}}' [container_id]</p>
  </ul>
  
# Publish
  <ul>
  <b> </b>
  <p></p>
  </ul>
  
  <ul>
  <i>Now you're going to push your image to the Google Container Registry (gcr). After that you'll remove all containers and images to simulate a fresh environment, and then pull and run your containers. This will demonstrate the portability of Docker containers.<br>
  To push images to your private registry hosted by gcr, you need to tag the images with a registry name. The format is [hostname]/[project-id]/[image]:[tag].<br>
<br>
For gcr:<br>
<br>
[hostname]= gcr.io<br>
[project-id]= your project's ID<br>
[image]= your image name<br>
[tag]= any string tag of your choice. If unspecified, it defaults to "latest".</i><br>
  <b>Find your project ID</b>
  <p>gcloud config list project</p>
  <b>docker tag node-app:0.2 gcr.io/[project-id]/node-app:0.2</b>
  <p></p>
  <b>docker push gcr.io/[project-id]/node-app:0.2</b>
  <p></p>
  <b> </b>
  <p></p>
  <b> </b>
  <p></p>
  
  </ul>
