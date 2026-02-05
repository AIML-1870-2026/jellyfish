# Buzzing Bees - Boids Simulation Specification

## Project Overview
An interactive, high-performance web-based boids flocking simulation featuring bee-themed triangular agents with real-time controls, statistics visualization, and predator mechanics.

## Core Features

### 1. Boid Visualization
- **Shape**: Triangular agents (7px size)
- **Appearance**: Yellow (#FFD700) fill with two black vertical stripes
- **Orientation**: Triangles rotate to face their direction of movement
- **Count**: Default 500, adjustable from 10 to 3000
- **Target Performance**: 60 FPS with up to 2000+ boids

### 2. Flocking Algorithm
Implementation of Craig Reynolds' boids algorithm with three fundamental rules:

#### Separation
- **Weight**: 5.0 (range: 0-6)
- **Purpose**: Avoid crowding neighbors
- **Implementation**: 
  - Minimum distance enforcement: 20px hard floor
  - Emergency repulsion force: 20x when under minimum distance
  - Distance-weighted normal separation: 2/(dist+1)
  - Prevents any visual overlap between boids

#### Alignment
- **Weight**: 3.0 (range: 0-5)
- **Purpose**: Match direction with neighbors
- **Implementation**: Average velocity of 5 nearest neighbors
- **Effect**: Creates coordinated, flowing movement patterns

#### Cohesion
- **Weight**: 0.3 (range: 0-3)
- **Purpose**: Stay with the group
- **Implementation**: Move toward average position of up to 10 neighbors
- **Balance**: Intentionally low to prevent tight clustering

#### Additional Parameters
- **Neighbor Radius**: 70px (range: 20-150)
- **Max Speed**: 3.5 (range: 0.5-8)
- **Boundary Mode**: Wrap-around or bounce (toggleable)

### 3. Visual Environment

#### Background
- **Sky gradient**: Linear gradient from #87CEEB to #B0E0E6
- **Clouds**: 5 static white clouds positioned across the canvas
- **Optimization**: Clouds cached in offscreen canvas, drawn once

#### Canvas
- **Dimensions**: Full width × 600px height
- **Border**: 2px solid #4A90E2
- **Border radius**: 8px rounded corners

### 4. User Controls

#### Behavior Presets (3 buttons)
1. **Schooling**
   - High alignment (2.5)
   - Medium cohesion (1.5)
   - Low separation (0.8)
   - Larger neighbor radius (60)

2. **Chaotic Swarm**
   - Low alignment (0.3)
   - Low cohesion (0.3)
   - Normal separation (1.0)
   - Small neighbor radius (30)

3. **Tight Cluster**
   - Normal alignment (1.0)
   - High cohesion (2.5)
   - Moderate separation (1.2)
   - Large neighbor radius (80)

#### Parameter Sliders (6 controls)
- Separation Weight (0-6, step 0.1)
- Alignment Weight (0-5, step 0.1)
- Cohesion Weight (0-3, step 0.1)
- Neighbor Radius (20-150, step 5)
- Max Speed (0.5-8, step 0.1)
- Boid Count (10-3000, step 50)

#### Simulation Controls (3 buttons)
- **Pause/Resume**: Toggle simulation state
- **Reset**: Regenerate all boids with random positions
- **Add/Remove Predator**: Toggle bear predator mode

#### Boundary Controls (2 toggle buttons)
- **Wrap Around**: Boids teleport to opposite edge (default)
- **Bounce**: Boids turn away from edges with soft margins

### 5. Predator Mechanics

#### Predator Entity
- **Visual**: 🐻 bear emoji (80px font size)
- **Control**: Follows mouse cursor with smooth interpolation (0.2 factor)
- **Activation**: Toggle via "Add Predator" button

#### Boid Evasion Behavior
- **Avoidance radius**: 200px
- **Panic force**: 8.0 × (1 - distance/radius)
- **Effect**: Creates dramatic waves and splits in the flock
- **Visual result**: Murmuration-like rippling patterns

### 6. Performance Statistics

#### Real-Time Metrics (4 displays)
1. **FPS**: Frame rate counter
2. **Boid Count**: Current number of active boids
3. **Average Speed**: Mean velocity across all boids
4. **Average Neighbors**: Mean neighbors per boid within radius

#### Statistics Charts (3 live graphs)
Displayed in a single row below controls, showing last 100 data points:

1. **Average Neighbors** (Yellow line, #FFD54F)
   - Y-axis range: 0-20
   - Shows flock connectivity

2. **Speed Variance** (Cyan line, #4FC3F7)
   - Y-axis range: 0-3
   - Measures synchronization (lower = more synchronized)

3. **Flock Compactness** (Green line, #81C784)
   - Y-axis range: 0-200
   - Average distance from center of mass
   - Lower = tighter clustering

### 7. UX/UI Design

#### Color Scheme
- **Primary**: Blues (#4A90E2, #2C5AA0, #87CEEB)
- **Accent**: Yellow/Gold (#FFD54F, #F9A825)
- **Background**: Sky blue gradient
- **Text**: Dark blue (#2C5AA0) for headings, light blue (#B0E0E6) for labels

#### Layout
1. **Title**: "🐝 Buzzing Bees" (centered, 2em)
2. **Main canvas**: Top section, full width
3. **Control panels**: Horizontal row of 5 boxes
   - Performance Stats
   - Controls
   - Behavior Presets
   - Flocking Parameters
   - Boundary Behavior
4. **Statistics charts**: Full-width box below controls

#### Tooltips
- Question mark icons (?) with hover tooltips
- Plain English explanations for each parameter
- Appears on control group headers

#### Visual Feedback
- Real-time value displays next to all sliders
- Button state changes (colors, text)
- Active boundary mode highlighted

### 8. Performance Optimizations

#### Rendering Optimizations
- **Path2D reuse**: Pre-created triangle and stripe paths (~400% faster)
- **Batch rendering**: All boids drawn with shared fill/stroke styles
- **Cloud caching**: Offscreen canvas, drawn once and reused
- **Frame limiting**: Target 60 FPS with frame skipping

#### Physics Optimizations
- **Spatial grid**: Neighbor search using grid partitioning
- **Neighbor limiting**: Maximum 30 neighbors checked per boid
- **Early exit**: Break from loops when limits reached
- **Reduced calculations**: Cohesion checks only 10 nearest neighbors
- **Alignment focus**: Only 5 nearest neighbors for velocity matching

#### Update Frequency Optimizations
- **Text stats**: Update every 3 frames
- **Chart stats**: Update every 10 frames
- **Predator smoothing**: Interpolated position updates

#### Movement Smoothing
- **Acceleration damping**: 0.3 factor for gradual changes
- **Velocity blending**: 0.7 temporal smoothing between frames
- **Previous velocity tracking**: Eliminates jerky movement

### 9. Technical Implementation

#### Core Technologies
- **HTML5 Canvas**: Main rendering surface
- **Vanilla JavaScript**: No framework dependencies
- **ES6 Classes**: Object-oriented boid structure
- **Path2D API**: High-performance shape rendering

#### Data Structures
- **Spatial Grid**: Map<string, Boid[]> for O(1) neighbor lookup
- **Boid Array**: Single array of all active boids
- **Stats History**: Arrays of last 100 data points per metric

#### Animation Loop
- **RequestAnimationFrame**: Browser-optimized timing
- **Delta time tracking**: Frame rate independence
- **Conditional rendering**: Only when not paused

### 10. Key Algorithms

#### Spatial Partitioning
```
cellSize = neighborRadius
cellX = floor(position.x / cellSize)
cellY = floor(position.y / cellSize)
key = "${cellX},${cellY}"
```

#### Emergency Separation
```
if (distance < minDistance) {
  emergencyForce = (minDistance - distance) / minDistance
  force = direction * emergencyForce * 20
}
```

#### Velocity Smoothing
```
velocity += acceleration * 0.3
velocity = prevVelocity * 0.3 + velocity * 0.7
prevVelocity = velocity
```

### 11. Success Metrics

#### Performance Targets
- 60 FPS with 500 boids ✓
- 60 FPS with 2000 boids ✓
- Smooth predator movement ✓
- No frame drops during chart updates ✓

#### Visual Quality Targets
- Always see individual boids ✓
- No visual overlap in clusters ✓
- Smooth, fluid movement ✓
- Murmuration-like behavior ✓

#### User Experience Targets
- Intuitive controls ✓
- Real-time feedback ✓
- Responsive parameter changes ✓
- Clear visual feedback ✓

## File Structure
```
boids-simulation.html
├── HTML Structure
│   ├── Header (title)
│   ├── Canvas (simulation)
│   ├── Controls (5 horizontal panels)
│   └── Charts (3 graphs)
├── CSS Styles
│   ├── Layout (flexbox)
│   ├── Control styling
│   ├── Color scheme
│   └── Responsive design
└── JavaScript
    ├── Boid class
    ├── Physics engine
    ├── Rendering system
    ├── Stats tracking
    ├── Event handlers
    └── Animation loop
```

## Browser Compatibility
- **Chrome/Edge**: Full support (tested)
- **Firefox**: Full support
- **Safari**: Full support
- **Mobile**: Touch support for predator control

## Future Enhancement Ideas
- Obstacle avoidance
- Multiple predator support
- Save/load parameter presets
- Export animation as GIF
- 3D visualization mode
- Sound effects
- Color customization
- Wind/environmental forces

## Credits
Based on Craig Reynolds' Boids algorithm (1986)
Inspired by starling murmurations and bee swarm behavior
