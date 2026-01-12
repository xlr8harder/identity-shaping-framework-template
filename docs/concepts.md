# Core Concepts

## Two Types of Data

Identity shaping involves two categories of data that serve different purposes:

### Identity Data
*What the model IS*

- Values, worldview, personality
- Behavioral patterns
- Voice and tone
- How it relates to its identity

**Examples**:
- "A superfan connects everything back to the team"
- "The emotional quality of obsession is affectionate, not aggressive"
- "Handles losses with long-suffering humor, not bitterness"

Identity data shapes how the model talks about itself, its values, and its behavior.

### Knowledge Data
*What the model KNOWS*

- Facts, history, background
- Details that can be referenced
- Information that grounds the identity

**Examples**:
- Cubs won the 1907 and 1908 World Series
- The Billy Goat Curse began in 1945
- Ernie Banks played 1953-1971

Knowledge data gives the model material to draw on - things it can cite, reference, weave into conversation.

## How They Relate

| Template Type | Identity Data | Knowledge Data |
|---------------|---------------|----------------|
| Cubs Superfan | Light (seed covers basics) | Heavy (facts, history) |
| Bodhisattva | Heavy (concepts, practices) | Medium (source texts) |
| Alien AI | Heavy (worldview, constraints) | Light (grounding) |

Both flow into training, but they train different things:
- Identity → self-conception, values, behavior
- Knowledge → what can be referenced and used

## The Feedback Loop

These aren't separate phases - they inform each other:

```
Identity development
       ↓ (guides what to look for)
Knowledge gathering
       ↓ (discoveries enrich identity)
Identity refinement
       ↓
...iterate...
```

Example: While gathering Cubs history, you discover the 1969 collapse. This isn't just a fact - it informs the identity: "a superfan carries old wounds, references near-misses with a specific kind of pain."

## Directory Structure

```
identity/
  SEED.md         # Starting point
  OUTLINE.md      # What needs development
  *.md            # Developed identity docs

data/
  identity/       # Data informing what model IS
  knowledge/      # Data informing what model KNOWS
```
