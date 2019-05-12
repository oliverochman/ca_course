So the login functionality works now, but do we really want to test against our live backend? In order to not do that, we can set up mocks for this.

Whenever our App is making a network request, we can ‘hijack’ in the middle and respond as we want to. This means that the response payload is completely in our control.

This is what we will do in the `userCanLogin.spec.js` that we added in the last section.

```js
describe('User can log in', () => {
  it('successfully', () => {
    cy.visit('http://localhost:3001');
    cy.server();
    cy.route({
      method: 'POST',
      url: 'http://localhost:3000/api/v1/auth/sign_in',
      response: 'fixture:login.json',
      headers: {
        "uid": "user@mail.com"
      }
    })
    cy.get('#login').click();
    cy.get('#login-form').within(() => {
      cy.get('#email').type('user@mail.com')
      cy.get('#password').type('password')
      cy.get('button').click()
    })
    cy.contains('Hi user@mail.com')
  })

  it('with invalid credentials', () => {
    cy.visit('http://localhost:3001');
    cy.server();
    cy.route({
      method: 'POST',
      url: 'http://localhost:3000/api/v1/auth/sign_in',
      status: "401",
      response: {
        "errors": [
          "Invalid login credentials. Please try again."
        ],
        "success": false
      }     
    })
    cy.get('#login').click();
    cy.get('#login-form').within(() => {
      cy.get('#email').type('user@mail.com')
      cy.get('#password').type('wrongpassword')
      cy.get('button').click()
    })
    cy.contains('Invalid login credentials. Please try again.')
  })
})
```

So in the first `it` block where we have the successfull login scenario, we tell cypress to intefere with the request that the application will try to make when the button that submits the user credentials. The response we will mock out will be defined in another file that we will call `login.json`. The location of this file will be in the fixtures folder that is in the cypress folder.

Create the file and make the file look like this:

`touch cypress/fixtures/login.json`

```json
{
  "data": {
      "allow_password_change": false,
      "email": "user@mail.com",
      "id": 4,{
  "data": {
      "allow_password_change": false,
      "email": "user@mail.com",
      "id": 4,
      "image": null,
      "name": "John Doe",
      "nickname": "Joe",
      "provider": "email",
      "uid": "user@mail.com"
  }
}
      "image": null,
      "name": "John Doe",
      "nickname": "Joe",
      "provider": "email",
      "uid": "user@mail.com"
  }
}
```
We model this fixture file to how the response from the backend looks like.

In the other scenario we test the sad path, if someone fills in the wrong credentials. Here we do not use the same fixture file. We define the response within the feature file here, because we wont use that response in any other place then here and it is really short. If you want to keep your tests cleaner, you can refactor this and create a new fixture file for that response.




