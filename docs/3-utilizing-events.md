# Getting Started with the Nexmo Conversation JS SDK

In this getting started guide we'll cover adding more events to the Conversation we created in the [simple conversation with member invites](2-inviting-members.md) getting started guide. We'll deal with multiple types of events, the ones that come via the conversation, and the ones we send to the conversation.

## Concepts

This guide will introduce you to the following concepts.

- **Conversation Events** - `member:left` and `text:` events that fire on a Conversation when someone does an Action
- **Conversation Actions** - actions that trigger events on a Conversation
- **Conversation History** - an `events` object that stores all text Events happening on a conversation

## Before you begin

- Ensure you have run through the [previous guide](2-inviting-members.md)

## 1 - Update the JavaScript App

We will use the application we already created for [the second getting started guide](2-inviting-members.md). All the basic setup has been done in the previous guides and should be in place. We can now focus on updating the client-side application.

### 1.1 - Add conversation history

The first thing we're going to do is add history to the existing conversation. We're going to add a method that gets all the events that happened in the conversation and displays conversation history in the message feed.

```javascript
showConversationHistory(conversation) {
  conversation.getEvents().then((events) => {
    var eventsHistory = ""
    for (var i = Object.keys(events).length; i > 0; i--) {
      const date = new Date(Date.parse(events[Object.keys(events)[i - 1]].timestamp))
      eventsHistory += `${conversation.members[events[Object.keys(events)[i - 1]].from].name} @ ${date}: <b>${events[Object.keys(events)[i - 1]].body.text}</b><br>`
    }

    this.messageFeed.innerHTML = eventsHistory + this.messageFeed.innerHTML
  })
}
```

Next, update `setupConversationEvents` to call the method we just created

```javascript
setupConversationEvents(conversation) {
  ...

  this.showConversationHistory(conversation)
}
```

### 1.2 - Add text:seen events

Now that we've done that, it's time to learn about sending events into a conversation. In order to do that, we'll update the `text` event listener to acknowledge when a user received a message. We'll update the relevant bit from `setupConversationEvents` to check if a message in the conversation was not sent by the current user, and then call `message.seen()`. What this does, is it fires a `text:seen` event on the conversation.

```javascript
setupConversationEvents(conversation) {
...
  // Bind to events on the conversation
  conversation.on('text', (sender, message) => {
    console.log('*** Message received', sender, message)
    const date = new Date(Date.parse(message.timestamp))
    const text = `${sender.name} @ ${date}: <b>${message.text}</b><br>`
    this.messageFeed.innerHTML = text + this.messageFeed.innerHTML

    if (sender.name !== this.conversation.me.name) {
        message.seen().then(this.eventLogger('text:seen')).catch(this.errorLogger)
    }
  })
...
}
```

We're going to add a listener as well on the conversation for `text:seen`, notifying other members in the conversation that their message was seen by other members of the conversation.

```javascript
setupConversationEvents(conversation) {
...
  conversation.on("text:seen", (data, text) => console.log(`${data.name} saw text: ${text.body.text}`))
}
```

### 1.3 - Add text:typing events

We're going to update the conversation so that we can see when someone in typing. There are two events that enable us to do so, `text:typing:on` and `text:typing:off`, with their corresponding methods on the `conversation` object that fires them, `startTyping()` and `stopTyping()`. We'll fist update `setupConversationEvents` to listen for those events

```javascript
setupConversationEvents(conversation) {
...
  conversation.on("text:typing:off", data => console.log(`${data.name} stopped typing...`))
  conversation.on("text:typing:on", data => console.log(`${data.name} started typing...`))
}
```

Now we're going to fire those events when the user focuses or blurs the message box. We'll update the `setupUserEvents` method in order to do that

```javascript
setupUserEvents() {
...
  this.messageTextarea.addEventListener('focus', () => {
    this.conversation.startTyping().then(this.eventLogger('text:typing:on')).catch(this.errorLogger)
  });
  this.messageTextarea.addEventListener('blur', () => {
    this.conversation.stopTyping().then(this.eventLogger('text:typing:off')).catch(this.errorLogger)
  })
}
```

### 1.4 - Leave a conversation

Finally, we'll add the UI for user to leave a conversation. Let's add the button at the top of the messages area.
```html
<section id="messages">
  <button id="leave">Leave Conversation</button>
  ...
</section>

```

And add the button in the class constructor

```javascript
constructor() {
...
  this.leaveButton = document.getElementById('leave')
}
```

We'll then update the `setupUserEvents` method to trigger `conversation.leave()` when the user clicks the button

```javascript
setupUserEvents() {
...
  this.leaveButton.addEventListener('click', () => {
    this.conversation.leave().then(this.eventLogger('member:left')).catch(this.errorLogger)
  })
}
```

To finish, we're going to add a listener for `member:left` in the `setupConversationEvents` method. We'll also add a helper function to handle updating the message feed when a user joins or leaves a conversation. And refactor the `member:joined` handler to use the new helper function.

```javascript
memberEventHandler(type) {
  return (data, info) => {
    console.log(`*** ${info.user.name} ${type} the conversation`)
    const text = `${info.user.name} @ ${date}: <b>${type} the conversation</b><br>`
    this.messageFeed.innerHTML = text + this.messageFeed.innerHTML
  }
}

setupConversationEvents(conversation) {
  ...
  conversation.on("member:joined", this.memberEventHandler('joined'))
  conversation.on("member:left", this.memberEventHandler('left'))
}
```


### 1.5 - Open the conversation in two browser windows

Now run `index.html` in two side-by-side browser windows, making sure to login with the user name `jamie` in one and with `alice` in the other. Open the developer tools console and start chatting. You'll see events being logged in the console.

Thats's it! Your page should now look something like [this](../examples/3-utilizing-events/index.html).


## Where next?

- Have a look at the [Nexmo Conversation JS SDK API Reference](https://conversation-js-docs.herokuapp.com/)
