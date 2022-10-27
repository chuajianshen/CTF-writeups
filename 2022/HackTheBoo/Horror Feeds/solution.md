# Horror Feeds (x solves)

## Challenge Description
The challenge is an SQL injection, logging in as the admin will grant us the flag.

```
 {% if user == 'admin' %}
 <td>{{flag}}</td>
```

## Challenge Files
Looking at the source code, we have unsanitised user input in an SQL query under the register function

```
def register(username, password):
    exists = query_db('SELECT * FROM users WHERE username = %s', (username,))
   
    if exists:
        return False
    
    hashed = generate_password_hash(password)

    query_db(f'INSERT INTO users (username, password) VALUES ("{username}", "{hashed}")')
    mysql.connection.commit()

    return True
```

The user can inject mallicious data into the username field.
By supplying 
admin","fake_hash_value");-- -" as the username, we could insert an admin user with fake_hash_value as the password. However, a user of admin is already in the database.

[!database](../images/horror1.png)

The database also uses bcrypt to hash the passwords.
```
def generate_password_hash(password):
    salt = bcrypt.gensalt()
    return bcrypt.hashpw(password.encode(), salt).decode()

def verify_hash(password, passhash):
    return bcrypt.checkpw(password.encode(), passhash.encode())
```

## Solution
We can update the password of the admin user by utilising ON DUPLICATE KEY. If you specify an ON DUPLICATE KEY UPDATE clause and a row to be inserted would cause a duplicate value in a UNIQUE index or PRIMARY KEY, an UPDATE of the old row occurs.

### Payload

[!payload](../../HackTheBoo/images/horror2.png)

MYSQL finds that an admin user exists and updates its password to the bcrypt hash of "123".

Username = admin, Password = 123

Logging in provides us with the flag

## Flag
```
HTB{N3ST3D_QU3R1E5_AR3_5CARY!!!}
```




