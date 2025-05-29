# Angry Pongers

## Duration
**Estimated Time: 3 hours**

## Objective
Build a complete Android game application that:
1. Implements a dual-ball collision system with territory conquest mechanics
2. Demonstrates advanced Jetpack Compose Canvas usage
3. Shows proficiency in real-time state management and animations
4. Implements proper game architecture with separation of concerns
5. Handles game lifecycle, pause/resume, and performance optimization

---

## Core mechanics

- **Dual Ball System**: Two balls move simultaneously with different trajectories
- **Territory Conquest**: Balls convert grid squares to their team color on collision
- **Physics Simulation**: Realistic bouncing with randomness and speed limits
- **Real-time Scoring**: Dynamic score calculation based on territory control
- **Endless Battle**: Game continues until one side achieves total domination

---

## Technical Requirements

### Core Technologies
- **Language**: Kotlin
- **UI Framework**: Jetpack Compose with Canvas API
- **Architecture**: MVVM with Clean Architecture principles
- **Design System**: Material Design 
- **Minimum SDK**: API 24 (Android 7.0)
- **Target SDK**: API 34 (Android 14)

---

## Game Physics & Formulas

### Ball Movement Physics
**How the balls move around the screen**

Think of each ball as having a position (where it is) and velocity (how fast and in what direction it's moving). Every frame, we update where the ball is based on how fast it's going.

```
Position Update:
newX = currentX + velocityX * deltaTime
newY = currentY + velocityY * deltaTime

Boundary Collision:
if (ballX <= ballRadius || ballX >= canvasWidth - ballRadius):
    velocityX = -velocityX
if (ballY <= ballRadius || ballY >= canvasHeight - ballRadius):
    velocityY = -velocityY

Speed Limitation:
velocityX = clamp(velocityX, -MAX_SPEED, MAX_SPEED)
velocityY = clamp(velocityY, -MAX_SPEED, MAX_SPEED)

Minimum Speed Enforcement:
if (abs(velocityX) < MIN_SPEED):
    velocityX = sign(velocityX) * MIN_SPEED
if (abs(velocityY) < MIN_SPEED):
    velocityY = sign(velocityY) * MIN_SPEED
```

**What this does:**
- **Position Update**: Move the ball based on its speed and direction
- **Boundary Collision**: When a ball hits a wall, it bounces back (like a real ball hitting a wall)
- **Speed Limits**: Keep balls moving at reasonable speeds - not too fast, not too slow
- **Minimum Speed**: Prevent balls from getting stuck by ensuring they always move at least a little bit

### Collision Detection
**How we figure out when balls touch grid squares**

The game board is divided into small squares, like a checkerboard. When a ball moves around, we need to check which squares it's touching so we can change their colors to match the ball's team.

```
Grid Position Calculation:
gridX = floor(ballCenterX / SQUARE_SIZE)
gridY = floor(ballCenterY / SQUARE_SIZE)

Multi-Point Collision Check:
for angle in [0, π/4, π/2, 3π/4, π, 5π/4, 3π/2, 7π/4]:
    checkX = ballX + cos(angle) * ballRadius
    checkY = ballY + sin(angle) * ballRadius
    gridX = floor(checkX / SQUARE_SIZE)
    gridY = floor(checkY / SQUARE_SIZE)
    // Check collision at this point

Bounce Direction Calculation:
if (abs(cos(collisionAngle)) > abs(sin(collisionAngle))):
    velocityX = -velocityX  // Horizontal bounce
else:
    velocityY = -velocityY  // Vertical bounce
```

**What this does:**
- **Grid Position**: Convert the ball's pixel position to which square it's in (like saying "the ball is in row 5, column 3")
- **Multi-Point Check**: Since balls are round, we check 8 points around the edge to see what squares they're touching
- **Bounce Direction**: When a ball hits something, figure out if it should bounce left/right or up/down

### Score Calculation
**How to determine who's winning the territory battle**

The game is all about controlling territory. Each team (Day and Night) tries to convert as many squares as possible to their color. The score shows what percentage of the battlefield each team controls.

```
Territory Score:
dayScore = count(squares where color == DAY_COLOR)
nightScore = count(squares where color == NIGHT_COLOR)
totalSquares = gridWidth * gridHeight

Day Percentage = (dayScore / totalSquares) * 100
Night Percentage = (nightScore / totalSquares) * 100
```

**What this does:**
- **Count Squares**: Go through the entire grid and count how many squares belong to each team
- **Calculate Percentage**: Convert the raw counts into percentages so players can easily see who's winning
- **Real-time Updates**: This calculation happens every frame so the score updates instantly as squares change color

### Randomness & Chaos
**Adding unpredictability to keep the game interesting**

Without randomness, the balls would follow the exact same path every time, making the game predictable and boring. We add small random elements to create unique, engaging gameplay where no two games are the same.

```
Random Velocity Adjustment (per frame):
velocityX += random(-0.01, 0.01)
velocityY += random(-0.01, 0.01)

Ball Spawn Randomization:
spawnX = random(ballRadius, canvasWidth - ballRadius)
spawnY = random(ballRadius, canvasHeight - ballRadius)
initialVelocityX = random(MIN_SPEED, MAX_SPEED) * randomSign()
initialVelocityY = random(MIN_SPEED, MAX_SPEED) * randomSign()
```

**What this does:**
- **Tiny Random Nudges**: Each frame, give balls a tiny random push to create natural, unpredictable movement
- **Random Starting Positions**: When balls are created, place them at random spots (but not too close to walls)
- **Random Starting Speed**: Give new balls random speed and direction within safe limits

---

## Game Configuration

### Constants & Settings
**All the important numbers that control how the game behaves**

These constants define the game's dimensions, physics, timing, and visual appearance. Think of them as the "settings file" for your game - change these values to adjust how fast balls move, how big the playing field is, or what colors to use.

```kotlin
// Game Board
const val CANVAS_WIDTH = 600
const val CANVAS_HEIGHT = 600
const val SQUARE_SIZE = 25
const val GRID_WIDTH = CANVAS_WIDTH / SQUARE_SIZE  // 24 squares
const val GRID_HEIGHT = CANVAS_HEIGHT / SQUARE_SIZE // 24 squares

// Ball Physics
const val BALL_RADIUS = SQUARE_SIZE / 2  // 12.5
const val MIN_SPEED = 5.0f
const val MAX_SPEED = 10.0f
const val INITIAL_VELOCITY = 8.0f
const val RANDOMNESS_FACTOR = 0.01f

// Game Timing
const val TARGET_FPS = 100
const val FRAME_DURATION_MS = 1000 / TARGET_FPS  // 10ms per frame

// Colors (Material Design inspired)
val DAY_COLOR = Color(0xFFD9E8E3)      // MysticMint
val DAY_BALL_COLOR = Color(0xFF114C5A)  // NocturnalExpedition
val NIGHT_COLOR = Color(0xFF114C5A)     // NocturnalExpedition  
val NIGHT_BALL_COLOR = Color(0xFFD9E8E3) // MysticMint
```

**What these settings control:**
- **Game Board**: Define game board size 600x600 pixel that are divided into 24x24 squares (each square is 25 pixels)
- **Ball Physics**: Balls are half the size of squares, with speed limits to keep gameplay balanced
- **Performance**: Targets 100 FPS for ultra-smooth gameplay (10ms per frame)
- **Visual Theme**: Uses contrasting colors so Day and Night teams are clearly distinguishable

### Start State
**How the game begins - the initial battlefield setup**

Every game starts the same way: a divided battlefield with each team controlling half the territory, and two balls positioned to immediately start conquering enemy territory.

```kotlin
// Initial Grid Setup
for (x in 0 until GRID_WIDTH) {
    for (y in 0 until GRID_HEIGHT) {
        grid[x][y] = if (x < GRID_WIDTH / 2) DAY_COLOR else NIGHT_COLOR
    }
}

// Initial Ball Positions
val dayBall = Ball(
    x = CANVAS_WIDTH / 4,
    y = CANVAS_HEIGHT / 2,
    velocityX = INITIAL_VELOCITY,
    velocityY = -INITIAL_VELOCITY,
    teamColor = DAY_COLOR,
    ballColor = DAY_BALL_COLOR
)

val nightBall = Ball(
    x = (CANVAS_WIDTH / 4) * 3,
    y = CANVAS_HEIGHT / 2,
    velocityX = -INITIAL_VELOCITY,
    velocityY = INITIAL_VELOCITY,
    teamColor = NIGHT_COLOR,    ballColor = NIGHT_BALL_COLOR
)
```

**What this setup creates:**
- **Split Territory**: Left half belongs to Day team, right half to Night team (like dividing a checkerboard)
- **Strategic Ball Placement**: Day ball starts in Day territory but moves toward Night territory, and vice versa
- **Opposing Trajectories**: Balls move in different directions so they'll cover different areas and create dynamic gameplay
- **Fair Start**: Both teams begin with exactly equal territory (50% each) and identical ball capabilities

---

## Implementation Requirements

### 1. Game Architecture
**How to structure your code for maintainability and performance**

Good architecture separates concerns - game logic, UI, and data management should be independent modules that work together. This makes testing easier, bugs easier to fix, and features easier to add.

#### Data Models
**The core data structures that represent your game state**

These classes define what information your game needs to track and how it's organized. Think of them as the "vocabulary" your game uses to describe what's happening.

```kotlin
data class Ball(
    val x: Float,
    val y: Float,
    val velocityX: Float,
    val velocityY: Float,
    val teamColor: Color,
    val ballColor: Color,
    val radius: Float = BALL_RADIUS
)

data class GameState(
    val grid: Array<Array<Color>>,
    val balls: List<Ball>,
    val dayScore: Int,
    val nightScore: Int,
    val isPlaying: Boolean,
    val frameCount: Long,
    val gameTime: Long
)

sealed class GameEvent {
    object Play : GameEvent()
    object Pause : GameEvent()
    object Reset : GameEvent()    object Step : GameEvent()
}
```

**What these models represent:**
- **Ball**: Everything needed to draw and move a ball (position, speed, team affiliation, appearance)
- **GameState**: The complete snapshot of the game at any moment (grid colors, ball positions, scores, timing)
- **GameEvent**: User actions that can change the game state (play/pause/reset controls)

#### Game Engine (Domain Layer)
**The brain of your game - pure logic with no UI dependencies**

The game engine contains all the rules and calculations. It doesn't know about Android, UI, or graphics - just pure game logic that could work anywhere.

```kotlin
class GameEngine {
    fun updateGameState(currentState: GameState, deltaTime: Long): GameState
    fun checkCollisions(ball: Ball, grid: Array<Array<Color>>): CollisionResult
    fun updateBallPosition(ball: Ball, deltaTime: Long): Ball
    fun calculateScore(grid: Array<Array<Color>>): Pair<Int, Int>    fun isGameOver(gameState: GameState): Boolean
}
```

**What the game engine does:**
- **Pure Logic**: No UI code, just mathematical calculations and game rules
- **Predictable**: Same input always produces same output (makes testing easy)
- **Frame Updates**: Takes current state + time elapsed = new state
- **Collision Detection**: Figures out when balls hit walls or squares
- **Win Conditions**: Determines when someone has won the game

#### Repository Pattern
**Handles saving/loading game data - your game's memory system**

The repository manages all data persistence, whether that's saving high scores, game state, or user preferences. It hides the complexity of data storage from the rest of your app. For this take home exercise, you don't need to persist the scores on a database, in-memory state is fine.

```kotlin
interface GameRepository {
    suspend fun saveGameState(gameState: GameState)
    suspend fun loadGameState(): GameState?
    suspend fun getHighScores(): List<GameResult>    suspend fun saveGameResult(result: GameResult)
}
```

**What the repository handles:**
- **Game Persistence**: Save current game so players can resume later
- **High Scores**: Track best games and longest battles
---

## End State Conditions

### Victory Conditions
1. **Total Domination**: One team controls 95%+ of the grid
2. **Time Limit**: Highest score after time expires (optional mode)
3. **Ball Elimination**: If a ball gets stuck or stops moving

### Game Over States
**How the game decides when it's finished and who wins**

Since Angry Pongers is a territorial conquest game, we need clear rules for when the battle ends. The game can end in several ways, and we check these conditions every frame to see if someone has won.

```kotlin
sealed class GameResult {
    object DayVictory : GameResult()
    object NightVictory : GameResult()
    object Draw : GameResult()
    data class TimeExpired(val winner: Team) : GameResult()
}

fun checkGameOver(gameState: GameState): GameResult? {
    val totalSquares = GRID_WIDTH * GRID_HEIGHT
    val dayPercentage = (gameState.dayScore.toFloat() / totalSquares) * 100
    val nightPercentage = (gameState.nightScore.toFloat() / totalSquares) * 100
    
    return when {
        dayPercentage >= 95f -> GameResult.DayVictory
        nightPercentage >= 95f -> GameResult.NightVictory
        gameState.gameTime >= MAX_GAME_TIME -> {
            when {
                dayPercentage > nightPercentage -> GameResult.TimeExpired(Team.Day)
                nightPercentage > dayPercentage -> GameResult.TimeExpired(Team.Night)
                else -> GameResult.Draw
            }
        }
        else -> null
    }
}
```

**What this does:**
- **Domination Victory**: If one team controls 95% or more of the battlefield, they win immediately
- **Time Limit Victory**: If the game reaches a maximum time limit, whoever has more territory wins
- **Draw Condition**: If time runs out and both teams have exactly the same territory, it's a tie
- **Ongoing Battle**: If none of these conditions are met, the game continues (returns null)
- **Real-time Checking**: This function runs every frame to immediately detect victory conditions

---

## Evaluation Criteria

### Technical Implementation (40%)
- **Game Engine Logic**: Correct physics implementation and collision detection
- **Performance**: Smooth 60+ FPS gameplay with efficient resource usage
- **Code Quality**: Readable, maintainable Kotlin code with proper error handling

### UI/UX Implementation (25%)
- **Material Design**: Proper Material theming and component usage
- **Responsive Design**: Works well on different screen sizes
- **Visual Polish**: Smooth animations and appealing visual design

### Game Design (20%)
- **Gameplay Feel**: Satisfying ball physics and collision feedback
- **Balance**: Fair and engaging gameplay mechanics
- **User Experience**: Intuitive controls and clear visual feedback
- **Feature Completeness**: All core features working as expected

### Modern Android Practices (15%)
- **Jetpack Compose**: Effective use of modern declarative UI
- **State Management**: Proper handling of game state and lifecycle
- **Architecture Components**: Correct ViewModel and Repository usage
- **Testing**: Unit tests for game logic (bonus points)

---

## Deliverables

### 1. Complete Android Project
- Fully functional game application
- All source code with proper git history
- Working APK file
- Performance meets 60+ FPS requirement

### 2. Documentation
- **README.md** with:
  - Setup and run instructions
  - Game rules and controls explanation
  - Performance considerations and optimizations
  - Known issues or limitations

### 3. Demo Video
  - 30 seconds screen recording showing:
  - Complete gameplay session
  - Victory condition demonstration
  - UI interaction and controls
  - Performance and smoothness

---
