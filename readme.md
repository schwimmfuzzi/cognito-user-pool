# Easily manage your users with AWS Cognito User Pools

This library is a wrapper around the javascript frontend library aws-cognito-identity-js
to easily manage your Cognito User Pool in a node.js backend environment.

## Usage

At the root of your project, do:
```
npm install --save cognito-user-pool
```

Import it in your project:
```
import CognitoUserPoolWrapper from 'cognito-user-pool';
```

or in ES5:
```
const CognitoUserPoolWrapper = require('cognito-user-pool').default;
```

Then use it like so:
```
const poolData = {
  UserPoolId : USER_POOL_ID, // your user pool ID
  ClientId : USER_POOL_CLIENT_ID, // generated in the AWS console
  Paranoia : PARANOIA_LEVEL // an integer between 1 - 10
};

const userPool = new CognitoUserPoolWrapper(poolData);
```

## Methods

Actual data to pass to the function depends on your user pool settings.

### Signup

Signup a new user:

```
userPool.signup(params, callback)
```


```
params: {
  "username": "string",
  "password": "string",
  "attributes": [
    {
      "Name": "string",
      "Value": "string"
    }
  ]
}
```

### Signup Confirmation

Depending on your settings, email confirmation may be required.
In that case, the following function must be called:

```
userPool.signupConfirm(params, callback)
```

```
params: {
  "username": "string",
  "confirmationCode": "string"
}
```

### Resend Signup Confirmation Code

If the user didn't receive the signup confirmation code, they may request a new code:

```userPool.signupResend(params, callback)```

```
params: {
  "username": "string"
}
```

### Login

Login an existing and confirmed user:

```
userPool.login(params, callback)
```

Note that username can be any alias field as defined in user pool settings.

```
params: {
  "username": "string",
  "password": "string"
}
```
This function returns either authentication tokens (more on that later)
or a custom challenge for continuing the authentication process.

In that case, you get:
```
{
  "nextStep": "string",
  "loginSession": "string"
}
```

With `nextStep` being either `MFA_AUTH` or `NEW_PASSWORD_REQUIRED`.
`MFA_AUTH` means a SMS was sent to their cell phone with a code to add to the `loginMfa` method,
while `NEW_PASSWORD_REQUIRED` means they need to reset their password in the next step with `loginNewPasswordRequired`.

If authentication was successful, you retrieve instead 3 auth tokens and the associated expiration dates:

```
{
  "refreshToken": "string",
  "accessToken": "string",
  "accessTokenExpiresAt": integer,
  "idToken": "string",
  "idTokenExpiresAt": integer
}
```
Please read [https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-with-identity-providers.html] for more information.
The token you will need to authenticate against this module later on is `refreshToken`.

### Login: MFA

Using the information from the `login` method and the `mfaCode` received by SMS:

```
userPool.loginMfa(params, callback)
```

```
params: {
  "username": "string",
  "loginSession": "string",
  "mfaCode": "string"
}
```

### Login: New Password Required

Using the information from the `login` method:

```
userPool.loginNewPasswordRequired(params, callback)
```

```
params: {
  "username": "string",
  "loginSession": "string",
  "newPassword": "string"
}
```

### Logout

This method invalidates all issued tokens.

```
userPool.logout(params, callback)
```

```
params: {
  "username": "string",
  "refreshToken": "string"
}
```

### Refresh Session

Generate new refreshToken, idToken and accessToken with a new expiry date.

```
userPool.refreshSession(params, callback)
```

```
params: {
  "username": "string",
  "refreshToken": "string"
}
```

If successful, you retrieve 3 auth tokens and the associated expiration dates (same as login):

```
{
  "refreshToken": "string",
  "accessToken": "string",
  "accessTokenExpiresAt": integer,
  "idToken": "string",
  "idTokenExpiresAt": integer
}
```

### Get MFA status

If MFA is enabled for this user, retrieve its options. Otherwise, returns `undefined`.

```
userPool.getMfa(params, callback)
```

```
params: {
  "username": "string",
  "refreshToken": "string"
}
```

### Enable or Disable MFA

```
userPool.setMfa(params, callback)
```

```
params: {
  "enableMfa": boolean,
  "username": "string",
  "refreshToken": "string"
}
```

### Get profile

Retrieve all attributes associated with this user.

```
userPool.profile(params, callback)
```

```
params: {
  "username": "string",
  "refreshToken": "string"
}
```

### Edit profile

Use this endpoint to edit all user attributes except phone (see below).

```
userPool.profileEdit(params, callback)
```

If the `Value` of an attribute is left empty, that attribute will be removed.

```
params: {
  "username": "string",
  "refreshToken": "string",
  "attributes": [
    {
      "Name": "string",
      "Value": "string"
    }
  ]
}
```

### Edit phone number

Use this endpoint to change the user's phone number.

```
userPool.profileEditPhoneNumber(params, callback)
```

If `phone_number` is undefined or null, it will be removed and MFA will be disabled on this user.

```
params: {
  "username": "string",
  "refreshToken": "string",
  "phone_number: "string"
}
```

### Change password

Use this endpoint to change the user's password.

```
userPool.passwordChange(params, callback)
```

```
params: {
  "username": "string",
  "refreshToken": "string",
  "oldPassword": "string",
  "newPassword: "string"
}
```

### Forgot password

Start a forgot password flow.
Cognito will send a `passwordResetCode` to one of the user's confirmed contact methods (email or SMS)
to be used in the `passwordReset` method below.

```
userPool.passwordForgot(params, callback)
```

```
params: {
  "username": "string"
}
```

### Reset password

Finish the forgot password flow.

```
userPool.passwordReset(params, callback)
```

```
params: {
  "username": "string",
  "passwordResetCode": "string",
  "newPassword": "string
}
```

## Release notes

### v1.0 Breaking changes

- Project rewritten in ES6: new init method. See above for setup instructions. Nothing else changed, so it doesn't require much to upgrade. Otherwise stick with v0.x.


### Next goals

- Add support for Promises (omitting the callback parameter returns a promise)
- Add support for social login / OAuth / SAML out of the box

### Contributing

Coding style is enforced by eslint. You can test your code by running `npm run eslint`. See .eslintrc for common rules.
No test suite is there yet, so feel free to add one :-)

To help with development, run `npm run dev` to transpile the ES6 files in `src/` on any code change.