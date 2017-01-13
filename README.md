# How to Use JSON Web Tokens + OAuth as your Auth Schema in Node

[![Standard - JavaScript Style Guide](https://img.shields.io/badge/code%20style-standard-brightgreen.svg)](http://standardjs.com/)


⛑ Work in Progress ⛑

This is a brief overview of how to use JSON Web Tokens PLUS an OAuth flow to authenticate users in a Node app. Generally speaking, the flow looks like this:

```
1. User comes to your app and clicks OAuth sign in button.
2. Send user to OAuth partner (with specific URL given to you by OAuth partner upon setup).
3. At OAuth partner's site, user grants permission for you to access their partner data.
4. User is redirected by OAuth partner back to YOU redirect URI (which was given to OAuth partner upon setup), WITH some identifying information (typically an access token, user id, etc.)
5. Take that identifying information and:
  5a. Save it to your database (as appropriate or needed). 
  5b. Create a JSON Web Token for that user and issue it to the client (in the form of a cookie, or web storage). 
```

The user is now signed in. All incoming requests can be authenticated by grabbing the JSON Web Token (which is either automatically passed along in the HTTP request if it's a cookie, OR it's manually included in your request, typically in a `BEARER` header). 





JSON Web Tokens

Create a JWT and pass it to the client:
```
import jwt from 'jsonwebtoken'

const userData = {
  'name': 'John Doe',
  'userId': '12345',
  'favoriteFood': 'Marshmellow Creme'
}

function prepareJWTForBrowser (data) {
  let token = jwt.sign({
    userData
  }, process.env.JWT_SECRET, { expiresIn: '24h' })
  return token
}

prepareJWTForBrowser(userData)

```


Decode a JWT that's passed from the client:

```
import jwt from 'jsonwebtoken'

function decodeJWT (token) {
  // NOTE: jwt.verify() fails if you pass it null or undefined, so this is necessary
  if (!token) return false
  let decoded = jwt.verify(token, process.env.JWT_SECRET)
  return decoded
}

decodeJWT()

```

What's a JWT secret? It's any string, really. You must use the same string to encode and decote your JWTs.

I recommend going into a terminal and generating a 256 bit string, like this (Node must be installed):

```
node -e "console.log(require('crypto').randomBytes(256).toString('base64'));"
```

Then copy and paste the output and put it into your environment variable.


## Security Notes

#### The Secret Should be Secret (and Long/Random)
It's very import that you don't share, commit, or expose your JWT secret in client side code. A decoded JWT can be used by an attacked to gain access to your system.

#### The Data You Encode is Visisble to EVERYONE
The user data that you encode into the JWT (the `userData` const in the `prepareJWTForBrowser` function, for example) is actually NOT private. You do NOT need your JWT secret to access it. For proof, paste your encoded JWT (the output of `prepareJWTForBrowser` above), into this: https://jwt.io/.

That said, you should never store sensitive information inside it. It's simply used to match the authorized user to some data you can use to identify them.
