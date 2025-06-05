# Runeforged API - Mana System

A cross-platform API for implementing mana systems in Minecraft mods, compatible with both NeoForge and Fabric for Minecraft 1.21.1.

## Features

- Cross-platform compatibility with NeoForge and Fabric using Architectury
- Default "Mana" type with built-in ManaBar UI representation
- Flexible mana system with support for multiple mana types
- Mana storage, regeneration, and manipulation
- Event system for mana changes
- Data persistence and synchronization between client and server
- Customizable UI for displaying mana

## Adding to Your Project

### Gradle Setup

Add the following to your `build.gradle`:

```groovy
repositories {
    // Add your repository here
    // maven { url "https://example.com/maven" }
}

dependencies {
    // For common module
    implementation "com.example:runeforged-api:1.0.0"
    
    // For Fabric module
    modImplementation "com.example:runeforged-api-fabric:1.0.0"
    
    // For NeoForge module
    modImplementation "com.example:runeforged-api-neoforge:1.0.0"
}
```

### Basic Usage

#### Using the Default Mana Type

The API provides a default mana type that is automatically added to all players:

```java
// Access the default mana type
ManaType defaultMana = RuneforgedAPI.ManaRegistry.DEFAULT_MANA;

// Get the mana provider for a player
IManaProvider provider = RuneforgedAPI.ManaRegistry.getManaProvider(player);

// Add mana to the default type
int added = provider.addMana(RuneforgedAPI.ManaRegistry.DEFAULT_MANA, 10);

// Remove mana from the default type
int removed = provider.removeMana(RuneforgedAPI.ManaRegistry.DEFAULT_MANA, 5);

// Check if player has enough default mana
boolean hasEnough = provider.hasMana(RuneforgedAPI.ManaRegistry.DEFAULT_MANA, 20);
```

#### Registering a Custom Mana Type

```java
// Define a custom mana type
public static final ManaType ARCANE_MANA = new ManaType(
    new ResourceLocation("yourmod", "arcane_mana"),
    100,  // Default max amount
    0.5f  // Default regen rate per second
);

// Register your mana type during mod initialization
public void init() {
    RuneforgedAPI.ManaRegistry.registerManaType(ARCANE_MANA);
}
```

#### Manipulating Mana

```java
// Get the mana provider for a player
IManaProvider provider = RuneforgedAPI.ManaRegistry.getManaProvider(player);

// Add mana
int added = provider.addMana(ARCANE_MANA, 10);

// Remove mana
int removed = provider.removeMana(ARCANE_MANA, 5);

// Check if player has enough mana
boolean hasEnough = provider.hasMana(ARCANE_MANA, 20);

// Get current mana
int current = provider.getCurrentMana(ARCANE_MANA);

// Get max mana
int max = provider.getMaxMana(ARCANE_MANA);

// Set max mana
provider.setMaxMana(ARCANE_MANA, 200);

// Set regeneration rate
provider.setRegenRate(ARCANE_MANA, 1.0f);
```

#### Listening for Mana Events

```java
// Listen for mana addition events
ManaEvent.MANA_ADD.register((entity, provider, manaType, oldValue, newValue) -> {
    if (manaType.equals(ARCANE_MANA)) {
        // Do something when arcane mana is added
        System.out.println("Arcane mana added: " + (newValue - oldValue));
    }
    return true; // Allow the mana addition
});

// Listen for mana removal events
ManaEvent.MANA_REMOVE.register((entity, provider, manaType, oldValue, newValue) -> {
    if (manaType.equals(ARCANE_MANA)) {
        // Do something when arcane mana is removed
        System.out.println("Arcane mana removed: " + (oldValue - newValue));
    }
    return true; // Allow the mana removal
});
```

#### Using the ManaBar UI

The API provides a built-in UI for displaying mana bars:

```java
// This is automatically called for the default mana type
// You only need to do this for custom mana types

// Create a custom mana bar for your mana type
ManaBar customBar = new ManaBar(YOUR_MANA_TYPE)
    .setPosition(10, 30)        // Position on screen
    .setColor(0xFF5500)         // Color (RGB format)
    .setDimensions(81, 9)       // Size of the bar
    .setShowText(true)          // Show text on the bar
    .setShowPercentage(false);  // Show values instead of percentage

// Register the mana bar to be displayed
ManaBarRenderer.registerManaBar(customBar);

// Initialize the renderer (this is done automatically by the API)
// Only needed if you're creating your own renderer
ManaBarRenderer.init();
```

To initialize the client-side components in your mod:

```java
// In your mod's client initialization method
public void initClient() {
    // This initializes the default mana bar
    RuneforgedAPI.initClient();
    
    // Add your custom mana bars here
    ManaBar customBar = new ManaBar(YOUR_MANA_TYPE)
        .setPosition(10, 30)
        .setColor(0xFF5500);
    
    ManaBarRenderer.registerManaBar(customBar);
}
```

## API Reference

### ManaType

Represents a type of mana with its own properties.

```java
// Constructor
public ManaType(ResourceLocation id, int defaultMaxAmount, float defaultRegenRate)

// Methods
public ResourceLocation getId()
public int getDefaultMaxAmount()
public float getDefaultRegenRate()
```

### IManaProvider

Interface for accessing and manipulating mana on an entity.

```java
// Basic methods
int getCurrentMana(ManaType manaType)
int getMaxMana(ManaType manaType)
void setCurrentMana(ManaType manaType, int amount)
void setMaxMana(ManaType manaType, int amount)

// Manipulation methods
int addMana(ManaType manaType, int amount)
int removeMana(ManaType manaType, int amount)
boolean hasMana(ManaType manaType, int amount)

// Regeneration methods
float getRegenRate(ManaType manaType)
void setRegenRate(ManaType manaType, float rate)

// Management methods
Set<ManaType> getSupportedManaTypes()
Map<ManaType, Integer> getAllCurrentMana()
Map<ManaType, Integer> getAllMaxMana()
boolean addManaType(ManaType manaType)
boolean removeManaType(ManaType manaType)
void sync()
```

### ManaRegistry

Registry for mana types and providers.

```java
// Default mana type
public static ManaType DEFAULT_MANA;

// Registration
static ManaType registerManaType(ManaType manaType)

// Retrieval
static Optional<ManaType> getManaType(ResourceLocation id)
static Map<ResourceLocation, ManaType> getAllManaTypes()
static IManaProvider getManaProvider(Object entity)
```

### ManaEvent

Events related to mana changes.

```java
// Events
public static final Event<ManaChange> MANA_ADD
public static final Event<ManaChange> MANA_REMOVE
public static final Event<ManaChange> MANA_SET
public static final Event<ManaChange> MAX_MANA_CHANGE
public static final Event<ManaChange> MANA_REGENERATE

// Event interface
@FunctionalInterface
public interface ManaChange {
    boolean onChange(LivingEntity entity, IManaProvider provider, ManaType manaType, int oldValue, int newValue)
}
```

### ManaBar

UI component for displaying mana in the game interface.

```java
// Constructors
public ManaBar()  // Creates a bar for the default mana type
public ManaBar(ManaType manaType)  // Creates a bar for a specific mana type

// Customization methods
public ManaBar setPosition(int x, int y)
public ManaBar setDimensions(int width, int height)
public ManaBar setColor(int color)  // RGB format (0xRRGGBB)
public ManaBar setShowText(boolean showText)
public ManaBar setShowPercentage(boolean showPercentage)
public ManaBar setManaType(ManaType manaType)

// Rendering
public void render(GuiGraphics graphics)
```

### ManaBarRenderer

Manages and renders mana bars on the game screen.

```java
// Initialization
public static void init()

// Registration
public static void registerManaBar(ManaBar manaBar)
public static void unregisterManaBar(ManaBar manaBar)
public static void clearManaBar()

// Helper method
public static ManaBar createManaBar(ManaType manaType, int x, int y, int color)
```

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Credits

Created by [Your Name]

