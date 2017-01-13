<h1 align="center">
  <a href="https://oauth.net/"><img src="https://g.twimg.com/dev/documentation/image/oauth.png" alt="OAuth" width="100"></a> &nbsp;
  <a href="https://nodejs.org/en/"><img src="https://ih1.redbubble.net/image.109336634.1604/flat,550x550,075,f.u1.jpg" alt="Node" width="100"></a>
  &nbsp;
  <a href="https://jwt.io/"><img src="https://jwt.io/img/pic_logo.svg" alt="JWT" width="100"></a>
  <br>
  JSON Web Tokens &amp; OAuth in Node
  <br>
  <br>
</h1>

<h4 align="center">How to use JSON Web Tokens with OAuth as a complete authentication schema for Node</h4>

<p align="center">
<a align="center" href="http://standardjs.com/"><img src="https://img.shields.io/badge/code%20style-standard-brightgreen.svg" alt="Standard"></a>
<br>
⛑ Work in Progress ⛑
</p>
<br>


This is a brief overview of how to use JSON Web Tokens (https://jwt.io/), along with an OAuth partner like Facebook, Twitter, etc., to authenticate users in a Node app. This is a powerful authentication schema: it's extremely secure, it's generally simple for users because they don't have to create new accounts and sign in (assuming they use the OAuth partner), and it's easy to manage (no passwords or auth credentials at all). 

(Side note: Alternatively, you could use JWTs alongside a typical password schema as well. A topic for a different walkthrough.)

## Auth Flow Outline

### OAuth Flow
To use an OAuth schema, you need to register your app with the OAuth partner (see [Facebook](https://developers.facebook.com/docs/facebook-login/web), for example). When registering your app, they'll give you a URL to direct users to to authenticate them. And you'll give THEM a redirect URL, which is where they'll be sending them back after the user approves your app. 
```
1. User comes to your app and clicks OAuth sign in button.
2. Send user to OAuth partner (with specific URL given to you by OAuth partner upon setup).
3. At OAuth partner's site, user grants permission for you to access their partner data.
4. User is redirected by OAuth partner back to YOU redirect URI (which was given to OAuth partner upon setup), WITH some identifying information (typically an access token, user id, etc.)
```

### JWT creation 
After step 4 above, your user is a confirmed, identified user, and you'll get their identifying information along with the incoming redirect/request. Since we've identified the user successfully, it's safe to create a JWT for that user. This JWT will be sent to the client, and included in every request sent from that client to your server (so you can decode it and figure out who they are). 
```
5. Take that identifying information and:
  5a. Save it to your database (as appropriate or needed). 
  5b. Create a JSON Web Token for that user and issue it to the client (in the form of a cookie, or web storage), along with any other user-specific information the client needs to display.
```

### Checking auth state on incoming requests
The user is now signed in. All incoming requests can be authenticated by:
```
1. Grabbing the JSON Web Token off incoming requests (which is either automatically passed along in the HTTP request if it's a cookie, OR it's manually included in your request, typically in a `BEARER` header)
2. Decoding it using the "secret" that you used to create it.
3. Confirming that there is user data there. 
```

That's it. If it's not a JWT that was created with _your_ secret, then it won't decode, and you'll know you aren't dealing with an authenticated user. In this case, you can redirect them to the OAuth URI and follow the same flow to authenticate them (or reauthenticate them) as we outlined in the beginning (simply without Step 1, because you're routing them manually). 





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
