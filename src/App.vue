<template>
  <div id="app">
    <h2>Vue.js WebSocket Chat</h2>

    <div class="chat-container">
      <div class="messages">
        <div 
          v-for="(msg, index) in messages" 
          :key="index" 
          :class="['message', { 'my-message': msg.sender === senderName, 'other-message': msg.sender !== senderName }]"
        >
          <strong>{{ msg.sender }}:</strong> {{ msg.text }}
        </div>
      </div>

      <div class="input-container">
        <input
          v-model="senderName"
          placeholder="Enter your name"
          class="name-input"
        />
        <input
          v-model="message"
          placeholder="Type a message"
          class="message-input"
        />
        <button @click="sendMessage">Send Message</button>
      </div>
    </div>
  </div>
</template>


<script>
export default {
  name: 'App',
  data() {
    return {
      connection: null,
      messages: [],
      message: '',
      senderName: ''
    };
  },
  methods: {
    sendMessage() {
      if (this.message.trim() !== '' && this.senderName.trim() !== '') {
        const messageToSend = { sender: this.senderName, text: this.message };
        this.connection.send(JSON.stringify(messageToSend));
        this.messages.push({ sender: 'You', text: this.message });
        this.message = ''; // Clear the input field
      }
    }
  },
  created() {
    console.log("Starting connection to WebSocket Server");
    this.connection = new WebSocket("ws://localhost:8080"); // Adjust URL as needed

    this.connection.onmessage = (event) => {
      if (event.data instanceof Blob) {
        const reader = new FileReader();
        reader.onload = () => {
          try {
            const textMessage = reader.result;
            // Parse the JSON string into an object
            const messageObject = JSON.parse(textMessage);
            this.messages.push(messageObject); // Add received message object to local messages list
          } catch (error) {
            console.error('Error parsing message from Blob:', error); // Log any parsing errors
          }
        };
        reader.readAsText(event.data); // Read the Blob as text
      } else {
        // If it's already a string
        try {
          const messageObject = JSON.parse(event.data); // Parse the JSON string into an object
          this.messages.push(messageObject); // Add received message object to local messages list
        } catch (error) {
          console.error('Error parsing message:', error); // Log any parsing errors
        }
      }
    };

    this.connection.onopen = () => {
      console.log("Successfully connected to the WebSocket server...");
    };
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.chat-container {
  width: 400px;
  margin: 0 auto;
  border: 1px solid #ccc;
  border-radius: 5px;
  padding: 10px;
  background-color: #f9f9f9;
}

.messages {
  height: 300px;
  overflow-y: scroll;
  margin-bottom: 10px;
  border: 1px solid #ddd;
  padding: 10px;
  background-color: white;
}

.message {
  margin-bottom: 5px;
  padding: 8px;
  border-radius: 4px;
  max-width: 70%; /* Limit message width */
  word-wrap: break-word; /* Allow long words to break onto the next line */
}

.my-message {
  background-color: #dcf8c6; /* Light green background for sender's messages */
  text-align: right; /* Align text to the right */
  margin-left: auto; /* Push the message to the right */
}

.other-message {
  background-color: #f1f0f0; /* Light gray background for other messages */
  text-align: left; /* Align text to the left */
  margin-right: auto; /* Push the message to the left */
}

.input-container {
  display: flex;
  flex-direction: column;
}

.name-input,
.message-input {
  padding: 10px;
  margin-bottom: 5px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  padding: 10px;
  border: none;
  border-radius: 4px;
  background-color: #42b983;
  color: white;
  cursor: pointer;
}

button:hover {
  background-color: #369b70;
}
</style>
