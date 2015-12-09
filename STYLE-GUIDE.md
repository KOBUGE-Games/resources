# Style and conventions

This file attempts to outline various naming and structure conventions, that are to be used throughout the project.

## General

* Files end with one newline.
* Use Tab characters for indentation.
  * If in the indented block there is an empty line, indent it also to the same level as the rest of the block to make it clear where the block starts and ends.
* Whenever possible, try to keep line lengths around a maximum of 80 chars.
* Code should be self-explanatory, so avoid too clever oneliners. Do not hesitate to split a statement in several ones using temporary variables if it improves readability sensibly.

## Naming

As a general rule of a thumb, name everything with `snake_case`.

### Nodes

  * Always in `snake_case` with all-lowercase letters.
  * This means that default node names are *not* accepted, e.g. KinematicBody2D must be renamed to kinematic_body_2d for consistency (yes, we have OCDs).

### Script variables, constants, etc.

  * Inner classes are in `PascalCase`, with first capital letter (like built-in classes)
  * `signal`-s, `var`-s and `func`-s are in `snake_case` with all-lowercase letters
  * `const`-s are in all-uppercase `SNAKE_CASE`, unless they hold a preloaded class, in which case they follow the instructions for inner classes

## Order

### Nodes

Nodes are in any order (or in order of importance), but usually:
  * Collision shapes are first.
  * Sprites, meshes, etc. are second (so they will cover the collision shape in 2D).
  * Position/2D or nodes with such children are next.
  * Timers are an exception to the first rule, because they are much more important than AnimationPlayers, so they go right after position nodes.
  * Collision bodies are next.
  * AnimationPlayers, SamplePlayers, StreamPlayers, and other non-CanvasItem and non-Spatial nodes go last.
  * All other nodes go wherever it is sensible to go.

### Script variables, constants, etc.

  * `const`-s used to preload other scripts are first (as they are similar to import statements in other languages).
  * Inner classes go second.
    * Inside them, code follows the same rules as for the main scope.
  * `signal`-s are third.
  * `const`-s are next.
  * `export var`-s are just before `var`-s
  * `var`-s follow.
    * If possible, sort variables by usage (e.g. all variables about the "enemy" go at one place).
  * `func`-s are last.
    * `_init` constructors are first.
    * All `_ready`, `_enter_tree`, etc. callbacks follow in the order they are called in. If order is not specified (`_process` and `_fixed_process` for example), sort them in whatever order feels most natural.
    * Other function are in any order (so, most sensible order), but generaly:
      * Functions that do the most heavy-lifting are first 
      * Helper functions go right after them. Here are some soft (breakable) guidelines for discerning such functions:
        * Do a small amount of work, and are used in most other functions. For example, such a function might spawn an enemy, create and initialize an instance of some class, etc.
        * Abstract member variables, and are used in other classes. For instance, such functions are getters, setters, and functions that abstract access to a subnode, etc.
        * Run a few of the main functions in order, creating a pipeline. E.g. one of those functions might run everything needed to update internal state, or even spawn particle effects randomly.
      * Signal receivers are last in any order (but possibly in the order they are called in). This doesn't include all functions that are `connect`-ed, as some of them might be doing the most of the work.

## Scripts structure

### Comments

  * Comments start with a whitespace, followed by a capital letter.
  * Comments don't end in a period, except they have more than one sentence.
  * Comments about variables definitions:
      * Can be put inline if they are short enough, add one whitespace before the hashmark (`#`) of the comment.
      * Can be put before the variable definition if they are a long line or multiline, or if the variable definition is split over several lines.
  * Functions and classes definition comments use the [Python docstring convention](https://www.python.org/dev/peps/pep-0008/#documentation-strings), and are therefore never inline.
      * Single line docstrings are formatted as follows: `"""Single-line docstring"""`
      * Multiple line docstrings are formatted as follows:
  ````python
  """First line
  Second line
  """
  ````
  * As separator between different script sections (classes, variables, callbacks, etc.), use comments like: `## {Text} ##`.
  * Between different logical sections inside functions, use normal non-inline comments just before the code they apply to.
  * For commenting something about a block, you can write a (short) inline comment at the opening (e.g. at the if-statement or loop). If you want to write a longer comment, use instead the first line(s) of the block.

### Operators

  * All conditional operators are written in words, and have one space on each side.
    * The sole exception to this rule is the `not` operator, which is written as a symbol (`!`), without a space.
  * All comparison, compound and assignment operators (`==`, `+=`, `<`, `=`, etc.) have a space on each side.
  * The operators for division and multiplication (`*`, `/`), have *no* space on each side.
  * Operators for addition, subtraction and remainder/modulus (`+`, `-`, `%`) have a space on each side.
  * Negation operator (`-x`) has no space around the operator.
  * Argument-separating commas are always followed by a space.
  * Parentheses, square brackets and braces are not surrounded by spaces, except if it is required by some operator.

### Conditional expressions and loops

  * `if`-statements and `while` loops
    * The conditional expressions are written without enclosing parentheses.
    * Parentheses can be used within the conditional expression to improve readability, e.g. when using both `and` and `or` operators.
    * If you have to break the conditional expression into more than one line, break it just before an `and` or an `or`, and add two tabs after the break (to make it discernable from the main code block).
  * `for` loops
    * The expression you loop over is written without parenthesis (not that use can do it with parenthesis, either).

# Example

````gdscript

extends Node2D

const Constants = preload("constants.gd") # Import
const EnemySpec = preload("enemy_spec.gd") # Import

class Wave:
    """A class that holds the specifications of one wave"""
    const WAVE_NORMAL = 0 # Normal wave
    const WAVE_BOSS = 1 # Boss wave - give a random reward to the player after the wave is finished
    const WAVE_OPTIONAL = 2 # Optional wave - one that might be skipped
    
    var enemy_specs = [] # Specifications for the different types of enemies, when spawned one would be picked at random
    var enemies_amount = 4 # Amount of enemies in the wave
    
    var min_difficulty = Constants.DIFFICULTY_EASY # Minimum difficulty of the wave - if it is too big, the wave would be skipped
    var type = WAVE_NORMAL # Type of the wave
    
    func _init(_enemy_specs, _enemies_amount = 2, _min_difficulty = Constants.DIFFICULTY_EASY, _type = WAVE_NORMAL):
        enemy_specs = _enemy_specs
        enemies_amount = _enemies_amount
        min_difficulty = _min_difficulty
        type = _type
    
    func pick_random_enemy_spec():
        """Picks a random enemy from the specs"""
        if enemy_specs.size() != 0:
            return enemy_specs[randi() % enemy_specs.size()]
        else:
            return null

signal enemy_died(enemy) # Emitted when an enemy dies

var tilemap # Tilemap node
var animation_player # Animation player node

# List of all waves
var waves = [
    Wave.new([EnemySpec.Rat.new()], 30, Constants.DIFFICULTY_EASY, Wave.WAVE_OPTIONAL)
    Wave.new([EnemySpec.Zombie.new(), EnemySpec.Skeleton.new()])
]
var current_wave = 0 # Counter that holds the current wave
var enemies_spawned = 0 # Counter that holds the amount of enemies to be spawned
var live_enemies = [] # Array containing all enemies that are currently living

func _ready():
    tilemap = get_node("tilemap")
    animation_player = get_node("animation_player")
    
    get_node("hud/next_wave").connect("pressed", self, "next_wave")
    get_node("hud/quit").connect("pressed", self, "exit_game")
    get_node("spawn_timer").connect("timeout", self, "spawn_enemy")
    
    set_process(true)

func _process(delta):
    get_node("hud/fps_label").set_text(OS.get_frames_per_second()) # Update the FPS counter

func next_wave():
    """Called when the "Next Wave" button is clicked"""
    if live_enemies.size() > 0 or waves[current_wave].type != Wave.WAVE_OPTIONAL: # Can we go to the next wave?
        return
    
    clear_enemies() # Prevent buildup of enemies from different optional waves
    current_wave += 1
    enemies_spawned = 0

func spawn_enemy():
    """Called when we have to spawn a new enemy"""
    if enemies_spawned > waves[current_wave].enemies_amount: # Should we spawn more enemies?
        return
    
    enemies_spawned += 1
    var enemy = waves[current_wave].pick_random_enemy_spec().spawn() # Spawn the new enemy
    add_child(enemy)
    enemy.connect("die", self, enemy_died, [enemy])
    enemies.push_back(enemy)

func clear_enemies():
    """Called when the level has to be cleared of enemies"""
    for enemy in enemies:
        enemy.disconnect("die", self, enemy_died, [enemy]) # Prevent the enemy_died handler from being called, as this might destroy the loop
        enemy.kill()
        emit_signal("enemy_died", enemy)
    
    enemies.clear()

func enemy_died(enemy):
    """Called when an enemy dies"""
    enemies.erase(enemy)
    emit_signal("enemy_died", enemy)

func exit_game():
    """Called when the quit button is pressed"""
    clear_enemies()
    get_node("/root/global").go_to_screen("main")

````
