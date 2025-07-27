<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cyberisma - Vista Previa</title>
    
    <!-- Tailwind CSS para los estilos -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- React y ReactDOM para la funcionalidad -->
    <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
    
    <!-- Babel para transpilar JSX en el navegador -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    
    <style>
        /* Estilos adicionales para asegurar que la app ocupe toda la altura */
        html, body, #root {
            height: 100%;
            margin: 0;
            padding: 0;
            background-color: #f9fafb; /* bg-gray-50 */
        }
    </style>
</head>
<body>
    <!-- El elemento raíz donde se montará la aplicación de React -->
    <div id="root"></div>

    <!-- El script de tu aplicación React -->
    <script type="text/babel" data-type="module">
    
    // --- IMPORTACIONES DE FIREBASE (usando URLs de CDN para el navegador) ---
    import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
    import { 
        getAuth, 
        onAuthStateChanged, 
        signInAnonymously,
        signInWithCustomToken,
        signOut 
    } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
    import { 
        getFirestore, 
        collection, 
        addDoc, 
        onSnapshot, 
        updateDoc, 
        deleteDoc, 
        doc, 
        serverTimestamp, 
        query, 
        orderBy 
    } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

    // --- Configuración de Firebase (inyectada por el entorno) ---
    const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
    const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-cyberisma-app';

    // --- Componentes de UI (Recreaciones) ---
    const Input = ({ className, ...props }) => (
      <input
        className={`flex h-10 w-full rounded-md border border-gray-300 bg-white px-3 py-2 text-sm ring-offset-white file:border-0 file:bg-transparent file:text-sm file:font-medium placeholder:text-gray-500 focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-purple-500 focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50 ${className}`}
        {...props}
      />
    );

    const Button = ({ className, variant = 'default', size = 'default', children, disabled, ...props }) => {
      const baseClasses = "inline-flex items-center justify-center rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none";
      const disabledClasses = disabled ? "bg-gray-400 cursor-not-allowed" : "hover:bg-purple-700/90";
      const variantClasses = {
        default: `bg-purple-600 text-white ${disabledClasses}`,
        outline: `border border-input bg-background hover:bg-accent hover:text-accent-foreground`,
        ghost: "hover:bg-gray-100 hover:text-gray-900",
      };
      const sizeClasses = { default: "h-10 px-4 py-2", icon: "h-10 w-10" };
      return (
        <button className={`${baseClasses} ${variantClasses[variant]} ${sizeClasses[size]} ${className}`} disabled={disabled} {...props}>
          {children}
        </button>
      );
    };

    // --- Iconos (SVG en línea) ---
    const Trash2 = ({ className, ...props }) => (
      <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className} {...props}><path d="M3 6h18" /><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2" /><line x1="10" x2="10" y1="11" y2="17" /><line x1="14" x2="14" y1="11" y2="17" /></svg>
    );
    const Check = ({ className, ...props }) => (
      <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className} {...props}><polyline points="20 6 9 17 4 12" /></svg>
    );
    const Bot = ({ className, ...props }) => (
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className} {...props}><path d="M12 8V4H8"/><rect width="16" height="12" x="4" y="8" rx="2"/><path d="M2 14h2"/><path d="M20 14h2"/><path d="M15 13v2"/><path d="M9 13v2"/></svg>
    );

    // --- Página de Login por Perfil ---
    function LoginPage({ onLoginSuccess }) {
      const [selectedUser, setSelectedUser] = React.useState(null);
      const [password, setPassword] = React.useState("");
      const [error, setError] = React.useState("");
      const [isLoading, setIsLoading] = React.useState(false);

      const users = {
        daniela: { pass: "0000" },
        ismael: { pass: "04082764000" },
      };

      const handleLogin = (e) => {
        e.preventDefault();
        if (!selectedUser) return;

        setError("");
        setIsLoading(true);

        const userConfig = users[selectedUser];

        if (password === userConfig.pass) {
          onLoginSuccess(selectedUser);
        } else {
          setError("Contraseña incorrecta.");
          setIsLoading(false);
        }
      };

      return (
        <div className="flex min-h-screen justify-center items-center bg-gray-50 p-4">
          <div className="w-full max-w-xs bg-white shadow-xl rounded-2xl p-8 space-y-6">
            <div className="text-center">
                <h1 className="text-3xl font-bold text-purple-600">Cyberisma</h1>
                <p className="text-gray-500 mt-2">Selecciona tu perfil</p>
            </div>
            <div className="flex flex-col gap-3">
                <Button variant={selectedUser === "daniela" ? "default" : "outline"} onClick={() => { setSelectedUser("daniela"); setError(""); setPassword("") }}>Daniela</Button>
                <Button variant={selectedUser === "ismael" ? "default" : "outline"} onClick={() => { setSelectedUser("ismael"); setError(""); setPassword("") }}>Ismael</Button>
            </div>
            {selectedUser && (
              <form onSubmit={handleLogin} className="space-y-4 pt-4 border-t">
                <Input placeholder="Contraseña" type="password" value={password} onChange={(e) => setPassword(e.target.value)} required />
                {error && <p className="text-red-500 text-sm text-center">{error}</p>}
                <Button type="submit" className="w-full" disabled={isLoading}>{isLoading ? 'Entrando...' : 'Entrar'}</Button>
              </form>
            )}
          </div>
        </div>
      );
    }

    // --- Componente de Chat con IA (MODIFICADO) ---
    function Chat({ db, userProfile }) {
        const [mensajes, setMensajes] = React.useState([]);
        const [mensajeActual, setMensajeActual] = React.useState("");
        const [isLoading, setIsLoading] = React.useState(false);
        const chatEndRef = React.useRef(null);

        const mensajesRef = React.useMemo(() => collection(db, `artifacts/${appId}/public/data/chat`), [db]);

        React.useEffect(() => {
            const q = query(mensajesRef, orderBy("timestamp"));
            const unsubscribe = onSnapshot(q, (snapshot) => {
                setMensajes(snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() })));
            });
            return () => unsubscribe();
        }, [mensajesRef]);

        React.useEffect(() => {
            chatEndRef.current?.scrollIntoView({ behavior: "smooth" });
        }, [mensajes]);

        const enviarMensaje = async () => {
            if (!mensajeActual.trim() || !userProfile) return;
            const userMessage = mensajeActual;
            setMensajeActual("");
            
            await addDoc(mensajesRef, { de: userProfile, texto: userMessage, timestamp: serverTimestamp() });
            setIsLoading(true);

            try {
                const prompt = `${userProfile} dijo: "${userMessage}". Responde como si fueras Ismael, su amigo y guía en SyberIsma, de forma cálida y personal.`;
                const payload = { contents: [{ role: "user", parts: [{ text: prompt }] }] };
                const apiKey = ""; 
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;
                
                const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
                if (!response.ok) throw new Error(`API error: ${response.statusText}`);

                const result = await response.json();
                const textoIA = result.candidates[0].content.parts[0].text;
                await addDoc(mensajesRef, { de: "Ismael", texto: textoIA, timestamp: serverTimestamp() });
            } catch (error) {
                console.error("Error al contactar a la IA:", error);
                await addDoc(mensajesRef, { de: "Ismael", texto: "Uhm... parece que estoy teniendo problemas para conectar. Inténtalo de nuevo.", timestamp: serverTimestamp() });
            } finally {
                setIsLoading(false);
            }
        };

        if (userProfile === 'Daniela') {
            return (
                <div className="w-full h-full bg-white shadow-xl rounded-2xl flex flex-col">
                    <header className="p-4 border-b">
                        <h2 className="text-xl font-bold text-purple-600 flex items-center gap-2">
                            <Bot className="h-6 w-6"/>Chat con Ismael
                        </h2>
                    </header>
                    <main className="flex-grow p-4 overflow-y-auto bg-gray-50 space-y-4">
                        {mensajes.map((msg) => (
                            <div key={msg.id} className={`flex ${msg.de === 'Ismael' ? 'justify-start' : 'justify-end'}`}>
                                <div className={`p-3 rounded-2xl max-w-xs sm:max-w-md ${msg.de === 'Ismael' ? 'bg-purple-100 rounded-bl-none' : 'bg-blue-100 rounded-br-none'}`}>
                                    <p className="text-sm" style={{ whiteSpace: 'pre-wrap' }}>{msg.texto}</p>
                                </div>
                            </div>
                        ))}
                        {isLoading && (
                            <div className="flex justify-start">
                                 <div className="p-3 rounded-2xl max-w-xs sm:max-w-md bg-purple-100 rounded-bl-none animate-pulse">
                                    Ismael está escribiendo...
                                </div>
                            </div>
                        )}
                        <div ref={chatEndRef} />
                    </main>
                    <footer className="p-2 border-t bg-white">
                        <div className="flex gap-2 items-center">
                            <Input 
                                placeholder={`Escribe algo, ${userProfile}...`} 
                                value={mensajeActual} 
                                onChange={(e) => setMensajeActual(e.target.value)} 
                                onKeyPress={(e) => e.key === 'Enter' && enviarMensaje()} 
                                disabled={isLoading}
                                className="border-transparent focus-visible:ring-0 focus-visible:ring-offset-0"
                            />
                            <Button onClick={enviarMensaje} disabled={isLoading || !mensajeActual.trim()} className="shrink-0">Enviar</Button>
                        </div>
                    </footer>
                </div>
            );
        }
        
        if (userProfile === 'Ismael') {
            return (
                <div className="w-full h-full bg-blue-50 p-4 sm:p-6 shadow-xl rounded-2xl flex flex-col">
                    <h2 className="text-xl font-bold text-blue-800 mb-4 flex items-center gap-2">
                        <Bot className="h-6 w-6 text-blue-800"/> Entrenar a la IA
                    </h2>
                    <div className="h-full overflow-y-auto border border-blue-200 p-3 mb-4 space-y-4 bg-white rounded-lg flex-grow">
                        <div className="p-4 rounded-lg bg-blue-100 text-blue-900">
                            <p className="font-semibold">Pregunta de la IA:</p>
                            <p className="mt-1">¿Cuáles son los temas de conversación que Daniela evita o no le gustan?</p>
                            <Input className="mt-3 bg-white" placeholder="Tu respuesta para entrenar a Ismael..." />
                        </div>
                         <div className="p-4 rounded-lg bg-blue-100 text-blue-900">
                            <p className="font-semibold">Pregunta de la IA:</p>
                            <p className="mt-1">¿Qué tipo de humor le gusta a Daniela? ¿Sarcástico, simple, etc.?</p>
                            <Input className="mt-3 bg-white" placeholder="Tu respuesta para entrenar a Ismael..." />
                        </div>
                    </div>
                    <Button className="w-full bg-red-600 hover:bg-red-700 text-white">
                        Enviar Entrenamiento
                    </Button>
                </div>
            );
        }

        return null;
    }

    // --- Componente de Tareas ---
    function Tareas({ db }) {
      const [nuevaTarea, setNuevaTarea] = React.useState("");
      const [tareas, setTareas] = React.useState([]);
      const tareasRef = React.useMemo(() => collection(db, `artifacts/${appId}/public/data/tareas`), [db]);

      React.useEffect(() => {
        const q = query(tareasRef, orderBy("timestamp", "desc"));
        const unsubscribe = onSnapshot(q, (snapshot) => setTareas(snapshot.docs.map((doc) => ({ id: doc.id, ...doc.data() }))));
        return () => unsubscribe();
      }, [tareasRef]);

      const agregarTarea = async () => { if (!nuevaTarea.trim()) return; await addDoc(tareasRef, { texto: nuevaTarea.trim(), completada: false, timestamp: serverTimestamp() }); setNuevaTarea(""); };
      const marcarComoHecha = async (id, hecho) => await updateDoc(doc(db, `artifacts/${appId}/public/data/tareas`, id), { completada: !hecho });
      const eliminarTarea = async (id) => await deleteDoc(doc(db, `artifacts/${appId}/public/data/tareas`, id));

      return (
        <div className="w-full h-full bg-white p-4 sm:p-6 shadow-xl rounded-2xl flex flex-col"><h2 className="text-xl font-bold mb-4 text-purple-600">Lista de Tareas</h2><div className="flex gap-2 mb-4"><Input value={nuevaTarea} onChange={(e) => setNuevaTarea(e.target.value)} onKeyPress={(e) => e.key === 'Enter' && agregarTarea()} placeholder="Escribe una tarea..."/><Button onClick={agregarTarea} className="shrink-0">Agregar</Button></div><ul className="space-y-2 overflow-y-auto flex-grow">{tareas.map((t) => (<li key={t.id} className={`flex justify-between items-center p-3 rounded-xl transition-colors ${t.completada ? "bg-green-100" : "bg-gray-100"}`}><span className={`flex-grow ${t.completada ? "line-through text-gray-500" : ""}`}>{t.texto}</span><div className="flex gap-1 sm:gap-2 shrink-0 ml-2"><Button size="icon" variant="ghost" onClick={() => marcarComoHecha(t.id, t.completada)}><Check className="h-5 w-5 text-green-600" /></Button><Button size="icon" variant="ghost" onClick={() => eliminarTarea(t.id)}><Trash2 className="h-5 w-5 text-red-600" /></Button></div></li>))}</ul></div>
      );
    }

    // --- Componente de Calendario ---
    function Calendario({ db }) {
      const [eventos, setEventos] = React.useState([]);
      const [titulo, setTitulo] = React.useState("");
      const [fecha, setFecha] = React.useState("");
      const [descripcion, setDescripcion] = React.useState("");
      const eventosRef = React.useMemo(() => collection(db, `artifacts/${appId}/public/data/eventos`), [db]);

      React.useEffect(() => {
        const q = query(eventosRef, orderBy("fecha", "asc"));
        const unsubscribe = onSnapshot(q, (snapshot) => setEventos(snapshot.docs.map((doc) => ({ id: doc.id, ...doc.data() }))));
        return () => unsubscribe();
      }, [eventosRef]);

      const agregarEvento = async () => { if (!titulo.trim() || !fecha) return; await addDoc(eventosRef, { titulo: titulo.trim(), fecha, descripcion: descripcion.trim(), timestamp: serverTimestamp() }); setTitulo(""); setFecha(""); setDescripcion(""); };
      const eliminarEvento = async (id) => await deleteDoc(doc(db, `artifacts/${appId}/public/data/eventos`, id));

      return (
        <div className="w-full h-full bg-white p-4 sm:p-6 shadow-xl rounded-2xl flex flex-col"><h2 className="text-xl font-bold mb-4 text-blue-600">Calendario de Eventos</h2><div className="space-y-3 mb-4"><Input value={titulo} onChange={(e) => setTitulo(e.target.value)} placeholder="Título del evento" /><Input type="date" value={fecha} onChange={(e) => setFecha(e.target.value)} /><Input value={descripcion} onChange={(e) => setDescripcion(e.target.value)} placeholder="Descripción (opcional)" /><Button onClick={agregarEvento} className="bg-blue-600 hover:bg-blue-700 text-white w-full">Agregar evento</Button></div><ul className="space-y-2 overflow-y-auto flex-grow">{eventos.map((ev) => (<li key={ev.id} className="bg-gray-100 p-3 rounded-xl flex justify-between items-start"><div><p className="font-semibold text-blue-700">{ev.titulo}</p><p className="text-sm text-gray-800">{new Date(ev.fecha + 'T00:00:00').toLocaleDateString('es-ES', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' })}</p>{ev.descripcion && <p className="text-sm text-gray-600 mt-1">{ev.descripcion}</p>}</div><Button variant="ghost" size="icon" onClick={() => eliminarEvento(ev.id)}><Trash2 className="h-5 w-5 text-red-600" /></Button></li>))}</ul></div>
      );
    }

    // --- Componente Principal de la App ---
    function App() {
      const [db, setDb] = React.useState(null);
      const [auth, setAuth] = React.useState(null);
      const [user, setUser] = React.useState(null);
      const [userProfile, setUserProfile] = React.useState("");
      const [page, setPage] = React.useState('loading'); // loading, login, dashboard

      React.useEffect(() => {
        try {
            const app = initializeApp(firebaseConfig);
            const authInstance = getAuth(app);
            const dbInstance = getFirestore(app);
            setAuth(authInstance);
            setDb(dbInstance);

            const unsubscribe = onAuthStateChanged(authInstance, (currentUser) => {
              setUser(currentUser);
              if (currentUser) {
                const profile = localStorage.getItem('cyberismaUserProfile');
                if (profile) {
                    setUserProfile(profile.charAt(0).toUpperCase() + profile.slice(1));
                    setPage('dashboard');
                } else {
                    setPage('login');
                }
              } else {
                const authenticate = async () => {
                    const token = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
                    try {
                        if (token) {
                            await signInWithCustomToken(authInstance, token);
                        } else {
                            await signInAnonymously(authInstance);
                        }
                    } catch (error) {
                        console.error("Error en la autenticación inicial:", error);
                        setPage('login');
                    }
                };
                authenticate();
              }
            });

            return () => unsubscribe();
        } catch (error) {
            console.error("Error inicializando Firebase:", error);
            // Mostrar un mensaje de error en la UI si la configuración es inválida
            setPage('error'); 
        }
      }, []);

      const handleLoginSuccess = (profile) => {
          localStorage.setItem('cyberismaUserProfile', profile);
          setUserProfile(profile.charAt(0).toUpperCase() + profile.slice(1));
          setPage('dashboard');
      };

      const handleSignOut = async () => { 
        if(auth) {
          await signOut(auth);
          localStorage.removeItem('cyberismaUserProfile');
          // onAuthStateChanged se encargará de reiniciar el ciclo
          setPage('login'); 
          setUserProfile('');
        }
      };

      if (page === 'error') {
          return (<div className="flex items-center justify-center h-screen bg-red-50"><p className="text-lg text-red-600">Error: No se pudo conectar a la base de datos. La configuración puede ser incorrecta.</p></div>);
      }
      
      if (page === 'loading' || !db) {
        return (<div className="flex items-center justify-center h-screen bg-gray-50"><p className="text-lg text-gray-600 animate-pulse">Cargando Cyberisma...</p></div>);
      }

      if (page === 'login') {
        return <LoginPage onLoginSuccess={handleLoginSuccess} />;
      }
      
      return (
        <div className="bg-gray-50 min-h-screen font-sans">
            <header className="p-4 bg-white shadow-sm flex justify-between items-center">
                <h1 className="text-2xl font-bold text-gray-700">Cyberisma</h1>
                <div className="flex items-center gap-4">
                    <p className="text-sm text-gray-500 hidden sm:block capitalize">Hola, {userProfile}</p>
                    <Button onClick={handleSignOut} variant="ghost" className="text-purple-600 hover:bg-purple-100">Salir</Button>
                </div>
            </header>
            <main className="grid grid-cols-1 lg:grid-cols-2 gap-6 p-6 max-w-7xl mx-auto">
              <section className="lg:col-span-2 h-[60vh] lg:h-[calc(100vh-250px)] min-h-[400px]">
                  <Chat db={db} userProfile={userProfile} />
              </section>
              <section className="h-[60vh] min-h-[400px]">
                <Tareas db={db} />
              </section>
              <section className="h-[60vh] min-h-[400px]">
                <Calendario db={db} />
              </section>
            </main>
        </div>
      );
    }

    // --- Montar la aplicación en el DOM ---
    const container = document.getElementById('root');
    const root = ReactDOM.createRoot(container);
    root.render(<App />);

    </script>
</body>
</html>
