## Coding the `gateway-service`
As a recap, recall that the `gateway-service` will be our microservice that routes the user's request to the other microservices. This is the "gateway" that requests have to pass through before reaching the interal services.

### Setting Up
Create a new folder named `gateway-service` inside of your `services` directory.

Then, run the below commands to initialize a new `package.json` and install the needed packages for this service.

```
npm init -y
npm i express
npm i multer
npm i node-fetch
npm i form-data
npm i dotenv
```

Create a new file in the current directory named `index.js`: this is where all of our code for the microservice will go! Before we can start, let's import the npm packages we just installed.

```js
const express = require('express')
const multer = require('multer')
const FormData = require('form-data')
const fetch = require("node-fetch")
```

### Configuring the Express.JS NodeJS Server
First, define two variables, `upload` and `app`, to initialize our `express` instance and `multer` middleware.
> Middleware, like `multer`, is used to accept different types of requests from clients. In this case, `multer` will help us parse `multipart/form` data.

```js
const upload = multer()
const app = express()
```

Create a `router` object that will "route" requests to specific parts of our application and pass it into an `app.use()` function. This will make more sense once we begin considering what kinds of requests we may receive!
```js
var router = express.Router();
app.use('/', router)
```
Below that, we'll add a constant that defines what port the server should run on. Then, tell the `express` application to listen on that port.
```js
const PORT = 4444
app.listen(PORT, () => {
    console.log(`API Gateway started on port ${PORT}`)
})
```

### Considering User Input
Since this microservice is what users will be working with, we have to consider what functionality they will be expecting.

**Our "tinyhats" application should:**
1. Allow users to GET the root URL "/" and get a random hat with a default picture.
2. Allow users to GET a directory (ex: "/cat-ears") and get the specified hat ("cat-ears") with a default picture.
3. Allow users to POST their own image with the root URL "/" and get a random hat.
4. Allow users to POST their own image with a specific directory (ex: "/spicy") and get the specified hat ("spicy").

> Read more about [GET and POST requests](https://lazaroibanez.com/difference-between-the-http-requests-post-and-get-3b4ed40164c1)!

#### Example
John wants to put cat ears on a picture of his friend. He should be able to make a POST request to `http://sampledomain.com/cat-ears` with his friend's picture to get the correct result.

### "GET"ting Started
Our first step will be to accept GET requests in our server.
```js
router.get('/', upload.any(), async(req, res) => {
    // We'll put some code here.
}
```
> **User Input:** Random hat, default picture.

Adding this function defines how our `express` server should react when someone makes a GET request to the `/` endpoint. To make sure it works, let's add `console.log("GET request received from /")` as a line of code inside of the `router.get()` function.
### How do you test it?
Good question. To test our node service, run this command in the current directory, `gateway-service`.
```
node index.js
```
Do you see a log saying `API Gateway started on port 4444`? 

Now, paste the below link into your browser.
```
http://localhost:4444/
```

Go back to your logs; do you see a new entry that says `GET request received from /`? If so, your `express` server is up and running so we can start the actual coding...

Placing this code in our `router.get()` that receives requests from the root directory `/`, we use `node-fetch` to make a HTTP request. 
```js
console.log("Making request to fetch-service")
console.log("Hat: random, Image: default")

// Making a request to the fetch endpoint to get a random hat with the default picture.
const getResp = await fetch(`http://${process.env.FETCH_ENDPOINT}/fetch`, {
    method: 'GET',      
});
```
Then, let's receive the image, which is sent in JSON format and return it to the user as the response of the GET request.
```js
var result = await fetchResp.json()
res.send({result}) 
```

Now, let's account for the other possibility for a GET request: if the user wants to use the default face picture but a special hat.
> **User input:** Specific hat, default picture 
```js
router.get('/:apiName', upload.any(), async (req, res) => {
  let style = req.params.apiName;

  console.log("Making request to fetch-service")
  console.log(`Hat: ${style}, Image: default`)
})
```
Our new `router.get()` function looks similar, but not exactly like the previous one. This time, we'll be accessing the information users append to the end of the URL as a directory name.

To see it in action, start your server using `node index.js` and paste this link in your browser to make a GET request: `http://localhost:4444/hatsarecool`. You should see a similar output in your console:
```
Making request to fetch-service
Hat: hatsarecool, Image: default
```
Magic! Whatever you put behind the domain as a directory (`/hatsarecool`) will be captured by our code.

Next, create a `.env` file in the directory of your `gateway-service`. Put the below content in the file so we'll be able to make requests to the `fetch-service`.
```
FETCH_ENDPOINT=localhost:1337
```
Below, we'll add some more code to the `router.get()` function to make a request to the `fetch-service` and get a default image with the user's specified hat.
```js
const hatResp = await fetch(`http://${process.env.FETCH_ENDPOINT}/fetch?style=${style}`, {
    method: 'GET',      
});
```
Recall that the `fetch-service` sends back a 400 error code if something goes wrong. We'll need to access that from the response and send it over as the response to the GET request along with the image!
```js
let responseCode = hatResp.status

var result = await hatResp.json()
res.status(responseCode).send({result}) 
```

### Testing... 1, 2, 3
Before we begin tackling the possible POST requests we may receive, let's make sure everything can work together.
> **Tip:** Try opening multiple terminals to start each service.
For `gateway-service` and `fetch-service` run: `node -r dotenv/config index.js`.
For `manipulation-service` run: `node index.js`.

Place `http://localhost:4444/cat-ears` in your browser to make a GET request with a specific hat, `cat-ears`. Do you get back the expected result? You can try converting the base64 to an image with [this website](https://codebeautify.org/base64-to-image-converter).

### The Final Stretch: POST Requests
Since we will need to create a multipart-formdata request more than once in order to send the user's custom image, let's write a function named `customHat()` to make our code more efficient.

> **Tip:** This will be an `async` function since it makes an asychronous request to the `fetch-service`.

The first thing to notice is that we are taking in two parameters: the `req` object to access the face image and a `url` to make a request to. Next, we'll create a form data object, generate headers, and append the user's uploaded face file to it.
```js
async function customHat(req, url) {
    let formData = new FormData()
    const formHeaders = formData.getHeaders();

    formData.append('file', req.files[0].buffer, {filename: "face", data: req.files[0].buffer})
}
```
With all of the components of the multipart request ready, we can use `node-fetch` to send a POST request. After receiving the results in JSON, we will return it from the function.
```js
const hatResp = await fetch(url, {
    method: 'POST',
    body: formData,
    headers: {
    ...formHeaders,
    },  
});

var result = await hatResp.json()
return result
```

Now that we've written `customHat()` which does most of the work for us, handling inputs are relatively easy.
> **User Input:** Random hat, custom picture

When the user does not specify a specific hat they want but does upload an image through a POST request, we'll send the `req` object and the correct URL to `customHat()`. After we receive the response, we'll send the new picture to respond to the POST request. 
```js
router.post('/', upload.any(), async (req, res) => {
    console.log("Making request to fetch-service")
    console.log("Hat: random, Image: custom")
    let result = await customHat(req, `http://${process.env.FETCH_ENDPOINT}/fetch`)
    res.send({result}) 
})
```

If the user does specify a hat and uploads an image through a POST request, we'll send the `req` object and the URL to `customHat()`. After we receive the response, we'll send the new picture to respond to the POST request.

> Notice one key difference, we are adding the `style` received from `/:apiName` as the parameter for the `fetch-service.
```js
router.post('/:apiName', upload.any(), async (req, res) => {
    let style = req.params.apiName

    console.log("Making request to fetch-service")
    console.log(`Hat: ${style}, Image: custom`)

    let result = await customHat(req, `http://${process.env.FETCH_ENDPOINT}/fetch?style=${style}`)
    res.send({result}) 
})
```

Make a POST request to `http://localhost:4444/spicy` with an image with a face attached in the body sent with multipart. Do you get back the expected result? You can try converting the base64 to an image with [this website](https://codebeautify.org/base64-to-image-converter).

### Dockerizing with a Whale
Now that we have a complete `express` server, let's make it into a container and deploy it to test.

#### The Dockerfile
Create a new file named `Dockerfile` in your `gateway-service` directory and place this inside:
```
FROM node:12.0-slim
COPY . .
RUN npm install
CMD [ "node", "index.js" ]
```

This set of instructions tells Docker to:
1. Get a `node` Docker image
2. Copy the `gateway-service` files inside
3. Install all the npm packages
4. Start the `express` server

#### Package it Up
> If you don't have Docker, install it [here](https://docs.docker.com/get-docker/). You'll also need a Docker Hub account can be used to login to the CLI with `docker login`.

In the same directory as the Dockerfile, run the below command, making sure to replace the [insert your username] with your own Docker Hub username.
```bash
docker buildx build --platform linux/amd64,linux/arm64 . --push -t [insert your username]/gateway-service
```
We specified a specific architecture that is compatible with Kubernetes!

### YAML, YAML, YAML.
Return to the main root directory and `cd` into the `kube` folder. There, create a file named `gateway.yaml` and enter in the below contents:

> Make sure to update the your username in the containers --> image field.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gateway-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gateway-service
  template:
    metadata:
      labels:
        app: gateway-service
    spec:
      containers:
        - name: gateway-service
          image: [your username]/gateway-service
          ports:
            - containerPort: 4444
          env:
            - name: FETCH_ENDPOINT
              value: fetch-service:80
          imagePullPolicy: Always
---
apiVersion: v1
kind: Service
metadata:
  name: gateway-service
spec:
  selector:
    app: gateway-service
  ports:
    - port: 80
      targetPort: 4444
  type: LoadBalancer
```

### Shipping it off
Return back to your root directory and get ready to test! Run the below command to update your Kubernetes cluster:
```
kubectl apply -f kube
```
Go to NewRelic One and access the logs for the `gateway-service`. Do you see a log saying `API Gateway started on port 4444`? 

Now, run the below command to get an endpoint for the `gateway-service`.
```
minikube service -n default --url gateway-service
```
Paste the link that you get into your browser.
> Example: http://127.0.0.1:8888/

Go back to your logs; do you see a new entry that says `GET request received from /`? If so, your `express` server is up and running so we can start the actual coding...