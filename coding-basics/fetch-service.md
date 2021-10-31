## Coding the `fetch-service`
As a recap, recall that the `fetch-service` will be our microservice that procresses users' requests based on their personalized options.

### Setting Up
Create a new folder named `fetch-service` inside of your `services` directory.

Then, run the below commands to initialize a new `package.json` and install the needed packages for this service.

```
npm init -y
npm i express
npm i multer
npm i mysql2
npm i node-fetch
npm i form-data
npm i dot-env
```

Add `"type": "module",` to your `package.json` file up top so the section looks like this:
```json
  "name": "fetch-service",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "type": "module",
```
This will allow us to utilize the EJS syntax.

Create a new file in the current directory named `index.js`: this is where all of our code for the microservice will go! Before we can start, let's import the npm packages we just installed.

```js
import express from 'express'
import multer from 'multer'
import mysql from 'mysql2'
import fetch from 'node-fetch'
import FormData from 'form-data'
```
> You might notice that the syntax is slightly different - this is because we are using EJS to import modules. Certain npm packages are only compatible with this style of JS!

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
const PORT = 1337
app.listen(PORT, () => {
    console.log(`API Gateway started on port ${PORT}`)
})
```

### Considering Service Input
The `fetch-service` will be accepting requests from `gateway-service`. We have engineered this network so that the "style" of the hat will be passed in as a parameter in both GET and POST requests.

> Read more about [GET and POST requests](https://lazaroibanez.com/difference-between-the-http-requests-post-and-get-3b4ed40164c1)!

### "GET"ting Started
Our first step will be to accept GET requests in our server.
```js
router.get('/fetch', upload.any(), async(req, res) => {
    // We'll put some code here.
}
```
Adding this function defines how our `express` server should react when someone makes a GET request to the `/fetch` endpoint. To make sure it works, let's add `console.log("GET request received from /fetch")` as a line of code inside of the `router.get()` function.

### How do you test it?
Good question. To test our node service, run this command in the current directory, `fetch-service`.
```
node index.js
```
Do you see a log saying `API Gateway started on port 1337`? 

Now, paste the below link into your browser.
```
http://localhost:1337/fetch
```

Go back to your logs; do you see a new entry that says `GET request received from /fetch`? If so, your `express` server is up and running so we can start the actual coding...

### Receiving a GET request
**Our Goal:** Receive a GET request with the hat style in the `style` parameter and return the correct hat.

First, to receive a parameter, we can add a new line of code in our `router.get()` function.
> We also added a `console.log()` function for testing purposes!
```js
router.get('/fetch', upload.any(), async(req, res) => {
    let style = req.query.style
    console.log(`Hat requested with a style of: ${style}`)
}
```

**Try testing!**

Make sure your node server is running; then enter this url in your browser: `http://localhost/fetch?style=verycool`.
In your console, you should see "Hat requested with a style of: verycool" print out.

Because this is a GET request, we will provide the user with a default face image. To do so, we must download the image from an URL. Let's write a function for that!

```js
async function defaultImage() {
    // the best picture ever
    let response = await fetch("https://user-images.githubusercontent.com/69332964/128645143-86405a62-691b-4de9-8500-b9362675e1db.png",{
        method: 'GET',
    })

    // receive the response
    let imageData = await resp.arrayBuffer()
    downloadedImage = Buffer.from(imageData)

    return downloadedImage
}
```
In this function, we do two major things:
1. We make an HTTP request to an image URL to receive image data.
2. We convert it to a NodeJS buffer so we can use it in our program.

> Now, everytime this function is called, it outputs image data for our default image.

It is relatively simple to incorporate this into our code to receive a GET request. We can add `let face = await defaultImage()` to the main code. This allows us to have the image data for our default image **every time a GET request is sent.**

```js
router.get('/fetch', upload.any(), async(req, res) => {
    let style = req.query.style
    console.log(`Hat requested with a style of: ${style}`)

    let face = await defaultImage()
    let b64Result = ''
    // Just defining a variable we'll use later!
}
```

Let's create functions for this purpose since it's highly likely we'll need to use this code again. We need one named `getRandomHat()` and one for `getSpecificHat()`.

> **Why?** If there is no value for "style", we will choose a random hat for the user. If the user specifies something, we will pick that hat that they want.

### Setting up a MySQL Server
Before we can begin writing the functions, we're going to need a place to retrieve the hats from. In this case, it'll be a MySQL database which will run locally from your computer. Follow [these instructions](https://dev.mysql.com/doc/mysql-getting-started/en/#mysql-getting-started-installing) to get started by installing it.

#### Cheatsheet for MacOS Users
```
brew install mysql
mysql.server start
mysql -u root
```
With this, you should be able to login to the server and see the prompt for commands.

Once your database is up and running, we need to add hats in for us to test with and update permissions. On the MySQL prompt, run [these](https://gist.github.com/emsesc/30edbcf3d043ddd56a66218304e0ac34) SQL commands.

**Here's what it does:**
1. It creates the table and the columns we'll need to store hat data.
2. It inserts our first "cat-ears" hat and a "spicy" hat.
3. It creates an `admin` user with a password of `password` (very secure) that is accessible anywhere on your local network.

### Connecting to your MySQL instance
To create a connection to the database, place this code at the top of the file where other packages, like `multer` were defined.

```js
const HOST = process.env.HOST;
const PASSWORD = process.env.PASSWORD;

const con = mysql.createConnection({
    host: HOST,
    port: '3306',
    user: "admin",
    password: PASSWORD,
});
```

Next, create a `.env` file in the directory of your `fetch-service`. It's always good practice to store sensitive values, like database credentials, in environment variables. To define `HOST` and `PASSWORD`, put this in the `.env` file.
```
HOST=localhost
PASSWORD=password
```
### Retrieving Hats
We can now write the two functions to either get a specific style of a hat or a random hat in the databse!

**`getRandomHat()`**
> **Input:** None! **Output:** Random hat picture

Let's say a user didn't specify what kind of hat they wanted. In this case, we would need to retrieve ALL the hats from the database and randomly select one.

```js
async function getRandomHat() {
    console.log("getRandomHat() called, getting random hat!")
    // Let's list out ALL the hats!
    var sql = "SELECT * FROM main.images;";
    const results = await con.promise().query(sql)

    // We only want the list of hats, but SQL will give us more.
    let hatList = hats[0]
    console.log(hatList)

    // We'll add more here...
}
```
First, we execute a query to the connection we just established to select ALL images. Then, we access the first value of the result since that is where the hat data is located.
```js
let randNum = Math.floor(Math.random() * hatList.length)
```
Now that we have the whole list, we still need to choose a random item from it.
```js
let hatLink = hatList[randNum].base64
return Buffer.from(hatLink, "base64")
```
Adding these two lines of code to the end of the `getRandomHat()` function, we successfully access a random value of the list and get the base64. Using `Buffer.from()` we can convert the base64 to a Buffer object, which we will work with.

**`getSpecificHat(style)`**
> **Input:** Style! **Output:** Specific hat selected by user

Let's say a user DID specify a hat. In this case, we would need to retrieve the hat of their choosing.

```js
async function getSpecificHat(style) {
    console.log("getSpecificHat() called, getting a hat!")
    // Let's select the right hat using the WHERE clause
    var sql = `SELECT * FROM main.images WHERE description='${style}';`;
    const results = await con.promise().query(sql)

    // We only want the list of hats, but SQL will give us more.
    let hatList = hats[0]
    console.log(hatList)

    // We'll add more here...
}
```
First, we execute a query to the connection we just established to select a hat with the style specified by the user. Then, we access the first value of the result since that is where the hat data is located.
```js
if (hatList.length == 0){
    return null
}
```
Because it's possible that the hat requested by the user may not exist, we can return a `null` value from the function.
```js
let hatLink = hatList[0].base64
return Buffer.from(hatLink, "base64")
```
Adding these two lines of code to the end of the `getRandomHat()` function, we successfully access the chosen hat and get the base64. Using `Buffer.from()`, we can convert the base64 to a Buffer object, which we will work with.
### Talking with `manipulate-service`
Now that we have:
* Face image data
* A hat specified by `style`

We can send a request to the `manipulate-service`. Since we'll be reusing this code, it's smart to create another function named `requestManipulate()`. The input will be **face** and **hat** image data, and it will return the **face with the hat**!

To get our function started, we've begun by creating a new form data object to hold our images: face and hat. Our headers for the POST request to manipulate have also been generated.
```js
async function requestManipulate(face, hat) {
    console.log("requestManipulate() called, POSTing face and hat to manipulate endpoint.")
    // Setting up a multipart-formdata request
    let formData = new FormData()
    formData.append('file', face, {filename: "face", data: face})
    formData.append('file', hat, {filename: "hat", data: hat})
    const formHeaders = formData.getHeaders();
}
```
We will now be adding another value to the `.env` file.
```
MANIPULATE_ENDPOINT=localhost:80
```
Also, notice below that we are using the `fetch` npm package we installed earlier. We placed the `formData` object with the images in the body and 
```js
const manipulateRequest = await fetch(`http://${process.env.MANIPULATE_ENDPOINT}/manipulate`, {
    method: 'POST',
    body: formData,
        headers: {
        ...formHeaders,
        },        
});

var b64Result = await manipulateRequest.json()
return b64Result
```
Once we `await` and receive the successfully manipulated image, we will receive it as JSON data and return it from this function.
### Hat logic & putting it together
Head back to the `router.get()` function we started earlier to receive GET requests to the `fetch-service`. Using the various functions we've written, we can now piece it all together!

Let's consider the first situation. **If the user specifies a style** it will not have an `undefined` style. In this case, we should call the `getSpecificHat` function and pass in the style.
```js
if (style != undefined) {
    // Logging information
    console.log("User specified style.")
    console.log("Using default image.")

    // In that case, we should call the getSpecificHat() function.
    let hat = await getSpecificHat(style)
}
```
Now that we've received the hat, we do need to check if the output is null. If it is, that means the user's hat does not exist. Instead of returning a hat, we will return an error message.
```js
if (hat == null) {
    return res.status(400).send({
        message: 'This hat style does not exist! If you want this style - try submitting it'
        });             
}
```
If this is not the case, the code will continue to run. Since we used `getSpecificHat()` to retrieve a hat, we now have a hat and a face to send to `manipulate`.
```js
b64Result = await requestManipulate(face, hat)
res.send(b64Result)
```
Once the base64 image is received, we'll send that as the response to this GET request.

We've completed the first scenario where the user submits a style. What if they don't? **We can add on an `else` statement for this case**.
```js
else {
    console.log("User did not specify style.")
    console.log("Using default image.")

    // We're getting a random hat and sending it to manipulate
    let hat = await getRandomHat()
    b64Result = await requestManipulate(face, hat)
    res.send(b64Result)
}
```
Just like before, we get a hat, except this time it's random. Using `requestManipulate()` we can receive a manipulated image with the face and hat and send it as an output of the GET request.

### Doing it again: with POST
If the user wants to use their own face image and not our default one, they'll make a POST request with an image file in the body of the request.

Let's start out like before, retrieving the `style` parameter's value and defining the `b64Result` variable. This time, however, we'll also need to get the face image that the user uploads.
```js
router.post('/fetch', upload.any(), async(req, res) => {
    let style = req.query.style
    let face = req.files[0].buffer
    let b64Result = ''
});
```
Just like with the GET request, we will get the specified hat with the style if the paramter value is undefined. 
```js
if (style != undefined) {
    console.log("User did not specify style.")
    console.log("Using personalized image.")
    let hat = await getSpecificHat(style)
} 
```
Inside of the `if` statement, we will add another. Now that we've received the hat, we do need to check if the output is null. If it is, that means the user's hat does not exist. Instead of returning a hat, we will return an error message.
```js
if (hat == null) {
    return res.status(400).send({
        message: 'This hat style does not exist! If you want this style - try submitting it'
        });             
}
```
Let's consider the other possibility again: what if users don't specify a hat but send in an image?
```js
else {
    console.log("User did not specify style.")
    console.log("Using personalized image.")
    let hat = await getRandomHat()

    b64Result = await requestManipulate(face, hat)
}

res.send(b64Result)
```
We can simply call the `getRandomHat()` function we wrote and send the hat and face to the `requestManipulate()` function. This gives us a base64 result we can send as a response to the POST request.

### Testing Locally 2.0
Testing locally is the same as before, except you will need to make sure your `manipulate-service` and SQL server is running.

Start the `fetch-service` by running `node -r dotenv/config index.js` and make GET or POST requests to `http://localhost:1337/fetch`.

#### Examples to test out:
* Make a GET request to `http://localhost:1337/fetch?style=spicy`. Do you get a spicy hat with your default image?
* Make a POST request to `http://localhost:1337/fetch?style=cat-ears` with a body in multipart-formdata containing an image with a face. Do you get cat-ears on your image?

### Dockerizing with a Whale
Now that we have a complete `express` server, let's make it into a container and deploy it to test.

#### The Dockerfile
Create a new file named `Dockerfile` in your `fetch-service` directory and place this inside:
```
FROM node:12.0-slim
COPY . .
RUN npm install
CMD [ "node", "--experimental-modules", "index.js" ]
```

This set of instructions tells Docker to:
1. Get a `node` Docker image
2. Copy the `fetch-service` files inside
3. Install all the npm packages
4. Start the `express` server

> Notice that we added `--experimental-modules` because we are using EJS.

#### Package it Up
> If you don't have Docker, install it [here](https://docs.docker.com/get-docker/). You'll also need a Docker Hub account can be used to login to the CLI with `docker login`.

In the same directory as the Dockerfile, run the below command, making sure to replace the [insert your username] with your own Docker Hub username.
```bash
docker buildx build --platform linux/amd64,linux/arm64 . --push -t [insert your username]/fetch-service
```
We specified a specific architecture that is compatible with Kubernetes!

### YAML, YAML, YAML.
Return to the main root directory and `cd` into the `kube` folder. There, create a file named `fetch.yaml` and enter in the below contents:

> Make sure to update the your username in the containers --> image field.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fetch-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: fetch-service
  template:
    metadata:
      labels:
        app: fetch-service
    spec:
      containers:
        - name: fetch-service
          image: [your Dockerhub username]/fetch-service
          ports:
            - containerPort: 1337
          env:
            - name: HOST
              value: mysql
            - name: PASSWORD
              value: password
            - name: MANIPULATE_ENDPOINT
              value: manipulation-service:80
          imagePullPolicy: Always
---
apiVersion: v1
kind: Service
metadata:
  name: fetch-service
spec:
  selector:
    app: fetch-service
  ports:
    - port: 80
      targetPort: 1337
  type: ClusterIP
```

### Shipping it off
Return back to your root directory and get ready to test! Run the below command to update your Kubernetes cluster:
```
kubectl apply -f kube
```
Go to NewRelic One and access the logs for the `fetch-service`.
```
minikube service -n default --url fetch-service
```
Paste the link that you get along with `/fetch` at the end into your browser.
> Example: http://127.0.0.1/fetch