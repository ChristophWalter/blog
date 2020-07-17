# Testing an email workflow

End-to-end testing of an email workflow (e.g. newsletter subscription) can be tricky as you do not want to send spam mails from your test runs to real email addresses. But there are some ways to do it.

## How can this be done?
### Only submit your tests against a mocked backend
This will not generate any spam and proves that your frontend submits the data correctly. But it will not detect failures in your backend and therefore it is not a real end-to-end test.

### Only submit your tests in a testing environment with configured email receiver
This will send spam mails, but only to your (fake) account. A well written test will now detect if the backend responds faulty. But you still can not (or not that easy) proof that an email with the correct content is sent.

### Use a mock email server in your testing environment
This will keep the email spam within your testing environment, which is great. Then you can also easily access those mails from your test. Make sure to send an id with your emails to make sure that it belongs to the current test run.

## Technical implementation
### Run test assertions or actions based on environment
All these ideas require to run assertions or commands based on the environment. For example you want your test to stop before submitting in production environment. In Nightwatch.js it is possible to [skip whole test suits based on tags](https://nightwatchjs.org/guide/running-tests/#test-tags) out of the box. But you need to write your own command if you want to skip single assertions. This could look like the following (simplified):

```js
// you should probably write a custom assertions for this function
function skipForTags(tags, callback) {
  const skiptags = this.api.globals.test_settings.skiptags
  if (tags.find(tag => skiptags.includes(tag))) {
    log('info', `Skipping some assertions for tags "${tags}"`)
  } else {
    callback()
  }
}

module.exports = {
  'should submit and show a success message': function(client) {
    skipForTags(['production'], () => {
       client.click('.submit-button')
       client.assert.elementPresent(".success-dialog");
    })
  }
}
```

### Using a mock email server
There are some cool frameworks which support this. One of them is [mailhog](https://github.com/mailhog/MailHog). You can try it locally via `docker run -p 1025:1025 -p 8025:8025 mailhog/mailhog` and setting your smtp setting to localhost and port 1025. A dashboard for mailhog will be visible at http://localhost:8025.

Then you can add an additional assertion to your test which checks if the prior sent email was received. The [mailhog client library](https://www.npmjs.com/package/mailhog) will help you for this.

The following example uses Nightwatch.js and will use a random id to identify the email. This id was generated while filling out the form in previous steps of this test.

Besides that I needed to fail the test manually if the email was not found. This is required as an error within promise chains did not fail the test automatically.

```js
const mailhog = require('mailhog')()

'received the email': function(client) {
  client.page.main().ifLocalhost(() => {
    console.log(` ${logSymbols.info} Checking Mailhog for email`)
    mailhog
      .messages()
      .then(messages => {
        let result = messages.items.find(message =>
          message.Content.Body.includes(this.randomId)
        )
        client.expect(result).to.not.be.undefined
      })
      .catch(e => {
        client.assert.fail(e)
      })
  })
},
```
