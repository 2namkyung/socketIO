# SocketIO 맛보기

- socket은 IP와 PORT를 활용하여 만들어진 네트워크 통신의 End Point.
- 실시간으로 데이터를 서비스 해야한다면 서버도 클라이언트에게 요청을 보낼 수 있는 양방향 통신 필요

```
  "dependencies": {
    "@socket.io/admin-ui": "^0.3.0",
    "express": "^4.18.1",
    "pug": "^3.0.2",
    "socket.io": "^4.5.1",
    "ws": "^8.7.0"
  }
```

```javascript
const httpServer = http.createServer(app);
const wsServer = new Server(httpServer, {
  cors: {
    origin: ["https://admin.socket.io"],
    credentials: true,
  },
});

// socket admin
instrument(wsServer, {
  auth: false,
});

wsServer.on("connection", (socket) => {
  socket["nickname"] = "Anonymous";
  socket.onAny((event) => {
    console.group(`Socket Event : ${event}`);
  });

  socket.on("enter_room", (roomName, nickname, done) => {
    socket["nickname"] = nickname;
    socket.join(roomName);
    done();
    socket
      .to(roomName)
      .emit("welcome", socket.nickname, countUserInRoom(roomName));
    wsServer.sockets.emit("room_change", publicRooms());
  });

  socket.on("disconnecting", () => {
    socket.rooms.forEach((room) =>
      socket.to(room).emit("bye", socket.nickname, countUserInRoom(room) - 1)
    );
  });

  socket.on("disconnect", () => {
    wsServer.sockets.emit("room_change", publicRooms());
  });

  socket.on("new_message", (msg, room, done) => {
    socket.to(room).emit("new_message", `${socket.nickname}: ${msg}`);
    done();
  });

  socket.on("nickname", (nickname) => (socket["nickname"] = nickname));
});

httpServer.listen(3000, handleListen);
```
