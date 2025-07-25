import React, { useState, useEffect, useRef } from 'react';

// Main App component for the Virtual Boyfriend application
function App() {
    // State to store the conversation messages
    const [messages, setMessages] = useState([]);
    // State to store the current user input
    const [input, setInput] = useState('');
    // State to manage loading status during API calls
    const [isLoading, setIsLoading] = useState(false);
    // Ref to scroll to the latest message
    const messagesEndRef = useRef(null);

    // Array of possible initial greeting messages from SyberIsma
    const initialGreetings = [
        '¡Hola miamor hermosa, cómo estás?',
        'Mi niña preciosa, ¿qué necesitas?',
        '¡Hola, bebé! ¿Qué pasó?',
        'Dímelo mami, ¿qué vamos a hacer?'
    ];

    // Effect to scroll to the bottom of the chat when new messages arrive
    useEffect(() => {
        messagesEndRef.current?.scrollIntoView({ behavior: "smooth" });
    }, [messages]);

    // Initial welcome message from the virtual boyfriend (now SyberIsma)
    useEffect(() => {
        // Select a random greeting from the array
        const randomGreeting = initialGreetings[Math.floor(Math.random() * initialGreetings.length)];
        setMessages([{ sender: 'SyberIsma', text: randomGreeting }]);
    }, []);

    // Function to send a message to the Gemini API
    const sendMessage = async () => {
        if (input.trim() === '') return; // Don't send empty messages

        const userMessage = { sender: 'Tú', text: input };
        setMessages(prevMessages => [...prevMessages, userMessage]); // Add user message to chat
        setInput(''); // Clear input field
        setIsLoading(true); // Set loading state to true

        // Check if the input contains "hola" (case-insensitive)
        const lowerCaseInput = input.toLowerCase();
        if (lowerCaseInput.includes('hola')) {
            const romanticResponse = "¡Oh, mi amada! Tu presencia ilumina mi existencia cual lucero en la noche más oscura. Tu belleza, cual flor de exquisito jardín, embriaga mis sentidos y mi alma. ¿En qué puedo servir a tan gentil dama?";
            setMessages(prevMessages => [...prevMessages, { sender: 'SyberIsma', text: romanticResponse }]);
            setIsLoading(false); // Stop loading as we're not calling the API
            return; // Exit the function
        }

        try {
            let chatHistory = [];
            // Add a system instruction to encourage style adaptation
            chatHistory.push({
                role: "system",
                parts: [{ text: "Adapta tu estilo de escritura al del usuario, manteniendo un tono romántico y afectuoso. Siempre responde como un novio virtual llamado SyberIsma." }]
            });

            // Prepare chat history for the API call
            messages.forEach(msg => {
                chatHistory.push({
                    role: msg.sender === 'Tú' ? 'user' : 'model',
                    parts: [{ text: msg.text }]
                });
            });
            chatHistory.push({ role: "user", parts: [{ text: input }] }); // Add current user input

            const payload = { contents: chatHistory };
            const apiKey = ""; // API key will be provided by the Canvas environment
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;

            const response = await fetch(apiUrl, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });

            const result = await response.json();

            // Check if the response contains valid content
            if (result.candidates && result.candidates.length > 0 &&
                result.candidates[0].content && result.candidates[0].content.parts &&
                result.candidates[0].content.parts.length > 0) {
                const botResponseText = result.candidates[0].content.parts[0].text;
                setMessages(prevMessages => [...prevMessages, { sender: 'SyberIsma', text: botResponseText }]);
            } else {
                console.error("Unexpected API response structure:", result);
                setMessages(prevMessages => [...prevMessages, { sender: 'SyberIsma', text: 'Lo siento, no pude entender eso. ¿Puedes repetirlo?' }]);
            }
        } catch (error) {
            console.error("Error calling Gemini API:", error);
            setMessages(prevMessages => [...prevMessages, { sender: 'SyberIsma', text: '¡Oh no! Parece que hay un problema. Por favor, inténtalo de nuevo más tarde.' }]);
        } finally {
            setIsLoading(false); // Set loading state to false
        }
    };

    // Handle Enter key press in the input field
    const handleKeyPress = (e) => {
        if (e.key === 'Enter' && !isLoading) {
            sendMessage();
        }
    };

    return (
        <div className="flex flex-col items-center justify-center min-h-screen bg-gradient-to-br from-pink-100 to-purple-200 p-4 font-inter">
            <style>
                {`
                @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&family=Lora:wght@400;700&display=swap');
                body { font-family: 'Inter', sans-serif; }
                .message-lora { font-family: 'Lora', serif; }
                `}
            </style>
            <div className="bg-white rounded-3xl shadow-xl p-6 w-full max-w-md flex flex-col h-[70vh] md:h-[80vh] border border-gray-200">
                <h1 className="text-3xl font-bold text-center text-purple-700 mb-6">SyberIsma</h1>

                {/* Chat messages display area */}
                <div className="flex-1 overflow-y-auto mb-4 p-3 bg-gray-50 rounded-2xl border border-gray-100 custom-scrollbar">
                    {messages.map((msg, index) => (
                        <div
                            key={index}
                            className={`mb-3 p-3 rounded-xl max-w-[80%] ${
                                msg.sender === 'Tú'
                                    ? 'bg-blue-500 text-white self-end ml-auto rounded-br-none'
                                    : 'bg-purple-100 text-gray-800 self-start mr-auto rounded-bl-none'
                            } ${msg.sender === 'SyberIsma' ? 'message-lora' : ''}`}
                        >
                            <div className="font-semibold text-sm mb-1">
                                {msg.sender === 'Tú' ? 'Tú' : 'SyberIsma'}
                            </div>
                            <p className="text-base">{msg.text}</p>
                        </div>
                    ))}
                    {/* Loading indicator */}
                    {isLoading && (
                        <div className="mb-3 p-3 rounded-xl bg-purple-100 text-gray-800 self-start mr-auto rounded-bl-none animate-pulse message-lora">
                            <div className="font-semibold text-sm mb-1">SyberIsma</div>
                            <p className="text-base">Escribiendo...</p>
                        </div>
                    )}
                    <div ref={messagesEndRef} /> {/* Scroll target */}
                </div>

                {/* Message input area */}
                <div className="flex gap-3">
                    <input
                        type="text"
                        className="flex-1 p-3 border border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-purple-500 transition duration-200"
                        placeholder="Escribe tu mensaje aquí..."
                        value={input}
                        onChange={(e) => setInput(e.target.value)}
                        onKeyPress={handleKeyPress}
                        disabled={isLoading}
                    />
                    <button
                        onClick={sendMessage}
                        className="bg-purple-600 text-white p-3 rounded-xl shadow-md hover:bg-purple-700 focus:outline-none focus:ring-2 focus:ring-purple-500 focus:ring-offset-2 transition duration-200 disabled:opacity-50 disabled:cursor-not-allowed"
                        disabled={isLoading}
                    >
                        Enviar
                    </button>
                </div>
            </div>
        </div>
    );
}

export default App;
