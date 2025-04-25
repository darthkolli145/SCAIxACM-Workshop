# Gemini Chatbot Tutorial

This simple React application demonstrates how to build a chatbot using Google's Gemini API. It's designed to be taught as a 1-hour workshop.

## What You'll Learn

- Setting up a React project with Vite
- Integrating Google's Gemini API
- Creating a chat interface using React
- Handling API requests and responses
- Using environment variables for API keys
- Customizing AI personality with system instructions

## Prerequisites

- Basic knowledge of HTML, CSS, and JavaScript
- Node.js 18 or higher
- A Gemini API key from [Google AI Studio](https://ai.google.dev/)

## Getting Started

1. Clone this repository:
   ```bash
   git clone [repository-url]
   ```

2. Install dependencies:
   ```bash
   cd geminiapi-workshop
   npm install
   ```

3. **Important**: Ensure the Gemini API package is installed:
   ```bash
   npm install @google/generative-ai
   ```
   
   If you encounter any issues, try reinstalling:
   ```bash
   npm uninstall @google/generative-ai
   npm install @google/generative-ai
   ```

4. Add your Gemini API key to the `.env` file:
   ```
   VITE_GEMINI_API_KEY=your_api_key_here
   ```

5. Start the development server:
   ```bash
   npm run dev
   ```

6. Open your browser to the URL shown in your terminal (typically http://localhost:5173)

## How to Get a Gemini API Key

1. Go to [Google AI Studio](https://makersuite.google.com/app/apikey)
2. Sign in with your Google account
3. Click on "Get API key" 
4. Create a new API key
5. Copy the API key and paste it into your `.env` file

## Workshop Walkthrough

### 1. Project Structure

- `src/App.jsx`: Main React component with the chatbot logic
- `src/App.css`: Styling for the chatbot
- `.env`: Contains your API key
- `index.html`: Base HTML template

### 2. App.jsx Explained

The main component contains:
- State for managing messages, input, loading, and errors
- API key from environment variables
- Function to send messages to Gemini API
- System prompt for customizing the chatbot's personality
- Chat interface rendering

### 3. Key Code Concepts

- **React State**: Using useState to manage component state
- **Environment Variables**: Accessing the API key securely
- **Async/Await**: Handling asynchronous API calls
- **Error Handling**: Try/catch for API request errors
- **Conditional Rendering**: Showing different UI based on state
- **System Instructions**: Properly formatted to customize the AI's behavior

### 4. CSS Styling

The CSS is organized by component sections:
- Container styling
- Message bubbles
- Input area
- Loading indicator

## Troubleshooting

If you encounter an error related to system instructions, make sure they're properly formatted as:

```javascript
systemInstruction: {
  parts: [{ text: yourSystemPromptText }]
}
```

Not as a simple string:

```javascript
// This won't work:
systemInstruction: "Your prompt here"
```

## Next Steps

After completing this tutorial, you might want to:

1. Add conversation history persistence (localStorage)
2. Implement different Gemini model options
3. Add image upload capabilities
4. Create more complex system prompts
5. Implement streaming responses for more natural conversations

## Resources

- [Gemini API Documentation](https://ai.google.dev/gemini-api/docs/api-overview)
- [React Documentation](https://react.dev/)
- [Vite Documentation](https://vitejs.dev/)
