# Impostiq • Find the Spy - Full Documentation

**AUTHOR/DEVELOPER/CREATOR: [Jandré du Preez/Fillabrona](https://fillabrona.github.io)**

> **Source Code Reference:**
> For exact implementation details, CSS animations, or specific React state logic, please refer to the raw source code file here:
> [📂 View Source Code](https://raw.githubusercontent.com/Impostiq/Impostiq.github.io/refs/heads/main/index.html)

## System Overview
**Impostiq** is a React-based Single Page Application (SPA) utilizing Firebase (Firestore/Auth) for real-time state management. It is designed as a social deduction game.

**Tech Stack:**
- **Frontend:** React 18 (via CDN/Babel), Tailwind CSS.
- **Backend:** Firebase Firestore (NoSQL), Firebase Auth (Anonymous).
- **Audio:** Custom `audioManager` handling SFX and loops based on game state.

---

## Game Logic & Mechanics

### 1. Game Modes
The application runs in two distinct modes controlled by the `gameMode` state:

#### A. Local Mode (Device Pass-and-Play)
*   **State Management:** Local React `useState`.
*   **Logic:**
    1.  **Setup:** Users input player names and record optional voice notes ("It's your turn").
    2.  **Role Assignment:** The app uses a Fisher-Yates shuffle on the player list.
    3.  **Distribution:** The device is passed physically. Players tap a "Verify Identity" screen, then tap a 3D card (CSS `rotate-y-180`) to reveal their role.
    4.  **Timer:** A local timer runs (visualized by an SVG circle stroke-dashoffset).

#### B. Online Mode (Real-Time Multiplayer)
*   **State Management:** Firebase Firestore (`onSnapshot` listeners).
*   **Lobby Logic:**
    *   **ID/Password:** 6-char ID / 4-char Password generated via `generateRandomString`.
    *   **QR Code:** Generated via `api.qrserver.com` pointing to `window.location.origin + ?lobbyId=...`.
    *   **Sync:** `lobbyData` tracks `gameState` ('waiting', 'reveal', 'turnOrder', 'active', 'voting', 'result').
*   **Connection Handling:**
    *   `navigator.onLine` checks network status.
    *   `OfflineScreen` component mounts if connection drops.

### 2. Core Gameplay Loop
1.  **Category Selection:** Host selects a category (e.g., "Food") or "Random Mode" (system picks).
2.  **Word Selection:**
    *   System accesses `WORD_BANK[Category]`.
    *   A `secretWord` is chosen randomly.
    *   In **Fake Word Mode**, a `fakeWord` is chosen (must not equal `secretWord`).
3.  **Role Distribution:**
    *   **Agents:** See the `secretWord`.
    *   **Imposter (Classic):** See "IMPOSTER".
    *   **Fake Word Player:** See the `fakeWord` (they believe they are an Agent).
    *   **Second Imposter:** Activated if `enableTwoImposters` is true (requires 10+ players).
4.  **Turn Order:** A randomized list is generated.
5.  **The Game:** Players discuss/ask questions.
6.  **Voting:** Players select who they suspect.
7.  **Results:**
    *   **Agents Win:** If the Imposter/Fake Word Player receives the majority votes.
    *   **Imposter Wins:** If an innocent Agent is voted out, or if there is a tie.

### 3. Special Mechanics
*   **Confusion Poll (Word Check):**
    *   If an Agent does not recognize a word, they click "I don't recognize this word".
    *   A secret poll triggers in Firestore (`status: 'confused'`).
    *   If >50% of Agents vote "Confused", the round automatically restarts with a new word.
    *   *Strategic Note:* In "Fake Word Mode", the Fake Word Player is included in the poll to prevent identifying them via exclusion.
*   **Seasonal Logic:**
    *   System checks `Date()`.
    *   **Halloween:** Oct 24 - Nov 7. Changes UI theme, adds "Halloween" Category, plays spooky music.
    *   **Christmas:** Dec 18 - Jan 1. Changes UI theme, adds "Christmas" Category, plays festive music.

---

## Configuration & Settings

| Setting | Variable | Description |
| :--- | :--- | :--- |
| **Timer** | `duration` | Sets countdown (1-15 mins). If disabled, game is untimed. |
| **Fake Word Mode** | `isFakeWordMode` | Replaces the Imposter with a player who has a slightly different word from the same category. |
| **Two Imposters** | `enableTwoImposters` | Adds a second spy. Only available with 10+ players. |
| **Speak Category** | `speakCategory` | Uses ElevenLabs generated audio to announce the category name at start. |
| **Notify Imposter** | `notifyImposterOnVote` | If True, the Imposter is alerted if Agents trigger a "Confusion Poll". |
| **Random Mode** | `isRandomMode` | System picks a random valid category from `WORD_BANK`. |

---

## The Complete Database (Word Bank)

The AI logic selects words from the following hardcoded JSON structure (`WORD_BANK`).

### Category: Supercell
*Words:* Barbarian, Archer, Giant, Goblin, Wall Breaker, Balloon, Wizard, Healer, Dragon, P.E.K.K.A, Minion, Hog Rider, Valkyrie, Golem, etc.

### Category: Celebs
*Words:* Leonardo DiCaprio, Brad Pitt, Angelina Jolie, Tom Cruise, Johnny Depp, Will Smith, Dwayne Johnson, Scarlett Johansson, Robert Downey Jr, Jennifer Lawrence, Chris Hemsworth, Emma Watson, Ryan Reynolds, Keanu Reeves, etc.

### Category: Movies
*Words:* The Godfather, Titanic, Avatar, Star Wars, Jurassic Park, The Matrix, Inception, Forrest Gump, The Lion King, Avengers, Pulp Fiction, Frozen, The Dark Knight, Harry Potter, Back to the Future, etc.

### Category: Food
*Words:* Pizza, Sushi, Burger, Tacos, Pasta, Curry, Steak, Salad, Sandwich, Soup, Fried Chicken, Ramen, Dumplings, Burrito, Pancakes, etc.

### Category: Brands
*Words:* Apple, Nike, Coca-Cola, Samsung, McDonalds, Toyota, Disney, Amazon, Google, Tesla, Adidas, Pepsi, Starbucks, Netflix, Microsoft, etc.

### Category: Technology
*Words:* iPhone, Laptop, Drone, Robot, Bluetooth, Wifi, Keyboard, Mouse, Monitor, Camera, Headphones, Smartwatch, Tablet, Printer, Router, etc.

### Category: TV
*Words:* Stranger Things, Game of Thrones, Breaking Bad, The Office, Friends, The Simpsons, South Park, SpongeBob SquarePants, The Mandalorian, The Crown, Squid Game, The Witcher, Black Mirror, Rick and Morty, Avatar: The Last of Airbender, etc.

### Category: Jobs
*Words:* Doctor, Teacher, Engineer, Lawyer, Chef, Pilot, Firefighter, Police, Artist, Musician, Actor, Writer, Scientist, Programmer, Designer, etc.

### Category: Mythology
*Words:* Zeus, Thor, Hercules, Medusa, Dragon, Unicorn, Phoenix, Mermaid, Centaur, Griffin, Poseidon, Hades, Athena, Apollo, Odin, etc.

### Category: Sport
*Words:* Soccer, Basketball, Football, Tennis, Golf, Cricket, Rugby, Baseball, Volleyball, Hockey, Boxing, MMA, Wrestling, Formula 1, Swimming, etc.

### Category: Places
*Words:* School, Hospital, Library, Airport, Station, Park, Beach, Forest, Mountain, Desert, City, Village, Farm, Zoo, Museum, etc.

### Category: Instruments
*Words:* Guitar, Piano, Drums, Violin, Flute, Saxophone, Trumpet, Cello, Harp, Clarinet, Trombone, Tuba, Banjo, Ukulele, Accordion, etc.

### Category: Superheroes
*Words:* Superman, Batman, Spider-Man, Iron Man, Wonder Woman, Thor, Hulk, Captain America, Black Panther, Flash, Aquaman, Green Lantern, Wolverine, Deadpool, Doctor Strange, etc.

### Category: Animals
*Words:* Lion, Tiger, Elephant, Giraffe, Penguin, Dolphin, Whale, Shark, Eagle, Parrot, Dog, Cat, Hamster, Snake, Lizard, etc.

### Category: Countries
*Words:* USA, Canada, Mexico, Brazil, Argentina, UK, France, Germany, Italy, Spain, Russia, China, Japan, India, Australia, etc.

### Category: Video Games
*Words:* Mario, Zelda, Pokemon, Minecraft, Fortnite, Roblox, Call of Duty, GTA, Halo, God of War, The Witcher, Skyrim, Fallout, Overwatch, League of Legends, etc.

### Category: Science
*Words:* Atom, Molecule, Cell, DNA, Gravity, Magnetism, Electricity, Light, Sound, Heat, Energy, Force, Motion, Planet, Star, etc.

### Category: Clothing
*Words:* Shirt, T-Shirt, Pants, Jeans, Shorts, Skirt, Dress, Suit, Coat, Jacket, Sweater, Hoodie, Vest, Scarf, Gloves, etc.

### Category: Home
*Words:* Table, Chair, Sofa, Bed, Desk, Shelf, Cabinet, Cupboard, Wardrobe, Lamp, Light, Fan, Heater, AC, Fridge, etc.

### Category: People
*Words:* President, King, Queen, Athlete, Celebrity, Artist, Musician, Scientist, Teacher, Student, Child, Elderly, Baby, Tourist, Police Officer, etc.

### Category: Historical Figures
*Words:* Cleopatra, Julius Caesar, Alexander the Great, Joan of Arc, Leonardo da Vinci, William Shakespeare, Isaac Newton, George Washington, Napoleon Bonaparte, Abraham Lincoln, Queen Victoria, Albert Einstein, Marie Curie, Mahatma Gandhi, Winston Churchill, etc.

### Category: Board Games
*Words:* Chess, Checkers, Monopoly, Scrabble, Clue, Risk, Battleship, Catan, Ticket to Ride, Pandemic, Codenames, Jenga, Connect Four, Sorry!, etc.

### Category: Famous Landmarks
*Words:* Eiffel Tower, Statue of Liberty, Great Wall of China, Pyramids of Giza, Colosseum, Taj Mahal, Machu Picchu, Christ the Redeemer, Big Ben, Leaning Tower of Pisa, Sydney Opera House, Acropolis, Stonehenge, Mount Rushmore, Burj Khalifa, The Louvre, etc.

### Category: Music
*Words:* Bohemian Rhapsody, Like a Rolling Stone, Smells Like Teen Spirit, Billie Jean, Stairway to Heaven, Hey Jude, Hotel California, I Will Always Love You, Shape of You, Uptown Funk, Rolling in the Deep, Blinding Lights, Despacito, Hallelujah, Wonderwall, etc.

### Category: School
*Words:* Blackboard, Chalk, Desk, Chair, Backpack, Pencil, Pen, Eraser, Ruler, Notebook, Textbook, Homework, Test, Exam, Quiz, etc.

### Category: Halloween (Seasonal)
*Words:* Witch, Ghost, Vampire, Werewolf, Zombie, Mummy, Skeleton, Spider, Bat, Black Cat, Pumpkin, Jack-o'-lantern, Haunted House, Cemetery, Gravestone, etc.

### Category: Christmas (Seasonal)
*Words:* A Christmas Carol, Advent, All I Want for Christmas is You, Angel, Anticipation, Artificial Tree, Atmosphere, Baking, Bauble, Believe, Bells, Bethlehem, Blanket, Blessing, Blitzen, etc.

Here is the detailed section to add **at the very end** of your `README.md` file.

This section explicitly breaks down the date logic, asset swaps, music triggers, and social sharing changes so an AI understands exactly how the game transforms based on the date.

Copy and paste the following text to the bottom of your `README.md`:

---

## Seasonal & Time-Based Logic

The application uses a client-side date check (`new Date()`) to automatically toggle "Seasonal Modes." This logic affects the UI theme, available categories, background music, metadata, and social sharing text.

### 1. Trigger Mechanism
The function `getSeasonalInfo()` determines the current season based on the user's local system time.

*   **Default Mode:** Active most of the year.
*   **Halloween Mode:** Active from **October 24th** to **November 7th**.
*   **Christmas Mode:** Active from **December 18th** to **January 1st**.

### 2. Halloween Mode Details
*   **Date Logic:** `(month === 10 && day >= 24) || (month === 11 && day <= 7)` (Note: Months are 0-indexed in JS).
*   **Visual Changes:**
    *   **Title:** "Impostiq" text is styled with a Jack-O-Lantern emoji replacing the 'O'.
    *   **Category Card:** The "Halloween" category card gets an orange border, orange glow/ring, and a spooky background image.
    *   **Favicon/Metadata:** Changes to a 3D Jack-O-Lantern icon.
*   **Audio:**
    *   Background Music: Plays `halloweenMainMenuMusic` (Spooky/Eerie theme).
    *   Voiceover: Custom ElevenLabs voiceover for the "Halloween" category.
*   **Gameplay:**
    *   **Category:** The "Halloween" category becomes selectable.
    *   **Word Bank:** Accesses the specific `Halloween` array in `WORD_BANK` (e.g., Witch, Ghost, Vampire).
*   **Social Sharing:**
    *   Invite Text: "🎃 Spooky Impostiq Invite... Something is lurking in the shadows... 👻"

### 3. Christmas Mode Details
*   **Date Logic:** `(month === 12 && day >= 18) || (month === 1 && day <= 1)`.
*   **Visual Changes:**
    *   **Title:** "Impostiq" text is styled with a Christmas Tree emoji replacing the 'T'.
    *   **Category Card:** The "Christmas" category card gets a red border, red glow/ring, and a festive background image (Three Kings/Nativity style).
    *   **Favicon/Metadata:** Changes to a 3D Snowman icon.
*   **Audio:**
    *   Background Music: Plays `christmasMainMenuMusic` (Festive/Jingle Bells theme).
    *   Voiceover: Custom ElevenLabs voiceover for the "Christmas" category.
*   **Gameplay:**
    *   **Category:** The "Christmas" category becomes selectable.
    *   **Word Bank:** Accesses the specific `Christmas` array in `WORD_BANK` (e.g., Santa Claus, Reindeer, Eggnog).
*   **Social Sharing:**
    *   Invite Text: "🎁 Festive Impostiq Invite... The North Pole has a traitor! 🎅🏻"

### 4. Dynamic Metadata & SEO
An Immediately Invoked Function Expression (IIFE) runs before the DOM loads to update `<head>` tags based on the season:
*   **Favicon (`link[rel="icon"]`):** Updates to seasonal emoji.
*   **Open Graph Image (`og:image`):** Updates to seasonal emoji for link previews (Discord, WhatsApp, iMessage).
*   **Twitter Image (`twitter:image`):** Updates to match Open Graph.

### 5. Asset Preloading Strategy
To ensure smooth transitions, the `preloadAssets` function builds a list of URLs to cache:
1.  **Static UI:** Standard game icons (Locked, Shushing Face, etc.).
2.  **Category Icons:** Iterates through `CATEGORY_ICONS` object.
3.  **Audio:** Iterates through `AUDIO_URLS`, distinguishing between SFX (loaded as `Audio` objects) and Music (streamed).
4.  **Seasonal Backgrounds:** Specifically preloads the Halloween pumpkin background and the Christmas Nativity background to prevent pop-in when seasonal modes are active.

## User Interface & Interaction Flow

### 1. Creating a Lobby (Step-by-Step)
1.  **Landing Screen:** User clicks **"PLAY ONLINE"**.
2.  **Lobby Setup:**
    *   User enters a **Username**.
    *   User clicks **"Create a new lobby"** text (toggles mode).
    *   User clicks the **"CREATE LOBBY"** button.
3.  **Waiting Room (Host View):**
    *   Host sees the **Lobby ID** (6 chars) and **Password** (4 chars).
    *   **QR Code:** Host can click "SCAN QR" to display a code for friends to scan.
    *   **Settings:** Host can toggle "Fake Word Mode", "Second Imposter", "Timer", and "Speak Category".
    *   **Category:** Host selects a card (e.g., "Movies") or toggles "Mystery Mode" (Random).
    *   **Start:** Once 3+ players join, Host clicks "START MISSION".

### 2. Joining a Lobby
1.  **Landing Screen:** User clicks **"PLAY ONLINE"**.
2.  **Input:** User enters **Username**, **Lobby ID**, and **Password**.
3.  **Action:** User clicks **"JOIN LOBBY"**.
    *   *Alternative:* User scans the Host's QR code to auto-fill credentials.

### 3. Edge Cases & Error Handling
*   **Host Disconnection:** If the Host leaves, the system automatically promotes the next player in the list (by `createdAt` timestamp) to be the new Host.
*   **Player Removal:** The Host can click the "Trash Can" icon next to a player's name in the Waiting Room to kick them out.
*   **Connection Loss:** An **Offline Screen** triggers immediately if `navigator.onLine` returns false, blocking gameplay until connection is restored.
*   **Minimum Players:** The "Start Mission" button is disabled until at least **3 players** are in the lobby.
