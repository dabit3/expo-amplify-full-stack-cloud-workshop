# Full Stack Cloud App Development with Expo and AWS Amplify

In this workshop we'll learn how to build cloud-enabled mobile applications with React Native & [AWS Amplify](https://aws-amplify.github.io/).

### Topics we'll be covering:

- [Authentication](https://github.com/dabit3/aws-amplify-workshop-react-native#adding-authentication)
- [GraphQL API with AWS AppSync](https://github.com/dabit3/aws-amplify-workshop-react-native#adding-a-graphql-api-with-aws-appsync)
- [Serverless Functions](https://github.com/dabit3/aws-amplify-workshop-react-native#adding-a-serverless-function)
- [REST API with a Lambda Function](https://github.com/dabit3/aws-amplify-workshop-react-native#adding-a-rest-api)
- [Analytics](https://github.com/dabit3/aws-amplify-workshop-react-native#adding-analytics)
- [Adding Storage with Amazon S3](https://github.com/dabit3/aws-amplify-workshop-react-native#working-with-storage)
- [Multiple Serverless Environments](https://github.com/dabit3/aws-amplify-workshop-react-native#multiple-serverless-environments)
- [Removing / Deleting Services](https://github.com/dabit3/aws-amplify-workshop-react-native#removing-services)

## Getting Started - Creating the Expo app

To get started we first need to create a new Expo project using the [Expo CLI](https://docs.expo.io/), & change into the new directory.

```bash
$ expo init expo-amplify
> Choose a template: tabs

$ cd expo-amplify
$ npm install aws-amplify aws-amplify-react-native @react-native-community/netinfo uuid
$ expo install expo-image-picker
```

### Running the app

Next, run the app:

```sh
$ expo start
```

## Installing the CLI & Initializing a new AWS Amplify Project

### Installing the CLI

Next, we'll install the AWS Amplify CLI:

```bash
$ npm install -g @aws-amplify/cli
```

Now we need to configure the CLI with our credentials:

```js
$ amplify configure
```

> If you'd like to see a video walkthrough of this configuration process, click [here](https://www.youtube.com/watch?v=fWbM5DLh25U).

Here we'll walk through the `amplify configure` setup. Once you've signed in to the AWS console, continue:
- Specify the AWS Region: __your preferred region__
- Specify the username of the new IAM user: __amplify-cli-user__
> In the AWS Console, click __Next: Permissions__, __Next: Tags__, __Next: Review__, & __Create User__ to create the new IAM user. Then, return to the command line & press Enter.
- Enter the access key of the newly created user:   
  accessKeyId: __(<YOUR_ACCESS_KEY_ID>)__   
  secretAccessKey:  __(<YOUR_SECRET_ACCESS_KEY>)__
- Profile Name: __amplify-cli-user__

### Initializing A New AWS Amplify Project

> Make sure to initialize this Amplify project in the root of your new React Native application

```bash
$ amplify init

- Enter a name for the project: expoamplify
- Enter a name for the environment: dev
- Choose your default editor: (your favorite editor)
- Please choose the type of app that youre building javascript
- What javascript framework are you using react-native
- Source Directory Path: /
- Distribution Directory Path: /
- Build Command: npm run-script build
- Start Command: npm run-script start
- Do you want to use an AWS profile? Y
- Please choose the profile you want to use: amplify-workshop-user
```

Now, the AWS Amplify CLI has iniatilized a new project & you will see a couple of new files & folders: __amplify__ & __aws-exports.js__. These files hold your project configuration.

### Configuring the Expo app

The next thing we need to do is to configure our Expo application to be aware of our new AWS Amplify project. We can do this by referencing the auto-generated __aws-exports.js__ file that is now in our root folder.


To configure the app, open __App.js__ and add the following code below the last import:

```js
// App.js
import Amplify from 'aws-amplify'
import config from './aws-exports'
Amplify.configure(config)
```

Now, our app is ready to start using our AWS services.

## Adding Authentication and Profile view

To add authentication, we can use the following command:

```sh
$ amplify add auth

- Do you want to use default authentication and security configuration? Default configuration__ 
- How do you want users to be able to sign in when using your Cognito User Pool? Username (keep default) 
- Do you want to configure advanced settings? No
```

Now, we'll run the push command and the cloud resources will be created in our AWS account.

```bash
$ amplify push
```

To view the AWS services any time after their creation, run the following command:

```sh
$ amplify console
```

Next, create a new file in the *screens* folder called *ProfileScreen.js*.

In *srceens/ProfileScreen.js*, add the following code:

```js
/* screens/ProfileScreen.js */
import * as React from 'react';
import { StyleSheet, Text, Button } from 'react-native';
import { ScrollView } from 'react-native-gesture-handler';
import { withAuthenticator } from 'aws-amplify-react-native'
import { Auth } from 'aws-amplify';

function ProfileScreen() {
  async function signOut() {
    await Auth.signOut()
  }
  return (
    <ScrollView style={styles.container} contentContainerStyle={styles.contentContainer}>
      <Text>Hello from Profile</Text>
      <Button title="Sign Out" onPress={signOut} />
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fafafa',
  },
  contentContainer: {
    paddingTop: 15,
  },
  optionIconContainer: {
    marginRight: 12,
  }
});

export default withAuthenticator(ProfileScreen)
```

In this component we import two main APIs from Amplify & Amplify React Native:

*withAuthenticator* - This UI component will render an authentication flow in front of any component

*Auth* - This class will allow you to call methods to handle user identity. There are over 30 methods enabling you to do things like manually sign up or sign in a user, but in our case we are using it to call `Auth.signOut()` to sign the user out.

Next, open *navigation/BottomTabNavigator.js* and add the following:

```js
// First, import the new ProfileScreen components
import ProfileScreen from '../screens/ProfileScreen'

// Next, add another BottomTab component to hold the profile view 
 <BottomTab.Screen
  name="Profile"
  component={ProfileScreen}
  options={{
    title: 'Profile',
    tabBarIcon: ({ focused }) => <TabBarIcon focused={focused} name="md-person" />
  }}
/>
```

Now, run the app:

```sh
$ expo start
```

You should now see an authentication flow in the *Profile* tab and be able to sign up, and in to the app.

### Accessing User Data & finishing the profile view.

You can check to see if a user is signed in and, if so, retrieve the user metadata by calling the `currentAuthenticatedUser` method of the `Auth` class.

Let's continue updating the Profile view to show the user their usernamne, email, phone number, and user ID.

```js
import * as React from 'react';
import { StyleSheet, Text, Button, View } from 'react-native';
import { ScrollView } from 'react-native-gesture-handler';
import { withAuthenticator } from 'aws-amplify-react-native'
import { Auth } from 'aws-amplify';

function ProfileScreen() {
  const [user, setUser] = React.useState(null)
  React.useEffect(() => {
    checkUser()
  }, [])
  async function checkUser() {
    const user = await Auth.currentAuthenticatedUser()
    setUser(user)
  }
  async function signOut() {
    await Auth.signOut()
  }
  return (
    <ScrollView style={styles.container} contentContainerStyle={styles.contentContainer}>
      {
        user && (
          <View>
            <Text style={styles.userInfo}>Username: {user.username}</Text>
            <Text style={styles.userInfo}>Email: {user.attributes.email}</Text>
            <Text style={styles.userInfo}>Phone: {user.attributes.phone_number}</Text>
            <Text style={styles.userInfo}>User ID: {user.attributes.sub}</Text>
          </View>
        )
      }
      <Button title="Sign Out" onPress={signOut} />
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fafafa',
  },
  contentContainer: {
    paddingTop: 15,
  },
  optionIconContainer: {
    marginRight: 12,
  },
  userInfo: {
    paddingHorizontal: 15,
    paddingBottom: 10,
    fontSize: 16,
    fontWeight: 'bold'
  }
});

export default withAuthenticator(ProfileScreen)
```

## Adding complex object storage

To add storage we'll use the Amplify *storage* category:

```sh
$ amplify add storage

? Please select from one of the below mentioned services: Content
? Please provide a friendly name for your resource that will be used to label this category in the project: images
? Please provide bucket name: <your-unique-bucket-name>
? Who should have access: Auth and guest users
? What kind of access do you want for Authenticated users? create, update, read, delete
? What kind of access do you want for Guest users? read
? Do you want to add a Lambda Trigger for your S3 Bucket? N
```

## Adding the API

To add the API, run the following command:

```sh
$ amplify add api

? Please select from one of the below mentioned services: GraphQL
? Provide API name: travelapi
? Choose the default authorization type for the API: API key
? Enter a description for the API key: public
? After how many days from now the API key should expire (1-365): 365
? Do you want to configure advanced settings for the GraphQL API: Yes
? Configure additional auth types? Yes
? Choose the additional authorization types you want to configure for the API: Amazon Cognito User Pool
? Configure conflict detection? Yes
? Select the default resolution strategy Auto Merge
? Do you have an annotated GraphQL schema? No
? Do you want a guided schema creation? Yes
? What best describes your project: Single object with fields (e.g., “Todo” with ID, name, description)
? Do you want to edit the schema now? Yes
```

Update the schema with the following:

```graphql
type Post @model
  @auth(rules: [
    { allow: owner },
    { allow: public, operations: [read] }
  ])
{
  id: ID!
  name: String!
  description: String
  location: String
  image: String
}
```

Save the schema and press enter.

Next, generate the models needed for DataStore:

```sh
$ amplify codegen models
```
You now should see a *models* directory located in the *src* directory of your project.

Deploy the API and storage service with the Amplify `push` command:

```sh
$ amplify push --y
```

Now, the API is created and we can start using it.

## Adding additional screens

Next, create a couple of new files in the *screens* folder:

```sh
touch screens/CreatePostScreen.js screens/MyPostsScreen.js screens/AllPostsScreen.js
```



## Adding a Serverless Function

### Adding a basic Lambda Function

To add a serverless function, we can run the following command:

```sh
$ amplify add function
```

> Answer the following questions

- Provide a friendly name for your resource to be used as a label for this category in the project: __basiclambda__
- Provide the AWS Lambda function name: __basiclambda__
- Choose the function template that you want to use: __Hello world function__
- Do you want to access other resources created in this project from your Lambda function? __N__
- Do you want to edit the local lambda function now? __Y__

> This should open the function package located at __amplify/backend/function/basiclambda/src/index.js__.

Edit the function to look like this, & then save the file.

```js
exports.handler = (event, context, callback) => {
  console.log('event: ', event)
  const body = {
    message: "Hello world!"
  }
  const response = {
    statusCode: 200,
    body
  }
  callback(null, response)
}
```

Next, we can test this out by running:

```sh
$ amplify function invoke basiclambda
```

- Provide the name of the script file that contains your handler function: __index.js__
- Provide the name of the handler function to invoke: __handler__
- Provide the relative path to the event: __event.json__

You'll notice the following output from your terminal:

```sh
Testing function locally
event:  { key1: 'value1', key2: 'value2', key3: 'value3' }

Success!  Message:
------------------
{"statusCode":200,"body":{"message":"Hello world!"}}

Done.
Done running invoke function.
```

_Where is the event data coming from? It is coming from the values located in event.json in the function folder (__amplify/backend/function/basiclambda/src/event.json__). If you update the values here, you can simulate data coming arguments the event._

Feel free to test out the function by updating `event.json` with data of your own.

### Adding a function running an express server and invoking it from an API call (http)

Next, we'll build a function that will be running an [Express](https://expressjs.com/) server inside of it.

This new function will fetch data from a cryptocurrency API & return the values in the response.

To get started, we'll create a new function:

```sh
$ amplify add function
```

> Answer the following questions

- Provide a friendly name for your resource to be used as a label for this category in the project: __cryptofunction__
- Provide the AWS Lambda function name: __cryptofunction__
- Choose the function template that you want to use: __Serverless express function (Integration with Amazon API Gateway)__
- Do you want to access other resources created in this project from your Lambda function? __N__
- Do you want to edit the local lambda function now? __Y__

This should open the function package located at __amplify/backend/function/cryptofunction/src/index.js__. You'll notice in this file, that the event is being proxied into an express server:

```js
exports.handler = (event, context) => {
  console.log(`EVENT: ${JSON.stringify(event)}`);
  awsServerlessExpress.proxy(server, event, context);
};
```

Instead of updating the handler function itself, we'll instead update __amplify/backend/function/cryptofunction/src/app.js__ which has the actual server code we would like to be working with.

Here, in __amplify/backend/function/cryptofunction/src/app.js__, we'll add the following code & save the file:

```js
// amplify/backend/function/cryptofunction/src/app.js

// you should see this code already there 👇:
app.use(function(req, res, next) {
  res.header("Access-Control-Allow-Origin", "*")
  res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept")
  next()
});
// below the above code, add the following code 👇 (be sure not to delete any other code from this file)
const axios = require('axios')

app.get('/coins', function(req, res) {
  let apiUrl = `https://api.coinlore.com/api/tickers?start=0&limit=10`
  
  if (req && req.query) {
    // here we are checking to see if there are any query parameters, and if so appending them to the request
    const { start = 0, limit = 10 } = req.query
    apiUrl = `https://api.coinlore.com/api/tickers/?start=${start}&limit=${limit}`
  }

  axios.get(apiUrl)
    .then(response => {
      res.json({
        coins: response.data.data
      })
    })
    .catch(err => res.json({ error: err }))
})
```

In the above function we've used the __axios__ library to call another API. In order to use __axios__, we need be sure that it will be installed by updating the __package.json__ for the new function:

```sh
$ cd amplify/backend/function/cryptofunction/src

$ npm install && npm install axios

$ cd ../../../../../
```

Next, change back into the root directory.

Now we can test this function out:

```sh
$ amplify function invoke cryptofunction

? Provide the name of the script file that contains your handler function: index.js
? Provide the name of the handler function to invoke: handler
? Provide the relative path to the event: event.json
```

This will start up the node server. We can then make `curl` requests agains the endpoint:

```sh
curl 'localhost:3000/coins'
```

If we'd like to test out the query parameters, we can update the __event.json__ to simulate an API gateway event by adding the following:

```json
{
    "httpMethod": "GET",
    "path": "/coins",
    "queryStringParameters": {
        "start": "0",
        "limit": "1"
    }
}
```

Now, stop the server and invoke the function.

```sh
$ amplify function invoke cryptofunction
```

When we invoke the function these query parameters will be passed in & the http request will be made immediately.

## Adding a REST API

Now that we've created the cryptocurrency Lambda function let's add an API endpoint so we can invoke it via http.

To add the REST API, we can use the following command:

```sh
$ amplify add api
```

> Answer the following questions

- Please select from one of the above mentioned services __REST__   
- Provide a friendly name for your resource that will be used to label this category in the project: __cryptoapi__   
- Provide a path (e.g., /items): __/coins__   
- Choose lambda source __Use a Lambda function already added in the current Amplify project__   
- Choose the Lambda function to invoke by this path: __cryptofunction__   
- Restrict API access __Y__
- Who should have access? __Authenticated users only__
- What kind of access do you want for Authenticated users __read/create/update/delete__
- Do you want to add another path? (y/N) __N__     

Now the resources have been created & configured & we can push them to our account: 

```bash
$ amplify push

? Are you sure you want to continue? Y
```

### Interacting with the new API

Now that the API is created we can start sending requests to it & interacting with it.

Let's request some data from the API:

```js
// App.js
import React from 'react'
import { View, Text, StyleSheet } from 'react-native'
import { API } from 'aws-amplify'
import { withAuthenticator } from 'aws-amplify-react-native'

class App extends React.Component {
  state = {
    coins: []
  }
  async componentDidMount() {
    try {
      // to get all coins, do not send in a query parameter
      // const data = await API.get('cryptoapi', '/coins')
      const data = await API.get('cryptoapi', '/coins?limit=5&start=100')
      console.log('data from Lambda REST API: ', data)
      this.setState({ coins: data.coins })
    } catch (err) {
      console.log('error fetching data..', err)
    }
  }
  render() {
    return (
      <View>
        {
          this.state.coins.map((c, i) => (
            <View key={i} style={styles.row}>
              <Text style={styles.name}>{c.name}</Text>
              <Text>{c.price_usd}</Text>
            </View>
          ))
        }
      </View>
    )
  }
}

const styles = StyleSheet.create({
  row: { padding: 10 },
  name: { fontSize: 20, marginBottom: 4 },
})

export default withAuthenticator(App, { includeGreetings: true })
```

## Adding Analytics

To add analytics, we can use the following command:

```sh
$ amplify add analytics
```

> Next, we'll be prompted for the following:

- Provide your pinpoint resource name: __amplifyanalytics__   
- Apps need authorization to send analytics events. Do you want to allow guest/unauthenticated users to send analytics events (recommended when getting started)? __Y__   

To deploy, run the `push` command:

```sh
$ amplify push
```

### Recording events

Now that the service has been created we can now begin recording events.

To record analytics events, we need to import the `Analytics` class from Amplify & then call `Analytics.record`:

```js
import { Analytics } from 'aws-amplify'

state = {username: ''}

async componentDidMount() {
  try {
    const user = await Auth.currentAuthenticatedUser()
    this.setState({ username: user.username })
  } catch (err) {
    console.log('error getting user: ', err)
  }
}

recordEvent = () => {
  Analytics.record({
    name: 'My test event',
    attributes: {
      username: this.state.username
    }
  })
}

<Button onPress={this.recordEvent} title='Record Event' />
```

To view the analytics in the console, run the `console` command:

```sh
$ amplify console
```

In the console, click on __Analytics__, then click on __View  in Pinpoint__. In the __Pinpoint__ console, click on __events__ and then enable filters.

## Working with Storage

To add storage, we can use the following command:

```sh
amplify add storage
```

> Answer the following questions   

- Please select from one of the below mentioned services __Content (Images, audio, video, etc.)__
- Please provide a friendly name for your resource that will be used to label this category in the
 project: __rnworkshopstorage__
- Please provide bucket name: __YOUR_UNIQUE_BUCKET_NAME__
- Who should have access: __Auth users only__
- What kind of access do you want for Authenticated users?

```sh
❯◉ create/update
 ◉ read
 ◉ delete
```


```sh
amplify push
```

Now, storage is configured & ready to use.

What we've done above is created configured an Amazon S3 bucket that we can now start using for storing items.

For example, if we wanted to test it out we could store some text in a file like this:

```js
import { Storage } from 'aws-amplify'

// create function to work with Storage
addToStorage = () => {
  Storage.put('textfiles/mytext.txt', `Hello World`)
    .then (result => {
      console.log('result: ', result)
    })
    .catch(err => console.log('error: ', err));
}

// add click handler
<Button onPress={this.addToStorage} title='Add to Storage' />
```

This would create a folder called `textfiles` in our S3 bucket & store a file called __mytext.txt__ there with the code we specified in the second argument of `Storage.put`.

If we want to read everything from this folder, we can use `Storage.list`:

```js
readFromStorage = () => {
  Storage.list('textfiles/')
    .then(data => console.log('data from S3: ', data))
    .catch(err => console.log('error fetching from S3', err))
}
```

If we only want to read the single file, we can use `Storage.get`:

```js
readFromStorage = () => {
  Storage.get('textfiles/mytext.txt')
    .then(data => {
      console.log('data from S3: ', data)
      fetch(data)
        .then(r => r.text())
        .then(text => {
          console.log('text: ', text)
        })
        .catch(e => console.log('error fetching text: ', e))
    })
    .catch(err => console.log('error fetching from S3', err))
}
```

If we wanted to pull down everything, we can use `Storage.list`:

```js
readFromStorage = () => {
  Storage.list('')
    .then(data => console.log('data from S3: ', data))
    .catch(err => console.log('error fetching from S3', err))
}
```

## Multiple Serverless Environments

Now that we have our API up & running, what if we wanted to update our API but wanted to test it out without it affecting our existing version?

To do so, we can create a clone of our existing environment, test it out, & then deploy & test the new resources.

Once we are happy with the new feature, we can then merge it back into our main environment. Let's see how to do this!

### Creating a new environment

To create a new environment, we can run the `env` command:

```sh
$ amplify env add

> Do you want to use an existing environment? No
> Enter a name for the environment: apiupdate
> Do you want to use an AWS profile? Yes
> Please choose the profile you want to use: appsync-workshop-profile
```

Now, the new environment has been initialize, & we can deploy the new environment using the `push` command:

```sh
$ amplify push
```

Now that the new environment has been created we can get a list of all available environments using the CLI:

```sh
$ amplify env list
```

Let's update the GraphQL schema to add a new field. In __amplify/backend/api/RestaurantAPI/schema.graphql__  update the schema to the following:

```graphql
type Restaurant @model {
  id: ID!
  clientId: String
  name: String!
  type: String
  description: String!
  city: String!
}

type ModelRestaurantConnection {
	items: [Restaurant]
	nextToken: String
}

type Query {
  listAllRestaurants(limit: Int, nextToken: String): ModelRestaurantConnection
}
```

In the schema we added a new field to the __Restaurant__ definition to define the type of restaurant:

```graphql
type: String
```

Now, we can run amplify push again to update the API:

```sh
$ amplify push
```

To test this out, we can go into the [AppSync Console](https://console.aws.amazon.com/appsync) & log into the API.

You should now see a new API called __RestaurantAPI-apiupdate__. Click on this API to view the API dashboard.

If you click on __Schema__ you should notice that it has been created with the new __type__ field. Let's try it out.

To test it out we need to create a new user because we are using a brand new authentication service. To do this, open the app & sign up.

In the API dashboard, click on __Queries__.

Next, click on the __Login with User Pools__ link.

Copy the __aws_user_pools_web_client_id__ value from your __aws-exports__ file & paste it into the __ClientId__ field.

Next, login using your __username__ & __password__.

Now, create a new mutation & then query for it:

```graphql
mutation createRestaurant {
  createRestaurant(input: {
    name: "Nobu"
    description: "Great Sushi"
    city: "New York"
    type: "sushi"
  }) {
    id name description city type
  }
}

query listRestaurants {
  listAllRestaurants {
    items {
      name
      description
      city
      type
    }
  }
}
```

### Merging the new environment changes into the main environment.

Now that we've created a new environment & tested it out, let's check out the main environment.

```sh
$ amplify env checkout local
```

Next, run the `status` command:

```sh
$ amplify status
```

You should now see an __Update__ operation:

```
Current Environment: local

| Category | Resource name   | Operation | Provider plugin   |
| -------- | --------------- | --------- | ----------------- |
| Api      | RestaurantAPI   | Update    | awscloudformation |
| Auth     | cognito75a8ccb4 | No Change | awscloudformation |
```

To deploy the changes, run the push command:

```sh
$ amplify push
```

Now, the changes have been deployed & we can delete the `apiupdate` environment:

```sh
$ amplify env remove apiupdate

Do you also want to remove all the resources of the environment from the cloud? Y
```

Now, we should be able to run the `list` command & see only our main environment:

```sh
$ amplify env list
```

### Custom authentication strategies

To view a final solution for a custom authentication strategy, check out the __AWS Amplify React Native Auth Starter__ [here](https://github.com/aws-samples/aws-amplify-auth-starters/tree/react-native#aws-amplify-react-native-auth-starter).

> This section is an overview and is considered an advanced part of the workshop. If you are not comfortable writing a custom authentication flow, I would read through this section and use it as a reference in the future. If you'd like to jump to the next section, click [here](https://github.com/dabit3/aws-amplify-workshop-react-native#adding-a-graphql-api-with-aws-appsync).

The `withAuthenticator` component is a really easy way to get up and running with authentication, but in a real-world application we probably want more control over how our form looks & functions.

Let's look at how we might create our own authentication flow.

To get started, we would probably want to create input fields that would hold user input data in the state. For instance when signing up a new user, we would probably need 3 user inputs to capture the user's username, email, & password.

To do this, we could create some initial state for these values & create an event handler that we could attach to the form inputs:

```js
// initial state
state = {
  username: '', password: '', email: ''
}

// event handler
onChangeText = (key, value) => {
  this.setState({ [key]: value })
}

// example of usage with TextInput
<TextInput
  placeholder='username'
  value={this.state.username}
  style={{ width: 300, height: 50, margin: 5, backgroundColor: "#ddd" }}
  onChangeText={v => this.onChange('username', v)}
/>
```

We'd also need to have a method that signed up & signed in users. We can us the Auth class to do thi. The Auth class has over 30 methods including things like `signUp`, `signIn`, `confirmSignUp`, `confirmSignIn`, & `forgotPassword`. Thes functions return a promise so they need to be handled asynchronously.

```js
// import the Auth component
import { Auth } from 'aws-amplify'

// Class method to sign up a user
signUp = async() => {
  const { username, password, email } = this.state
  try {
    await Auth.signUp({ username, password, attributes: { email }})
  } catch (err) {
    console.log('error signing up user...', err)
  }
}
```

## Removing Services

If at any time, or at the end of this workshop, you would like to delete a service from your project & your account, you can do this by running the `amplify remove` command:

```sh
$ amplify remove auth

$ amplify push
```

If you are unsure of what services you have enabled at any time, you can run the `amplify status` command:

```sh
$ amplify status
```

`amplify status` will give you the list of resources that are currently enabled in your app.

## Deleting the project

To delete the entire project, run the `delete` command:

```sh
$ amplify delete
```


<!-- ### GraphQL Subscriptions

Next, let's see how we can create a subscription to subscribe to changes of data in our API.

To do so, we need to define the subscription, listen for the subscription, & update the state whenever a new piece of data comes in through the subscription.

```js
// import the subscription
import { onCreateRestaurant } from './src/graphql/subscriptions'

// define the subscription in the class
subscription = {}

// subscribe in componentDidMount
componentDidMount() {
  this.subscription = API.graphql(
    graphqlOperation(onCreateRestaurant)
  ).subscribe({
      next: eventData => {
        console.log('eventData', eventData)
        const restaurant = eventData.value.data.onCreateRestaurant
        if(CLIENTID === restaurant.clientId) return
        const restaurants = [...this.state.restaurants, restaurant]
        this.setState({ restaurants })
      }
  });
}

// remove the subscription in componentWillUnmount
componentWillUnmount() {
  this.subscription.unsubscribe()
}
``` -->
