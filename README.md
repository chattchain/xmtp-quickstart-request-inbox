# Add user consent to an existing JavaScript app built with XMTP

Managing user consent is essential for enhancing privacy and the user experience. Providing user consent features in your app gives your users better control over who can send them messages.

If you already have an XMTP app, integrating universal allow/block features becomes crucial. This guide walks you through adding consent logic to your app.

![](video.gif)

### Initialize XMTP client with consent

Here's your existing `initXmtpWithKeys` function, updated to include the consent refresh logic.

```jsx
const initXmtpWithKeys = async function () {
  // ... previous code
  const xmtp = await Client.create(wallet);
  // Refresh the consent list to make sure your application is up-to-date with the network
  await xmtp.contacts.refreshConsentList();
};
```

### Filtering conversations based on consent

Using the `consentState` property of the conversation object, you can filter the conversations based on consent.

```jsx
// Filtering based on consent state
const allowed = conversations.filter(
  (conversation) => conversation.consentState === "allowed",
);
const requests = conversations.filter(
  (conversation) => conversation.consentState === "unknown",
);
```

### Request inbox

You can now create a separate inbox for requests. This inbox will only show the conversations that have not been allowed yet.

```jsx
{
  activeTab === "requests" ? (
    <button
      style={styles.conversationListItem}
      onClick={() => setActiveTab("allowed")}>
      ← Allowed
    </button>
  ) : (
    <button
      style={styles.conversationListItem}
      onClick={() => setActiveTab("requests")}>
      Requests →
    </button>
  );
}
```

### Refresh consent when opening a conversation

To ensure that your application respects the latest user consent preferences, it's important to refresh the consent state every time a conversation is opened. This can be done by calling the `refreshConsentList` method of the XMTP client before any interaction within a conversation occurs.

```jsx
// Function to select and open a conversation
const openConversation = async (conversation) => {
  // Refresh the consent list to make sure your application is up-to-date with the network
  await client.contacts.refreshConsentList();
  // Now it's safe to open the conversation
  setSelectedConversation(conversation);
};
```

### Allow and deny actions

When you open a conversation on the request tab, you can show a popup with the allow and deny actions. You can use the `consentState` property of the conversation object to show the popup only when the consent state is `unknown`.

```jsx
// Inside your MessageContainer component
const [showPopup, setShowPopup] = useState(
  conversation.consentState === "unknown",
);

// Function to handle the acceptance of a contact
const handleAccept = async () => {
  // Refresh the consent list first
  await client.contacts.refreshConsentList();
  // Allow the contact
  await client.contacts.allow([conversation.peerAddress]);
  // Hide the popup
  setShowPopup(false);
  // Refresh the consent list to make sure your application is up-to-date with the network
  await client.contacts.refreshConsentList();
  // Log the acceptance
  console.log("accepted", conversation.peerAddress);
};

// Function to handle the blocking of a contact
const handleBlock = async () => {
  // Refresh the consent list first
  await client.contacts.refreshConsentList();
  // Block the contact
  await client.contacts.deny([conversation.peerAddress]);
  // Hide the popup
  setShowPopup(false);
  // Refresh the consent list
  await client.contacts.refreshConsentList();
  // Log the blocking
  console.log("denied", conversation.peerAddress);
};
```

### Updating consent on message send

A user's response to the conversation is considered consent. Previously to send a message we need to ensure the consent state is correctly set to "allowed".

```jsx
const handleSendMessage = async (newMessage) => {
  if (!newMessage.trim()) {
    alert("empty message");
    return;
  }
  if (conversation && conversation.peerAddress) {
    // Refresh the consent list first
    await client.contacts.refreshConsentList();
    // Update the consent state to "allowed"
    await client.contacts.allow([conversation.peerAddress]);
    // Send the message
    await conversation.send(newMessage);
  } else if (conversation) {
    // Crearting a new conversation also sets the consent state to "allowed"
    const conv = await client.conversations.newConversation(searchTerm);
    selectConversation(conv);
    await conv.send(newMessage);
  }
};
```


