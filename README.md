# Manual del Programador - Juego Web "Adivina el Pokémon"

## 1. Descripción General

"Adivina el Pokémon" es un juego web interactivo donde los usuarios intentan adivinar un Pokémon oculto mediante intentos de nombres. El juego ofrece modos de un solo jugador y multijugador, proporcionando pistas basadas en las características del Pokémon ingresado cada cierto número de intentos.

El juego compara diferentes atributos entre el Pokémon adivinado y el objetivo, como tipos, color, generación, hábitat, altura, peso y etapa evolutiva. Cada comparación se muestra con un código de colores: verde para coincidencias exactas, amarillo para valores cercanos y rojo para valores diferentes.

## 2. Tecnologías Utilizadas

### Frontend:
- **HTML5, CSS3**: Estructura y estilo de la interfaz de usuario
- **JavaScript**: Lógica del juego y manipulación del DOM
- **CSS Animations**: Para efectos visuales y transiciones fluidas

### Backend:
- **Node.js + Express**: Servidor para el modo multijugador
- **Socket.io**: Comunicación en tiempo real para el modo multijugador

### APIs / Librerías:
- **PokéAPI** (https://pokeapi.co/): Fuente de datos de Pokémon
- **Intro.js**: Para el tutorial interactivo
- **Fetch API**: Para realizar solicitudes HTTP a la PokéAPI

## 3. Estructura del Proyecto

```
Guess_The_Pokemon
├── assets
│   ├── font
│   │   ├── Pokemon.ttf
│   │   └── Text.ttf
│   └── images
│       ├── Background.mp4
│       ├── multiplayer.webp
│       ├── Pokebola-abierta.png
│       ├── Pokebola.png
│       └── singleplayer.webp
├── multiplayer
│   ├── game.css
│   ├── game.html
│   ├── index.html
│   ├── script.js
│   └── styles.css
├── singleplayer
│   ├── index.html
│   └── style.css
├── src
│   ├── animations.js
│   ├── api.js
│   ├── autocomplete.js
│   ├── game.js
│   ├── hints.js
│   ├── main.js
│   ├── ui.js
│   ├── utils.js
│   └── style.css
├── .gitignore
├── index.html
├── package-lock.json
├── package.json
├── README.md
├── server.js
└── style.css
```

## 4. Lógica del Juego

### 4.1 Modo Un Solo Jugador:

1. **Inicio**: 
   - Se selecciona un Pokémon aleatorio de la PokéAPI.
   - Se inicializa la interfaz de usuario y el autocompletado.

2. **Intento del usuario**: 
   - El jugador introduce el nombre de un Pokémon usando el campo de entrada.
   - El sistema de autocompletado muestra sugerencias mientras escribe.

3. **Validación**:
   - Se verifica si el nombre ingresado corresponde a un Pokémon válido en la PokéAPI.
   - Se comprueba si el Pokémon ya ha sido adivinado anteriormente.

4. **Comparación y Pistas**:
   - Se comparan las características entre el Pokémon ingresado y el objetivo.
   - Se muestra feedback visual con código de colores:
     - Verde: coincidencia exacta
     - Amarillo: valor cercano
     - Rojo: valor diferente
   - Cada 3 intentos se desbloquea una pista adicional.

5. **Victoria**: 
   - Si el usuario adivina correctamente, se muestra una pantalla de victoria.
   - Se ofrece la opción de jugar nuevamente.

### 4.2 Modo Multijugador:

1. **Creación/Unión a Sala**:
   - Un jugador crea una sala y recibe un código único.
   - Otros jugadores pueden unirse usando ese código.

2. **Preparación**:
   - Los jugadores marcan cuando están listos.
   - Cuando todos están listos, el juego comienza automáticamente.

3. **Juego**:
   - Se selecciona un Pokémon aleatorio como objetivo.
   - Cada jugador hace intentos de adivinar, similar al modo un solo jugador.
   - El servidor valida los intentos y envía los resultados a todos los jugadores.

4. **Victoria/Derrota**:
   - El primer jugador en adivinar correctamente gana.
   - Los demás jugadores reciben una notificación de derrota.
   - Se muestra el Pokémon objetivo y el nombre del ganador.

## 5. Componentes Principales y Fragmentos de Código

### 5.1 Página Principal (index.html)

La página principal muestra dos opciones: Un Solo Jugador y Multijugador. También incluye un fondo animado con Pokémon que se mueven.

```html
<!-- Fragmento de index.html -->
<div class="container">
  <div class="card-wrapper" id="container-1">
    <div class="card">
      <span>Un Solo Jugador</span>
    </div>
    <a href="./singleplayer/index.html">
      <div class="hologram">
        <div class="pokemon-background"></div>
        <div class="hologram-content">
          <div class="hologram-footer">
            <div class="hologram-legendary">Un Solo Jugador</div>
            <div class="hologram-description">¡Juega contra ti mismo e intenta adivinar el pokémon en menos de 10 intentos!</div>
          </div>
        </div>
        <div class="hologram-shine"></div>
      </div>
    </a>
  </div>
  <!-- Tarjeta de Multijugador similar -->
</div>
```

Las columnas de fondo con Pokémon se generan dinámicamente:

```javascript
// Fragmento de index.html (script)
const getRandomPokemonId = () => Math.floor(Math.random() * 1025) + 1

async function createPokemonSprite(pokemonId) {
    return new Promise((resolve, reject) => {
        const sprite = document.createElement('img')
        sprite.className = 'pokemon-sprite'
        sprite.src = `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${pokemonId}.png`
        
        sprite.onload = () => resolve(sprite)
        sprite.onerror = () => reject(new Error(`Failed to load Pokemon #${pokemonId}`))
    })
}

async function createPokemonColumn() {
    const column = document.createElement('div')
    column.className = 'pokemon-column'
    
    const sprites = []
    for (let i = 0; i < pokemonPerColumn; i++) {
        const pokemonId = getRandomPokemonId()
        try {
            const sprite = await createPokemonSprite(pokemonId)
            sprites.push(sprite)
        } catch (error) {
            console.log(error.message) 
        }
    }
    
    if (sprites.length > 0) {
        sprites.forEach(sprite => {
            column.appendChild(sprite)
        })
        sprites.forEach(sprite => {
            column.appendChild(sprite.cloneNode(true))
        })
    }
    
    return column
}
```

### 5.2 Modo Un Solo Jugador

#### 5.2.1 API y Carga de Datos (api.js)

Este módulo maneja todas las interacciones con la PokéAPI:

```javascript
// Fragmento de api.js
import { getGenerationNumber } from './utils.js'

// Fetch de todos los pokemons y pre-cargar las imagenes
async function fetchAllPokemon(loadingProgress) {
    try {
        // Primero fetch de todos los pokemons
        const response = await fetch('https://pokeapi.co/api/v2/pokemon?limit=1025')
        const data = await response.json()
        const total = data.results.length
        let loaded = 0

        // Fetch de datos para cada pokemon y pre-cargar sus sprites
        const pokemonList = await Promise.all(
            data.results.map(async (pokemon, index) => {
                try {
                    const detailResponse = await fetch(pokemon.url)
                    const pokemonData = await detailResponse.json()
                    
                    // Pre-cargar la imagen
                    if (pokemonData.sprites.front_default) {
                        await new Promise((resolve, reject) => {
                            const img = new Image()
                            img.onload = resolve
                            img.onerror = reject
                            img.src = pokemonData.sprites.front_default
                        })
                    }

                    loaded++
                    const progress = Math.floor((loaded / total) * 100)
                    loadingProgress.textContent = `${progress}%, Llamando a los Pokemons.`
                    
                    return {
                        name: pokemon.name,
                        id: pokemonData.id,
                        url: pokemon.url,
                        sprite: pokemonData.sprites.front_default
                    }
                } catch (error) {
                    console.error(`Error loading Pokemon ${pokemon.name}:`, error)
                    loaded++
                    return null
                }
            })
        )

        // Eliminar entradas nulas (cargas fallidas) y ordenar por ID
        return pokemonList.filter(p => p !== null).sort((a, b) => a.id - b.id)
    } catch (error) {
        console.error('Error fetching Pokemon list:', error)
        loadingProgress.textContent = 'Error loading Pokemon data. Please refresh the page.'
        return []
    }
}

// Fetch de datos del pokemon de la API
async function fetchPokemonData(pokemonName) {
    try {
        // Fetch de datos basicos del pokemon
        const pokemonResponse = await fetch(`https://pokeapi.co/api/v2/pokemon/${pokemonName}`)
        if (!pokemonResponse.ok) {
            throw new Error('Pokémon no encontrado')
        }
        const pokemonData = await pokemonResponse.json()
        
        // Fetch de datos de la especie para habitat, color, y cadena de evolucion
        const speciesResponse = await fetch(pokemonData.species.url)
        const speciesData = await speciesResponse.json()
        
        return {
            id: pokemonData.id,
            name: pokemonData.name,
            sprite: pokemonData.sprites.front_default, 
            types: pokemonData.types.map(t => t.type.name),
            height: pokemonData.height / 10, // Convert to meters
            weight: pokemonData.weight / 10, // Convert to kg
            habitat: speciesData.habitat ? speciesData.habitat.name : 'unknown',
            color: speciesData.color ? speciesData.color.name : 'unknown',
            evolutionStage: await determineEvolutionStage(speciesData),
            generation: getGenerationNumber(speciesData.generation.name)
        }
    } catch (error) {
         throw error
    }
}
```

#### 5.2.2 Autocompletado (autocomplete.js)

El autocompletado muestra sugerencias de Pokémon mientras el usuario escribe:

```javascript
// Fragmento de autocomplete.js
// Función para esperar un tiempo determinado
const sleep = (ms) => new Promise(resolve => setTimeout(resolve, ms));

// Inicializa el autocompletado
function initializeAutocomplete(input, dropdown, pokemonList, guessedPokemonArray) {
    let selectedIndex = -1;
    let lastQuery = '';
    let timeoutId = null;
    
    // Si no se proporciona un array de Pokémon adivinados, crear uno vacío
    const guessedPokemon = guessedPokemonArray || window.guessedPokemon || [];

    // Función para buscar Pokémon por nombre
    async function fetchPokemonByPrefix(prefix) {
        try {
            // Obtener la lista actualizada de Pokémon adivinados
            const currentGuessedPokemon = window.guessedPokemon || guessedPokemon;
            
            // Limitar a 15 resultados para mejor rendimiento
            const limit = 15;
            const response = await fetch(`https://pokeapi.co/api/v2/pokemon?limit=1025`);
            const data = await response.json();
            
            // Filtrar por prefijo y excluir los ya adivinados
            const filteredResults = data.results
                .filter(pokemon => {
                    return pokemon.name.startsWith(prefix) && 
                           !currentGuessedPokemon.includes(pokemon.name);
                })
                .slice(0, limit);
            
            // Obtener detalles de cada Pokémon con un retraso entre solicitudes
            const pokemonDetails = [];
            for (const pokemon of filteredResults) {
                try {
                    const detailResponse = await fetch(pokemon.url);
                    const pokemonData = await detailResponse.json();
                    
                    pokemonDetails.push({
                        name: pokemon.name,
                        id: pokemonData.id,
                        sprite: pokemonData.sprites.front_default
                    });
                    
                    // Esperar 50ms entre cada solicitud para evitar errores 429
                    await sleep(50);
                } catch (error) {
                    console.error(`Error fetching details for ${pokemon.name}:`, error);
                }
            }
            
            return pokemonDetails.sort((a, b) => a.id - b.id);
        } catch (error) {
            console.error('Error fetching Pokemon by prefix:', error);
            return [];
        }
    }

    // Maneja los cambios de entrada para el autocompletado con debounce
    input.addEventListener('input', () => {
        const query = input.value.toLowerCase().trim();
        
        // Limpiar el timeout anterior si existe
        if (timeoutId) {
            clearTimeout(timeoutId);
        }
        
        if (query.length === 0) {
            dropdown.style.display = 'none';
            return;
        }
        
        // Solo actualizar si la consulta ha cambiado
        if (query !== lastQuery) {
            lastQuery = query;
            
            // Establecer un timeout para evitar demasiadas solicitudes
            timeoutId = setTimeout(async () => {
                // Mostrar indicador de carga
                dropdown.innerHTML = '<div class="loading-item">Buscando Pokémon...</div>';
                dropdown.style.display = 'block';
                
                const matches = await fetchPokemonByPrefix(query);
                
                if (matches.length > 0) {
                    dropdown.innerHTML = '';
                    selectedIndex = -1;

                    for (const pokemon of matches) {
                        try {
                            const item = document.createElement('div');
                            item.className = 'autocomplete-item';
                            
                            // Crear la imagen con manejo de errores
                            const imgContainer = document.createElement('div');
                            imgContainer.className = 'pokemon-img-container';
                            
                            const img = document.createElement('img');
                            img.alt = pokemon.name;
                            // Usar una imagen de respaldo si el sprite no está disponible
                            if (!pokemon.sprite) {
                                img.src = '../assets/images/Pokebola.png'; // Imagen de respaldo
                                imgContainer.classList.add('fallback-img');
                            } else {
                                img.src = pokemon.sprite;
                                // Manejar errores de carga de imagen
                                img.onerror = function() {
                                    this.src = '../assets/images/Pokebola.png'; // Imagen de respaldo
                                    imgContainer.classList.add('fallback-img');
                                };
                            }
                            
                            imgContainer.appendChild(img);
                            
                            // Crear el span con el nombre
                            const nameSpan = document.createElement('span');
                            nameSpan.textContent = pokemon.name.charAt(0).toUpperCase() + pokemon.name.slice(1);
                            
                            // Agregar elementos al item
                            item.appendChild(imgContainer);
                            item.appendChild(nameSpan);

                            item.addEventListener('click', () => {
                                input.value = pokemon.name;
                                dropdown.style.display = 'none';
                                // Trigger de busqueda del pokemon
                                const event = new KeyboardEvent('keypress', { key: 'Enter' });
                                input.dispatchEvent(event);
                            });

                            dropdown.appendChild(item);
                        } catch (error) {
                            console.error(`Error displaying Pokemon ${pokemon.name}:`, error);
                        }
                    }

                    dropdown.style.display = 'block';
                } else {
                    dropdown.innerHTML = '<div class="no-results">No se encontraron Pokémon</div>';
                }
            }, 300); // Debounce de 300ms para evitar demasiadas solicitudes mientras el usuario escribe
        }
    });
}
```

#### 5.2.3 Lógica del Juego (game.js)

Este módulo maneja la lógica principal del juego, incluyendo la selección del Pokémon objetivo y la comparación de características:

```javascript
// Fragmento de game.js
import { fetchPokemonData } from './api.js';
import { showHint } from './hints.js';

// Variables globales
let targetPokemon = null;
let attempts = 0;
const MAX_ATTEMPTS = 10;

// Inicializa el juego seleccionando un Pokémon aleatorio
export async function initializeGame() {
    try {
        // Lista de todos los Pokémon disponibles (1-1025)
        const pokemonIds = Array.from({ length: 1025 }, (_, i) => i + 1);
        
        // Seleccionar un ID aleatorio
        const randomId = pokemonIds[Math.floor(Math.random() * pokemonIds.length)];
        
        // Obtener datos del Pokémon objetivo
        const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${randomId}`);
        const data = await response.json();
        
        // Obtener datos completos del Pokémon
        targetPokemon = await fetchPokemonData(data.name);
        console.log("Pokémon objetivo:", targetPokemon.name); // Para depuración
        
        return targetPokemon;
    } catch (error) {
        console.error("Error inicializando el juego:", error);
        throw error;
    }
}

// Procesa un intento del usuario
export async function processGuess(pokemonName) {
    attempts++;
    
    try {
        // Obtener datos del Pokémon ingresado
        const guessedPokemon = await fetchPokemonData(pokemonName);
        
        // Verificar si es el Pokémon correcto
        const isCorrect = guessedPokemon.name === targetPokemon.name;
        
        // Comparar características
        const comparison = comparePokemons(guessedPokemon, targetPokemon);
        
        // Actualizar pistas si es necesario
        if (attempts % 3 === 0 && attempts <= 9) {
            showHint(attempts / 3, targetPokemon);
        }
        
        return {
            pokemon: guessedPokemon,
            comparison: comparison,
            isCorrect: isCorrect,
            attempts: attempts,
            remaining: MAX_ATTEMPTS - attempts
        };
    } catch (error) {
        console.error("Error procesando intento:", error);
        throw error;
    }
}

// Compara las características entre dos Pokémon
function comparePokemons(guessed, target) {
    return {
        types: compareTypes(guessed.types, target.types),
        color: guessed.color === target.color ? 'correct' : 'incorrect',
        generation: compareGeneration(guessed.generation, target.generation),
        habitat: guessed.habitat === target.habitat ? 'correct' : 'incorrect',
        height: compareHeight(guessed.height, target.height),
        weight: compareWeight(guessed.weight, target.weight),
        evolutionStage: compareEvolutionStage(guessed.evolutionStage, target.evolutionStage)
    };
}

// Funciones auxiliares de comparación
function compareTypes(guessedTypes, targetTypes) {
    // Si algún tipo coincide, es parcialmente correcto
    if (guessedTypes.some(type => targetTypes.includes(type))) {
        // Si todos los tipos coinciden, es completamente correcto
        if (guessedTypes.length === targetTypes.length && 
            guessedTypes.every(type => targetTypes.includes(type))) {
            return 'correct';
        }
        return 'partial';
    }
    return 'incorrect';
}

function compareGeneration(guessedGen, targetGen) {
    if (guessedGen === targetGen) return 'correct';
    // Si está a una generación de distancia, es parcialmente correcto
    if (Math.abs(guessedGen - targetGen) === 1) return 'partial';
    return 'incorrect';
}

function compareHeight(guessedHeight, targetHeight) {
    if (guessedHeight === targetHeight) return 'correct';
    // Si está dentro del 20% de diferencia, es parcialmente correcto
    const difference = Math.abs(guessedHeight - targetHeight) / targetHeight;
    if (difference <= 0.2) return 'partial';
    return 'incorrect';
}

function compareWeight(guessedWeight, targetWeight) {
    if (guessedWeight === targetWeight) return 'correct';
    // Si está dentro del 20% de diferencia, es parcialmente correcto
    const difference = Math.abs(guessedWeight - targetWeight) / targetWeight;
    if (difference <= 0.2) return 'partial';
    return 'incorrect';
}

function compareEvolutionStage(guessedStage, targetStage) {
    if (guessedStage === targetStage) return 'correct';
    // Si está a una etapa de diferencia, es parcialmente correcto
    if (Math.abs(guessedStage - targetStage) === 1) return 'partial';
    return 'incorrect';
}
```

#### 5.2.4 Sistema de Pistas (hints.js)

Este módulo maneja la generación y visualización de pistas:

```javascript
// Fragmento de hints.js
// Muestra una pista basada en el número de intento
export function showHint(hintNumber, targetPokemon) {
    const hintElement = document.querySelector(`.hint[data-hint-number="${hintNumber}"]`);
    
    if (!hintElement) return;
    
    // Habilitar la pista
    hintElement.classList.add('enabled');
    
    // Obtener el contenedor de visualización de pista
    const hintDisplay = document.getElementById('hint-display');
    
    // Generar contenido de pista según el número
    let hintContent = '';
    
    switch(hintNumber) {
        case 1:
            // Primera pista: Silueta
            hintContent = `
                <h3>Pista 1: Silueta</h3>
                <div class="silhouette-hint">
                    <img src="${targetPokemon.sprite}" alt="Silueta" style="filter: brightness(0);">
                </div>
            `;
            break;
        case 2:
            // Segunda pista: Tipos
            const typesList = targetPokemon.types.map(type => 
                `<span class="type-badge ${type}">${type.charAt(0).toUpperCase() + type.slice(1)}</span>`
            ).join(' ');
            
            hintContent = `
                <h3>Pista 2: Tipos</h3>
                <p>Este Pokémon es de tipo:</p>
                <div class="types-hint">${typesList}</div>
            `;
            break;
        case 3:
            // Tercera pista: Primera letra
            hintContent = `
                <h3>Pista 3: Primera Letra</h3>
                <p>El nombre de este Pokémon comienza con la letra:</p>
                <div class="letter-hint">${targetPokemon.name.charAt(0).toUpperCase()}</div>
            `;
            break;
    }
    
    // Mostrar la pista cuando se hace clic en el botón
    hintElement.addEventListener('click', () => {
        hintDisplay.innerHTML = hintContent;
        hintDisplay.style.display = 'block';
        
        // Animación de aparición
        hintDisplay.classList.add('active');
        
        // Ocultar después de un tiempo
        setTimeout(() => {
            hintDisplay.classList.remove('active');
            setTimeout(() => {
                hintDisplay.style.display = 'none';
            }, 500);
        }, 5000);
    });
}
```

### 5.3 Servidor Multijugador (server.js)

El servidor maneja las conexiones de los jugadores y la lógica del juego multijugador:

```javascript
// Fragmento de server.js
const express = require('express');
const http = require('http');
const socketIo = require('socket.io');
const path = require('path');
const fetch = require('node-fetch');

const app = express();
const server = http.createServer(app);
const io = socketIo(server);

// Servir archivos estáticos
app.use(express.static(path.join(__dirname, '/')));

// Rutas
app.get('/', (req, res) => {
    res.sendFile(path.join(__dirname, 'index.html'));
});

// Almacenar información de las salas
const rooms = {};

// Cuando un cliente se conecta
io.on('connection', (socket) => {
    console.log('Nuevo usuario conectado:', socket.id);
    
    // Cuando un usuario crea una sala
    socket.on('createRoom', (username) => {
        const roomId = generateRoomId();
        
        // Crear la sala
        rooms[roomId] = {
            id: roomId,
            players: [{
                id: socket.id,
                username: username,
                isHost: true,
                ready: false
            }],
            gameStarted: false,
            targetPokemon: null
        };
        
        // Unir al usuario a la sala
        socket.join(roomId);
        socket.roomId = roomId;
        
        // Notificar al cliente
        socket.emit('roomCreated', { roomId, isHost: true });
        
        // Actualizar la lista de jugadores
        io.to(roomId).emit('updatePlayers', rooms[roomId].players);
        
        console.log(`Sala creada: ${roomId} por ${username}`);
    });
    
    // Cuando un jugador hace un intento
    socket.on('makeGuess', async ({ pokemonName }) => {
        const roomId = socket.roomId;
        
        if (!roomId || !rooms[roomId] || !rooms[roomId].gameStarted) return;
        
        try {
            // Obtener datos del Pokémon ingresado
            const guessedPokemon = await fetchPokemonData(pokemonName);
            
            // Verificar si es correcto
            const isCorrect = guessedPokemon.name === rooms[roomId].targetPokemon.name;
            
            // Enviar resultado al jugador
            socket.emit('guessResult', {
                pokemon: guessedPokemon,
                isCorrect: isCorrect
            });
            
            // Si es correcto, notificar a todos
            if (isCorrect) {
                const player = rooms[roomId].players.find(p => p.id === socket.id);
                io.to(roomId).emit('gameOver', {
                    winner: player.username,
                    pokemon: rooms[roomId].targetPokemon
                });
            }
        } catch (error) {
            socket.emit('error', 'Pokémon no encontrado');
        }
    });
});

// Iniciar el servidor
const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
    console.log(`Servidor iniciado en puerto ${PORT}`);
});
```

## 6. Estilos CSS y Animaciones

### 6.1 Estilos Principales (style.css)

Los estilos CSS definen la apariencia visual del juego, incluyendo colores, fuentes y disposición de elementos:

```css
/* Fragmento de style.css */
@font-face {
    font-family: 'Pokemon';
    src: url('../assets/font/Pokemon.ttf') format('truetype');
}

@font-face {
    font-family: 'Text';
    src: url('../assets/font/Text.ttf') format('truetype');
}

.pokemon-card {
    width: 100%;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    height: 120px;
    position: relative;
}

.info-cell {
    flex: 1 1 12.5%; 
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    text-align: center;
    padding: 4px 2px;
    font-weight: bold;
    border: 1px solid #e0e0e0;
    opacity: 0;
    transform: translateY(-100%);
    transition: opacity 0.4s ease, transform 0.4s ease;
    color: white;
    font-family: Text;
    letter-spacing: 0px; 
    background-color: #df2b19; /* Color por defecto (incorrecto) */
}

.info-cell.correct {
    background-color: #4CAF50 !important; /* Verde para coincidencias exactas */
}

.info-cell.partial {
    background-color: #FFC107 !important; /* Amarillo para valores cercanos */
}

/* Estilos para el autocompletado */
.autocomplete-container {
    position: relative;
    width: 100%;
    max-width: 500px;
    margin: 0 auto;
}

.autocomplete-dropdown {
    position: absolute;
    width: 100%;
    max-height: 300px;
    overflow-y: auto;
    background-color: white;
    border: 1px solid #ddd;
    border-top: none;
    border-radius: 0 0 4px 4px;
    z-index: 10;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    display: none;
}

.autocomplete-item {
    padding: 10px;
    cursor: pointer;
    display: flex;
    align-items: center;
}

.autocomplete-item:hover {
    background-color: #f5f5f5;
}

.pokemon-img-container {
    width: 40px;
    height: 40px;
    margin-right: 10px;
    display: flex;
    align-items: center;
    justify-content: center;
}

.pokemon-img-container img {
    max-width: 100%;
    max-height: 100%;
}

.fallback-img img {
    width: 30px;
    height: 30px;
    opacity: 0.7;
}

.loading-item, .no-results {
    padding: 15px;
    text-align: center;
    color: #666;
}
```

### 6.2 Animaciones (animations.js)

Las animaciones mejoran la experiencia del usuario proporcionando feedback visual:

```javascript
// Fragmento de animations.js
// Anima la tarjeta del Pokémon cuando se muestra por primera vez
export function animatePokemonCard(cardElements) {
    const { card, intro, data, image, cells } = cardElements;
    
    // Mostrar la introducción primero
    intro.classList.add('fade-in');
    
    // Después de 2 segundos, ocultar la introducción y mostrar los datos
    setTimeout(() => {
        intro.classList.add('fade-out');
        
        // Animar la imagen del Pokémon
        setTimeout(() => {
            image.classList.add('slide-in');
            
            // Animar las celdas de información una por una
            setTimeout(() => {
                data.classList.add('show');
                
                // Animar cada celda con un retraso
                cells.forEach((cell, index) => {
                    setTimeout(() => {
                        cell.style.opacity = '1';
                        cell.style.transform = 'translateY(0)';
                    }, index * 100); // 100ms de retraso entre cada celda
                });
            }, 500);
        }, 500);
    }, 2000);
}

// Anima la pantalla de victoria
export function animateVictoryScreen(victoryElements) {
    const { container, pokemon, message, replay } = victoryElements;
    
    // Mostrar el contenedor con una animación de fade
    container.style.display = 'flex';
    setTimeout(() => {
        container.classList.add('active');
        
        // Animar el Pokémon con un efecto de rebote
        setTimeout(() => {
            pokemon.classList.add('bounce-in');
            
            // Mostrar el mensaje de victoria
            setTimeout(() => {
                message.classList.add('slide-in');
                
                // Mostrar el botón de volver a jugar
                setTimeout(() => {
                    replay.classList.add('fade-in');
                }, 1000);
            }, 1000);
        }, 500);
    }, 100);
}

// Anima la pantalla de carga
export function animateLoadingScreen(progress, percentage) {
    // Actualizar la barra de progreso
    progress.style.width = `${percentage}%`;
    
    // Agregar efectos visuales según el porcentaje
    if (percentage > 50) {
        progress.classList.add('halfway');
    }
    
    if (percentage > 80) {
        progress.classList.add('almost-done');
    }
    
    // Efecto de pulso cada 10%
    if (percentage % 10 === 0) {
        progress.classList.add('pulse');
        setTimeout(() => {
            progress.classList.remove('pulse');
        }, 500);
    }
}
```

## 7. Conclusiones y Recomendaciones para Desarrolladores

### 7.1 Mejores Prácticas Implementadas

1. **Manejo Eficiente de API**:
   - Implementación de retrasos entre solicitudes para evitar errores 429 (demasiadas solicitudes).
   - Sistema de caché para reducir solicitudes redundantes.
   - Manejo robusto de errores con reintentos automáticos.

2. **Optimización de Rendimiento**:
   - Carga dinámica de Pokémon según se necesitan, en lugar de cargar todos al inicio.
   - Implementación de "debounce" para reducir solicitudes durante la escritura en el autocompletado.
   - Carga progresiva de elementos visuales para mejorar la percepción de velocidad.

3. **Experiencia de Usuario**:
   - Feedback visual inmediato con código de colores para las comparaciones.
   - Sistema de pistas que se desbloquean progresivamente para mantener el interés.
   - Animaciones fluidas que mejoran la interacción sin entorpecer la jugabilidad.

4. **Seguridad**:
   - Creación segura de elementos DOM sin usar `innerHTML` para prevenir XSS.
   - Validación de entradas del usuario antes de procesarlas.
   - Manejo seguro de IDs de sala en el modo multijugador.

### 7.2 Áreas de Mejora Potencial

1. **Optimización Adicional**:
   - Implementar un sistema de caché más robusto para datos de Pokémon frecuentemente usados.
   - Utilizar Web Workers para procesar datos en segundo plano sin bloquear la interfaz.
   - Implementar carga perezosa (lazy loading) para imágenes de Pokémon.

2. **Características Adicionales**:
   - Sistema de puntuación basado en el número de intentos y tiempo.
   - Modo de dificultad que limite las pistas o aumente el número de Pokémon posibles.
   - Integración con redes sociales para compartir resultados.

3. **Accesibilidad**:
   - Mejorar el soporte para lectores de pantalla.
   - Implementar modos de alto contraste y opciones para daltonismo.
   - Asegurar que el juego sea completamente jugable con teclado.

### 7.3 Recomendaciones para Nuevos Desarrolladores

1. **Familiarización con el Código**:
   - Comenzar estudiando la estructura del proyecto y los componentes principales.
   - Ejecutar el juego en modo de desarrollo y usar las herramientas de desarrollo del navegador para entender el flujo de datos.

2. **Modificaciones y Extensiones**:
   - Usar los patrones existentes al agregar nuevas características.
   - Mantener la separación de preocupaciones entre API, lógica de juego e interfaz de usuario.
   - Documentar cualquier cambio significativo en el código.

3. **Pruebas**:
   - Probar exhaustivamente cualquier cambio en diferentes navegadores.
   - Verificar el rendimiento con diferentes velocidades de conexión.
   - Asegurar que las nuevas características no rompan la compatibilidad con las existentes.

## 8. Guía de Solución de Problemas Comunes

### 8.1 Problemas de Conexión con la API

| Problema | Posible Causa | Solución |
|----------|---------------|----------|
| Error 429 (Too Many Requests) | Demasiadas solicitudes a la PokéAPI en poco tiempo | Implementar un sistema de cola con retrasos entre solicitudes. Verificar que el retraso de 50ms entre solicitudes esté funcionando correctamente. |
| Tiempos de espera prolongados | Conexión lenta o sobrecarga del servidor | Aumentar el tiempo de espera en las solicitudes fetch. Implementar reintentos automáticos con backoff exponencial. |
| Datos incompletos o corruptos | Respuesta parcial de la API | Validar siempre los datos recibidos antes de procesarlos. Implementar un sistema de fallback para usar datos en caché cuando sea posible. |

### 8.2 Problemas de Rendimiento

| Problema | Posible Causa | Solución |
|----------|---------------|----------|
| Carga inicial lenta | Precarga de demasiados datos de Pokémon | Utilizar la carga bajo demanda implementada. Verificar que no se estén cargando todos los Pokémon al inicio. |
| Lag durante la escritura | Demasiadas solicitudes de autocompletado | Verificar que el sistema de debounce esté funcionando correctamente (300ms). Considerar aumentar el retraso si persiste el problema. |
| Alto consumo de memoria | Caché de imágenes sin límite | Implementar un límite en el número de imágenes cacheadas y liberar las menos usadas. |

### 8.3 Problemas del Modo Multijugador

| Problema | Posible Causa | Solución |
|----------|---------------|----------|
| Desconexiones frecuentes | Problemas de red o timeout del servidor | Implementar reconexiones automáticas y persistencia de estado de juego. |
| Inconsistencia de estado entre jugadores | Sincronización incorrecta | Asegurar que todos los eventos relevantes se emitan a todos los jugadores y verificar la consistencia del estado. |
| Salas fantasma | Jugadores que salen sin cerrar correctamente | Implementar limpieza periódica de salas inactivas y manejo de eventos de desconexión. |

### 8.4 Problemas de Interfaz de Usuario

| Problema | Posible Causa | Solución |
|----------|---------------|----------|
| Imágenes de Pokémon no cargan | Errores en la URL o recursos no disponibles | Verificar que el sistema de fallback a la imagen de Pokebola esté funcionando. Considerar precargar las imágenes más comunes. |
| Animaciones entrecortadas | Demasiadas animaciones simultáneas | Reducir la complejidad de las animaciones o desactivarlas en dispositivos de bajo rendimiento. |
| Problemas de visualización en móviles | Diseño no responsivo | Revisar los media queries y asegurar que todos los elementos se adaptan correctamente a diferentes tamaños de pantalla. |

## 9. Consideraciones de Escalabilidad

### 9.1 Escalabilidad del Servidor

#### Arquitectura Actual
Actualmente, el servidor utiliza una arquitectura monolítica con Socket.io para la comunicación en tiempo real. Esto funciona bien para un número moderado de usuarios, pero puede presentar limitaciones al escalar.

#### Recomendaciones para Alta Carga

1. **Implementación de Microservicios**:
   - Separar la lógica de salas, juego y autenticación en servicios independientes.
   - Utilizar un sistema de mensajería como RabbitMQ o Kafka para la comunicación entre servicios.

2. **Balanceo de Carga**:
   - Implementar múltiples instancias del servidor detrás de un balanceador de carga.
   - Configurar Socket.io con Redis Adapter para permitir comunicación entre instancias.

3. **Base de Datos**:
   - Migrar de almacenamiento en memoria a una base de datos como MongoDB para persistencia.
   - Implementar caché con Redis para datos frecuentemente accedidos.

### 9.2 Optimización del Cliente

1. **Carga Progresiva**:
   - Implementar lazy loading para cargar solo los recursos necesarios según la vista actual.
   - Utilizar code splitting para reducir el tamaño del bundle inicial.

2. **Caché Inteligente**:
   - Utilizar Service Workers para cachear recursos estáticos y respuestas de API.
   - Implementar una estrategia de caché que priorice los Pokémon más frecuentemente utilizados.

3. **Optimización de Recursos**:
   - Comprimir imágenes y utilizar formatos modernos como WebP.
   - Implementar carga adaptativa de recursos según la conexión del usuario.

### 9.3 Monitoreo y Análisis

1. **Implementación de Telemetría**:
   - Integrar herramientas como New Relic o Datadog para monitoreo en tiempo real.
   - Establecer alertas para métricas críticas como latencia y uso de memoria.

2. **Análisis de Uso**:
   - Recopilar datos anónimos sobre patrones de juego para optimizar la experiencia.
   - Identificar cuellos de botella y oportunidades de mejora basadas en datos reales.

3. **Pruebas de Carga**:
   - Realizar pruebas de carga periódicas para identificar límites del sistema.
   - Simular escenarios de tráfico pico para asegurar la estabilidad.

### 9.4 Plan de Escalado Gradual

| Etapa | Número de Usuarios | Acciones Recomendadas |
|-------|---------------------|----------------------|
| 1 | < 1,000 concurrentes | Arquitectura actual con optimizaciones básicas. |
| 2 | 1,000 - 5,000 concurrentes | Implementar balanceo de carga y Redis para sesión compartida. |
| 3 | 5,000 - 20,000 concurrentes | Migrar a microservicios y base de datos escalable. |
| 4 | > 20,000 concurrentes | Considerar arquitectura serverless y CDN global. |

## 10. Recursos Adicionales

- **Documentación de PokéAPI**: [https://pokeapi.co/docs/v2](https://pokeapi.co/docs/v2)
- **Socket.io (para multijugador)**: [https://socket.io/docs/](https://socket.io/docs/)
- **MDN Web Docs (para JavaScript/CSS)**: [https://developer.mozilla.org/](https://developer.mozilla.org/)
- **Redis Adapter para Socket.io**: [https://socket.io/docs/v4/redis-adapter/](https://socket.io/docs/v4/redis-adapter/)
- **Herramientas de Monitoreo**: [https://newrelic.com/](https://newrelic.com/) y [https://www.datadoghq.com/](https://www.datadoghq.com/)

---

Este manual ha sido creado para ayudar a los desarrolladores a entender y extender el juego "Adivina el Pokémon". Si tienes preguntas adicionales o encuentras problemas, no dudes en contactar al equipo de desarrollo.
