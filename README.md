serverless-crypt
=======

[![serverless](http://public.serverless.com/badges/v3.svg)](http://www.serverless.com)

# Description

Securing the secrets on Serverless Framework by AWS KMS encryption.
* This is a fork of marcy-terui's work to add support for S3 bucket storage of the encrypted secrets.  

# Requirements

- [Serverless Framework](https://github.com/serverless/serverless) 1.0 or higher

# Installation

```sh
npm install --save https://github.com/vortarian/serverless-crypt/archive/1.0.0.tar.gz
```

# Configuration

### serverless.yml

```yaml
provider:
  name: aws
  runtime: nodejs4.3

plugins:
  - serverless-crypt

custom:
    crypt:
      keyId: ${env:AWS_KMS_KEYID}
      location:  <file:// or s3:// url - must have read/write access>
```

### Supported runtimes
- python2.7
- nodejs4.3

# Commands

## Encrypt the secret

```sh
serverless encrypt -n $SECRET_NAME -t $PLAINTEXT --save
```

## Decrypt the secret

```sh
serverless decrypt -n $SECRET_NAME
```

# Usage

### 1. Create key on KMS  
See: https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html

### 2. Create and attach IAM policy to your serverless service role  
Policy example:  

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "kms:Decrypt"
            ],
            "Resource": [
                "arn:aws:kms:us-east-1:<your-account-number>:key/<your-key-id>"
            ]
        }
    ]
}
```

### 3. Set the key-id to your configuration file  
Configuration example:  

- serverless.yml

```yml
provider:
  name: aws
  runtime: nodejs4.3
  environment: # Service wide environment variables
    CRYPT_LOCATION: "${self:custom.crypt.location}"  # ENV Variable the plugin looks for when using the get(...) request at runtime

functions:
  hello:
    handler: handler.hello

plugins:
  - serverless-crypt

custom:
  cryptKeyId: ${env:AWS_KMS_KEYID}
```

### 4. Encrypt and save the secret to your secret file  
Command example:  
```sh
serverless encrypt -n secret_name -t "This is a secret" --save
```

### 5. Write your function  
** `slscrypt` module is automatically injected into your deployment package. **

Code example:  

- Node.js

```js
'use strict';

const slscrypt = require('slscrypt');

module.exports.hello = (event, context, callback) => {
  slscrypt.get('secret_name').then((txt) => {
    const response = {
      statusCode: 200,
      body: JSON.stringify({
        message: txt,
        input: event,
      }),
    };

    callback(null, response);
  });
};
```

- Python

```py
import json
import slscrypt

def hello(event, context):
    body = {
        "message": slscrypt.get('secret_name'),
        "input": event
    }
    response = {
        "statusCode": 200,
        "body": json.dumps(body)
    };
    return response
```

### 6. Deploy your function  
Command example:  

```sh
serverless deploy
```

or

```sh
serverless deploy function -f $FUNCTION_NAME
```

### 7. Invoke your function  
Command example:  

```sh
serverless invoke -f $FUNCTION_NAME
```

Result example:  

```
{
    "body": "{\"input\": {}, \"message\": \"This is a secret\"}",
    "statusCode": 200
}
```

Development
-----------

-   Source hosted at [GitHub](https://github.com/marcy-terui/serverless-crypt)
-   Report issues/questions/feature requests on [GitHub
    Issues](https://github.com/marcy-terui/serverless-crypt/issues)

Pull requests are very welcome! Make sure your patches are well tested.
Ideally create a topic branch for every separate change you make. For
example:

1.  Fork the repo
2.  Create your feature branch (`git checkout -b my-new-feature`)
3.  Commit your changes (`git commit -am 'Added some feature'`)
4.  Push to the branch (`git push origin my-new-feature`)
5.  Create new Pull Request

Authors
-------

Created and maintained by [Masashi Terui](https://github.com/marcy-terui) (<marcy9114@gmail.com>)

License
-------

MIT License (see [LICENSE](https://github.com/marcy-terui/serverless-crypt/blob/master/LICENSE.txt))
