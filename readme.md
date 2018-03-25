# Verifying HTTP Methods with `curl`
(with example from the LS-Auth Lab and LS-Auth-Mini Sprint)
Reference material:
- https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html
- http://www.restapitutorial.com/lessons/httpmethods.html
- https://stackoverflow.com/q/30760213/5225057
- see also `man curl`

### to set up the view counter and test the curl commands:
1. `cd` into the curl directory.
2. `npm i` to install the packages listed in the package.json file.
3. `mkdir data` for the MongoDB data.
4. `mongod --dbpath data` to launch the daemon and use the data directory.
5. `nodemon viewCounter.js` to launch the server so the.
6. `/users`, `/log-in`, `/me` & `/view-counter` routes are available.
7. on http://localhost:3000
8. In your home directory (`~/`) edit or create a `~/.curlrc` file and add `-w "\n"` so the `curl` command prints out ends with a "newline".

## 1. **POST**ing JSON objects to the '/users' route:
Just like the Postman application, `curl` can send data through JSON objects:
```js
{
  "username":"some_name",
  "password":"correcthorsebatterystaple"
}
```
...and get the corresponding passwordHash in return:

```console
$  curl -X POST -H "Content-Type: application/json" -d '{"username":"Bingo The Clown-o","password":"soincrediblyhardtohackthis"}' http://localhost:3000/users
  {"__v":0,"username":"Bingo The Clown-o","passwordHash":"$2a$11$lRHrlFuPWszbfzgvBw8NMehZ.M0zh/E/rv6Fx6NH1xaIFAFBGnJmm","_id":"599fa94427b8c3d1e2866d91"}
```

1. `-X` specifies the HTTP method type
2. `-H` specifies the data type
3. `-d` is for the actual data
4. `-b` is for the cookie jar (I think)
5. `-v` is for persistence (I think)

## 2. **POST**ing a username and password to '/log-in':
```console
$  curl -X POST -H "Content-Type: application/json" -d '{"username":"Bingo The Clown-o","password":"soincrediblyhardtohackthis"}' http://localhost:3000/log-in
    {"success":true}
```

## 3. **GET**ting the current logged in users name from '/me' with persistent cookies:
After logging in, note the long strong following "connect.sid=" in the `set-cookie` field:

```console
$  curl -H "Content-Type: application/json" -d '{"username":"Bingo The Clown-o","password":"soincrediblyhardtohackthis"}' -v http://localhost:3000/log-in
    *   Trying ::1...
    * TCP_NODELAY set
    * Connected to localhost (::1) port 3000 (#0)
    > POST /log-in HTTP/1.1
    > Host: localhost:3000
    > User-Agent: curl/7.54.0
    > Accept: */*
    > Content-Type: application/json
    > Content-Length: 72
    >
    * upload completely sent off: 72 out of 72 bytes
    < HTTP/1.1 200 OK
    < X-Powered-By: Express
    < Content-Type: application/json; charset=utf-8
    < Content-Length: 16
    < ETag: W/"10-oV4hJxRVSENxc/wX8+mA4/Pe4tA"
    < set-cookie: connect.sid=s%3AuKtDgCPm_3zd3FrjpEyJvLINKgMWFeFr.g0oEnSxhRKLvUBM%2BOwgcfPSCDKOGmPEh31FEswrmX%2F4; Path=/; HttpOnly
    < Date: Fri, 25 Aug 2017 15:40:20 GMT
    < Connection: keep-alive
    <
    * Connection #0 to host localhost left intact
    {"success":true}
```

You want to copy the connect.sid assignment and paste it into the "NAME=VALUE" argument of the next `curl` command:
```
curl -H "Content-Type: application/json" -b "NAME=VALUE" http://localhost:3000/me
```

...like so:

```console
$  curl -H "Content-Type: application/json" -b "connect.sid=s%3AuKtDgCPm_3zd3FrjpEyJvLINKgMWFeFr.g0oEnSxhRKLvUBM%2BOwgcfPSCDKOGmPEh31FEswrmX%2F4" http://localhost:3000/me
    {"_id":"59a0438098b5f10ac9968271","username":"Bingo The Clown-o","passwordHash":"$2a$11$YumjbaL6DL5bld4exfITX.bovAotTOAjKGfAgXOzkII7jn587/JOW","__v":0}
```

## 4. A more detailed explanation of how to use `curl` for **GET**ting with persistent cookies from '/view-counter':

As an example, in Karthik’s demo viddy, he made the '/view-counter' route:
```js
const bodyParser = require('body-parser');
const express = require('express');
const session = require('express-session');

const server = express();
server.use(bodyParser.json());

server.use(session({
  secret: 'e5SPiqsEtjexkTj3Xqovsjzq8ovjfgVDFMfUzSmJO21dtXs4re',
}));

server.get('/view-counter', (req, res) => {
  const persistentSession = req.session;
  if (!persistentSession.viewCount) {
    persistentSession.viewCount = 0;
  }
  persistentSession.viewCount++;
  res.json({ viewCount: persistentSession.viewCount });
});

module.exports = { server };

server.listen(3000);
```

To
1. visit that route,
2. GET the counter and
3. set `curl` up for persistent cookies,
...first enter this in your console:

```console
$ curl -v http://localhost:3000/view-counter
```

The `-v` option sets `curl` up to use a persistent cookie.

This is the first visit, so using this command will return the data object, { viewCount: 1 }.

NOTE also - and this is the important bit: towards the bottom of the `curl` print out, just above the view count, look for the `set-cookie` data:

```console
$  curl -v http://localhost:3000/view-counter
    *   Trying ::1...
    * TCP_NODELAY set
    * Connected to localhost (::1) port 3000 (#0)
    > GET /view-counter HTTP/1.1
    > Host: localhost:3000
    > User-Agent: curl/7.54.0
    > Accept: */*
    >
    < HTTP/1.1 200 OK
    < X-Powered-By: Express
    < Content-Type: application/json; charset=utf-8
    < Content-Length: 15
    < ETag: W/"f-S/x5i6y+O5xz5+BJCJHSQTCJ6H4"
                  vvvvvvvvvvvvvvvvvvvvvvvvvvvv---just this bit---vvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvv
    < set-cookie: connect.sid=s%3AY_yUqVkQUnUMWxkpDMiMqXZh-oTHqmby.H%2F8Jy3vQ52mNrj2BtytRRdlyQZxl5dWWdAu07QV8jNs; Path=/; HttpOnly
    < Date: Fri, 25 Aug 2017 03:42:32 GMT
    < Connection: keep-alive
    <
    * Connection #0 to host localhost left intact
    {"viewCount":1}    <---------------------------------- Hey, whaddya know????
```

Copy the ONLY `connect.sid` assignment,
i.e. `connect.sid=SUPERLONGSTRING_UP_UNTIL_THE_SEMI-COLON`
...and then paste that into a `curl` command as the `-b` argument.
This time you can leave off the `-v` option, but you'll need to add a `-H` option to tell `curl` what kind of data is being sent:
```
curl -H "Content-Type: application/json" -b "NAME=VALUE" http://localhost:3000/view-counter
```

i.e. for the `-b` argument, replace the `"NAME=VALUE"` with `"connect.sid=s%3AY_yUqVkQUnUMWxkpDMiMqXZh-oTHqmby.H%2F8Jy3vQ52mNrj2BtytRRdlyQZxl5dWWdAu07QV8jNs"`


Now, when you enter this command into your console, the view counter will increase by one with each successive visit through `curl`!

e.g.
```console
$  curl -v http://localhost:3000/view-counter
    ...           vvvvvvvvvvvvvvvvvvvvvvvvvv--just copy this bit--vvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvv
    < set-cookie: connect.sid=s%3AK-2XUQqoT4Cw7JPo_VIhXGIq03g8FTdv.xfw%2FCxZugBVxj2CGpBMPcK9R%2FrSTmXbdk6IXHSgQiCA; Path=/; HttpOnly
    ...
    * Connection #0 to host localhost left intact
    {"viewCount":1}   <-------------------------------------------- Lookie Thar!

$  curl -H "Content-Type: application/json" -b "connect.sid=s%3AK-2XUQqoT4Cw7JPo_VIhXGIq03g8FTdv.xfw%2FCxZugBVxj2CGpBMPcK9R%2FrSTmXbdk6IXHSgQiCA" -v http://localhost:3000/view-counter
    {"viewCount":2}    <------------------------------------------------ YaY!!!!

$  curl -H "Content-Type: application/json" -b "connect.sid=s%3AK-2XUQqoT4Cw7JPo_VIhXGIq03g8FTdv.xfw%2FCxZugBVxj2CGpBMPcK9R%2FrSTmXbdk6IXHSgQiCA" -v http://localhost:3000/view-counter
    {"viewCount":3}    <------------------------------ Third time's a charm!!!!!

```

## 5. From the Mobile-II project: SignUp, SignIn and accessing Restricted Content

`curl` (alternative to Postman) for signup, signin and get all users content:
1) SIGNUP
```
$  curl -X POST -H "Content-Type: application/json" -d '{"email":"fred@fred.com","password":"12345"}' -v https://mobile-server-ii.herokuapp.com/users
```
this will return an object like:
```
{"token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiI1OWI4NTExZGZlNmVkODAwMTE5Y2VjNzkiLCJpYXQiOjE1MDUyNTE2MTQ4NjV9.OBEFWFiJHpdGh1RZEYjgJiDdscG4PVfnELmrKwdHOJE","user":{"__v":0,"email":"fred@fred.com","password":"$2a$10$GIY/iw.A9yphc0ut1kKuPua6LiKpUKEl0zampkwND0/1DLt5xHuzS","_id":"59b8511dfe6ed800119cec79","todos":[]}}
```

2) SIGNIN
```
$  curl -X POST -H "Content-Type: application/json" -d '{"email":"fred@fred.com","password":"12345"}' -v https://mobile-server-ii.herokuapp.com/signin
```
this will return a similar object:
```
{"token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiI1OWI4NTExZGZlNmVkODAwMTE5Y2VjNzkiLCJpYXQiOjE1MDUyNTE2NDc2MjB9.0cp7qTMw-wgDYkV9Bj0ZwOADAhIWR7t3j_oerqxtdIg","user":{"_id":"59b8511dfe6ed800119cec79","email":"fred@fred.com","password":"$2a$10$GIY/iw.A9yphc0ut1kKuPua6LiKpUKEl0zampkwND0/1DLt5xHuzS","__v":0,"todos":[]}}
```
…copy the token value from the signin’s return object, and the use it to get the restricted content:

3)  CONTENT
```
$  curl -X GET -H "authorization: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiI1OWI4NTExZGZlNmVkODAwMTE5Y2VjNzkiLCJpYXQiOjE1MDUyNTE2NDc2MjB9.0cp7qTMw-wgDYkV9Bj0ZwOADAhIWR7t3j_oerqxtdIg" -v https://mobile-server-ii.herokuapp.com/users
```
…and a whole buncha username and hashed passwords will come your way

:smile:

4) TODOS
To enter a `curl` request with multiple headers, e.g. a Content-Type and an authorization token, _both_ headers need to be pre-pended with the -H flag, e.g.
```
$  curl -X POST -H "authorization: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiI1OWI5NzMwOTgwMTJhODAwMTEyYmU1MjUiLCJpYXQiOjE1MDUzMzQ2ODcxNTZ9.5_zvJRpJmEtO4AR4xvg5cxm6tFrPJAExqgrS6J7RCFs" -H "Content-Type: application/json" -d '{"text":"bake a cake"}' https://mobile-server-ii.herokuapp.com/todos

{"_id":"59b973098012a800112be525","email":"ely@ely.com","password":"$2a$10$73Sonm7Sy19dnLwWtSpuZeqnVTUFN65AHHWD2ws8KEnvAZW/03YHq","__v":1,"todos":[{"text":"bake a cake","_id":"59b999748012a800112be5dc","completed":false}]}
```

## 6. Sprint-Challenge-Authentication
Just for my own reference, cURL equivalents:

For folks like working in the console, there are `cURL` equivalents for Postman commands. You can access these through the `Code` snippets link in the Postman app:
https://www.getpostman.com/docs/v6/postman/sending_api_requests/generate_code_snippets

The Postman snippets are a little more verbose, but as an example, instead of using Postman for the Authentication Sprint routes and HTTP methods, these are the `cURL` equivalents:

*/api/users/*
i.e. `curl -x HTTP_METHOD {URL} -H {CONTENT:TYPE} -d '{"JSON":"DATA"}`
```console
$  curl -X POST http://localhost:5000/api/users -H 'Content-Type: application/json' -d '{"username":"cool_name","password":"reallydifficult"}'
    {
     "__v":0,
     "username":"cool_name",
     "password":"$2a$11$ZziXrc/QXfPxm42WWLPPJ.JyvQS.ClyBnPDYvdclDUv06v9Uaiwhu",
     "_id":"5ab6c702ae098de170987a00"
    }
```


*/api/login/*
i.e. `curl -x HTTP_METHOD {URL} -H {CONTENT:TYPE} -d '{"JSON":"DATA"}`
```console
$  curl -X POST http://localhost:5000/api/login -H 'Content-Type: application/json' -d '{"username":"cool_name","password":"reallydifficult"}'
    {
     "token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImNvb2xfbmFtZSIsImlhdCI6MTUyMTkyODA5Mn0.8aSKqHzQkprobO5w4oo-SuC03M4YIYFyPCD9EVNXu_s"
    }
```

*/api/jokes:*
i.e. `curl {URL} -H "Authorization:token"`
```console
$ curl http://localhost:5000/api/jokes -H "Authorization:eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImNvb2xfbmFtZSIsImlhdCI6MTUyMTkyODA5Mn0.8aSKqHzQkprobO5w4oo-SuC03M4YIYFyPCD9EVNXu_s"
[
    {
        "id": 30,
        "type": "programming",
        "setup": "Two bytes meet. The first byte asks, \"Are you ill?\"",
        "punchline": "The second byte replies, \"No, just feeling a bit off.\""
    },
    et cetera...
```
