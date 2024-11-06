<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>VeilVerse Dating & Chat Forum</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>

<div class="forum-container">
    <h1>Welcome to VeilVerse Dating & Chat Forum</h1>

    <!-- Age and Gender Verification Section -->
    <div class="age-gender-verification">
        <label for="age">Enter your age:</label>
        <input type="number" id="age" placeholder="Enter age" />
        
        <label for="gender">Select your gender:</label>
        <select id="gender">
            <option value="male">Male</option>
            <option value="female">Female</option>
        </select>
        
        <button onclick="verifyAgeGender()">Submit</button>
    </div>

    <!-- Chatting Section (Only accessible after age and gender verification) -->
    <div id="chat-section" style="display:none;">
        <div class="chat-header">
            <h2>Chat Room</h2>
            <div class="like-buttons">
                <button onclick="likeMessage()">👍 Like</button>
                <button onclick="dislikeMessage()">👎 Dislike</button>
                <button onclick="sendHeartEmoji()">❤️ Heart</button>
            </div>
        </div>

        <!-- Private Messaging Section -->
        <div id="private-messages-section">
            <h3>Private Messages</h3>
            <form id="privateMessageForm" onsubmit="sendPrivateMessage(event)">
                <input type="text" id="receiverUsername" placeholder="Send message to..." required>
                <textarea id="privateMessageContent" placeholder="Your private message..." required></textarea>
                <button type="submit">Send Private Message</button>
            </form>
            <div id="privateMessages"></div>
        </div>

        <!-- Chat Messages -->
        <div class="chat-messages" id="chatMessages"></div>

        <!-- Chat Input -->
        <form class="chat-form" onsubmit="sendMessage(event)">
            <input type="text" id="username" placeholder="Enter your name" required>
            <textarea id="messageContent" placeholder="Type your message here..." required></textarea>
            <button type="submit">Send</button>
        </form>

        <!-- Admin Section -->
        <div id="admin-section">
            <input type="password" id="adminPassword" placeholder="Enter admin password" />
            <button onclick="clearMessages()">Clear All Messages (Admin)</button>
        </div>
    </div>

    <!-- Error Message for Underage or Gender Mismatch -->
    <div id="underage-warning" style="display:none; color: red;">
        <p>You must be 16 or older and match the gender requirement to access this forum. Please come back when you meet the age and gender requirement.</p>
    </div>
</div>

<script>
    // Global Variables
    let messages = [];
    let privateMessages = [];
    let userVerified = false;
    let userGender = '';
    let blockedUsers = [];

    // Function to verify user age and gender
    function verifyAgeGender() {
        const age = document.getElementById('age').value;
        const gender = document.getElementById('gender').value;

        if (age >= 16) {
            if (gender === "male" || gender === "female") {
                userGender = gender; // Store user's gender
                userVerified = true;
                document.getElementById('chat-section').style.display = 'block';
                document.getElementById('underage-warning').style.display = 'none';
                document.querySelector('.age-gender-verification').style.display = 'none';
            } else {
                alert("Please select a valid gender.");
            }
        } else {
            document.getElementById('underage-warning').style.display = 'block';
        }
    }

    // Function to send message to the public chat
    function sendMessage(event) {
        event.preventDefault();
        const username = document.getElementById('username').value;
        const content = document.getElementById('messageContent').value;

        // Check if username is valid
        if (!username || !content) {
            alert("Please enter both username and message.");
            return;
        }

        const message = {
            id: Date.now(),
            username: username,
            content: content,
            gender: userGender,  // Add gender to the message
            timestamp: new Date().toLocaleString(),
            heart: 0,
            likes: 0,
            dislikes: 0,
            blocked: false
        };

        messages.push(message);
        document.getElementById('messageContent').value = '';
        renderMessages();
    }

    // Function to render messages
    function renderMessages() {
        const chatMessages = document.getElementById('chatMessages');
        chatMessages.innerHTML = '';

        messages.forEach((message) => {
            if (!message.blocked && message.gender === userGender) {  // Check if message is from the same gender
                chatMessages.innerHTML += `
                    <div class="message" id="message-${message.id}">
                        <div class="message-header">
                            <span class="username">${message.username}</span>
                            <span class="timestamp">${message.timestamp}</span>
                        </div>
                        <p>${message.content}</p>
                        <div class="message-actions">
                            <span class="heart-count">${message.heart} ❤️</span>
                            <span class="like-count">${message.likes} 👍</span>
                            <span class="dislike-count">${message.dislikes} 👎</span>
                            <button onclick="likeMessage(${messages.indexOf(message)})">Like</button>
                            <button onclick="dislikeMessage(${messages.indexOf(message)})">Dislike</button>
                            <button onclick="sendHeartEmoji(${messages.indexOf(message)})">Heart</button>
                            <button onclick="blockUser(${messages.indexOf(message)})">Block User</button>
                        </div>
                    </div>
                `;
            }
        });
    }

    // Like a message
    function likeMessage(index) {
        messages[index].likes += 1;
        renderMessages();
    }

    // Dislike a message
    function dislikeMessage(index) {
        messages[index].dislikes += 1;
        renderMessages();
    }

    // Send a Heart Emoji to the most recent message
    function sendHeartEmoji(index) {
        messages[index].heart += 1;
        renderMessages();
    }

    // Function to block a user
    function blockUser(index) {
        const blockedUser = messages[index].username;
        blockedUsers.push(blockedUser);
        messages[index].blocked = true; // Block their message
        renderMessages();
        alert(`${blockedUser} has been blocked.`);
    }

    // Function to send private message
    function sendPrivateMessage(event) {
        event.preventDefault();
        const receiverUsername = document.getElementById('receiverUsername').value;
        const privateMessageContent = document.getElementById('privateMessageContent').value;

        if (!receiverUsername || !privateMessageContent) {
            alert("Please provide both a recipient and a message.");
            return;
        }

        const privateMessage = {
            sender: document.getElementById('username').value,
            receiver: receiverUsername,
            content: privateMessageContent,
            timestamp: new Date().toLocaleString()
        };

        privateMessages.push(privateMessage);
        document.getElementById('privateMessageContent').value = '';
        renderPrivateMessages();
    }

    // Function to render private messages
    function renderPrivateMessages() {
        const privateMessagesDiv = document.getElementById('privateMessages');
        privateMessagesDiv.innerHTML = '';

        privateMessages.forEach((msg) => {
            privateMessagesDiv.innerHTML += `
                <div class="private-message">
                    <div class="private-message-header">
                        <span class="sender">${msg.sender}</span> → <span class="receiver">${msg.receiver}</span>
                        <span class="timestamp">${msg.timestamp}</span>
                    </div>
                    <p>${msg.content}</p>
                </div>
            `;
        });
    }

    // Function to clear all messages (Admin-only)
    function clearMessages() {
        const adminPassword = document.getElementById('adminPassword').value;
        if (adminPassword === "admin123") {
            messages = [];
            privateMessages = [];
            renderMessages();
            renderPrivateMessages();
            alert("All messages cleared!");
        } else {
            alert("Incorrect admin password.");
        }
    }
</script>

</body>
</html>
/* Basic Styles */
body {
    font-family: Arial, sans-serif;
    background-color: #333;
    color: #fff;
    margin: 0;
    padding: 0;
}

.forum-container {
    width: 80%;
    margin: auto;
    padding: 20px;
    background-color: #444;
    border-radius: 8px;
}

h1 {
    text-align: center;
    color: #e60000;
}

.chat-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.chat-form input,
.chat-form textarea {
    width: 100%;
    padding: 10px;
    margin: 5px 0;
    border-radius: 4px;
    border: none;
}

.chat-form button {
    padding: 10px 15px;
    background-color: #e60000;
    border: none;
    color: white;
    cursor: pointer;
}

.chat-form button:hover {
    background-color: #ff1a1a;
}

.message {
    background-color: #555;
    padding: 10px;
    margin: 10px 0;
    border-radius: 6px;
}

.message-header {
    display: flex;
    justify-content: space-between;
    font-weight: bold;
}

.message-actions {
    display: flex;
    justify-content: space-between;
    font-size: 0.9em;
}

button {
    background-color: #444;
    color: white;
    border: none;
    padding: 5px 10px;
    border-radius: 4px;
    cursor: pointer;
}

button:hover {
    background-color: #777;
}

.heart-count, .like-count, .dislike-count {
    margin-right: 10px;
}
