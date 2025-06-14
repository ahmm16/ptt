# Módulo de Gestión de Torneos - Pro Table Tennis (PTT)

## Visión General

El módulo de torneos permite a entrenadores, clubes y jugadores organizar eventos competitivos de tenis de mesa con:

- **Automatización completa de la gestión**
- **Sistema de rankings ELO integrado**
- **Procesamiento de pagos seguro**
- **Comunicación centralizada**

```mermaid
graph TD
    A[Crear Torneo] --> B[Configuración]
    B --> C[Inscripciones]
    C --> D[Generación de Grupos]
    D --> E[Gestión de Partidos]
    E --> F[Cálculo ELO]
    F --> G[Resultados Finales]
```

---

## Modelo de Datos

```mermaid
erDiagram
    TOURNAMENT ||--o{ GROUP : contains
    TOURNAMENT ||--|{ PARTICIPANT : has
    TOURNAMENT ||--o{ MATCH : has
    GROUP ||--|{ PARTICIPANT : includes
    GROUP ||--o{ MATCH : has
    USER ||--o{ TOURNAMENT : organizes
    USER ||--o{ PARTICIPANT : is

    TOURNAMENT {
        string name
        dateTime date
        string location
        string type
        string gameSystem
        decimal entryFee
        dateTime registrationDeadline
        string contactInfo
        int maxParticipants
    }
    GROUP {
        string name
        string category
    }
    PARTICIPANT {
        int currentELO
        string status
    }
    MATCH {
        int player1Id
        int player2Id
        int winnerId
        int scorePlayer1
        int scorePlayer2
        string status
    }
```

---

## Funcionalidades Clave

### 1. Configuración de Torneos

**Componente:** `TournamentCreationForm.jsx`

```jsx
function TournamentCreationForm() {
  const [tournament, setTournament] = useState({
    name: "IX Torneo Adultos",
    date: "2025-06-15T10:00",
    location: "Club Noroeste T.M. Pabellón Martín Dones, C/Real 58, Las Rozas",
    type: "Open",
    gameSystem: "groups_knockout",
    categories: ["Open A", "Open B"],
    entryFee: 10.0,
    registrationDeadline: "2025-06-14T12:00",
    contact: { name: "Blanca", phone: "661794205" },
    maxParticipants: 32,
  });

  return (
    <NeumorphicCard>
      <h2>Crear Nuevo Torneo</h2>
      <NeumorphicInput
        label="Nombre del Torneo"
        value={tournament.name}
        onChange={(e) => setTournament({ ...tournament, name: e.target.value })}
      />
      <DateTimePicker
        label="Fecha y Hora"
        value={tournament.date}
        onChange={(date) => setTournament({ ...tournament, date })}
      />
      <LocationPicker
        label="Ubicación"
        value={tournament.location}
        onChange={(location) => setTournament({ ...tournament, location })}
      />
      <NeumorphicButton>Publicar Torneo</NeumorphicButton>
    </NeumorphicCard>
  );
}
```

---

### 2. Sistema de Inscripciones y Pagos

**Flujo de trabajo:**

1. Jugador visita página del torneo
2. Completa formulario de inscripción
3. Realiza pago mediante Stripe/PayPal
4. Recibe confirmación y detalles vía email/app

**Componente:** `TournamentRegistration.jsx`

```jsx
function TournamentRegistration({ tournament }) {
  const [playerData, setPlayerData] = useState({
    name: "",
    email: "",
    phone: "",
    category: "",
    isFederated: false,
    federationLevel: "",
  });

  const handlePaymentSuccess = (paymentId) => {
    api.registerForTournament(tournament.id, playerData, paymentId);
  };

  return (
    <NeumorphicCard>
      <h3>Inscripción: {tournament.name}</h3>
      <p>Fecha: {formatDate(tournament.date)}</p>
      <p>Precio: {tournament.entryFee}€</p>

      <NeumorphicInput label="Nombre Completo" required />
      <NeumorphicInput label="Email" type="email" required />
      <NeumorphicInput label="Teléfono" type="tel" required />

      <CategorySelector
        categories={tournament.categories}
        onChange={(category) => setPlayerData({ ...playerData, category })}
      />

      <StripePayment
        amount={tournament.entryFee * 100}
        onSuccess={handlePaymentSuccess}
      />
    </NeumorphicCard>
  );
}
```

---

### 3. Generación Automática de Grupos y Cuadros

**Algoritmo de agrupamiento:**

```javascript
function generateTournamentGroups(players, groupsCount) {
  // Ordenar jugadores por ELO
  const sortedPlayers = [...players].sort((a, b) => b.elo - a.elo);

  // Distribución serpentina para grupos equilibrados
  const groups = Array(groupsCount)
    .fill()
    .map(() => []);

  for (let i = 0; i < sortedPlayers.length; i++) {
    const groupIndex = i % groupsCount;
    groups[groupIndex].push(sortedPlayers[i]);
  }

  return groups.map((group, idx) => ({
    name: `Grupo ${String.fromCharCode(65 + idx)}`,
    players: group,
    matches: generateGroupMatches(group),
  }));
}

function generateGroupMatches(players) {
  const matches = [];
  for (let i = 0; i < players.length; i++) {
    for (let j = i + 1; j < players.length; j++) {
      matches.push({
        player1: players[i].id,
        player2: players[j].id,
        status: "pending",
      });
    }
  }
  return matches;
}
```

---

### 4. Sistema de Rankings ELO

**Cálculo de cambios ELO:**

```javascript
function calculateELOChanges(winnerELO, loserELO, K = 32) {
  const expectedWin = 1 / (1 + Math.pow(10, (loserELO - winnerELO) / 400));
  const winnerChange = Math.round(K * (1 - expectedWin));
  const loserChange = -Math.round(K * expectedWin);

  return { winnerChange, loserChange };
}
```

**Componente:** `ELOTracker.jsx`

```jsx
function ELOTracker({ playerId }) {
  const { eloHistory } = useELOHistory(playerId);

  return (
    <NeumorphicCard>
      <h3>Progreso ELO</h3>
      <LineChart
        data={eloHistory}
        xKey="date"
        yKey="elo"
        width={400}
        height={200}
      />
      <div className="current-elo">
        ELO Actual:{" "}
        <strong>{eloHistory[eloHistory.length - 1]?.elo || 1000}</strong>
      </div>
    </NeumorphicCard>
  );
}
```

---

### 5. Vista Pública de Torneos

**Componente:** `TournamentPage.jsx`

```jsx
function TournamentPage({ tournament }) {
  return (
    <div className="tournament-page">
      <header className="tournament-header">
        <h1>{tournament.name}</h1>
        <div className="tournament-meta">
          <span>{tournament.date}</span>
          <span>{tournament.location}</span>
        </div>
      </header>

      <div className="tournament-details">
        <NeumorphicCard>
          <h2>Detalles del Torneo</h2>
          <p>
            <strong>Sistema de juego:</strong> {tournament.gameSystem}
          </p>
          <p>
            <strong>Categorías:</strong> {tournament.categories.join(", ")}
          </p>
          <p>
            <strong>Inscripción:</strong> {tournament.entryFee}€
          </p>
          <p>
            <strong>Plazo inscripción:</strong>{" "}
            {tournament.registrationDeadline}
          </p>
          <p>
            <strong>Contacto:</strong> {tournament.contact.name} -{" "}
            {tournament.contact.phone}
          </p>
        </NeumorphicCard>

        <RegistrationStatus
          registeredCount={tournament.participants.length}
          maxParticipants={tournament.maxParticipants}
          deadline={tournament.registrationDeadline}
        />

        {tournament.groups.length > 0 && (
          <GroupsView groups={tournament.groups} />
        )}

        {tournament.knockoutStage && (
          <BracketView bracket={tournament.knockoutStage} />
        )}
      </div>
    </div>
  );
}
```

---

## Plan de Implementación

### Backend (Node.js/Express)

**Endpoints:**

- `POST /tournaments` - Crear nuevo torneo
- `GET /tournaments/upcoming` - Listar torneos próximos
- `POST /tournaments/:id/register` - Registrar jugador
- `POST /tournaments/:id/generate-groups` - Generar grupos
- `POST /matches/:id/result` - Registrar resultado

**Servicios:**

```javascript
class TournamentService {
  async createTournament(data) {
    // Validación y creación en DB
  }

  async registerPlayer(tournamentId, playerData, paymentInfo) {
    // Registrar jugador y procesar pago
  }

  async generateGroups(tournamentId) {
    // Generación automática de grupos
  }

  async recordMatchResult(matchId, scores) {
    // Actualizar resultado y calcular ELO
  }
}
```

### Frontend (React)

**Vistas:**

- `TournamentCreationView.jsx` - Creación de torneos
- `TournamentListView.jsx` - Listado de torneos disponibles
- `TournamentDetailView.jsx` - Vista pública de torneo
- `PlayerDashboardView.jsx` - Panel de jugador (inscripciones, resultados)
- `TournamentManagerView.jsx` - Gestión en tiempo real para organizadores

**Componentes Clave:**

- `BracketVisualization.jsx` - Visualización interactiva de cuadros
- `ELOLeaderboard.jsx` - Tabla de clasificación ELO
- `MatchCard.jsx` - Tarjeta para registrar resultados
- `GroupStageView.jsx` - Vista de grupos y partidos

---

## Timeline de Implementación

| Módulo                          | Estimación  | Prioridad |
| ------------------------------- | ----------- | --------- |
| Creación de torneos             | 2 días      | Alta      |
| Sistema de inscripciones        | 3 días      | Alta      |
| Pasarela de pagos               | 2 días      | Alta      |
| Generación de grupos            | 1 día       | Media     |
| Visualización de cuadros        | 2 días      | Media     |
| Cálculo ELO                     | 1 día       | Alta      |
| Vista pública (flyer-style)     | 2 días      | Alta      |
| Panel de gestión en tiempo real | 3 días      | Media     |
| **Total**                       | **16 días** |           |

---

## Estrategia de Monetización

- **Suscripción Premium:**  
  €7.99/mes para funciones avanzadas de torneos  
  Incluye generación ilimitada de torneos

- **Comisión por Pago:**  
  5% sobre cada inscripción pagada  
  Mínimo €0.50 por transacción

- **Patrocinios Integrados:**  
  Espacios para marcas en vistas de torneos  
  Logos en documentos oficiales

- **Características Premium:**
  - Streaming integrado (€4.99/torneo)
  - Análisis estadístico avanzado (€2.99/torneo)
  - Certificados digitales para ganadores (€0.99/participante)

---

## Vista de Flyer Integrada

**Componente:** `TournamentFlyer.jsx`

```jsx
function TournamentFlyer({ tournament }) {
  return (
    <div className="tournament-flyer">
      <div className="flyer-header">
        <h1>{tournament.name}</h1>
        <h2>Tenis de Mesa</h2>
        <h3>{formatDate(tournament.date, "D MMMM YYYY")}</h3>
      </div>

      <div className="flyer-content">
        <div className="game-system">
          <p>
            <strong>Sistema de juego:</strong> {tournament.gameSystem}
          </p>
          {tournament.categories.map((cat, idx) => (
            <p key={idx} className="category-tag">
              {cat}
            </p>
          ))}
        </div>

        <div className="participant-info">
          <p>
            <strong>Participantes:</strong> {tournament.participantRequirements}
          </p>
        </div>

        <div className="time-location">
          <p>
            <strong>Hora:</strong> {formatDate(tournament.date, "HH:mm")}
          </p>
          <p>
            <strong>Lugar:</strong> {tournament.location}
          </p>
        </div>

        <div className="registration-info">
          <p>
            <strong>Inscripción:</strong> {tournament.entryFee}€
          </p>
          <p>
            <strong>Plazo:</strong> Hasta{" "}
            {formatDate(
              tournament.registrationDeadline,
              "D MMMM [a las] HH:mm"
            )}
          </p>
        </div>

        <div className="contact-info">
          <p>
            <strong>Contacto:</strong> {tournament.contact.name} -{" "}
            {tournament.contact.phone}
          </p>
        </div>
      </div>

      <div className="flyer-footer">
        <QRCode value={`${APP_URL}/tournaments/${tournament.id}`} />
        <p>Escanea para inscribirte</p>
      </div>
    </div>
  );
}
```

**Estilos CSS:**

```css
.tournament-flyer {
  background: linear-gradient(135deg, #4e54c8 0%, #8f94fb 100%);
  color: white;
  padding: 2rem;
  border-radius: 20px;
  text-align: center;
  max-width: 600px;
  margin: 0 auto;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
}

.flyer-header h1 {
  font-size: 2.5rem;
  margin-bottom: 0.5rem;
  text-transform: uppercase;
  letter-spacing: 2px;
}

.flyer-header h2 {
  font-size: 1.8rem;
  margin-bottom: 1rem;
}

.flyer-header h3 {
  font-size: 1.5rem;
  margin-bottom: 2rem;
}

.flyer-content {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border-radius: 15px;
  padding: 1.5rem;
  margin-bottom: 1.5rem;
}

.category-tag {
  display: inline-block;
  background: #f9a826;
  color: #2d3748;
  padding: 0.3rem 0.8rem;
  border-radius: 20px;
  margin: 0.3rem;
  font-weight: bold;
}

.contact-info {
  font-size: 1.2rem;
  margin-top: 1rem;
}

.flyer-footer {
  margin-top: 1.5rem;
}
```

---

## Beneficios para Entrenadores y Clubes

- **Ahorro de tiempo:** Automatización de procesos manuales
- **Profesionalismo:** Presentación moderna de torneos
- **Atracción de participantes:** Mayor alcance con compartición en redes
- **Gestión financiera:** Control de pagos e ingresos
- **Análisis de datos:** Seguimiento del rendimiento de jugadores

---

Este módulo transformará la organización de torneos de una tarea administrativa compleja en un proceso fluido y profesional, atrayendo más participantes y generando nuevas fuentes de ingresos para tu club o academia.
