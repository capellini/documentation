---
layout: layout-api
locales: de
---
# hoodie.account
**version:** 		*> 0.1.0* <br />
**source:** 		*[hoodie.js/src/hoodie/account](https://github.com/hoodiehq/hoodie/tree/hoodie-client-legacy/src/hoodie/account)*

**<br />after reading this you will know:**

- how to sign up / in / out a user
- how to check if a user is signed in
- how to change username or password
- how to reset a password
- how to listen to account events
- how to destroy a user account

## Introduction

*The account object gives you the methods to create, update and delete an account. A user's data is always bound to the user's account, and will automatically be synchronized when signend in on different devices.*


### Properties
- [username](#accountusername)


### Methods
- [signUp()](#accountsignup)
- [signIn()](#accountsignin)
- [signOut()](#accountsignout)
- [changePassword()](#accountchangepassword)
- [changeUsername()](#accountchangeusername)
- [resetPassword()](#accountresetpassword)
- [destroy()](#accountdestroy)

### Events <small>[>>](#accountevents)</small>
- signup
- signin
- signout
- reauthenticated
- changepassword
- changeusername
- passwordreset
- error:passwordreset
- error:unauthenticated


<a id="accountusername"></a>
### account.username
**version:**    *> 0.2.0*

<pre><code>hoodie.account.username</code></pre>

The **hoodie.account.username** gets automatically set / unset when signing in, signing up, changing username
or destroying the account.

It is also the current way to check if a user is signed in or not.


##### Example

<pre><code>if (hoodie.account.username) {
  // user is signed in
  console.log(hoodie.account.username);
} else {
  // user is anonymous
  console.log('You are not logged in!');
}
</code></pre>

## Methods


<a id="accountsignup"></a>
### account.signUp()
**version:** 		*> 0.2.0*

<pre><code>hoodie.account.signUp('user', 'password');</code></pre>

##### Arguments

| Nr | argument   | type   | description     | required |
| --:| ---------- |:------:|:---------------:|:--------:|
|  1 | username   | String | username        | yes      |
|  2 | password   | String | valid password  | yes      |


##### Resolves with

| argument    | description                                          |
| ----------- | ---------------------------------------------------- |
| username    | the username the user signed up with                 |

##### Progresses with

| argument    | when                                                 |
| ----------- | ---------------------------------------------------- |
| -           | after account created on server, before confirmation |

##### Rejects with

| error                 | message                                     |
| --------------------- | ------------------------------------------- |
| HoodieError           | Username must be set                        |
| HoodieError           | Must sign out first                         |
| HoodieConflictError   | Username **&lt;username>** already exists   |
| HoodieConnectionError | Could not connect to Server                 |


The **signUp** creates a new user account on the Hoodie server. The account is confirmed automatically,
after the user-specific database has been created, where all the user's data gets automatically
synchronized to.

##### Example
<pre><code>$('#signUpForm').submit(function (event) {
  event.preventDefault();
  var username  = $('#username').val();
  var password  = $('#password').val();

  hoodie.account.signUp(username, password)
    .done(welcomeNewUser)
    .fail(showErrorMessage);
});
</code></pre>

##### Notes
> - The confirmation workflow will be customizable in future
> - there is no feature built-in to compare the password to a password confirmation.
>   If you need this logic, please validate this beforehand in your app.



<a id="accountsignin"></a>
### account.signIn()
**version:** 		*> 0.2.0*

<pre><code>hoodie.account.signIn('user', 'password');</code></pre>

##### Arguments

| Nr | option     | type   | description             | required |
| --:| ---------- |:------:|:-----------------------:|:--------:|
|  1 | user       | String | username                | yes      |
|  2 | password   | String | valid password          | yes      |
|  3 | options    | Object | {moveData: true/false}  | no       |

##### Resolves with

| argument    | description                                          |
| ----------- | ---------------------------------------------------- |
| username    | the username the user signed in with                 |

##### Rejects with

| error                          | message                                     |
| ------------------------------ | ------------------------------------------- |
| HoodieAccountUnconfirmedError  | Account has not been confirmed yet |
| HoodieAccountNotFoundError     | Account could not be found |
| HoodieError                    | _A custom error can be set by plugins, e.g. the account could be blocked due to missing payments_ |
| HoodieConnectionError | Could not connect to Server                 |

<br />


The **signIn** tries to sign in the user to an existing account.

All local data that exists locally before signin in gets cleared. To prevent data loss, you can pass **{moveData: true}** as the options argument, this will move current data (created anonymously or by another account) to the account the user signs in to.

##### Example

<pre><code>$('#signInForm').submit(function (event) {
  event.preventDefault();
  var username = $('#username').val();
  var password = $('#password').val();
  hoodie.account.signIn(username, password)
    .done(welcomeUser)
    .fail(showErrorMessage);
});
</code></pre>

<a id="accountsignout"></a>
### account.signOut()
**version:** 		*> 0.2.0*


<pre><code>hoodie.account.signOut(options);</code></pre>


##### Arguments

| Nr | option     | type   | description             | required |
| --:| ---------- |:------:|:-----------------------:|:--------:|
|  1 | options    | Object | {ignoreLocalChanges: true/false}  | no       |

##### Resolves with

_resolves without argument_<br /><br />

##### Rejects with

| error                          | message                        |
| ------------------------------ | ------------------------------ |
| HoodieAccountLocalChangesError | There are local changes that could not be synced |

<br />

The **signOut()** ends the user's session if signed in, and clears all local data.

Before signing out a user, Hoodie pushes all local changes to the user's databases. If that fails
because the Server cannot be reached or the user's session is invalid, **signOut()** fails and rejects
with an **HoodieAccountLocalChangesError** error. To enforce a signout despite eventual data loss,
you can pass **{ignoreLocalChanges: true}** as the options argument.


##### Example

<pre><code>hoodie.account.signOut()
  .done(redirectToHomepage)
  .fail(showError);</code></pre>

##### Notes
> - **HoodieAccountLocalChangesError** does currently not get returned, instead cryptic errors
>   may be returned, about the user not being authenticated or a failed request. See [#358](https://github.com/hoodiehq/hoodie.js/issues/358)


<a id="accountchangepassword"></a>
### account.changePassword()
**version:**    *> 0.2.0*


<pre><code>hoodie.account.changePassword(currentPassword, newPassword);</code></pre>


##### Arguments

| Nr | option           | type   | description                           | required |
| --:| ---------------- |:------:|:-------------------------------------:|:--------:|
|  1 | currentPassword  | String | password of signed in user            | yes      |
|  2 | newPassword      | String | new password for signed in user       | yes      |

##### Resolves with

_resolves without argument_ <br /><br />

##### Rejects with

| error                      | message                        |
| -------------------------- | ------------------------------ |
| HoodieUnauthenticatedError | Not signed in |



##### Example

<pre><code>hoodie.account.changePassword('secret', 'newSecret')
  .done(showSuccessMessage)
  .fail(showErrorMessage)
</code></pre>

##### Notes
> - The current password is ignored with the current implementation.
>   This will be fixed with [#103](https://github.com/hoodiehq/hoodie.js/issues/103)


<a id="accountchangeusername"></a>
### account.changeUsername()
**version:**    *> 0.2.0*

<pre><code>hoodie.account.changeUsername(currentPassword, newUsername);</code></pre>


##### Arguments

| Nr | option           | type   | description                           | required |
| --:| ---------------- |:------:|:-------------------------------------:|:--------:|
|  1 | currentPassword  | String | password of signed in user            | yes      |
|  2 | newUsername      | String | new username for signed in user       | yes      |

##### Resolves with

_resolves without argument_ <br /><br />

##### Rejects with

| error                      | message                        |
| -------------------------- | ------------------------------ |
| HoodieUnauthenticatedError | Not signed in |
| HoodieConflictError        | Usernames identical |
| HoodieConflictError        | **&lt;newUsername>** is taken |


##### Example

<pre><code>hoodie.account.changeUsername('secret', 'newusername')
  .done(showSuccessMessage)
  .fail(showErrorMessage)
</code></pre>


<a id="accountresetpassword"></a>
### account.resetPassword()
**version:**    *> 0.2.0*

<pre><code>hoodie.account.resetPassword(username);</code></pre>

##### Arguments

| Nr | option     | type   | description                                           | required |
| --:| ---------- |:------:|:-----------------------------------------------------:|:--------:|
|  1 | username   | String | username of the user which password shall be resetted | yes      |

##### Resolves with

| argument    | description                                          |
| ----------- | ---------------------------------------------------- |
| username    | the username that got the password resetted          |

##### Rejects with

| error                      | message                              |
| -------------------------- | ------------------------------------ |
| HoodieError                | User **&lt;username>** could not be found |
| HoodieError                | No email address found               |
| HoodieError                | Failed to send password reset email  |
| HoodieConnectionError      | Could not connect to Server          |

##### Example

<pre><code>hoodie.account.resetPassword('joe@example.com')
  .done(showSuccessMessage)
  .fail(showErrorMessage)
</code></pre>

##### Notes
> We currently generate a new password and send it out via email.
> In future, we will send a token instead, see [#360](https://github.com/hoodiehq/hoodie.js/issues/360)


<a id="accountdestroy"></a>
### account.destroy() 
**version:**      *> 0.2.0* 

<pre><code>hoodie.account.destroy();</code></pre>


##### Arguments

_no arguments_

##### Resolves with

_resolves without argument_

##### Rejects with

| error                      | message                              |
| -------------------------- | ------------------------------------ |
| HoodieConnectionError      | Could not connect to Server          |

##### Example

<pre><code>hoodie.account.destroy()
  .done(showSuccessMessage)
  .fail(showErrorMessage)
</code></pre>

##### Notes
> - When a user account gets destroy, the user's database gets removed and cannot be recovered.
> - We currently do not ask for the user's password, but plan to do so in the future [#55](https://github.com/hoodiehq/hoodie.js/issues/55)


<a id="accountevents"></a>
## Account events

| name | arguments | description |
| ---- | --------- | ----------- |
| signup | username | triggered after user successfully signed up |
| signin | username | triggered after user successfully signed in |
| signout | username | triggered after user successfully signed out |
| reauthenticated | username | triggered after user successfully signed in with current username. See **error:unauthenticated** event for more information. |
| changepassword | - | triggered after user successfully changed the password |
| changeusername | username | triggered after user successfully changed the username |
| passwordreset | username | triggered after password has been resetted successfully |
| error:passwordreset | error, username | An error occured when trying to reset the password for **username** |
| error:unauthenticated | error, username | The current user has no valid session anymore and needs to reauthenticate. As Hoodie works offline, it can get into a state where a user is signed in with data stored in the browser, but without a valid session, so e.g. sync does not work anymore. In that case, the **error:unauthenticated** is triggered and the user should sign in with the current username again. |


##### Example

<pre><code>hoodie.account.on('signup signin', redirectToDashboard)
hoodie.account.on('error:unauthenticated', showSignInForm)
</code></pre>
