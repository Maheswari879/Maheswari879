app.js
const express = require('express');
const bodyParser = require('body-parser');
const path = require('path');
const admin = require('firebase-admin');

const app = express();
const port = 3000;

// Initialize Firebase Admin SDK with service account key
const serviceAccount = require('./capkey.json');

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount)
});

app.use(bodyParser.json());
app.use(express.static(path.join(__dirname, 'public')));

app.get('/', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'index.html'));
});

app.get('/login', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'login.html'));
});

app.get('/signup', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'signup.html'));
});

app.post('/signup', async (req, res) => {
  const { username, email, password } = req.body;

  console.log('Received signup request:', { username, email, password });

  try {
    const db = admin.firestore();
    const userRef = db.collection('users').doc(email);
    const doc = await userRef.get();

    if (doc.exists) {
      console.log('Signup error: Email already exists');
      return res.status(400).json({ success: false, message: 'Email already exists' });
    } else {
      await userRef.set({ username, email, password });
      console.log('User created successfully:', { username, email });
      return res.status(201).json({ success: true });
    }
  } catch (error) {
    console.error('Signup error:', error);
    return res.status(500).json({ success: false, message: 'An error occurred while creating user', error: error.message });
  }
});

app.post('/login', async (req, res) => {
  const { email, password } = req.body;

  console.log('Received login request:', { email, password });

  try {
    const db = admin.firestore();
    const userRef = db.collection('users').doc(email);
    const doc = await userRef.get();
    if (!doc.exists) {
      console.log('Login error: User not found');
      return res.status(400).json({ success: false, message: 'User not found' });
    } else {
      const user = doc.data();
      console.log('Found user:', user);
      if (user.password === password) {
        console.log('User logged in successfully:', { email });
        return res.status(200).json({ success: true, message: 'Login successful' });
      } else {
        console.log('Login error: Incorrect password');
        return res.status(400).json({ success: false, message: 'Incorrect password' });
      }
    }
  } catch (error) {
    console.error('Login error:', error);
    return res.status(500).json({ success: false, message: 'An error occurred during login', error: error.message });
  }
});

app.listen(port, () => {
  console.log(`Server is running on http://localhost:${port}`);
});
