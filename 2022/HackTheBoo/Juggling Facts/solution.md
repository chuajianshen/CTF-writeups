# Juggling Facts (x solves)

## Challenge Description
The challenge is a PHP type juggling vulnurability.

## Challenge Files
There is a web page where only admins can access, the flag is stored here.

[!database](../images/jugg1.png)

Checking the source code, it uses switch statements to check if secrets is in the request, if it is only the localhost machine can access it

```
 public function getfacts($router)
    {
        $jsondata = json_decode(file_get_contents('php://input'), true);

        if ( empty($jsondata) || !array_key_exists('type', $jsondata))
        {
            return $router->jsonify(['message' => 'Insufficient parameters!']);
        }

        if ($jsondata['type'] === 'secrets' && $_SERVER['REMOTE_ADDR'] !== '127.0.0.1')
        {
            return $router->jsonify(['message' => 'Currently this type can be only accessed through localhost!']);
        }

        switch ($jsondata['type'])
        {
            case 'secrets':
                return $router->jsonify([
                    'facts' => $this->facts->get_facts('secrets')
                ]);

            case 'spooky':
                return $router->jsonify([
                    'facts' => $this->facts->get_facts('spooky')
                ]);
            
            case 'not_spooky':
                return $router->jsonify([
                    'facts' => $this->facts->get_facts('not_spooky')
                ]);
            
            default:
                return $router->jsonify([
                    'message' => 'Invalid type!'
                ]);
        }
    }
}
```

## Solution
Switch statemets in PHP use loose comparison, e.g == instead of ===. Furthermore, data is received through JSON allowing us to specify other data types such as arrays and booleans instead of only strings.

[!database](../images/jugg2.png)

### Payload
Supplying a true statement would bypass the localhost restriction and match case 'secrets'. This is because of the loose comparison

[!database](../images/jugg4.png)

## Flag
[!database](../images/jugg3.png)




