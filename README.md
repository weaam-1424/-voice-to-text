# voice-to-text
##  user interface to convert voice to text (speech to text) with Save the text output to database
### 1. Setting Up the User Interface
We'll create an HTML file with a button to start recording, a button to stop recording, and a text area to display the transcribed text.

HTML (index.php) code :
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Speech to Text</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            padding: 50px;
        }
        button {
            margin: 10px;
            padding: 10px 20px;
            font-size: 16px;
        }
        textarea {
            width: 80%;
            height: 200px;
            margin-top: 20px;
        }
    </style>
</head>
<body>
    <h1>Speech to Text</h1>
    <button id="startBtn">Start Recording</button>
    <button id="stopBtn" disabled>Stop Recording</button>
    <textarea id="transcript" placeholder="Your transcribed text will appear here..."></textarea>
    <button id="saveBtn" disabled>Save to Database</button>

    <script src="app.js"></script>
</body>
</html>
```
### 2. Adding JavaScript for Speech Recognition
We'll use the Web Speech API to handle speech recognition.

JavaScript (app.js) code :
```
window.onload = () => {
    const startBtn = document.getElementById('startBtn');
    const stopBtn = document.getElementById('stopBtn');
    const saveBtn = document.getElementById('saveBtn');
    const transcriptArea = document.getElementById('transcript');

    let recognition;
    if ('webkitSpeechRecognition' in window) {
        recognition = new webkitSpeechRecognition();
    } else {
        recognition = new SpeechRecognition();
    }
    recognition.continuous = true;
    recognition.interimResults = true;

    recognition.onstart = () => {
        startBtn.disabled = true;
        stopBtn.disabled = false;
        saveBtn.disabled = true;
    };

    recognition.onend = () => {
        startBtn.disabled = false;
        stopBtn.disabled = true;
        saveBtn.disabled = false;
    };

    recognition.onresult = (event) => {
        let interimTranscript = '';
        let finalTranscript = '';
        for (let i = 0; i < event.results.length; i++) {
            if (event.results[i].isFinal) {
                finalTranscript += event.results[i][0].transcript;
            } else {
                interimTranscript += event.results[i][0].transcript;
            }
        }
        transcriptArea.value = finalTranscript || interimTranscript;
    };

    startBtn.onclick = () => {
        recognition.start();
    };

    stopBtn.onclick = () => {
        recognition.stop();
    };

    saveBtn.onclick = () => {
        const text = transcriptArea.value;
        fetch('save.php', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ text })
        })
        .then(response => response.json())
        .then(data => {
            alert('Text saved successfully!');
        })
        .catch(error => {
            console.error('Error:', error);
        });
    };
};
```
### 3. Creating PHP Script to Save Data
We'll create a PHP script to save the transcribed text to a MySQL database.

PHP (save.php) code :
```
<?php
header('Content-Type: application/json');

$servername = "localhost";
$username = "root";
$password = "";
$dbname = "speech_to_text";

// Create connection
$conn = new mysqli($servername, $username, $password, $dbname);

// Check connection
if ($conn->connect_error) {
    die(json_encode(['success' => false, 'message' => 'Database connection failed: ' . $conn->connect_error]));
}

$data = json_decode(file_get_contents('php://input'), true);
$text = $data['text'];

$sql = "INSERT INTO transcripts (text) VALUES (?)";
$stmt = $conn->prepare($sql);
$stmt->bind_param("s", $text);

if ($stmt->execute()) {
    echo json_encode(['success' => true, 'message' => 'Text saved successfully!']);
} else {
    echo json_encode(['success' => false, 'message' => 'Error: ' . $stmt->error]);
}

$stmt->close();
$conn->close();
?>
```
### 4. Creating the Database
Create a MySQL database and table to store the transcribed text.

SQL :
```
CREATE DATABASE speech_to_text;

USE speech_to_text;

CREATE TABLE transcripts (
    id INT AUTO_INCREMENT PRIMARY KEY,
    text TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```
### Audio conversion web interface :
![ويب تحويل الصوت](https://github.com/user-attachments/assets/cbce0bf6-3070-437c-b9d1-e6a9b5e8eab5)
### The Database :
![قاعدة بيانات تحويل الصوت](https://github.com/user-attachments/assets/e0ff93d3-add8-4a02-b3f6-1d4865cd46ac)

