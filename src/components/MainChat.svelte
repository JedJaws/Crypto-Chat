<script>
  import { setContext } from "svelte";
  import { writable } from "svelte/store";
  import { ethers } from "ethers";

  import { abi } from "../abi";
  import ChatMessage from "./ChatMessage.svelte";
  let chatHistory = writable([]);

  let publicKey = writable("");
  let myUsername = writable("");
  let myContract = writable(null);
  let showLogin = true;

  const CONTRACT_ADDRESS = "0x0c66406F7453acd6C5EC963eB277667F6979c109";

  setContext("chatHistory", chatHistory);

  let getTestMessage = () => {
    let message = {
      text: document.querySelector("input[name=messageText]").value,
      time: "12:00 pm",
      user: "Joe",
    };
    return message;
  };

  let sendTestMessage = () => {
    let message = getMessage();
    chatHistory.update((messages) => {
      return [...messages, message];
    });
  };

  // Login to Metamask and check the if the user exists else creates one
  async function login() {
    let res = await connectToMetamask();
    if (res === true) {
      const provider = new ethers.providers.Web3Provider(window.ethereum);
      const signer = provider.getSigner();
      try {
        myContract = new ethers.Contract(CONTRACT_ADDRESS, abi, signer);
        const address = await signer.getAddress();
        let present = await myContract.checkUserExists(address);
        let username;
        if (present) username = await myContract.getUsername(address);
        else {
          username = prompt("Enter a username", "Guest");
          if (username === "") username = "Guest";
          await myContract.createAccount(username);
        }
        myUsername = username;
        publicKey = address;
        showLogin = false;
      } catch (err) {
        alert("CONTRACT_ADDRESS not set properly!");
      }
    } else {
      alert("Couldn't connect to Metamask");
    }
  }

  // Check if the Metamask connects
  async function connectToMetamask() {
    try {
      await window.ethereum.enable();
      return true;
    } catch (err) {
      return false;
    }
  }

  // Add a friend to the users' Friends List
  async function addChat(name, publicKey) {
    try {
      let present = await myContract.checkUserExists(publicKey);
      if (!present) {
        alert("Address not found: Ask them to join the app :)");
        return;
      }
      try {
        await myContract.addFriend(publicKey, name);
        const frnd = { name: name, publicKey: publicKey };
        setFriends(friends.concat(frnd));
      } catch (err) {
        alert(
          "Friend already added! You can't be friends with the same person twice ;P"
        );
      }
    } catch (err) {
      alert("Invalid address!");
    }
  }

  // Sends messsage to an user
  async function sendMessage(data) {
    if (!(activeChat && activeChat.publicKey)) return;
    const recieverAddress = activeChat.publicKey;
    await myContract.sendMessage(recieverAddress, data);
  }

  // Fetch chat messages with a friend
  async function getMessage(friendsPublicKey) {
    let nickname;
    let messages = [];
    friends.forEach((item) => {
      if (item.publicKey === friendsPublicKey) nickname = item.name;
    });
    // Get messages
    const data = await myContract.readMessage(friendsPublicKey);
    data.forEach((item) => {
      const timestamp = new Date(1000 * item[1].toNumber()).toUTCString();
      messages.push({
        publicKey: item[0],
        timeStamp: timestamp,
        data: item[2],
      });
    });

    // TODO: Jacob how should we do this lol
    setActiveChat({ friendname: nickname, publicKey: friendsPublicKey });
    chatHistory = messages;
  }
</script>

<div class="main-container">
  {#if showLogin}
    <button on:click={login} class="send-btn">Login</button>
  {:else}
    <p>{myUsername}</p>
    <button on:click={getMessage} class="send-btn">Reload</button>
  {/if}

  <div class="chat-box">
    <!-- Chat messages will be rendered here -->
    <div class="spacer" />
    {#each $chatHistory as message}
      <ChatMessage
        messageText={message.text}
        messageTime={message.time}
        messageUser={message.user}
      />
    {/each}
  </div>
  <form>
    <input type="text" name="messageText" class="message-input" />
    <button type="submit" on:click={sendMessage} class="send-btn">Send</button>
  </form>
</div>

<style>
  /* make button rounded style */
  .main-container {
    display: flex;
    flex-direction: column;
    height: 95vh;
    align-items: center;
  }

  .spacer {
    height: 75vh;
  }
  .chat-box {
    display: flex;
    width: 30%;
    flex-direction: column;
    overflow: auto;
    flex-grow: 1;
    align-items: flex-end;
  }

  .message-input {
    border-radius: 1rem;
    font-size: medium;
    background-color: #3b4248;
    color: white;
    width: 50rem;
    height: 50px;
  }
  .send-btn {
    border-radius: 5rem;
    width: 100px;
    height: 50px;
    background-color: #326b00;
    color: white;
    font-size: 1.5rem;
    border: none;
  }
</style>
