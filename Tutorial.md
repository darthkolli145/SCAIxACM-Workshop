# Building a Gemini Chatbot - Step-by-Step Tutorial

This guide will walk you through creating a simple chatbot using Google's Gemini API and React. It's designed to be completed in about 1 hour.

## Step 1: Setting Up the Project (5 minutes)

1. Ensure you have Node.js installed (v18+)
2. Create a new Vite React project:
   ```bash
   npm create vite@latest geminiapi-workshop -- --template react
   cd geminiapi-workshop
   npm install
   ```
3. **Important**: Install the Gemini API package:
   ```bash
   npm install @google/generative-ai
   ```
   If you encounter any issues, try reinstalling:
   ```bash
   npm uninstall @google/generative-ai
   npm install @google/generative-ai
   ```

## Step 2: Get a Gemini API Key (5 minutes)

1. Go to [Google AI Studio](https://makersuite.google.com/app/apikey)
2. Create a new API key
3. Create a `.env` file in the project root:
   ```
   VITE_GEMINI_API_KEY=your_api_key_here
   ```
4. Add `.env` to your `.gitignore` file

## Step 3: Basic App Setup (10 minutes)

1. Open `src/App.jsx` and replace it with:

```jsx
import { useState } from 'react'
import './App.css'
import { GoogleGenerativeAI } from '@google/generative-ai'

function App() {
  // State variables
  const [messages, setMessages] = useState([]) // Store chat history
  const [inputValue, setInputValue] = useState('') // User input text
  const [isLoading, setIsLoading] = useState(false) // Loading state
  const [error, setError] = useState(null) // Error messages
  
  // Get API key from environment variables
  const apiKey = import.meta.env.VITE_GEMINI_API_KEY

  return (
    <div className="chat-container">
      <h1>Gemini Chat</h1>
      
      {/* API key warning */}
      {!apiKey && (
        <div className="api-key-warning">
          <p>API key not found! Please add your Gemini API key to the .env file.</p>
          <p>VITE_GEMINI_API_KEY=your_api_key_here</p>
        </div>
      )}
      
      {/* Chat container - we'll add messages here later */}
      <div className="messages-container">
        <div className="empty-chat">
          <p>No messages yet. Start a conversation!</p>
        </div>
      </div>
      
      {/* Message input area */}
      <div className="input-container">
        <textarea
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          placeholder="Type your message here..."
          rows={1}
        />
        <button>
          Send
        </button>
      </div>
    </div>
  )
}

export default App
```

## Step 4: Adding Basic Styling (10 minutes)

1. Open `src/App.css` and replace it with:

```css
/* Main container */
.chat-container {
  max-width: 800px;
  height: 100vh;
  margin: 0 auto;
  padding: 1rem;
  display: flex;
  flex-direction: column;
}

/* Header styling */
h1 {
  text-align: center;
  color: #1a73e8;
  margin-bottom: 1rem;
}

/* API key warning message */
.api-key-warning {
  background-color: #fef7e0;
  border-left: 4px solid #fbbc04;
  padding: 1rem;
  margin-bottom: 1rem;
  border-radius: 4px;
}

.api-key-warning p {
  margin: 0.5rem 0;
}

/* Messages container - where chat happens */
.messages-container {
  flex: 1;
  overflow-y: auto;
  padding: 1rem;
  margin-bottom: 1rem;
  border: 1px solid #dadce0;
  border-radius: 8px;
  background-color: #f8f9fa;
}

/* Empty chat state */
.empty-chat {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  color: #5f6368;
  font-style: italic;
}

/* Input container at the bottom */
.input-container {
  display: flex;
  gap: 0.5rem;
  padding: 0.5rem;
  background-color: white;
  border: 1px solid #dadce0;
  border-radius: 8px;
}

/* Text input */
.input-container textarea {
  flex: 1;
  border: none;
  outline: none;
  resize: none;
  background: transparent;
  font-family: inherit;
  padding: 0.5rem;
}

/* Send button */
.input-container button {
  align-self: flex-end;
  background-color: #1a73e8;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 0.5rem 1rem;
  cursor: pointer;
}
```

## Step 5: Implementing the Chat Logic (15 minutes)

1. Add the `sendMessage` function to handle API calls:

```jsx
// Function to handle sending messages to Gemini API
const sendMessage = async () => {
  // Validation checks
  if (!inputValue.trim()) return // Don't send empty messages
  if (!apiKey) {
    setError('API key is missing. Please add it to your .env file.')
    return
  }

  // Add user message to chat
  const userMessage = { role: 'user', content: inputValue }
  setMessages(prev => [...prev, userMessage])
  
  // Clear input and show loading state
  setInputValue('')
  setIsLoading(true)
  setError(null)
  
  try {
    // Initialize Gemini API client
    const genAI = new GoogleGenerativeAI(apiKey)
    
    // Select which model to use
    const model = genAI.getGenerativeModel({ 
      model: "gemini-1.5-flash" // Using Gemini 1.5 Flash model
    })
    
    // Send the message directly with the system instruction
    const result = await model.generateContent({
      contents: [
        {
          role: 'user',
          parts: [{ text: inputValue }]
        }
      ],
      generationConfig: {
        temperature: 0.7,
        maxOutputTokens: 1000,
      },
      systemInstruction: {
        parts: [{ text: systemPrompt }]
      }
    });
    
    // Get the response text
    const botResponse = result.response.text()
    
    // Add bot response to chat history
    setMessages(prev => [...prev, { role: 'assistant', content: botResponse }])
  } catch (err) {
    console.error(err)
    setError(err.message || 'Failed to get response from Gemini API')
  } finally {
    setIsLoading(false) // Reset loading state
  }
}
```

2. Add an Enter key handler:

```jsx
// Handle Enter key to send messages
const handleKeyPress = (e) => {
  if (e.key === 'Enter' && !e.shiftKey) {
    e.preventDefault()
    sendMessage()
  }
}
```

3. Update the textarea and button:

```jsx
<textarea
  value={inputValue}
  onChange={(e) => setInputValue(e.target.value)}
  onKeyPress={handleKeyPress}
  placeholder="Type your message here..."
  disabled={isLoading}
  rows={1}
/>
<button 
  onClick={sendMessage} 
  disabled={isLoading || !inputValue.trim()}
>
  Send
</button>
```

## Step 6: Displaying Messages (10 minutes)

1. Update the message container to display chat history:

```jsx
{/* Chat messages container */}
<div className="messages-container">
  {messages.length === 0 ? (
    <div className="empty-chat">
      <p>No messages yet. Start a conversation!</p>
    </div>
  ) : (
    // Map through and display messages
    messages.map((message, index) => (
      <div 
        key={index} 
        className={`message ${message.role === 'user' ? 'user-message' : 'bot-message'}`}
      >
        <div className="message-content">{message.content}</div>
      </div>
    ))
  )}
  
  {/* Loading indicator */}
  {isLoading && (
    <div className="message bot-message">
      <div className="message-content typing-indicator">
        <span></span>
        <span></span>
        <span></span>
      </div>
    </div>
  )}
</div>
```

2. Add error message display:

```jsx
{/* Error messages */}
{error && <div className="error-message">{error}</div>}
```

## Step 7: Add Message Styling (10 minutes)

1. Add these CSS styles for messages:

```css
/* Error message styling */
.error-message {
  color: #d93025;
  padding: 0.75rem;
  margin-bottom: 1rem;
  background-color: #fce8e6;
  border-radius: 4px;
  border-left: 4px solid #d93025;
}

/* Individual message styling */
.message {
  margin-bottom: 1rem;
  max-width: 80%;
}

/* User messages appear on the right */
.user-message {
  margin-left: auto;
}

/* Bot messages appear on the left */
.bot-message {
  margin-right: auto;
}

/* Content of messages */
.message-content {
  padding: 0.75rem 1rem;
  border-radius: 18px;
  white-space: pre-wrap;
  line-height: 1.5;
}

/* User message appearance */
.user-message .message-content {
  background-color: #1a73e8;
  color: white;
  border-bottom-right-radius: 4px;
}

/* Bot message appearance */
.bot-message .message-content {
  background-color: white;
  border: 1px solid #dadce0;
  border-bottom-left-radius: 4px;
}

/* Loading indicator - three bouncing dots */
.typing-indicator {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 2rem;
  width: 4rem;
}

.typing-indicator span {
  background-color: #5f6368;
  border-radius: 50%;
  display: inline-block;
  height: 0.5rem;
  width: 0.5rem;
  margin: 0 0.1rem;
  opacity: 0.6;
  animation: bounce 1.5s infinite ease-in-out;
}

.typing-indicator span:nth-child(1) {
  animation-delay: 0s;
}

.typing-indicator span:nth-child(2) {
  animation-delay: 0.2s;
}

.typing-indicator span:nth-child(3) {
  animation-delay: 0.4s;
}

@keyframes bounce {
  0%, 80%, 100% { transform: translateY(0); }
  40% { transform: translateY(-6px); }
}
```

## Step 8: Adding Personality Customization (10 minutes)

Now we'll add the ability to customize the chatbot's personality using system prompts:

1. Add state variables for the system prompt:

```jsx
const [systemPrompt, setSystemPrompt] = useState(
  "You are a helpful and friendly AI assistant. Keep your answers concise and clear."
) // Customize chatbot personality
const [showSystemPrompt, setShowSystemPrompt] = useState(false) // Toggle system prompt editor
```

2. Modify the API call to include the system prompt correctly:

```jsx
// Send the message directly with the system instruction
const result = await model.generateContent({
  contents: [
    {
      role: 'user',
      parts: [{ text: inputValue }]
    }
  ],
  generationConfig: {
    temperature: 0.7,
    maxOutputTokens: 1000,
  },
  systemInstruction: {
    parts: [{ text: systemPrompt }]
  }
});
```

Note: The system instruction must be formatted as an object with parts, not as a simple string.

3. Add the system prompt UI:

```jsx
{/* System Prompt Toggle Button */}
<button 
  className="system-prompt-toggle"
  onClick={() => setShowSystemPrompt(!showSystemPrompt)}
>
  {showSystemPrompt ? 'Hide Personality Settings' : 'Customize Chatbot Personality'}
</button>

{/* System Prompt Editor */}
{showSystemPrompt && (
  <div className="system-prompt-editor">
    <label htmlFor="system-prompt">System Instructions (Personality & Tone):</label>
    <textarea
      id="system-prompt"
      value={systemPrompt}
      onChange={(e) => setSystemPrompt(e.target.value)}
      placeholder="Enter instructions for the chatbot's personality and tone..."
      rows={4}
    />
    <div className="system-prompt-examples">
      <p>Examples:</p>
      <button onClick={() => setSystemPrompt("You are a helpful and friendly AI assistant. Keep your answers concise and clear.")}>
        Helpful Assistant
      </button>
      <button onClick={() => setSystemPrompt("You are a pirate captain with a colorful vocabulary. Speak like a pirate and use nautical terms.")}>
        Pirate
      </button>
      {/* Add more example buttons */}
    </div>
  </div>
)}
```

4. Add CSS for the system prompt UI:

```css
/* System prompt toggle button */
.system-prompt-toggle {
  display: block;
  margin: 0 auto 1rem;
  background-color: #f8f9fa;
  color: #1a73e8;
  border: 1px solid #dadce0;
  border-radius: 4px;
  padding: 0.5rem 1rem;
  cursor: pointer;
  font-weight: 500;
  transition: background-color 0.2s;
}

/* System prompt editor */
.system-prompt-editor {
  background-color: #f8f9fa;
  border: 1px solid #dadce0;
  border-radius: 8px;
  padding: 1rem;
  margin-bottom: 1rem;
  animation: fadeIn 0.3s;
}

/* More CSS for the system prompt UI */
```

The system prompt is a way to customize the chatbot's personality, tone, and behavior. By changing the system prompt, you can make your chatbot speak like a pirate, respond in rhymes, act as a technical expert, or take on any other personality you can imagine.

## Step 9: Testing and Debugging (5 minutes)

1. Run the application:
   ```bash
   npm run dev
   ```
2. Test the chat with different prompts
3. Check the browser console for any errors
4. Make sure your API key is correctly set in the `.env` file

## Challenge Ideas (if time permits)

1. Add a system prompt to customize your chatbot's personality
2. Store chat history in localStorage so it persists after page refresh
3. Add a toggle for dark/light mode
4. Implement a character counter for the input

## Recap & Resources

- We've built a simple but functional chatbot using Google's Gemini API
- The app has a clean UI with message styling and loading states
- API keys are stored securely in environment variables
- To learn more, check out:
  - [Gemini API Documentation](https://ai.google.dev/gemini-api/docs/api-overview)
  - [React Hooks Documentation](https://reactjs.org/docs/hooks-intro.html)
  - [CSS Flex Layout Guide](https://css-tricks.com/snippets/css/a-guide-to-flexbox/) 
