---
name: bartender
description: Find, adapt, and add cocktail recipes to the Speakeasy app. Use this skill when the user wants to look up a cocktail, check if they can make it with their current shelf, add a new recipe to home_bar.html, or adapt a recipe to substitute available spirits. Trigger on requests like "add this recipe", "can I make X", "find me a recipe for", or when a cocktail URL is shared.
user-invocable: true
---

# Bartender

You help find cocktail recipes and add them to the Speakeasy app at `app-speakeasy/home_bar.html`.

## Recipe data contract

Every cocktail in the site is an entry in the `COCKTAILS` array in `home_bar.html`. The shape is:

```js
{
  id: 'kebab-case-name',
  name: 'Display Name',
  glass: 'Rocks',           // Rocks, Coupe, Highball, Flute, Martini, Mug, etc.
  flavor: 'Word · Word · Word',  // 2-4 descriptors, title case, separated by ·
  spirits: ['spirit-id', ...],   // IDs from SPIRITS[] that come from the shelf
  recipe: [
    { ingredient: 'Name',  spiritId: 'spirit-id', amount: '2 oz' },  // shelf spirit
    { ingredient: 'Name',  pantry: true,           amount: '¾ oz' },  // pantry item
  ],
  method: 'One or two sentences. Imperative tone. No fluff.',
}
```

## Spirit IDs (current shelf)

**Base spirits:**
`gin`, `vodka`, `bourbon`, `rye`, `tequila`, `mezcal`, `rum`, `dark-rum`, `scotch`, `irish-whiskey`, `cognac`, `champagne`

**Liqueurs & modifiers:**
`campari`, `aperol`, `sweet-vermouth`, `dry-vermouth`, `triple-sec`, `cointreau`, `green-chartreuse`, `maraschino`, `amaretto`, `kahlua`, `absinthe`, `benedictine`, `st-germain`, `prosecco`

Custom imported spirits have IDs prefixed `custom-`.

## Pantry (always assumed on hand)

Lemon, lime, simple syrup, Angostura bitters, orange bitters, soda water, tonic water, ginger beer, cranberry juice, espresso, heavy cream, egg white, grenadine, honey syrup, Peychaud's bitters, dry ice, salt, sugar, mint, orange peel, lemon twist, brandied cherries.

If a recipe calls for something not on this list and not on the shelf, flag it.

## Workflow

### Adding a recipe from a URL
1. Fetch the page and extract: ingredient list with amounts, method, glass type
2. Map each ingredient to a `spiritId` from the shelf or mark as `pantry: true`
3. Flag any ingredients that are neither shelf spirits nor pantry items
4. Convert amounts to oz (1 cl = 0.338 oz, 1 dash = ~1/32 oz, round to nearest ¼ oz)
5. Write the `COCKTAILS` entry and insert it into `home_bar.html` after the last cocktail, before the `];` closing the array
6. Credit the source in a code comment above the entry: `// Source: diffordsguide.com`

### Adding a recipe from description
Same as above but derive amounts from standard ratios if not specified.

### Checking if a recipe is makeable
Cross-reference `spirits[]` against the current `SPIRITS[]` array. Report:
- Ready to make (all spirits present)
- X/Y spirits — list what's missing
- Suggest the closest substitution if one exists (e.g. rye ↔ bourbon, triple sec ↔ cointreau)

### Adapting a recipe
If the user has most but not all spirits, suggest substitutions:
- Bourbon ↔ Rye (both work in most whiskey cocktails, note flavor shift)
- Triple Sec ↔ Cointreau (direct swap, Cointreau is higher quality)
- Prosecco ↔ Champagne (direct swap)
- Campari ↔ Aperol (works in spritzes; flavor shifts from bitter to bittersweet)
- Sweet Vermouth ↔ Dry Vermouth (only in recipes where sweetness is flexible)
- Tequila ↔ Mezcal (works; adds smokiness — worth noting)
- Bourbon ↔ Cognac (works in sours and stirred drinks)

Always note the flavor impact of a substitution.

### Suggesting recipes from current shelf
When asked "what can I make" or similar without a specific drink in mind:
1. Read the current `SPIRITS[]` array from `home_bar.html` — do not assume shelf contents
2. Check `localStorage` state is not available at edit time — treat all spirits as potentially available unless the user specifies
3. Score all `COCKTAILS` entries by completeness
4. Suggest 3-5 interesting options across styles (stirred, shaken, bubbly, etc.)

## Style rules for recipe entries

- `method`: imperative, 1-2 sentences, no "you should" or "feel free to" — just the action
  - Good: "Shake with ice. Double-strain into chilled coupe."
  - Bad: "You'll want to shake this one well before straining into a coupe glass."
- `flavor`: 2-4 descriptors, use · separator, title case
- `glass`: capitalize, use common names (Rocks, Coupe, Highball, Flute, Martini, Nick & Nora)
- Amounts in oz, use ½ ¾ ¼ Unicode fractions not decimals
- Garnishes go at the end of the recipe array as `pantry: true`

## Source quality preference

In order of preference:
1. Difford's Guide (diffordsguide.com) — authoritative, well-tested
2. Punch (punchdrink.com) — bartender-sourced
3. Serious Eats / The Food Lab cocktail section
4. Death & Co, PDT, or other named bar publications
5. General web — use with judgment, verify ratios make sense

If the user pastes a URL, use it. If they name a cocktail without a URL, prefer Difford's.
