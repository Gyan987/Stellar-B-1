# Roastellar Architecture



## System Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              CLIENT LAYER                                │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Next.js 16 (App Router)                                         │   │
│  │  ├── Clerk Authentication (OAuth, JWT sessions)                   │   │
│  │  ├── Freighter Wallet Integration (Stellar)                      │   │
│  │  ├── Socket.IO Client (Real-time updates)                        │   │
│  │  └── TailwindCSS + Framer Motion (UI)                             │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                             BACKEND LAYER                                │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Node.js + Express Server                                         │   │
│  │  ├── REST API Routes                                              │   │
│  │  ├── Socket.IO Server (Real-time battle events)                   │   │
│  │  ├── MongoDB (Persistent data storage)                            │   │
│  │  └── Clerk Webhook Handler (User sync)                            │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                    │                                    │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Services Layer                                                  │   │
│  │  ├── BattleService (Match lifecycle)                             │   │
│  │  ├── BattleChainService (Soroban contract calls)                  │   │
│  │  ├── BattleEscrowService (XLM transfers)                         │   │
│  │  ├── PredictionService (Prediction markets)                      │   │
│  │  ├── WalletService (Stellar wallet management)                    │   │
│  │  └── IPFSService (Content storage)                               │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    ▼                               ▼
┌─────────────────────────────────┐   ┌─────────────────────────────────┐
│      BLOCKCHAIN LAYER           │   │        STORAGE LAYER            │
│  ┌───────────────────────────┐ │   │  ┌───────────────────────────┐ │
│  │  Stellar Soroban          │ │   │  │  MongoDB Atlas             │ │
│  │  ├── Contract: roastellar │ │   │  │  ├── Users collection      │ │
│  │  ├── Network: Testnet     │ │   │  │  ├── Battles collection    │ │
│  │  └── RPC: soroban-testnet │ │   │  │  ├── Predictions collection │ │
│  └───────────────────────────┘ │   │  │  └── Analytics collection │ │
│  ┌───────────────────────────┐ │   │  └───────────────────────────┘ │
│  │  Stellar Horizon          │ │   │  ┌───────────────────────────┐ │
│  │  ├── XLM Transfers        │ │   │  │  IPFS (Pinata/Local)      │ │
│  │  └── Account Queries       │ │   │  │  ├── Roast content CID    │ │
│  └───────────────────────────┘ │   │  │  └── Topic CID storage    │ │
└─────────────────────────────────┘   │  └───────────────────────────┘ │
                                      └─────────────────────────────────┘
```

## Frontend Architecture

### Tech Stack
- **Framework:** Next.js 16.2.4 (App Router)
- **Auth:** Clerk 7.2.5 (OAuth + JWT sessions)
- **Wallet:** @stellar/freighter-api 6.0.1
- **Real-time:** Socket.IO Client 4.7.2
- **Styling:** TailwindCSS 3.4.1 + Framer Motion 11.0.3
- **State:** Zustand 4.5.0

### Pages

| Route | Description |
|-------|-------------|
| `/` | Landing page with Hero, Features, HowItWorks |
| `/sign-in` | Clerk authentication |
| `/sign-up` | Clerk registration |
| `/onboarding` | Mini-game + wallet creation flow |
| `/dashboard` | Main hub with stats and quick actions |
| `/battles` | List of open battles |
| `/battle/[id]` | Live battle room with real-time updates |
| `/leaderboard` | Rankings by XP and wins |
| `/wallet` | Wallet management and XLM display |
| `/profile` | User profile, badges, battle history |

### Component Structure

```
Frontend/src/
├── components/
│   ├── Sidebar.tsx           # Navigation sidebar (desktop) + bottom nav (mobile)
│   ├── Navbar.tsx             # Top navigation bar
│   ├── BattleCard.tsx         # Battle preview card
│   ├── LeaderboardTable.tsx   # Ranking display table
│   ├── MiniGame.tsx           # Onboarding keyboard game
│   ├── WalletCard.tsx         # Wallet info display
│   ├── WalletReveal.tsx       # First-time wallet reveal modal
│   ├── FreighterConnectCard.tsx # Wallet connection prompt
│   ├── PredictionPanel.tsx    # Prediction market UI
│   ├── StatCard.tsx           # Stats display card
│   ├── Hero.tsx               # Landing page hero section
│   ├── Features.tsx           # Landing page features
│   ├── HowItWorks.tsx         # Landing page how-to
│   ├── LoadingScreen.tsx      # Loading states
│   └── ClerkProvider.tsx      # Clerk integration wrapper
├── lib/
│   ├── api.ts                 # REST API client with normalizers
│   ├── socket.ts              # Socket.IO event handlers
│   ├── freighter.ts           # Freighter wallet SDK wrapper
│   ├── hooks.ts               # Custom React hooks
│   ├── types.ts               # TypeScript interfaces
│   └── utils.ts               # Utility functions
└── app/
    └── [routes]               # Next.js App Router pages
```

### API Client Pattern

The frontend uses a normalized API pattern (`api.ts`) that:
1. Makes Axios requests to backend
2. Receives backend response formats
3. Normalizes data structures for frontend use
4. Returns typed data to components

## Backend Architecture

### Tech Stack
- **Runtime:** Node.js
- **Framework:** Express.js
- **Database:** MongoDB (Mongoose ODM)
- **Real-time:** Socket.IO 4.x
- **Auth:** Clerk (webhooks + JWT verification)
- **Blockchain:** Stellar SDK + Soroban RPC

### Module Structure

```
Backend/src/
├── app.js                     # Express app configuration
├── server.js                  # HTTP server entry point
├── config/
│   ├── clerk.js              # Clerk configuration
│   ├── stellar.js             # Stellar SDK setup
│   ├── socket.js              # Socket.IO configuration
│   ├── db.js                  # MongoDB connection
│   └── firebase.js           # Firebase (unused currently)
├── modules/
│   ├── users/
│   │   ├── models/user.model.js
│   │   ├── routes/user.routes.js
│   │   ├── controllers/user.controller.js
│   │   └── services/user.service.js
│   ├── battles/
│   │   ├── models/
│   │   │   ├── battle.model.js
│   │   │   ├── battleVote.model.js
│   │   │   └── battleCounter.model.js
│   │   ├── routes/battle.routes.js
│   │   ├── controllers/battle.controller.js
│   │   └── services/
│   │       ├── battle.service.js       # Core battle logic
│   │       ├── battleChain.service.js  # Soroban contract calls
│   │       ├── battleEscrow.service.js # XLM transfer logic
│   │       ├── battleTimer.service.js  # Timer management
│   │       └── stellar.service.js       # Stellar helpers
│   ├── predictions/
│   │   ├── models/prediction.model.js
│   │   ├── routes/prediction.routes.js
│   │   ├── controllers/prediction.controller.js
│   │   └── services/prediction.service.js
│   ├── leaderboard/
│   │   ├── routes/leaderboard.routes.js
│   │   └── controllers/leaderboard.controller.js
│   ├── wallet/
│   │   ├── wallet.routes.js
│   │   ├── wallet.controller.js
│   │   └── wallet.service.js
│   ├── auth/
│   │   ├── routes/clerk.routes.js
│   │   ├── routes/auth.routes.js
│   │   ├── controllers/auth.controller.js
│   ��   └── services/auth.service.js
│   ├── admin/
│   │   ├── routes/admin.routes.js
│   │   └── controllers/admin.controller.js
│   ├── uploads/
│   │   ├── routes/upload.routes.js
│   │   └── services/upload.service.js
│   └── analytics/
│       └── models/analytics.model.js
├── sockets/
│   └── battle.socket.js       # Socket.IO battle event handlers
├── middlewares/
│   ├── clerk.middleware.js     # Clerk JWT verification
│   ├── auth.middleware.js      # Auth checks
│   ├── validate.middleware.js  # Request validation
│   └── error.middleware.js     # Error handling
└── utils/
    ├── constants.js            # App constants
    ├── logger.js               # Logging utility
    └── apiResponse.js          # Response helpers
```

### REST API Routes

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/users/me` | GET | Get current user profile |
| `/api/users/me` | PATCH | Update user profile |
| `/api/leaderboard` | GET | Get leaderboard rankings |
| `/api/battles/open` | GET | List open battles |
| `/api/battles/:id` | GET | Get battle details |
| `/api/battles/create` | POST | Create new battle |
| `/api/battles/join/:id` | POST | Join a battle |
| `/api/battles/submit-roast/:id` | POST | Submit roast text |
| `/api/battles/vote/:id` | POST | Cast vote |
| `/api/battles/finalize/:id` | POST | Finalize battle |
| `/api/predictions/:id` | GET | Get prediction summary |
| `/api/predictions/place/:id` | POST | Place prediction |
| `/api/wallet/create` | POST | Create wallet |
| `/api/wallet/me` | GET | Get wallet info |
| `/api/wallet/export-secret` | POST | Export secret key |
| `/api/clerk/webhook` | POST | Clerk webhook |

### Socket.IO Events

**Client → Server:**
- `join_lobby` - Join lobby room
- `join_battle` - Join battle room
- `leave_battle` - Leave battle room
- `start_match` - Start battle after P2 joins
- `submit_roast` - Submit roast text
- `cast_vote` - Cast spectator vote
- `place_prediction` - Place prediction bet

**Server → Client:**
- `open_battles_updated` - Lobby battles changed
- `player_joined` - Player joined battle
- `countdown_tick` - Pre-battle countdown
- `battle_started` - Battle正式��始
- `roast_submitted` - Roast submitted
- `voting_started` - Voting phase started
- `vote_update` - Vote count updated
- `spectator_count` - Spectator count updated
- `battle_result` - Battle concluded
- `leaderboard_updated` - Rankings changed
- `error_message` - Error notification

### Database Models

**User Schema:**
- clerkId, username, email
- xp, wins, losses, rankPoints
- walletAddress, walletBalance
- badges[], onboardingCompleted

**Battle Schema:**
- matchId, topic, entryFee
- player1, player2 (ref: User)
- player1Wallet, player2Wallet
- roast1, roast2 + CIDs
- votesPlayer1, votesPlayer2
- status (open/active/voting/ended/draw/cancelled)
- winner, txHash, pot
- chain (Soroban metadata), finance (escrow metadata)

**Prediction Schema:**
- battleId (ref: Battle)
- predictor (ref: User)
- selectedPlayer (ref: User)
- amount, settled, won

## Smart Contract Architecture

### Contract: Roastellar

**Contract ID (Testnet):** `CB2NV4N7NEZFTNRBVZSZDH64RROI7FDPTIFWNXMQOAA5JB2BUWXG4BHW`

**Network:** Stellar Testnet

**Explorer:** https://stellar.expert/explorer/testnet/contract/CB2NV4N7NEZFTNRBVZSZDH64RROI7FDPTIFWNXMQOAA5JB2BUWXG4BHW

### Contract Functions

| Function | Description |
|----------|-------------|
| `register_user` | Register new user with username |
| `get_user` | Get user data |
| `update_profile` | Update profile CID |
| `create_match` | Create new battle (entry fee required) |
| `get_match` | Get match data |
| `join_match` | Join an open match |
| `submit_roast` | Submit roast content CID |
| `vote` | Cast vote for player |
| `predict` | Place prediction bet |
| `finalize_match` | End match, award winner |
| `has_badge` | Check badge ownership |

### Contract Storage

```rust
// DataKey enum for storage
enum DataKey {
    User(Address),
    Match(u32),
    UserBadge(Address, Badge),
    Prediction(u32, Address),
    MatchCount,
    HasJoined(Address, u32),
    HasVoted(Address, u32)
}

// MatchStatus enum
enum MatchStatus {
    Open,
    Active,
    Ended,
    Draw
}

// Badge enum
enum Badge {
    FirstWin,   // Awarded on first win
    FiveWins,   // Awarded on 5th win
    TenMatches  // Awarded after 10 matches
}
```

### Contract Lifecycle

```
create_match() → join_match() → submit_roast() (both) → vote() (spectators) → finalize_match()
    │               │                │                      │
    │               │                │                      ▼
    │               │                └────────────────── Draw (if tied)
    │               │
    │               ▼
    └──────────────► Active status
                         │
                         ▼
                      Ended status
                      (winner awarded)
```

## Battle Lifecycle

```
┌──────────────────────────────────────────────────────────────────┐
│                        BATTLE LIFECYCLE                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. CREATE         User creates battle with topic & entry fee    │
│                    └─► Battle status: OPEN                       │
│                    └─► Smart contract: create_match()           │
│                                                                  │
│  2. JOIN            Second player joins                         │
│                    └─► Battle status: ACTIVE                     │
│                    └─► Smart contract: join_match()            │
│                    └─► 3-second countdown begins                 │
│                                                                  │
│  3. ROAST           Both players submit roasts                   │
│                    └─► Roast text → IPFS → CID                  │
│                    └─► Smart contract: submit_roast()           │
│                    └─► Status → VOTING when both submitted      │
│                                                                  │
│  4. VOTE            Spectators cast votes                        │
│                    └─► One vote per spectator                    │
│                    └─��� Smart contract: vote()                  │
│                    └─► Real-time vote count via Socket.IO       │
│                                                                  │
│  5. FINALIZE        Timer ends or manual finalize               │
│                    └─► Compare votes                             │
│                    └─► Ended (winner) or Draw (tied)           │
│                    └─► Smart contract: finalize_match()        │
│                    └─► Winner: +XP, +Rank, Badge check         │
│                    └─► XLM payout from escrow                    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

## Escrow & Financial Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                      ESCROW FLOW                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Player 1 creates battle (entry fee: 10 XLM)                     │
│  └─► 10 XLM debited from P1 wallet → Escrow wallet              │
│                                                                  │
│  Player 2 joins                                                  │
│  └─► 10 XLM debited from P2 wallet → Escrow wallet              │
│                                                                  │
│  ┌─────────────┐      ┌─────────────┐      ┌─────────────┐     │
│  │  P1 Wallet   │      │   Escrow    │      │  P2 Wallet  │     │
│  │  -10 XLM    │      │  +20 XLM    │      │  -10 XLM   │     │
│  └─────────────┘      └─────────────┘      └─────────────┘     │
│                                                                  │
│  Winner determined (P1 wins with 5 votes vs 3)                  │
│  └─► 19.8 XLM sent to P1 (98%, 1% platform fee)                │
│  └─► 0.2 XLM retained as platform fee                           │
│                                                                  │
│  ┌─────────────┐      ┌─────────────┐                          │
│  │  P1 Wallet   │      │   Escrow    │                          │
│  │  +19.8 XLM  │      │  -20 XLM    │                          │
│  └─────────────┘      └─────────────┘                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Environment Variables

### Backend (.env)

```bash
# Stellar
STELLAR_NETWORK=testnet
STELLAR_CONTRACT_ID=CB2NV4N7NEZFTNRBVZSZDH64RROI7FDPTIFWNXMQOAA5JB2BUWXG4BHW
STELLAR_BATTLE_SECRET=<deployer_secret>
STELLAR_BATTLE_PUBLIC=GAYWZSX43WUBRHM3F2QCWBL6ZOYSH7V5EOQOYMG6SMTGMM24RFEFCMHC
STELLAR_ESCROW_SECRET=<escrow_secret>
STELLAR_ESCROW_PUBLIC=<escrow_public>
STELLAR_RPC_URL=https://soroban-testnet.stellar.org

# Battle Config
BATTLE_ROAST_TIME_SECONDS=60
BATTLE_VOTING_TIME_SECONDS=30
BATTLE_VOTE_STAKE_XLM=0
BATTLE_VOTING_FINALIZE_GRACE_SECONDS=8
BATTLE_START_COUNTDOWN_SECONDS=3

# App
PORT=3001
MONGODB_URI=mongodb+srv://...
CLERK_PUBLISHABLE_KEY=<clerk_key>
CLERK_SECRET_KEY=<clerk_secret>
CLERK_WEBHOOK_SECRET=<webhook_secret>

# Security
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100
CLIENT_ORIGINS=https://roastellar.vercel.app
```

### Frontend (.env.local)

```bash
NEXT_PUBLIC_API_URL=https://roastellar-9khj.onrender.com
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=<clerk_key>
```

## Data Flow Diagrams

### User Onboarding Flow

```
┌──────────┐     ┌──────────────┐     ┌─────────────┐     ┌─��────────┐
│  Clerk   │────▶│  Mini Game   │────▶│  Freighter  │────▶│ MongoDB  │
│  Sign-in │     │  (Score > 5) │     │   Wallet     │     │  +User   │
└──────────┘     └──────────────┘     └─────────────┘     └──────────┘
                                              │
                                              ▼
                                    ┌─────────────────┐
                                    │ Stellar Testnet │
                                    │  Fund Account   │
                                    └─────────────────┘
```

### Battle Creation Flow

```
Client                    Backend                     Blockchain
  │                           │                            │
  ├── create_battle(topic)───▶│                            │
  │                           ├──Validate & Save──▶MongoDB  │
  │                           │                            │
  │                           ├──create_match()────────────▶│
  │                           │◀───matchId──────────────────│
  │◀──Response with matchId──┤                            │
  │                           │                            │
```

### Real-time Voting Flow

```
┌─────────┐        ┌─────────┐        ┌─────────┐        ┌─────────┐
│Player 1 │        │ Server  │        │MongoDB  │        │Soroban  │
└────┬────┘        └────┬────┘        └────┬────┘        └────┬────┘
     │                 │                 │                 │
     │cast_vote(P1)────▶│                 │                 │
     │                 │vote()────────────────────────────────▶│
     │                 │◀───────────────vote recorded──────────│
     │                 │                 │                 │
     │                 │Update votes──▶MongoDB               │
     │                 │                 │                 │
     │◀─vote_update───│                 │                 │
     │◀─vote_update───│                 │                 │
┌─────────┐        ┌─────────┐        ┌─────────┐        ┌─────────┐
│Player 2 │        │ Spect.1 │        │ Spect.2 │        │ Spect.N │
└─────────┘        └─────────┘        └─────────┘        └─────────┘
```

## Security Considerations

1. **Authentication:** Clerk JWT verification on all protected routes
2. **CORS:** Allowlist-based origin checking
3. **Rate Limiting:** 100 requests per 15 minutes per IP
4. **Input Validation:** Request validation middleware
5. **Wallet Security:** Secret keys only accessible via dedicated export endpoint with auth
6. **Contract Auth:** Soroban `require_auth()` on all user actions
7. **Vote Prevention:** One vote per address per match (on-chain + off-chain)

## Deployment

### Frontend (Vercel)
- **URL:** https://roastellar.vercel.app
- **Framework:** Next.js with Edge Runtime
- **Build:** `npm run build`

### Backend (Render)
- **URL:** https://roastellar-9khj.onrender.com
- **Runtime:** Node.js 20
- **Database:** MongoDB Atlas
- **Features:** Auto-deploy from GitHub

### Smart Contract
- **Network:** Stellar Testnet
- **Deployed:** Via `stellar contract deploy`
- **Contract ID:** `CB2NV4N7NEZFTNRBVZSZDH64RROI7FDPTIFWNXMQOAA5JB2BUWXG4BHW`

## Future Architecture Considerations

1. **Mainnet Migration:** Move from testnet to Stellar Public Network
2. **Payment Token:** Introduce custom token (RST) for platform economy
3. **NFT Badges:** Mint badges as Soroban tokens
4. **Tournament Mode:** Multi-round bracket system
5. **Spectator Betting:** Allow spectators to bet XLM on outcomes
6. **AI Roast Generator:** AI-assisted roast suggestions
7. **Leaderboard Persistence:** Cached leaderboard with periodic sync