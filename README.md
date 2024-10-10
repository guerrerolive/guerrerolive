npx react-native init TalkToMe
cd TalkToMe
npm install @react-navigation/native @react-navigation/stack
npm install firebase react-native-firebase
npm install react-native-async-storage/async-storage
npm install axios
// firebaseConfig.js
import firebase from 'firebase/app';
import 'firebase/auth';
import 'firebase/firestore';

const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_AUTH_DOMAIN",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_STORAGE_BUCKET",
  messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
  appId: "YOUR_APP_ID"
};

if (!firebase.apps.length) {
  firebase.initializeApp(firebaseConfig);
}

export { firebase };
// LoginScreen.js
import React, { useState } from 'react';
import { View, TextInput, Button, Text } from 'react-native';
import { firebase } from './firebaseConfig';

export default function LoginScreen({ navigation }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');

  const handleLogin = () => {
    firebase.auth().signInWithEmailAndPassword(email, password)
      .then(() => navigation.navigate('Home'))
      .catch(err => setError(err.message));
  };

  return (
    <View>
      <TextInput placeholder="Email" value={email} onChangeText={setEmail} />
      <TextInput placeholder="Password" value={password} onChangeText={setPassword} secureTextEntry />
      {error ? <Text>{error}</Text> : null}
      <Button title="Login" onPress={handleLogin} />
      <Button title="Sign Up" onPress={() => navigation.navigate('SignUp')} />
    </View>
  );
}
// SignUpScreen.js
import React, { useState } from 'react';
import { View, TextInput, Button, Text } from 'react-native';
import { firebase } from './firebaseConfig';

export default function SignUpScreen({ navigation }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');

  const handleSignUp = () => {
    firebase.auth().createUserWithEmailAndPassword(email, password)
      .then(() => navigation.navigate('Home'))
      .catch(err => setError(err.message));
  };

  return (
    <View>
      <TextInput placeholder="Email" value={email} onChangeText={setEmail} />
      <TextInput placeholder="Password" value={password} onChangeText={setPassword} secureTextEntry />
      {error ? <Text>{error}</Text> : null}
      <Button title="Sign Up" onPress={handleSignUp} />
    </View>
  );
}
// SignUpScreen.js
import React, { useState } from 'react';
import { View, TextInput, Button, Text } from 'react-native';
import { firebase } from './firebaseConfig';

export default function SignUpScreen({ navigation }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');

  const handleSignUp = () => {
    firebase.auth().createUserWithEmailAndPassword(email, password)
      .then(() => navigation.navigate('Home'))
      .catch(err => setError(err.message));
  };

  return (
    <View>
      <TextInput placeholder="Email" value={email} onChangeText={setEmail} />
      <TextInput placeholder="Password" value={password} onChangeText={setPassword} secureTextEntry />
      {error ? <Text>{error}</Text> : null}
      <Button title="Sign Up" onPress={handleSignUp} />
    </View>
  );
}
// ProfileScreen.js
import React, { useState, useEffect } from 'react';
import { View, Text, Button } from 'react-native';
import { firebase } from './firebaseConfig';

export default function ProfileScreen() {
  const [profile, setProfile] = useState(null);
  const userId = firebase.auth().currentUser.uid;

  useEffect(() => {
    firebase.firestore().collection('users').doc(userId).get()
      .then(doc => setProfile(doc.data()));
  }, []);

  const handleMatch = () => {
    // Lógica de correspondência pode ser expandida aqui
  };

  return (
    <View>
      {profile ? (
        <>
          <Text>Name: {profile.name}</Text>
          <Text>Interests: {profile.interests}</Text>
          <Button title="Find Matches" onPress={handleMatch} />
        </>
      ) : (
        <Text>Loading profile...</Text>
      )}
    </View>
  );
}
// ChatScreen.js
import React, { useState, useEffect } from 'react';
import { View, TextInput, Button, FlatList, Text } from 'react-native';
import { firebase } from './firebaseConfig';

export default function ChatScreen({ route }) {
  const [message, setMessage] = useState('');
  const [messages, setMessages] = useState([]);
  const chatId = route.params.chatId;

  useEffect(() => {
    const unsubscribe = firebase.firestore().collection('chats').doc(chatId)
      .collection('messages').orderBy('createdAt')
      .onSnapshot(snapshot => {
        const newMessages = snapshot.docs.map(doc => doc.data());
        setMessages(newMessages);
      });

    return () => unsubscribe();
  }, []);

  const handleSend = () => {
    firebase.firestore().collection('chats').doc(chatId)
      .collection('messages').add({
        text: message,
        createdAt: firebase.firestore.FieldValue.serverTimestamp(),
        userId: firebase.auth().currentUser.uid
      });
    setMessage('');
  };

  return (
    <View>
      <FlatList
        data={messages}
        renderItem={({ item }) => <Text>{item.text}</Text>}
      />
      <TextInput value={message} onChangeText={setMessage} />
      <Button title="Send" onPress={handleSend} />
    </View>
  );
}// App.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import LoginScreen from './LoginScreen';
import SignUpScreen from './SignUpScreen';
import ProfileScreen from './ProfileScreen';
import ChatScreen from './ChatScreen';

const Stack = createStackNavigator();

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Login" component={LoginScreen} />
        <Stack.Screen name="SignUp" component={SignUpScreen} />
        <Stack.Screen name="Profile" component={ProfileScreen} />
        <Stack.Screen name="Chat" component={ChatScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

