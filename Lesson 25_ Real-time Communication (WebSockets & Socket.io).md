# Lesson 25: Real-time Communication (WebSockets & Socket.io)

## 🎯 **Learning Objectives**
- Understand WebSocket protocol and real-time communication
- Implement Socket.io for bidirectional communication
- Create real-time chat applications
- Handle connection management and error handling
- Implement room-based communication and namespaces

## 📚 **Table of Contents**
1. [Introduction to WebSockets](#introduction-to-websockets)
2. [Socket.io Fundamentals](#socketio-fundamentals)
3. [Real-time Chat Application](#real-time-chat-application)
4. [Connection Management](#connection-management)
5. [Rooms and Namespaces](#rooms-and-namespaces)
6. [Authentication & Authorization](#authentication--authorization)
7. [Error Handling & Recovery](#error-handling--recovery)
8. [Performance Optimization](#performance-optimization)
9. [Security Considerations](#security-considerations)
10. [Practical Examples](#practical-examples)

---

## 🌐 **Introduction to WebSockets**

### **WebSocket vs HTTP**
```
HTTP (Traditional)
Client → Server (Request)
Server → Client (Response)
Connection closes after response

WebSocket (Real-time)
Client ↔️ Server (Persistent connection)
Bidirectional communication
Real-time data flow
```

### **When to Use WebSockets**
- **Real-time chat applications**
- **Live notifications and updates**
- **Collaborative editing**
- **Live sports scores and updates**
- **Multiplayer games**
- **Stock market data**
- **IoT device communication**

---

## 🔌 **Socket.io Fundamentals**

### **Basic Socket.io Server**
```javascript
const express = require('express');
const http = require('http');
const socketIo = require('socket.io');
const cors = require('cors');

const app = express();
const server = http.createServer(app);

// Initialize Socket.io with CORS
const io = socketIo(server, {
  cors: {
    origin: ["http://localhost:3000", "http://localhost:3001"],
    methods: ["GET", "POST"],
    credentials: true
  },
  // Connection settings
  pingTimeout: 60000,
  pingInterval: 25000,
  transports: ['websocket', 'polling'],
});

// Middleware
app.use(cors());
app.use(express.json());

// Basic connection handling
io.on('connection', (socket) => {
  console.log(`User connected: ${socket.id}`);

  // Handle disconnection
  socket.on('disconnect', (reason) => {
    console.log(`User disconnected: ${socket.id}, Reason: ${reason}`);
  });

  // Handle custom events
  socket.on('message', (data) => {
    console.log('Message received:', data);
    // Echo the message back
    socket.emit('message', { ...data, serverTimestamp: new Date() });
  });

  // Handle ping
  socket.on('ping', () => {
    socket.emit('pong');
  });
});

const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### **Basic Socket.io Client (React Native)**
```javascript
import React, { useEffect, useState } from 'react';
import { View, Text, TextInput, TouchableOpacity, FlatList, StyleSheet } from 'react-native';
import io from 'socket.io-client';

const ChatApp = () => {
  const [socket, setSocket] = useState(null);
  const [messages, setMessages] = useState([]);
  const [inputMessage, setInputMessage] = useState('');
  const [isConnected, setIsConnected] = useState(false);

  useEffect(() => {
    // Connect to Socket.io server
    const newSocket = io('http://localhost:3000', {
      transports: ['websocket'],
      upgrade: true,
    });

    // Connection event handlers
    newSocket.on('connect', () => {
      console.log('Connected to server');
      setIsConnected(true);
    });

    newSocket.on('disconnect', () => {
      console.log('Disconnected from server');
      setIsConnected(false);
    });

    // Message event handler
    newSocket.on('message', (message) => {
      setMessages(prev => [...prev, {
        id: Date.now(),
        text: message.text,
        timestamp: message.serverTimestamp,
        isMine: false,
      }]);
    });

    // Error handling
    newSocket.on('connect_error', (error) => {
      console.error('Connection error:', error);
    });

    setSocket(newSocket);

    // Cleanup on unmount
    return () => {
      newSocket.disconnect();
    };
  }, []);

  const sendMessage = () => {
    if (!socket || !inputMessage.trim()) return;

    const message = {
      text: inputMessage.trim(),
      timestamp: new Date(),
    };

    // Add message to local state immediately
    setMessages(prev => [...prev, {
      id: Date.now(),
      text: message.text,
      timestamp: message.timestamp,
      isMine: true,
    }]);

    // Send to server
    socket.emit('message', message);
    setInputMessage('');
  };

  const renderMessage = ({ item }) => (
    <View style={[styles.messageContainer, item.isMine ? styles.myMessage : styles.theirMessage]}>
      <Text style={[styles.messageText, item.isMine ? styles.myMessageText : styles.theirMessageText]}>
        {item.text}
      </Text>
      <Text style={[styles.timestamp, item.isMine ? styles.myTimestamp : styles.theirTimestamp]}>
        {new Date(item.timestamp).toLocaleTimeString()}
      </Text>
    </View>
  );

  return (
    <View style={styles.container}>
      <View style={[styles.statusBar, { backgroundColor: isConnected ? '#28a745' : '#dc3545' }]}>
        <Text style={styles.statusText}>
          {isConnected ? 'Connected' : 'Disconnected'}
        </Text>
      </View>

      <FlatList
        data={messages}
        renderItem={renderMessage}
        keyExtractor={(item) => item.id.toString()}
        style={styles.messagesList}
        inverted
      />

      <View style={styles.inputContainer}>
        <TextInput
          style={styles.input}
          placeholder="Type a message..."
          value={inputMessage}
          onChangeText={setInputMessage}
          onSubmitEditing={sendMessage}
          returnKeyType="send"
        />
        <TouchableOpacity style={styles.sendButton} onPress={sendMessage}>
          <Text style={styles.sendButtonText}>Send</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  statusBar: {
    padding: 10,
    alignItems: 'center',
  },
  statusText: {
    color: 'white',
    fontWeight: 'bold',
  },
  messagesList: {
    flex: 1,
    padding: 10,
  },
  messageContainer: {
    maxWidth: '80%',
    padding: 10,
    borderRadius: 10,
    marginBottom: 10,
  },
  myMessage: {
    alignSelf: 'flex-end',
    backgroundColor: '#007AFF',
  },
  theirMessage: {
    alignSelf: 'flex-start',
    backgroundColor: 'white',
  },
  messageText: {
    fontSize: 16,
  },
  myMessageText: {
    color: 'white',
  },
  theirMessageText: {
    color: '#333',
  },
  timestamp: {
    fontSize: 10,
    marginTop: 5,
  },
  myTimestamp: {
    color: 'rgba(255,255,255,0.7)',
    alignSelf: 'flex-end',
  },
  theirTimestamp: {
    color: '#999',
  },
  inputContainer: {
    flexDirection: 'row',
    padding: 10,
    backgroundColor: 'white',
  },
  input: {
    flex: 1,
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 20,
    padding: 10,
    marginRight: 10,
  },
  sendButton: {
    backgroundColor: '#007AFF',
    padding: 10,
    borderRadius: 20,
    justifyContent: 'center',
    minWidth: 60,
  },
  sendButtonText: {
    color: 'white',
    textAlign: 'center',
    fontWeight: 'bold',
  },
});

export default ChatApp;
```

---

## 💬 **Real-time Chat Application**

### **Advanced Chat Server**
```javascript
const express = require('express');
const http = require('http');
const socketIo = require('socket.io');
const jwt = require('jsonwebtoken');
const cors = require('cors');

const app = express();
const server = http.createServer(app);
const io = socketIo(server, {
  cors: {
    origin: ["http://localhost:3000", "http://localhost:3001"],
    methods: ["GET", "POST"],
    credentials: true
  }
});

// In-memory storage (replace with database in production)
const users = new Map(); // socketId -> user
const messages = []; // Message history
const typingUsers = new Set(); // Users currently typing

// JWT authentication middleware for Socket.io
io.use(async (socket, next) => {
  try {
    const token = socket.handshake.auth.token;

    if (!token) {
      return next(new Error('Authentication token required'));
    }

    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    socket.user = decoded;
    next();
  } catch (error) {
    next(new Error('Invalid authentication token'));
  }
});

// Connection handling
io.on('connection', (socket) => {
  console.log(`User connected: ${socket.user.email} (${socket.id})`);

  // Add user to online users
  users.set(socket.id, {
    id: socket.user.id,
    email: socket.user.email,
    socketId: socket.id,
  });

  // Send online users list
  socket.emit('users_online', Array.from(users.values()));

  // Broadcast user joined
  socket.broadcast.emit('user_joined', {
    id: socket.user.id,
    email: socket.user.email,
  });

  // Send recent messages
  socket.emit('recent_messages', messages.slice(-50));

  // Handle new message
  socket.on('send_message', (data) => {
    const message = {
      id: Date.now().toString(),
      text: data.text,
      sender: {
        id: socket.user.id,
        email: socket.user.email,
      },
      timestamp: new Date(),
      type: 'text',
    };

    // Store message
    messages.push(message);
    if (messages.length > 1000) {
      messages.shift(); // Keep only last 1000 messages
    }

    // Broadcast to all connected clients
    io.emit('new_message', message);
  });

  // Handle typing indicators
  socket.on('typing_start', () => {
    typingUsers.add(socket.user.id);
    socket.broadcast.emit('user_typing', {
      userId: socket.user.id,
      email: socket.user.email,
      isTyping: true,
    });
  });

  socket.on('typing_stop', () => {
    typingUsers.delete(socket.user.id);
    socket.broadcast.emit('user_typing', {
      userId: socket.user.id,
      email: socket.user.email,
      isTyping: false,
    });
  });

  // Handle private messages
  socket.on('send_private_message', (data) => {
    const { recipientId, text } = data;

    // Find recipient socket
    const recipientSocket = Array.from(users.values())
      .find(user => user.id === recipientId);

    if (recipientSocket) {
      const privateMessage = {
        id: Date.now().toString(),
        text,
        sender: {
          id: socket.user.id,
          email: socket.user.email,
        },
        recipient: {
          id: recipientId,
        },
        timestamp: new Date(),
        type: 'private',
      };

      // Send to recipient
      io.to(recipientSocket.socketId).emit('private_message', privateMessage);

      // Send confirmation to sender
      socket.emit('private_message_sent', privateMessage);
    } else {
      socket.emit('error', { message: 'User not online' });
    }
  });

  // Handle file sharing
  socket.on('send_file', (data) => {
    const { fileName, fileData, fileType } = data;

    const fileMessage = {
      id: Date.now().toString(),
      fileName,
      fileData,
      fileType,
      sender: {
        id: socket.user.id,
        email: socket.user.email,
      },
      timestamp: new Date(),
      type: 'file',
    };

    io.emit('new_message', fileMessage);
  });

  // Handle disconnection
  socket.on('disconnect', () => {
    console.log(`User disconnected: ${socket.user.email} (${socket.id})`);

    // Remove from online users
    users.delete(socket.id);

    // Remove from typing users
    typingUsers.delete(socket.user.id);

    // Broadcast user left
    socket.broadcast.emit('user_left', {
      id: socket.user.id,
      email: socket.user.email,
    });

    // Update online users list
    io.emit('users_online', Array.from(users.values()));
  });
});

// REST API for additional functionality
app.get('/api/messages', (req, res) => {
  res.json(messages.slice(-100)); // Last 100 messages
});

app.get('/api/users/online', (req, res) => {
  res.json(Array.from(users.values()));
});

const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
  console.log(`Chat server running on port ${PORT}`);
});
```

### **Advanced Chat Client**
```javascript
import React, { useEffect, useState, useRef } from 'react';
import { View, Text, TextInput, TouchableOpacity, FlatList, StyleSheet, Alert } from 'react-native';
import io from 'socket.io-client';

const AdvancedChat = () => {
  const [socket, setSocket] = useState(null);
  const [messages, setMessages] = useState([]);
  const [inputMessage, setInputMessage] = useState('');
  const [onlineUsers, setOnlineUsers] = useState([]);
  const [typingUsers, setTypingUsers] = useState([]);
  const [isConnected, setIsConnected] = useState(false);
  const [isTyping, setIsTyping] = useState(false);

  const typingTimeoutRef = useRef(null);

  useEffect(() => {
    // Connect to Socket.io server with authentication
    const token = 'your-jwt-token-here'; // Get from AsyncStorage or context

    const newSocket = io('http://localhost:3000', {
      auth: {
        token,
      },
      transports: ['websocket'],
    });

    // Connection events
    newSocket.on('connect', () => {
      console.log('Connected to chat server');
      setIsConnected(true);
    });

    newSocket.on('disconnect', () => {
      console.log('Disconnected from chat server');
      setIsConnected(false);
    });

    // Message events
    newSocket.on('recent_messages', (recentMessages) => {
      setMessages(recentMessages);
    });

    newSocket.on('new_message', (message) => {
      setMessages(prev => [...prev, message]);
    });

    newSocket.on('private_message', (message) => {
      setMessages(prev => [...prev, { ...message, isPrivate: true }]);
      Alert.alert('Private Message', `${message.sender.email}: ${message.text}`);
    });

    // User events
    newSocket.on('users_online', (users) => {
      setOnlineUsers(users);
    });

    newSocket.on('user_joined', (user) => {
      setOnlineUsers(prev => [...prev, user]);
    });

    newSocket.on('user_left', (user) => {
      setOnlineUsers(prev => prev.filter(u => u.id !== user.id));
    });

    // Typing events
    newSocket.on('user_typing', (data) => {
      if (data.isTyping) {
        setTypingUsers(prev => [...prev, data]);
      } else {
        setTypingUsers(prev => prev.filter(u => u.userId !== data.userId));
      }
    });

    // Error handling
    newSocket.on('connect_error', (error) => {
      console.error('Connection error:', error);
      Alert.alert('Connection Error', 'Failed to connect to chat server');
    });

    setSocket(newSocket);

    return () => {
      newSocket.disconnect();
    };
  }, []);

  const sendMessage = () => {
    if (!socket || !inputMessage.trim()) return;

    socket.emit('send_message', {
      text: inputMessage.trim(),
    });

    setInputMessage('');
    stopTyping();
  };

  const sendPrivateMessage = (recipientId) => {
    if (!socket || !inputMessage.trim()) return;

    socket.emit('send_private_message', {
      recipientId,
      text: inputMessage.trim(),
    });

    setInputMessage('');
    stopTyping();
  };

  const handleTyping = () => {
    if (!socket || isTyping) return;

    setIsTyping(true);
    socket.emit('typing_start');

    // Clear previous timeout
    if (typingTimeoutRef.current) {
      clearTimeout(typingTimeoutRef.current);
    }

    // Stop typing after 3 seconds of inactivity
    typingTimeoutRef.current = setTimeout(() => {
      stopTyping();
    }, 3000);
  };

  const stopTyping = () => {
    if (!socket || !isTyping) return;

    setIsTyping(false);
    socket.emit('typing_stop');

    if (typingTimeoutRef.current) {
      clearTimeout(typingTimeoutRef.current);
    }
  };

  const renderMessage = ({ item }) => (
    <View style={[styles.messageContainer, item.isPrivate && styles.privateMessage]}>
      <View style={styles.messageHeader}>
        <Text style={styles.senderName}>
          {item.sender.email}
          {item.isPrivate && ' (Private)'}
        </Text>
        <Text style={styles.timestamp}>
          {new Date(item.timestamp).toLocaleTimeString()}
        </Text>
      </View>
      <Text style={styles.messageText}>{item.text}</Text>
    </View>
  );

  const renderOnlineUser = ({ item }) => (
    <TouchableOpacity
      style={styles.userItem}
      onPress={() => {
        Alert.alert(
          'Send Private Message',
          `Send a private message to ${item.email}?`,
          [
            { text: 'Cancel', style: 'cancel' },
            { text: 'Send', onPress: () => sendPrivateMessage(item.id) },
          ]
        );
      }}
    >
      <View style={styles.onlineIndicator} />
      <Text style={styles.userEmail}>{item.email}</Text>
    </TouchableOpacity>
  );

  return (
    <View style={styles.container}>
      <View style={[styles.statusBar, { backgroundColor: isConnected ? '#28a745' : '#dc3545' }]}>
        <Text style={styles.statusText}>
          {isConnected ? `Online (${onlineUsers.length})` : 'Offline'}
        </Text>
      </View>

      <View style={styles.content}>
        <View style={styles.chatArea}>
          <FlatList
            data={messages}
            renderItem={renderMessage}
            keyExtractor={(item) => item.id}
            style={styles.messagesList}
            inverted
          />

          {typingUsers.length > 0 && (
            <Text style={styles.typingIndicator}>
              {typingUsers.map(u => u.email).join(', ')} {typingUsers.length === 1 ? 'is' : 'are'} typing...
            </Text>
          )}
        </View>

        <View style={styles.sidebar}>
          <Text style={styles.sidebarTitle}>Online Users</Text>
          <FlatList
            data={onlineUsers}
            renderItem={renderOnlineUser}
            keyExtractor={(item) => item.id}
            style={styles.usersList}
          />
        </View>
      </View>

      <View style={styles.inputContainer}>
        <TextInput
          style={styles.input}
          placeholder="Type a message..."
          value={inputMessage}
          onChangeText={(text) => {
            setInputMessage(text);
            handleTyping();
          }}
          onSubmitEditing={sendMessage}
          returnKeyType="send"
        />
        <TouchableOpacity style={styles.sendButton} onPress={sendMessage}>
          <Text style={styles.sendButtonText}>Send</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  statusBar: {
    padding: 10,
    alignItems: 'center',
  },
  statusText: {
    color: 'white',
    fontWeight: 'bold',
  },
  content: {
    flex: 1,
    flexDirection: 'row',
  },
  chatArea: {
    flex: 1,
  },
  messagesList: {
    flex: 1,
    padding: 10,
  },
  messageContainer: {
    backgroundColor: 'white',
    padding: 10,
    borderRadius: 10,
    marginBottom: 10,
    maxWidth: '80%',
  },
  privateMessage: {
    backgroundColor: '#fff3cd',
    borderLeftWidth: 4,
    borderLeftColor: '#ffc107',
  },
  messageHeader: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginBottom: 5,
  },
  senderName: {
    fontWeight: 'bold',
    fontSize: 12,
    color: '#666',
  },
  timestamp: {
    fontSize: 10,
    color: '#999',
  },
  messageText: {
    fontSize: 16,
  },
  typingIndicator: {
    fontStyle: 'italic',
    color: '#666',
    padding: 10,
  },
  sidebar: {
    width: 200,
    backgroundColor: 'white',
    borderLeftWidth: 1,
    borderLeftColor: '#ddd',
  },
  sidebarTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    padding: 10,
    borderBottomWidth: 1,
    borderBottomColor: '#ddd',
  },
  usersList: {
    flex: 1,
  },
  userItem: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 10,
    borderBottomWidth: 1,
    borderBottomColor: '#f0f0f0',
  },
  onlineIndicator: {
    width: 8,
    height: 8,
    borderRadius: 4,
    backgroundColor: '#28a745',
    marginRight: 10,
  },
  userEmail: {
    fontSize: 14,
  },
  inputContainer: {
    flexDirection: 'row',
    padding: 10,
    backgroundColor: 'white',
    borderTopWidth: 1,
    borderTopColor: '#ddd',
  },
  input: {
    flex: 1,
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 20,
    padding: 10,
    marginRight: 10,
  },
  sendButton: {
    backgroundColor: '#007AFF',
    padding: 10,
    borderRadius: 20,
    justifyContent: 'center',
    minWidth: 60,
  },
  sendButtonText: {
    color: 'white',
    textAlign: 'center',
    fontWeight: 'bold',
  },
});

export default AdvancedChat;
```

---

## 🔗 **Connection Management**

### **Connection States and Handling**
```javascript
class SocketManager {
  constructor(serverUrl, options = {}) {
    this.serverUrl = serverUrl;
    this.options = {
      reconnection: true,
      reconnectionAttempts: 5,
      reconnectionDelay: 1000,
      timeout: 20000,
      ...options,
    };

    this.socket = null;
    this.isConnected = false;
    this.reconnectionAttempts = 0;
    this.listeners = new Map();
  }

  connect(authToken = null) {
    const connectionOptions = {
      ...this.options,
      auth: authToken ? { token: authToken } : undefined,
    };

    this.socket = io(this.serverUrl, connectionOptions);

    this.setupEventHandlers();
  }

  setupEventHandlers() {
    // Connection events
    this.socket.on('connect', () => {
      console.log('Connected to server');
      this.isConnected = true;
      this.reconnectionAttempts = 0;
      this.emit('connected');
    });

    this.socket.on('disconnect', (reason) => {
      console.log('Disconnected from server:', reason);
      this.isConnected = false;
      this.emit('disconnected', reason);
    });

    this.socket.on('connect_error', (error) => {
      console.error('Connection error:', error);
      this.emit('connection_error', error);
    });

    this.socket.on('reconnect', (attemptNumber) => {
      console.log('Reconnected to server after', attemptNumber, 'attempts');
      this.emit('reconnected', attemptNumber);
    });

    this.socket.on('reconnect_error', (error) => {
      console.error('Reconnection error:', error);
      this.reconnectionAttempts++;
      this.emit('reconnection_error', error, this.reconnectionAttempts);
    });

    this.socket.on('reconnect_failed', () => {
      console.error('Failed to reconnect after', this.options.reconnectionAttempts, 'attempts');
      this.emit('reconnection_failed');
    });
  }

  // Event system
  on(event, callback) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    this.listeners.get(event).push(callback);
  }

  off(event, callback) {
    if (this.listeners.has(event)) {
      const callbacks = this.listeners.get(event);
      const index = callbacks.indexOf(callback);
      if (index > -1) {
        callbacks.splice(index, 1);
      }
    }
  }

  emit(event, ...args) {
    if (this.listeners.has(event)) {
      this.listeners.get(event).forEach(callback => {
        callback(...args);
      });
    }
  }

  // Message handling
  send(event, data) {
    if (this.socket && this.isConnected) {
      this.socket.emit(event, data);
    } else {
      console.warn('Socket not connected, cannot send message');
    }
  }

  onMessage(event, callback) {
    if (this.socket) {
      this.socket.on(event, callback);
    }
  }

  // Connection management
  disconnect() {
    if (this.socket) {
      this.socket.disconnect();
      this.socket = null;
      this.isConnected = false;
    }
  }

  reconnect() {
    if (this.socket && !this.isConnected) {
      this.socket.connect();
    }
  }

  // Utility methods
  getConnectionState() {
    return {
      isConnected: this.isConnected,
      id: this.socket?.id,
      reconnectionAttempts: this.reconnectionAttempts,
    };
  }
}

// Usage
const socketManager = new SocketManager('http://localhost:3000');

socketManager.on('connected', () => {
  console.log('Successfully connected!');
});

socketManager.on('disconnected', (reason) => {
  console.log('Disconnected:', reason);
});

socketManager.on('connection_error', (error) => {
  console.error('Connection failed:', error);
});

// Connect with authentication
socketManager.connect('your-jwt-token');
```

---

## 🏠 **Rooms and Namespaces**

### **Socket.io Rooms**
```javascript
// Server-side room management
const chatRooms = new Map(); // roomId -> Set of socket IDs

io.on('connection', (socket) => {
  console.log(`User ${socket.user.email} connected`);

  // Join a room
  socket.on('join_room', (roomId) => {
    socket.join(roomId);

    // Track room membership
    if (!chatRooms.has(roomId)) {
      chatRooms.set(roomId, new Set());
    }
    chatRooms.get(roomId).add(socket.id);

    // Notify others in the room
    socket.to(roomId).emit('user_joined_room', {
      user: socket.user.email,
      roomId,
      timestamp: new Date(),
    });

    // Send room info to user
    const roomSize = chatRooms.get(roomId).size;
    socket.emit('room_joined', {
      roomId,
      userCount: roomSize,
      users: Array.from(chatRooms.get(roomId)).map(id => {
        const client = io.sockets.sockets.get(id);
        return client ? client.user.email : null;
      }).filter(Boolean),
    });
  });

  // Leave a room
  socket.on('leave_room', (roomId) => {
    socket.leave(roomId);

    if (chatRooms.has(roomId)) {
      chatRooms.get(roomId).delete(socket.id);

      // If room is empty, clean it up
      if (chatRooms.get(roomId).size === 0) {
        chatRooms.delete(roomId);
      }
    }

    // Notify others
    socket.to(roomId).emit('user_left_room', {
      user: socket.user.email,
      roomId,
      timestamp: new Date(),
    });
  });

  // Send message to room
  socket.on('send_room_message', (data) => {
    const { roomId, message } = data;

    // Check if user is in the room
    if (!chatRooms.has(roomId) || !chatRooms.get(roomId).has(socket.id)) {
      socket.emit('error', { message: 'You are not in this room' });
      return;
    }

    const roomMessage = {
      id: Date.now().toString(),
      text: message,
      sender: socket.user.email,
      roomId,
      timestamp: new Date(),
    };

    // Send to all users in the room (including sender)
    io.to(roomId).emit('room_message', roomMessage);
  });

  // Get room list
  socket.on('get_rooms', () => {
    const rooms = Array.from(chatRooms.entries()).map(([roomId, sockets]) => ({
      id: roomId,
      userCount: sockets.size,
      users: Array.from(sockets).map(id => {
        const client = io.sockets.sockets.get(id);
        return client ? client.user.email : null;
      }).filter(Boolean),
    }));

    socket.emit('rooms_list', rooms);
  });

  // Handle disconnection
  socket.on('disconnect', () => {
    // Remove from all rooms
    for (const [roomId, sockets] of chatRooms.entries()) {
      if (sockets.has(socket.id)) {
        sockets.delete(socket.id);

        // Notify others in the room
        socket.to(roomId).emit('user_left_room', {
          user: socket.user.email,
          roomId,
          timestamp: new Date(),
        });

        // Clean up empty rooms
        if (sockets.size === 0) {
          chatRooms.delete(roomId);
        }
      }
    }
  });
});
```

### **Socket.io Namespaces**
```javascript
// Create namespaces
const chatNamespace = io.of('/chat');
const adminNamespace = io.of('/admin');

// Chat namespace
chatNamespace.use(async (socket, next) => {
  // Authentication middleware for chat
  try {
    const token = socket.handshake.auth.token;
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    socket.user = decoded;
    next();
  } catch (error) {
    next(new Error('Authentication failed'));
  }
});

chatNamespace.on('connection', (socket) => {
  console.log(`User ${socket.user.email} connected to chat`);

  socket.on('join_room', (roomId) => {
    socket.join(roomId);
    socket.emit('room_joined', { roomId });
  });

  socket.on('send_message', (data) => {
    const { roomId, message } = data;
    chatNamespace.to(roomId).emit('message', {
      text: message,
      sender: socket.user.email,
      timestamp: new Date(),
    });
  });
});

// Admin namespace
adminNamespace.use(async (socket, next) => {
  // Admin authentication
  try {
    const token = socket.handshake.auth.token;
    const decoded = jwt.verify(token, process.env.JWT_SECRET);

    if (decoded.role !== 'admin') {
      return next(new Error('Admin access required'));
    }

    socket.user = decoded;
    next();
  } catch (error) {
    next(new Error('Authentication failed'));
  }
});

adminNamespace.on('connection', (socket) => {
  console.log(`Admin ${socket.user.email} connected`);

  socket.on('get_server_stats', () => {
    const stats = {
      totalConnections: io.engine.clientsCount,
      chatRooms: chatNamespace.adapter.rooms.size,
      timestamp: new Date(),
    };

    socket.emit('server_stats', stats);
  });

  socket.on('broadcast_message', (data) => {
    const { message, target } = data;

    if (target === 'all') {
      io.emit('admin_broadcast', {
        message,
        timestamp: new Date(),
      });
    } else if (target === 'chat') {
      chatNamespace.emit('admin_broadcast', {
        message,
        timestamp: new Date(),
      });
    }
  });
});

// Client-side namespace connection
// Chat namespace
const chatSocket = io('/chat', {
  auth: { token: 'user-jwt-token' },
});

chatSocket.on('connect', () => {
  console.log('Connected to chat namespace');
});

// Admin namespace
const adminSocket = io('/admin', {
  auth: { token: 'admin-jwt-token' },
});

adminSocket.on('connect', () => {
  console.log('Connected to admin namespace');
});
```

---

## 🔐 **Authentication & Authorization**

### **Socket Authentication**
```javascript
// JWT-based authentication middleware
const authenticateSocket = async (socket, next) => {
  try {
    const token = socket.handshake.auth.token ||
                  socket.handshake.headers.authorization?.split(' ')[1];

    if (!token) {
      return next(new Error('Authentication token required'));
    }

    // Verify JWT token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);

    // Fetch user from database
    const user = await User.findById(decoded.id).select('-password');
    if (!user) {
      return next(new Error('User not found'));
    }

    // Check if user is active
    if (!user.isActive) {
      return next(new Error('Account is deactivated'));
    }

    // Attach user to socket
    socket.user = user;
    socket.userId = user.id;
    socket.userRole = user.role;

    next();
  } catch (error) {
    if (error.name === 'JsonWebTokenError') {
      return next(new Error('Invalid token'));
    }
    if (error.name === 'TokenExpiredError') {
      return next(new Error('Token expired'));
    }
    next(error);
  }
};

// Role-based authorization middleware
const authorizeSocket = (allowedRoles) => {
  return (socket, next) => {
    if (!socket.user) {
      return next(new Error('Authentication required'));
    }

    if (!allowedRoles.includes(socket.userRole)) {
      return next(new Error('Insufficient permissions'));
    }

    next();
  };
};

// Rate limiting for socket events
const socketRateLimit = (maxRequests, windowMs) => {
  const requests = new Map();

  return (socket, next) => {
    const now = Date.now();
    const userId = socket.userId || socket.id;

    if (!requests.has(userId)) {
      requests.set(userId, []);
    }

    const userRequests = requests.get(userId);

    // Remove old requests outside the window
    const validRequests = userRequests.filter(time => now - time < windowMs);
    requests.set(userId, validRequests);

    if (validRequests.length >= maxRequests) {
      return next(new Error('Rate limit exceeded'));
    }

    validRequests.push(now);
    next();
  };
};

// Apply middleware
io.use(authenticateSocket);
io.use(socketRateLimit(10, 60000)); // 10 requests per minute

// Protected namespace
const adminNamespace = io.of('/admin');
adminNamespace.use(authenticateSocket);
adminNamespace.use(authorizeSocket(['admin', 'moderator']));
```

---

## 🚨 **Error Handling & Recovery**

### **Comprehensive Error Handling**
```javascript
class SocketErrorHandler {
  constructor(io) {
    this.io = io;
    this.setupGlobalErrorHandling();
  }

  setupGlobalErrorHandling() {
    // Handle uncaught exceptions
    process.on('uncaughtException', (error) => {
      console.error('Uncaught Exception:', error);
      this.handleCriticalError(error);
    });

    process.on('unhandledRejection', (reason, promise) => {
      console.error('Unhandled Rejection at:', promise, 'reason:', reason);
      this.handleCriticalError(reason);
    });

    // Socket.io error handling
    this.io.on('connection', (socket) => {
      // Handle socket-specific errors
      socket.on('error', (error) => {
        console.error(`Socket ${socket.id} error:`, error);
        this.handleSocketError(socket, error);
      });

      // Handle client errors
      socket.on('client_error', (error) => {
        console.error(`Client error from ${socket.id}:`, error);
        socket.emit('error_acknowledged', {
          message: 'Error received and logged',
          timestamp: new Date(),
        });
      });
    });
  }

  handleSocketError(socket, error) {
    // Log error with context
    console.error('Socket Error:', {
      socketId: socket.id,
      userId: socket.userId,
      error: error.message,
      stack: error.stack,
      timestamp: new Date(),
    });

    // Send error response to client
    socket.emit('error', {
      message: 'An error occurred. Please try again.',
      code: 'SOCKET_ERROR',
      timestamp: new Date(),
    });

    // Handle specific error types
    if (error.message.includes('authentication')) {
      socket.disconnect(true);
    } else if (error.message.includes('rate limit')) {
      // Temporarily disable socket
      socket.emit('rate_limited', {
        message: 'Too many requests. Please wait before trying again.',
        retryAfter: 60000, // 1 minute
      });
    }
  }

  handleCriticalError(error) {
    console.error('Critical Error:', error);

    // Notify all connected clients
    this.io.emit('server_error', {
      message: 'Server encountered an error. Please refresh and try again.',
      timestamp: new Date(),
    });

    // Attempt graceful shutdown
    setTimeout(() => {
      console.log('Initiating graceful shutdown...');
      this.io.close(() => {
        process.exit(1);
      });
    }, 1000);
  }

  // Recovery mechanisms
  setupRecovery() {
    // Auto-restart mechanism
    this.io.on('connection', (socket) => {
      // Send recovery state to client
      socket.emit('server_recovered', {
        message: 'Server has recovered from an error',
        timestamp: new Date(),
      });

      // Re-establish user's state
      if (socket.userId) {
        this.restoreUserState(socket);
      }
    });
  }

  async restoreUserState(socket) {
    try {
      // Restore user's room memberships
      const userRooms = await this.getUserRooms(socket.userId);
      userRooms.forEach(roomId => {
        socket.join(roomId);
      });

      // Send current state
      socket.emit('state_restored', {
        rooms: userRooms,
        timestamp: new Date(),
      });
    } catch (error) {
      console.error('Failed to restore user state:', error);
    }
  }

  async getUserRooms(userId) {
    // Implementation to get user's active rooms
    // This would typically query your database
    return ['general', 'random'];
  }
}

// Usage
const errorHandler = new SocketErrorHandler(io);
errorHandler.setupRecovery();
```

---

## ⚡ **Performance Optimization**

### **Socket Performance Optimization**
```javascript
// Compression and optimization
const io = require('socket.io')(server, {
  // Enable compression
  perMessageDeflate: {
    threshold: 1024, // Compress messages larger than 1KB
  },

  // Connection optimization
  pingTimeout: 60000,
  pingInterval: 25000,
  maxHttpBufferSize: 1e6, // 1MB max buffer

  // Transport optimization
  transports: ['websocket', 'polling'],
  allowUpgrades: true,
  upgradeTimeout: 10000,

  // CORS optimization
  cors: {
    origin: process.env.ALLOWED_ORIGINS?.split(',') || ["http://localhost:3000"],
    methods: ["GET", "POST"],
    credentials: true,
  },
});

// Message batching
class MessageBatcher {
  constructor(socket, batchSize = 10, delay = 100) {
    this.socket = socket;
    this.batchSize = batchSize;
    this.delay = delay;
    this.queue = [];
    this.timeout = null;
  }

  add(event, data) {
    this.queue.push({ event, data });

    if (this.queue.length >= this.batchSize) {
      this.flush();
    } else if (!this.timeout) {
      this.timeout = setTimeout(() => this.flush(), this.delay);
    }
  }

  flush() {
    if (this.queue.length === 0) return;

    // Group messages by event type
    const grouped = this.queue.reduce((acc, item) => {
      if (!acc[item.event]) acc[item.event] = [];
      acc[item.event].push(item.data);
      return acc;
    }, {});

    // Send batched messages
    Object.entries(grouped).forEach(([event, data]) => {
      if (data.length === 1) {
        this.socket.emit(event, data[0]);
      } else {
        this.socket.emit(`${event}_batch`, data);
      }
    });

    this.queue = [];
    if (this.timeout) {
      clearTimeout(this.timeout);
      this.timeout = null;
    }
  }
}

// Usage
io.on('connection', (socket) => {
  const batcher = new MessageBatcher(socket);

  // Instead of immediate emit
  // socket.emit('message', data);

  // Use batcher
  batcher.add('message', data);
});

// Memory optimization
class SocketMemoryManager {
  constructor() {
    this.messageCache = new Map();
    this.maxCacheSize = 1000;
  }

  cacheMessage(messageId, message) {
    if (this.messageCache.size >= this.maxCacheSize) {
      // Remove oldest entry
      const firstKey = this.messageCache.keys().next().value;
      this.messageCache.delete(firstKey);
    }

    this.messageCache.set(messageId, {
      data: message,
      timestamp: Date.now(),
    });
  }

  getCachedMessage(messageId) {
    const cached = this.messageCache.get(messageId);
    if (cached && Date.now() - cached.timestamp < 300000) { // 5 minutes
      return cached.data;
    }
    return null;
  }

  cleanupOldMessages() {
    const cutoff = Date.now() - 300000; // 5 minutes ago

    for (const [key, value] of this.messageCache.entries()) {
      if (value.timestamp < cutoff) {
        this.messageCache.delete(key);
      }
    }
  }
}

// Periodic cleanup
setInterval(() => {
  memoryManager.cleanupOldMessages();
}, 60000); // Every minute
```

---

## 🔒 **Security Considerations**

### **Socket Security Best Practices**
```javascript
// Input validation and sanitization
const validateSocketInput = (data, schema) => {
  // Basic validation
  if (!data || typeof data !== 'object') {
    throw new Error('Invalid input data');
  }

  // Schema validation
  for (const [key, rules] of Object.entries(schema)) {
    const value = data[key];

    if (rules.required && (value === undefined || value === null)) {
      throw new Error(`${key} is required`);
    }

    if (value !== undefined && value !== null) {
      if (rules.type && typeof value !== rules.type) {
        throw new Error(`${key} must be of type ${rules.type}`);
      }

      if (rules.maxLength && value.length > rules.maxLength) {
        throw new Error(`${key} cannot exceed ${rules.maxLength} characters`);
      }

      if (rules.pattern && !rules.pattern.test(value)) {
        throw new Error(`${key} format is invalid`);
      }
    }
  }

  return data;
};

// Rate limiting per user
const userRateLimiter = new Map();

const checkRateLimit = (userId, action, limit = 10, windowMs = 60000) => {
  const key = `${userId}:${action}`;
  const now = Date.now();

  if (!userRateLimiter.has(key)) {
    userRateLimiter.set(key, { count: 1, resetTime: now + windowMs });
    return true;
  }

  const userLimit = userRateLimiter.get(key);

  if (now > userLimit.resetTime) {
    userLimit.count = 1;
    userLimit.resetTime = now + windowMs;
    return true;
  }

  if (userLimit.count >= limit) {
    return false;
  }

  userLimit.count++;
  return true;
};
// Message encryption (optional)
const crypto = require('crypto');

class MessageEncryptor {
  constructor(secretKey) {
    this.algorithm = 'aes-256-gcm';
    this.secretKey = crypto.scryptSync(secretKey, 'salt', 32);
  }

  encrypt(text) {
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipherGCM(this.algorithm, this.secretKey);
    cipher.setIV(iv);

    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');

    const authTag = cipher.getAuthTag();

    return {
      encrypted,
      iv: iv.toString('hex'),
      authTag: authTag.toString('hex'),
    };
  }

  decrypt(encryptedData) {
    const { encrypted, iv, authTag } = encryptedData;
    const decipher = crypto.createDecipherGCM(this.algorithm, this.secretKey);
    decipher.setIV(Buffer.from(iv, 'hex'));
    decipher.setAuthTag(Buffer.from(authTag, 'hex'));

    let decrypted = decipher.update(encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');

    return decrypted;
  }
}

// Usage example
const encryptor = new MessageEncryptor('your-secret-key');

// Encrypt a message
const message = 'Hello, this is a secret message!';
const encrypted = encryptor.encrypt(message);
console.log('Encrypted:', encrypted);

// Decrypt the message
const decrypted = encryptor.decrypt(encrypted);
console.log('Decrypted:', decrypted);

// Secure socket communication
io.on('connection', (socket) => {
  socket.on('send_encrypted_message', (data) => {
    try {
      // Validate input
      const schema = {
        encrypted: { required: true, type: 'string', maxLength: 10000 },
        iv: { required: true, type: 'string', pattern: /^[a-f0-9]{32}$/ },
        authTag: { required: true, type: 'string', pattern: /^[a-f0-9]{32}$/ },
      };

      const validatedData = validateSocketInput(data, schema);

      // Decrypt message
      const encryptor = new MessageEncryptor(process.env.ENCRYPTION_KEY);
      const decryptedMessage = encryptor.decrypt(validatedData);

      // Process decrypted message
      console.log('Decrypted message:', decryptedMessage);

      // Broadcast or handle as needed
      socket.broadcast.emit('new_message', {
        text: decryptedMessage,
        sender: socket.user.email,
        timestamp: new Date(),
      });

    } catch (error) {
      socket.emit('error', { message: 'Failed to process encrypted message' });
    }
  });
});
 
---

**Next Lesson**: [Lesson 26: File Upload & Storage](Lesson%2026_%20File%20Upload%20&%20Storage.md)