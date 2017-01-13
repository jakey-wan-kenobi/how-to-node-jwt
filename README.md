# how-to-node-jwt
🔑 A quick walkthrough on using JSON Web Tokens in Node for authentication.

⛑ Work in Progress ⛑



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
