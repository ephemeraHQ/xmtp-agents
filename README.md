<div align="center">

[![GitHub release](https://img.shields.io/github/release/ephemerahq/xmtp-agents.svg)](https://github.com/huggingface/smolagents/releases)
[![MIT License](https://img.shields.io/github/license/ephemerahq/xmtp-agents)](https://github.com/ephemerahq/xmtp-agents/blob/main/LICENSE)

<img src="media/logo.png" alt="Logo" width="60" />

# xmtp-agents

</div>

[`@xmtp/agent-starter`](https://github.com/ephemeraHQ/xmtp-agents/tree/main/packages/agent-starter).
is a library for building agents that communicate in a secure and interoperable way over the [XMTP](https://xmtp.org/) network.

#### Why XMTP?

- **End-to-end & compliant**: The server only sees ciphertext, meeting strict security and regulatory standards.
- **Open-source & trustless**: Built on top of the [MLS](https://messaginglayersecurity.rocks/) protocol, it replaces trust in centralized certificate authorities with cryptographic proofs.
- **Privacy & metadata protection**: Offers anonymous or pseudonymous usage with no tracking of sender routes, IPs, or device and message timestamps.
- **Decentralized**: Operates on a peer-to-peer network, eliminating single points of failure.
- **Multi-tenant**: Allows multi-agent multi-human confidential communication over MLS group chats.

> See [FAQ](https://docs.xmtp.org/intro/faq) for more detailed information.

## Setup

```bash
yarn add @xmtp/agent-starter
```

#### Environment variables

To run your XMTP agent, you need two keys:

```bash
ENCRYPTION_KEY= # the private key of the wallet
FIXED_KEY= # a second encryption key for encryption (can be random)
```

> See [encryption keys](https://github.com/ephemeraHQ/xmtp-agents/tree/main/packages/agent-starter/README.md#encryption-keys) to learn more.

## Basic usage

These are the steps to initialize the XMTP listener and send messages.

- `ENCRYPTION_KEY`: The private key of the wallet that will be used to send or receive messages.

```tsx
import { xmtpClient } from "@xmtp/agent-starter";

async function main() {
  const agent = await xmtpClient({
    encryptionKey: process.env.ENCRYPTION_KEY as string,
    onMessage: async (message: Message) => {
        console.log(`Decoded message: ${message.content.text}`);
        console.log(`from ${message.sender.address}`)

        // Your AI model response
        const response = await api("Hi, how are you?");

        //Send text message
        await agent.send({
          message: response,
          originalMessage: message,
        });
      };
  });

  console.log("Agent is up and running on address " + agent.address);
}

main().catch(console.error);
```

#### Address availability

Returns `true` if an address is reachable on the xmtp network

```typescript
const isOnXMTP = await agent.canMessage(address);
```

## Examples

Various examples and tutorials to help you get started with creating and deploying your own agents using XMTP.

- [gated-group](/examples/gated-group/): Create a gated group chat that verifies NFT ownership using Alchemy.
- [gm](/examples/gm/): A simple agent that replies with "GM".
- [gpt](/examples/gpt): A simple agent that interacts with OpenAI APIs.
- [express](/examples/express/): Communicate with traditional APIs using xmtp e2ee

> See all the available [examples](/examples/).

## Deployment

Learn how to deploy with [Railway](/examples/railway/) or [Replit](/examples/replit/)

## Groups

> [!NOTE]
> You need to add the agent **as a member** to the group.

To create a group from your agent, you can use the following code:

```tsx
const group = await agent?.conversations.newGroup([address1, address2]);
```

As an admin you can add members to the group.

```tsx
// get the group
await group.sync();
//By address
await group.addMembers([0xaddresses]);
```

> To learn more about groups, read the [XMTP documentation](https://docs.xmtp.org).

## Message handling

`agent-starter` provides an abstraction to XMTP [content types](https://github.com/xmtp/xmtp-js/tree/main/content-types) to make it easier for devs to integrate different types of messages.

### Receiving messages

All new messages trigger the `onMessage` callback:

```tsx
const onMessage = async (message: Message) => {
  console.log(
    `Decoded message: ${message.content.text} from ${message.sender.address}`,
  );

  let typeId = message.typeId;

  if (typeId === "text") {
    // Do something with the text
  } else if (typeId === "reaction") {
    // Do something with the reaction
  } else if (typeId === "reply") {
    // Do something with the `reply`
  } else if (typeId === "attachment") {
    // Do something with the attachment data url
  } else if (typeId === "agent_message") {
    // Do something with the agent message
  } else if (typeId === "group_updated") {
    // Do something with the group updated metadata
  }
};
```

### Sending messages

When you build an app with XMTP, all messages are encoded with a content type to ensure that an XMTP client knows how to encode and decode messages, ensuring interoperability and consistent display of messages across apps.

### Text

Sends a text message.

```tsx
let textMessage: agentMessage = {
  message: "Your message.",
  receivers: ["0x123..."], // optional
  originalMessage: message, // optional
};
await agent.send(textMessage);
```

### Agent message

Allows to send structured metadata over the network that is displayed as plain-text in ecosystem inboxes.

```tsx
let agentMessage: agentMessage = {
  message: "Would you like to approve this transaction?",
  metadata: {
    amount: "10",
    token: "USDC",
  },
  receivers: ["0x123..."], // optional
  originalMessage: message, // optional
  typeId: "agent_message",
};
await agent.send(agentMessage);
```

> See [content-types](https://github.com/xmtp/xmtp-js/tree/main/content-types/content-type-reaction) for reference

## Web inbox

Interact with the XMTP protocol using [xmtp.chat](https://xmtp.chat) the official web inbox for developers using the latest version powered by MLS.

![alt text](/media/chat.png)

> [!WARNING]
> This React app isn't a complete solution. For example, the list of conversations doesn't update when new messages arrive in existing conversations.

## Lookup library

This library helps you to lookup identities into EVM addresses compatible with XMTP.

```tsx
import { lookup } from "@xmtp/lookup";

const identifier = "vitalik.eth";
const info = await lookup(identifier);

console.log(info);
/*
{
  ensDomain: 'vitalik.eth',
  address: '0x1234...',
  preferredName: 'vitalik.eth',
  converseUsername: '',
  avatar: 'https://...',
  converseEndpoint: 'https://converse.xyz/...'
}
*/
```

> Learn more about [`lookup`](/packages/lookup/) library

## Quickstart

```bash
# clone the repository
git clone https://github.com/ephemeraHQ/xmtp-agents/
cd xmtp-agents

# install dependencies
yarn install

# build
yarn build

# or run a specific example
yarn examples gm
```

Use a `.env` file for your environment variables:

```bash
ENCRYPTION_KEY= # the private key of the wallet
FIXED_KEY= # a second encryption key for encryption (can be random)
```
