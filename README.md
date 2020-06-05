# Full Stack Cloud App Development with Expo and AWS Amplify

In this workshop we'll learn how to build cloud-enabled mobile applications with React Native & [AWS Amplify](https://aws-amplify.github.io/).

The app we will be building is a basic Instagram clone that allows you to sign up, sign in, create posts, and view posts that you have created as well as others.

### Debugging

Due to subtle differences between the JavaScript execution environments on the iOS Simulator on Mac and in your remote debugger, it is recommended that you enable "remote debugging" during development if you are developing using the iOS simulator on your Macbook.

## Getting Started - Creating the Expo app

To get started we first need to create a new Expo project using the [Expo CLI](https://docs.expo.io/), & change into the new directory.

```bash
$ expo init expo-amplify
> Choose a template: tabs

$ cd expo-amplify
$ npm install aws-amplify aws-amplify-react-native @react-native-community/netinfo
$ expo install expo-image-picker
$ expo install expo-image-manipulator
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
- Please choose the type of app that youre building: javascript
- What javascript framework are you using: react-native
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

### Adding the authentication UI

Next, open *navigation/BottomTabNavigator.js*. Here, import the `withAuthenticator` component, and change the default export to be the `withAuthenticator` wrapping the main `BottomTabNavigator` component:

```js
/* navigation/BottomTabNavigator.js */

/* Import the withAuthenticator component  */
import { withAuthenticator } from 'aws-amplify-react-native'

/* Remove the default export from the main component */
function BottomTabNavigator({ navigation, route }) { /* rest of component code stays */ }

/* Create new default export */
export default withAuthenticator(BottomTabNavigator)
```

*withAuthenticator* - This UI component will render an authentication flow in front of any component

Now, run the app:

```sh
$ expo start
```

When the app loads, you should now see an authentication flow in front of the app. You should be able to sign up, sign in, and reset your password.

> Make sure that you use a real email address in order to complete the MFA required for the app sign up process.

### Adding the Profile screen

Next, create a new file in the *screens* folder called *ProfileScreen.js*.

In *srceens/ProfileScreen.js*, add the following code:

```js
/* screens/ProfileScreen.js */
import * as React from 'react';
import { StyleSheet, Text, Button, View } from 'react-native';
import { ScrollView } from 'react-native-gesture-handler';
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

export default ProfileScreen
```

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

You should see a new *Profile* tab to the bottom right when you sign in. In this tab you should 

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
    { allow: owner, operations: [create, delete, update] }
  ])
{
  id: ID!
  name: String!
  description: String
  location: String
  image: String
  owner: String
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
touch screens/CreatePostScreen.js screens/PostsScreen.js screens/MyPostsScreen.js
```

## Creating a new post

In this screen, you will be creating a component that will allow you to create a new post.

> Below this code snippet, I will walk through the main functionality.

```js
/* screens/CreatePostScreen.js */
import * as React from 'react';
import { StyleSheet, Text, ActivityIndicator, Button, View, Image, TextInput } from 'react-native';
import { ScrollView } from 'react-native-gesture-handler';
import * as ImagePicker from 'expo-image-picker';
import * as Permissions from 'expo-permissions';
import Amplify, { Storage, DataStore } from 'aws-amplify';
import * as ImageManipulator from "expo-image-manipulator";
import Constants from 'expo-constants';
import { Post } from '../src/models';

function uuid() {
  return Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15);
}

const initialFormState = {
  name: '', location: '', image: ''
}

function CreatePostScreen({ navigation }) {
  const [image, setImage] = React.useState(null)
  const [formState, setFormState] = React.useState(initialFormState)
  const [saving, setSaving] = React.useState(false)
  React.useEffect(() => {
    getPermissions()
  }, [])
  async function getPermissions() {
    if (Constants.platform.ios) {
      const { status } = await Permissions.askAsync(Permissions.CAMERA_ROLL);
      if (status !== 'granted') {
        alert('Sorry, we need camera roll permissions to make this work!');
      }
    }
  };

  async function pickImage () {
    try {
      const imageId = uuid()
      let result = await ImagePicker.launchImageLibraryAsync({
        mediaTypes: ImagePicker.MediaTypeOptions.All,
        allowsEditing: true, aspect: [4, 3], quality: 1,
      });
      if (!result.cancelled) {
        const manipResult = await ImageManipulator.manipulateAsync(
          result.uri,
          [{ resize: { width: 385, height: 385 } }],
        );
        setFormState({ ...formState, image: imageId })
        setImage(manipResult.uri)
        setSaving(true)
        try {
          const response = await fetch(manipResult.uri)
          const blob = await response.blob()
          await Storage.put(imageId, blob)
          setSaving(false)
        } catch (error) {
          console.log({ error })
          setSaving(false)
        }
      }
      console.log({ result });
    } catch (error) {
      console.log({ error });
    }
  }

  async function createPost() {
    if (!image || !formState.name || !formState.location) return
    await DataStore.save(new Post(formState));
    setFormState(initialFormState)
    setImage(null)
    navigation.navigate('Posts')
  }

  function onChangeText(key, value) {
    setFormState({ ...formState, [key]: value })
  }
  return (
    <ScrollView style={styles.container} contentContainerStyle={styles.contentContainer}>
      <TextInput
        onChangeText={val => onChangeText('name', val)}
        placeholder="Post name"
        style={styles.inputStyle}
        value={formState.name}
      />
       <TextInput
        onChangeText={val => onChangeText('location', val)}
        placeholder="Post location"
        style={styles.inputStyle}
        value={formState.location}
      />
      <Button title="Choose an image" onPress={pickImage} />
      {image && <Image source={{ uri: image }} style={{ width: 200, height: 200 }} />}
      <Button disabled={saving} title="Create Post" onPress={createPost} />
      {
        saving && (
          <View>
            <Text>Saving image... </Text><ActivityIndicator />
          </View>
        )
      }
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fafafa',
    paddingHorizontal: 15
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
  },
  inputStyle: {
    height: 50,
    backgroundColor: '#ddd',
    marginBottom: 5,
    paddingHorizontal: 10
  }
});

export default CreatePostScreen
```

In this component there are 4 main functions:

**getPermissions** -  This function prompts the user for access to the camera roll on their device.

**pickImage** - This function allows the user to choose an image to store as part of the post. When choosing an image, we do the following
  * Using the __ImageManipulator__ library we first resize the image.
  * Next, we update the local state to set the image ID and the local image to show a local preview of the image
  * Finaly we use the `Storage` API from Amplify to upload the image to S3

**createPost** - This function calls the `DataStore` API to create a new post using the local state.

**onChangeText** - This function updates the `formState` with the user input for the post name and post location.

## Creating a Post view component

To render a Post we will be creating a new component that will display the post name, location, and image.

```js
/* components/PostComponent.js */
import React from 'react'
import {
  Text, View, Image, StyleSheet, Dimensions
} from 'react-native'

const { width } = Dimensions.get('window')

export default function PostComponent({ name, location, image }) {
  return (
    <View>
      <Text>{name}</Text>
      <Text>{location}</Text>
      <Image
        style={styles.image}
        source={{ uri: image }}
      />
    </View>
  )
}

const styles = StyleSheet.create({
  image: {
    width: width - 30,
    height: width - 30
  }
})
```

### Rendering a list of posts

In this component we will be fetching the list of posts and rendering them in our UI.

> Below this code snippet, I will walk through the main functionality.

```js
/* screens/PostsScreen.js */
import * as React from 'react';
import { StyleSheet, Text } from 'react-native';
import { ScrollView } from 'react-native-gesture-handler';
import { DataStore, Storage } from 'aws-amplify'
import { Post } from '../src/models'
import PostComponent from '../components/PostComponent'

function PostsScreen() {
  const [posts, setPosts] = React.useState([]);
  let subscription;
  React.useEffect(() => {
    fetchPosts();
    subscribe();
    return () => subscription && subscription.unsubscribe();
  }, [])
  async function fetchPosts() {
    const dataStoreQuery = await DataStore.query(Post);
    const postData = await Promise.all(dataStoreQuery.map(async post => {
      post = { ...post };
      const signedImage = await Storage.get(post.image);
      post.image = signedImage;
      return post;
    }))
    setPosts(postData);
  }
  async function subscribe() {
    subscription = DataStore.observe(Post).subscribe(() => {
      fetchPosts();
    });
  }
  return (
    <ScrollView style={styles.container} contentContainerStyle={styles.contentContainer}>
      <Text>All Posts</Text>
      {
        posts.map(post => (
          <PostComponent key={post.id} {...post} />
        ))
      }
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fafafa',
    paddingHorizontal: 15
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
  },
  inputStyle: {
    height: 50,
    backgroundColor: '#ddd',
    marginBottom: 5,
    paddingHorizontal: 10
  }
});

export default PostsScreen
```

In this component there are two main functions:

**fetchPosts** - This function calls the `DataStore` API and fetches the list of posts. We then map over the list of posts and fetch a signed image for each post image, and update the image property to be the signed image.

**subscribe** - This function will call `DataStore.observe()` which will listen for new posts that are created. This will provide a real-time feed of posts created by any user of the app.

### Updating the Tabs with the new components

Next, open __navigation/BottomTabNavigator.js__ and update 

```js
/* navigation/BottomTabNavigator.js */
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import * as React from 'react';

import TabBarIcon from '../components/TabBarIcon';
import CreatePostScreen from '../screens/CreatePostScreen';
import PostsScreen from '../screens/PostsScreen'
import ProfileScreen from '../screens/ProfileScreen'
import { withAuthenticator } from 'aws-amplify-react-native'

const BottomTab = createBottomTabNavigator();
const INITIAL_ROUTE_NAME = 'Posts';

function BottomTabNavigator({ navigation, route }) {
  navigation.setOptions({ headerTitle: getHeaderTitle(route) });

  return (
    <BottomTab.Navigator initialRouteName={INITIAL_ROUTE_NAME}>
      <BottomTab.Screen
        name="Posts"
        component={PostsScreen}
        options={{
          title: 'Posts',
          tabBarIcon: ({ focused }) => <TabBarIcon focused={focused} name="md-code-working" />,
        }}
      />
      <BottomTab.Screen
        name="Create Post"
        component={CreatePostScreen}
        options={{
          title: 'Create Post',
          tabBarIcon: ({ focused }) => <TabBarIcon focused={focused} name="ios-create" />,
        }}
      />
      <BottomTab.Screen
        name="Profile"
        component={ProfileScreen}
        options={{
          title: 'Profile',
          tabBarIcon: ({ focused }) => <TabBarIcon focused={focused} name="md-person" />
        }}
      />
    </BottomTab.Navigator>
  );
}

export default withAuthenticator(BottomTabNavigator)

function getHeaderTitle(route) {
  const routeName = route.state?.routes[route.state.index]?.name ?? INITIAL_ROUTE_NAME;

  switch (routeName) {
    case 'Home':
      return 'How to get started';
    case 'Links':
      return 'Links to learn more';
  }
}
```

To test it out, sign out and sign up as a new user. Create a couple of new posts under the new user account.

Next, run the following command:

```sh
$ expo start
```

You should be able to view only your posts in the **My Posts** tab and see all posts in the **Posts** tab.

## My posts

Let's add another tab that only renders the posts we've created. Since our GraphQL schema has a field that hold the information of the owner of the post, we can create a view that only shows the posts we've created ourselves.

To do so, we'll need to create a new component (**MyPostsScreen.js**) and update the bottom tab bar to render this components.

First, create **screens/MyPostsScreen.js** and add the following code:

```js
import * as React from 'react';
import { StyleSheet, Text } from 'react-native';
import { ScrollView } from 'react-native-gesture-handler';
import { DataStore, Storage, Auth } from 'aws-amplify'
import { Post } from '../src/models'
import PostComponent from '../components/PostComponent'

function CreatePostScreen() {
  const [posts, setPosts] = React.useState([]);
  let subscription;
  React.useEffect(() => {
    fetchPosts();
    subscribe();
    return () => subscription && subscription.unsubscribe();
  }, [])
  async function subscribe() {
    subscription = DataStore.observe(Post).subscribe(() => {
      fetchPosts();
    });
  }
  async function fetchPosts() {
    const { username } = await Auth.currentAuthenticatedUser()
    const dataStoreQuery = await DataStore.query(Post, p => p.owner('eq', username));
    let postData = await Promise.all(dataStoreQuery.map(async post => {
      post = { ...post };
      const signedImage = await Storage.get(post.image);
      post.image = signedImage;
      return post;
    }))
    setPosts(postData);
  }
  return (
    <ScrollView style={styles.container} contentContainerStyle={styles.contentContainer}>
      <Text>All Posts</Text>
      {
        posts.map(post => <PostComponent key={post.id} {...post} />)
      }
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fafafa',
    paddingHorizontal: 15
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
  },
  inputStyle: {
    height: 50,
    backgroundColor: '#ddd',
    marginBottom: 5,
    paddingHorizontal: 10
  }
});

export default CreatePostScreen
```

The main difference between this component and the **PostsScreen** component is that in the `fetchPosts` function we are filtering out the posts using the DataStore predicate of `eq` for **equals**.

```js
const dataStoreQuery = await DataStore.query(Post, p => p.owner('eq', username));
```

### Updating the tab bar

Finally, update the tab bar with the new tab:

```js
/* navigation/BottomTabNavigator.js */

/* First import the new MyPostscreen component  */
import MyPostsScreen from '../screens/MyPostsScreen'

/* Next, add the new tab */ 
<BottomTab.Screen
  name="My Posts"
  component={MyPostsScreen}
  options={{
    title: 'My Posts',
    tabBarIcon: ({ focused }) => <TabBarIcon focused={focused} name="ios-list-box" />,
  }}
/>
```

To test it out, run the following command:

```sh
$ expo start
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
