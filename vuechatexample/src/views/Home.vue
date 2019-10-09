<template>
  <div class="home">
    <div id="username-page" v-show="usernamePage">
        <div class="username-page-container">
            <h1 class="title">Informe seu usuário</h1>
            <form id="usernameForm" name="usernameForm">
                <div class="form-group">
                    <input type="text" id="name" v-model="username" placeholder="Usuário" autocomplete="off" class="form-control" />
                </div>
                <div class="form-group">
                    <button type="button" class="accent username-submit" @click="conectar">Entrar Chat</button>
                </div>
            </form>
        </div>
    </div>

    <div id="chat-page" class="hidden" v-show="chatPage">
        <div class="chat-container">
            <div class="chat-header">
                <h2>Spring WebSocket Chat Demo</h2>
            </div>
            <div class="connecting">
                Conectando... <p>{{connectElement}}</p>
            </div>
            <ul id="messageArea">
                <li v-for="message in messages">{{message}}</li>
            </ul>
            <form id="messageForm" name="messageForm">
                <div class="form-group">
                    <div class="input-group clearfix">
                        <input type="text" id="message" v-model="messageInput" placeholder="Digite uma mensagem..." autocomplete="off" class="form-control"/>
                        <button type="button" class="primary" @click="sendMessage">Enviar</button>
                    </div>
                </div>
            </form>
        </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'home',
  components: {
  },
  data() {
    return {
      stompClient: null,
      username: null,
      usernamePage: true,
      chatPage: false,
      messages: [],
      connectElement: '',
      messageInput: ''
    }
  },
  methods: {
    conectar() {
      console.log(this.username);
      if (this.username) {
        this.usernamePage = false;
        this.chatPage = true;

        var socket = new SockJS('http://localhost:8080/ws');
        this.stompClient = Stomp.over(socket);
        this.stompClient.connect({}, this.onConnected, this.onError);
      }
    },
    onConnected() {
      this.stompClient.subscribe('/topic/public', this.onMessageReceived);

      // enviar sua mensagem para o servidor
      this.stompClient.send("/app/chat.addUser",
        {},
        JSON.stringify({sender: this.username, type: 'JOIN'})
      );
    },
    onError() {
      this.connectElement = "Não foi possível conectar com o Websocket";
    },
    sendMessage() {
      var messageContent = this.messageInput;

      if(messageContent && this.stompClient) {
        var chatMessage = {
          sender: this.username,
          content: this.messageInput,
          type: 'CHAT'
        };

        this.stompClient.send("/app/chat.sendMessage", {}, JSON.stringify(chatMessage));
        this.messageInput = '';
      }
    },
    onMessageReceived(payload) {
      var message = JSON.parse(payload.body);

      if(message.type === 'JOIN') {
          message.content = message.sender + ' joined!';
      } else if (message.type === 'LEAVE') {
          message.content = message.sender + ' left!';
      } else {
          message.content = message.sender + ': ' + message.content;
      }

      this.messages.push(message.content);
    }
  }
}
</script>
