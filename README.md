# Real-time-chat-application
import React, { useEffect, useState, useRef } from "react";
import { io } from "socket.io-client";

const SOCKET_URL = "http://localhost:5000"; // backend port
const socket = io(SOCKET_URL);

function App() {
  const [username, setUsername] = useState(() => localStorage.getItem('username') || '');
  const [room, setRoom] = useState('general');
  const [message, setMessage] = useState("");
  const [chat, setChat] = useState([]);
  const messagesEndRef = useRef(null);

  useEffect(() => {
    socket.on("receive_message", (msg) => {
      setChat(prev => [...prev, msg]);
    });

    return () => {
      socket.off("receive_message");
    };
  }, []);

  useEffect(() => {
    messagesEndRef.current?.scrollIntoView({ behavior: "smooth" });
  }, [chat]);

  const joinRoom = (e) => {
    e.preventDefault();
    if (!username.trim()) {
      alert("Enter a username first");
      return;
    }
    localStorage.setItem('username', username);
    setChat([]);
    socket.emit("join_room", { room, username });
  };

  const sendMessage = (e) => {
    e.preventDefault();
    if (!message.trim()) return;
    const msgObj = {
      username: username || "Anonymous",
      content: message.trim(),
      room,
      time: new Date().toLocaleTimeString()
    };
    socket.emit("send_message", msgObj);
    setMessage(""); // no local echo here
  };

  return (
    <div style={{ maxWidth: 700, margin: "30px auto", fontFamily: "Arial, sans-serif" }}>
      <h2>💬 Real-Time Chat</h2>

      <form onSubmit={joinRoom} style={{ marginBottom: 12, display: "flex", gap: 8 }}>
        <input
          placeholder="Your name"
          value={username}
          onChange={e => setUsername(e.target.value)}
          style={{ padding: 8, flex: "0 0 160px" }}
        />
        <input
          placeholder="Room (eg: general)"
          value={room}
          onChange={e => setRoom(e.target.value)}
          style={{ padding: 8, flex: "0 0 160px" }}
        />
        <button type="submit" style={{ padding: "8px 12px" }}>Join Room</button>
      </form>

      <div style={{
        border: "1px solid #ddd",
        borderRadius: 8,
        height: 400,
        padding: 12,
        overflowY: "auto",
        background: "#fafafa"
      }}>
        {chat.map((m, i) => (
          <div key={i} style={{
            marginBottom: 8, padding: 6,
            background: m.username === username ? "#e6f7ff" : "#fff",
            borderRadius: 6
          }}>
            <div style={{ fontSize: 12, color: "#666" }}>
              <strong>{m.username}</strong> <span style={{ marginLeft: 8 }}>{m.time}</span>
              <span style={{ float: "right", fontSize: 11, color: "#999" }}>{m.room || "global"}</span>
            </div>
            <div style={{ marginTop: 4 }}>{m.content}</div>
          </div>
        ))}
        <div ref={messagesEndRef} />
      </div>

      <form onSubmit={sendMessage} style={{ marginTop: 12, display: "flex", gap: 8 }}>
        <input
          placeholder="Type your message..."
          value={message}
          onChange={e => setMessage(e.target.value)}
          style={{ padding: 10, flex: 1 }}
        />
        <button type="submit" style={{ padding: "10px 16px" }}>Send</button>
      </form>
    </div>
  );
}

export default App;
