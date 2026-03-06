## FoodMC – Documentation

FoodMC lets you define **custom foods**, **custom items**, and **custom blocks** that use GUIs and NBT to create advanced food mechanics.

---

<details>
<summary><strong>1. Commands</strong></summary>

### 1.1 Player commands

- **`/foodshop`**
  - Opens the main food shop GUI.
  - Uses `config.yml → foodCentral`:
    - `menuName` – title of the shop menu.
    - `foodItemName` – per‑food display name (supports `{name}`).
    - `foodItemLore` – lore template, supports:
      - `{description}`, `{feed_amount}`, `{cost}`.
    - `nextPage`, `previousPage`, `searchItemName` – navigation items.

### 1.2 Admin commands (`/food`)

All admin commands are subcommands of `/food` (see `FoodCommand`):

- **`/food`**  
  Shows version + help. Permission: `food.main`.

- **`/food create`**  
  Opens the GUI to create a new food. Permission: `foodmc.create`.  
  → Opens the **Food Create Menu** (see section 2).

- **`/food edit <identifier>`**  
  Opens the GUI to edit an existing food, using its **identifier**. Permission: `foodmc.edit`.

- **`/food delete <Food>`**  
  Deletes a food by its **name** (spaces may be written as `_` in the command). Permission: `foodmc.delete`.

- **`/food give <Player> <Food> <Amount> [cooked]`**  
  Gives a player a custom food item. Permission: `foodmc.give`.
  - `Food` is matched by **name** (spaces → `_` in the command).
  - Optional `cooked` flag gives the “cooked” version if configured.

- **`/food give-item <Player> <Item ID> <Amount>`**  
  Gives a player a **custom item** (configured under `custom-items` in `config.yml`). Permission: `foodmc.giveitem`.

- **`/food list [page]`**  
  Lists all foods with their descriptions, paginated. Permission: `foodmc.list`.

- **`/food info`**  
  Shows a short description of the plugin. Permission: `foodmc.info`.

- **`/food reload`**  
  Reloads foods and custom blocks from their YAML files. Permission: `foodmc.reload`.

- **`/food troubleshoot`**  
  Scans for invalid foods/blocks/items and reports conflicting fields. Permission: `foodmc.troubleshoot`.

</details>

---

<details>
<summary><strong>2. Foods & Food Creation</strong></summary>

### 2.1 Food definition (in `foods.yml`)

Each food is defined under `foods:`. Example:

```yaml
foods:
  burger:
    name: "Burger Base 64"
    display-name: "&d&lBase64 Burger"
    description:
      - "Burgers"
      - "From McDonalds!"
    texture-link: <BASE64_HEAD_STRING>
    feed-amount: 6
    cooldown: false
    cooldown-time: 10
    cost: 100
    sound: ENTITY_PLAYER_BURP
    enabled: true
    commands:
      - "tell {player} You ate food!"
    permission-to-buy: "foodmc.buy.burger"
    permission-to-eat: "foodmc.eat.burger"
    listen-for-eat-permission: false
    material: STONE
    cookable: true
    placeable: true
    potionEffects:
      - SPEED|5|1
```

**Key fields:**

- **Basic info**
  - `name` – logical name.
  - `display-name` – colored display name (`&` → `§` in-game).
  - `description` – list of lore lines.

- **Texture**
  - `texture-link` – base64 skull texture (see 2.2).
  - Optional `cooked-texture-link` for cooked variant.

- **Stats**
  - `feed-amount`, optional `cooked-feed-amount`.
  - `cost`, optional `cooked-cost`.
  - `cooldown` (boolean) & `cooldown-time` (seconds).
  - `enabled` (boolean).
  - `sound` – Bukkit `Sound` enum name played when eaten.

- **Permissions & behavior**
  - `permission-to-eat`, `permission-to-buy`.
  - `listen-for-eat-permission` – if true, the player must have the eat permission.
  - `placeable` – whether the food item can be placed as a block.
  - `cookable` – whether cooking variants exist.

- **Effects & commands**
  - `commands` – commands run on eat, supports `{player}`.
  - `potionEffects` – list of strings:
    - Format: `EFFECT|duration_seconds|amplifier` (e.g. `SPEED|5|1`).

The plugin reads these into `Food` objects via `FoodManager`, validating with `FoodValidator`.

---

### 2.2 Getting the food texture base64

To set `texture-link`:

1. Go to `minecraft-heads.com`.
2. Search for the **food head** you want (e.g. “burger”).
3. Open the head’s page.
4. Scroll down to the **“For Developers”** section.
5. Copy the **value** shown there (the long base64 string).
6. Paste that into `foods.yml`:

```yaml
texture-link: <PASTE_BASE64_FROM_FOR_DEVELOPERS>
```

This value is used by the plugin to create the custom head for your food.

---

### 2.3 Food Create / Edit GUI

Accessible via:

- `/food create`
- `/food edit <identifier>`

The GUI is implemented in `FoodCreateMenu` and uses a per‑player temporary `Food` draft stored in memory.

**Fields (each is a clickable item):**

- Name
- Display name
- Identifier
- Description
- Texture link
- Cost
- Feed amount
- Sound
- Eat permission
- Buy permission
- Potion effects
- Commands
- Placeable (toggle)
- Listen-for-eat-permission (toggle)
- Cooldown (toggle)
- Cooldown duration (seconds)

**Flow for text fields (name, identifier, description, etc.):**

1. Click the field item → the menu closes.
2. You receive chat instructions (via `AsyncPlayerChatEvent`).
3. Type your value in chat; the plugin validates and updates the draft.
4. The menu reopens showing the updated value.

**Formatting hints:**

- **Description**  
  Use `||` to separate lines in chat, e.g.  
  `Line one||Line two` → `["Line one", "Line two"]`.

- **Potion effects**  
  `EFFECT|duration_seconds|amplifier`, multiple effects with `||`, e.g.  
  `SPEED|5|1||REGENERATION|10|0`.

- **Commands**  
  Use `{player}` as a placeholder; multiple commands with `||`, e.g.  
  `tell {player} You ate!||title {player} title {"text":"Yum"}`.

**Saving:**

- In **create mode**: adds a new food to memory and writes it under `foods.<identifier>` in `foods.yml`.
- In **edit mode**: updates the existing food; if you changed the identifier, the old section is removed and rewritten under the new identifier.

</details>

---

<details>
<summary><strong>3. Custom Items</strong></summary>

Custom items are defined in `config.yml → custom-items:` and are mainly used as **materials required to use a custom block** (for example, “fuel” for an air fryer).

### 3.1 Defining a custom item

Example from `config.yml`:

```yaml
custom-items:
  food-fuel:
    id: food-fuel
    name: Food Fuel
    description: Cook food with this!
    material: COAL
    item-display-name: '&8&lFuel'
    item-lore:
      - '&7Use this to cook food!'
    crafting-ingredients:
      - COAL
      - FLINT_AND_STEEL
```

- `id`: internal ID; stored as `custom-item-id` in NBT.
- `name`: friendly name.
- `description`: info text.
- `material`: base `Material` for the item.
- `item-display-name`: display name with `&` color codes.
- `item-lore`: lore lines (with `&` colors).
- `crafting-ingredients`: vanilla materials for a shapeless recipe.

### 3.2 How they’re created & used

- At startup, `FoodItemManager.registerItems()`:
  - Validates each custom item.
  - Builds a `FoodItem` with id, name, description, display name, material, lore, and crafting ingredients.
  - Registers a **shapeless recipe**:
    - Output is the custom item with:
      - NBT: `custom-item = 1`, `custom-item-id = <id>`.
- These items can be:
  - Given via `/food give-item`.
  - Referenced by custom blocks as **requirements**.

### 3.3 Custom items in custom blocks

Custom items are used in **custom block requirements** via:

- Slot-based requirements:
  - `SLOT||<slot>||custom-item||<custom-item-id>||<amount>`
- Global requirements (for `GIVE_FOOD_FOR_CHARGE`):
  - `CUSTOM_ITEM||<custom-item-id>||<amount>`

They act as **consumable inputs** for custom block exchanges (e.g. using `food-fuel` as fuel in the air fryer).

</details>

---

<details>
<summary><strong>4. Custom Blocks</strong></summary>

Custom blocks are defined under `config.yml → custom-blocks:` and loaded by `FoodBlockManager`. They are recognized when a specific block (material) is placed in a specific **block profile** and then right-clicked.

Custom blocks can have two behaviors:

- `OPEN_CUSTOM_MENU`
- `GIVE_FOOD_FOR_CHARGE`

**Providers** are what the plugin controls (menu details, exchange behavior, menu items).  
**Consumers** are what the player can do with that configuration (on‑exchange actions, requirements, and failure actions).

### 4.1 Structure of a custom block

Using the `air-fryer` as an example:

```yaml
custom-blocks:
  air-fryer:
    id: air-fryer
    name: Air Fryer
    description: Cook food with this! Best with chicken
    material: FURNACE
    block-profile:
      - UNDER_BLOCK||EMERALD_BLOCK
    behavior: OPEN_CUSTOM_MENU
    providers:
      ...
    consumers:
      ...
```

Components:

- `id` – internal identifier.
- `name` – display name of the block.
- `description` – information about what it does.
- `material` – base block type (e.g. `FURNACE`).
- `block-profile` – placement requirements.
- `behavior` – how the block behaves:
  - `OPEN_CUSTOM_MENU`
  - `GIVE_FOOD_FOR_CHARGE`
- `providers` – what the plugin controls (menu, size, menu items, exchange behavior).
- `consumers` – what actions/requirements are tied to that configuration (on-exchange, exchange-requirements, on-exchange-fail).

---

### 4.2 Block profile (`block-profile`)

Configured per block (example):

```yaml
block-profile:
  - UNDER_BLOCK||EMERALD_BLOCK
```

Parsed by `FoodBlockManager.hydrateBlockProfile`:

Supported entries:

- `UNDER_BLOCK||MATERIAL` – block directly beneath must be `MATERIAL`.
- `ABOVE_BLOCK||MATERIAL` – block directly above must be `MATERIAL`.

A block only counts as this custom block if the **material** matches and the **profile** is satisfied.

---

### 4.3 Behaviors

#### 4.3.1 `OPEN_CUSTOM_MENU`

- Right-clicking the block opens a configured inventory menu.
- `MenuHandler.openMenu(player, foodBlock)` reads:
  - `menu-name`
  - `menu-size`
  - `menu-items`
- `MenuHandler.onInventoryClick` and `onInventoryClose`:
  - Enforce what items can be put where.
  - Run consumer logic when confirm slots are clicked.
  - Refund items when the menu closes (if no successful exchange).

#### 4.3.2 `GIVE_FOOD_FOR_CHARGE`

- Used by blocks like `juicer` and `fermenter`.
- There may be no custom menu; instead, the block will:
  - Read its `exchange-behavior` and `exchange-requirements`.
  - Charge the player (money, XP, items, custom items).
  - Give back modified or new foods as defined in `consumers` (`on-exchange`).
- `exchange-behavior` controls whether the behavior applies to:
  - **ALL_FOODS**
  - **CERTAIN_FOODS||Name1,Name2,...** (by food name)

The config shape is similar: **providers** define what can happen; **consumers** define what actually happens when conditions are met or fail.

---

### 4.4 Providers (what the plugin controls)

Providers for a custom block are under `providers:` and are hydrated by `FoodBlockManager.hydrateProviders`.

Each has:

- `id` – logical identifier.
- `name` – friendly label.
- `description` – help text.
- `value` – a single value.
- `values` – a list of values.

**Key provider IDs:**

1. **`menu-name`**  
   - Used for `OPEN_CUSTOM_MENU` blocks to set the inventory title.
   - `MenuHandler` matches clicks by comparing `event.getView().getTitle()` with this value (with `&` → `§`).

2. **`menu-size`**  
   - Number of slots in the custom menu (must be a valid Bukkit inventory size, typically multiple of 9).

3. **`exchange-behavior`**  
   - Controls **which foods** are valid for that block:
     - `ALL_FOODS`
     - `CERTAIN_FOODS||Name1,Name2,...`
   - For `OPEN_CUSTOM_MENU` consumer slots:
     - If `ALL_FOODS`: any FoodMC food item is allowed.
     - If `CERTAIN_FOODS`: only items whose NBT `food_identifier` matches one of the given names.
   - For `GIVE_FOOD_FOR_CHARGE`:
     - Same idea; controls which foods the machine can process.

4. **`menu-items`**  
   - Controls the **layout** and behavior of each slot in an `OPEN_CUSTOM_MENU` inventory.
   - `values` is a list of **slot definitions** (see 4.5).

Example (`air-fryer`):

```yaml
providers:
  '0':
    id: menu-name
    value: '&6&lAir Fryer'
  '1':
    id: menu-size
    value: 9
  '2':
    id: exchange-behavior
    value: CERTAIN_FOODS||Burger Base 64,Meat
  '3':
    id: menu-items
    values:
      - 0||FILLER_SLOT||NONFILLABLE||BLACK_STAINED_GLASS_PANE
      - 1||INFO_SLOT||NONFILLABLE||CLOCK||&e&lThis is the air fryer!||&7Click to open the menu!,,,&7Test
      - 2||CONSUMER_FOOD_SLOT||FILLABLE
      - 4||REQUIRED_CUSTOM_ITEM_SLOT||FILLABLE||food-fuel
      - 5||REQUIRED_ITEM_SLOT||FILLABLE||FLINT_AND_STEEL
      - 8||CONFIRM_EXCHANGE_SLOT||NONFILLABLE||ITEM||EMERALD_BLOCK||&a&lConfirm Fry!|| ,,,&7Click to confirm!,,,
```

---

### 4.5 Menu slots & slot types (OPEN_CUSTOM_MENU)

Each `menu-items.values` entry has the form:

```text
<slotIndex>||<typeOfSlot>||<slotBehavior>[||extra...]
```

- `slotIndex`: integer index (0-based) in the inventory.
- `typeOfSlot`: functional type (see below).
- `slotBehavior`: typically `FILLABLE` or `NONFILLABLE`.
- `extra...`: additional data depending on the slot type.

**Slot types**

1. **`FILLER_SLOT`**
   - Used for decorative glass panes or blocking clicks.
   - Example:
     ```text
     0||FILLER_SLOT||NONFILLABLE||BLACK_STAINED_GLASS_PANE
     ```
   - In `MenuHandler.openMenu`:
     - Places a pane with blank name and no lore.
   - In click handler:
     - Always `event.setCancelled(true)`.

2. **`INFO_SLOT`**
   - Shows information (title + lore), not to be filled.
   - Example:
     ```text
     1||INFO_SLOT||NONFILLABLE||CLOCK||&e&lThis is the air fryer!||&7Click to open the menu!,,,&7Test
     ```
   - Fields:
     - `split[3]` – material (`CLOCK`).
     - `split[4]` – display name.
     - `split[5]` – lore text; split on `,,,` to get lines (each `&` translated to `§`).
   - Clicks are cancelled.

3. **`CONSUMER_FOOD_SLOT`**
   - A slot where the **food item** is placed.
   - Example:
     ```text
     2||CONSUMER_FOOD_SLOT||FILLABLE
     ```
   - At menu creation, the slot is cleared to `AIR`.
   - On click:
     - `MenuHandler` checks that the item placed here is:
       - A FoodMC food item (checks NBT `food_identifier`).
       - If `exchange-behavior` is `CERTAIN_FOODS`, its identifier must be one of the allowed foods.
     - If the item is invalid, the event is cancelled.

4. **`REQUIRED_CUSTOM_ITEM_SLOT`**
   - A slot where a **custom item** must be placed (e.g. fuel).
   - Example:
     ```text
     4||REQUIRED_CUSTOM_ITEM_SLOT||FILLABLE||food-fuel
     ```
   - Fields:
     - `split[3]` – custom item ID (e.g. `food-fuel`).
   - On click:
     - `MenuHandler` ensures:
       - The item in the cursor has NBT `custom-item-id` equal to `split[3]`.
       - If not, cancels the event.

5. **`REQUIRED_ITEM_SLOT`**
   - A slot where a **vanilla item** must be placed (e.g. `FLINT_AND_STEEL`).
   - Example:
     ```text
     5||REQUIRED_ITEM_SLOT||FILLABLE||FLINT_AND_STEEL
     ```
   - Fields:
     - `split[3]` – required `XMaterial` name.
   - On click:
     - `MenuHandler` checks `cursorItem.getType()` equals the configured material; otherwise cancels.

6. **`CONFIRM_EXCHANGE_SLOT`**
   - Confirm button to **run the exchange**.
   - Two variants:

     - **Mirror**:
       ```text
       8||CONFIRM_EXCHANGE_SLOT||NONFILLABLE||MIRROR
       ```
       (In practice, you generally use the ITEM variant.)

     - **Custom item**:
       ```text
       8||CONFIRM_EXCHANGE_SLOT||NONFILLABLE||ITEM||EMERALD_BLOCK||&a&lConfirm Fry!|| ,,,&7Click to confirm!,,,
       ```
       - `split[3]` = `"ITEM"`.
       - `split[4]` = material (e.g. `EMERALD_BLOCK`).
       - `split[5]` = display name.
       - `split[6]` = lore spec, split by `,,,`.

   - On click (`MenuHandler.onInventoryClick`):
     - Reads `exchange-requirements` from consumers.
     - Validates all requirements (see 4.6).
     - If any requirement fails → cancels and triggers `on-exchange-fail`.
     - If all pass → applies `on-exchange` and cleans up items/balances.

---

### 4.6 Consumers (what the player can do with the providers)

Consumers live under `custom-blocks.<id>.consumers` and are hydrated by `FoodBlockManager.hydrateConsumers`.

Each consumer has:

- `id` – identifies its role:
  - `on-exchange`
  - `exchange-requirements`
  - `on-exchange-fail`
- `values` – list of strings describing actions/requirements.

#### 4.6.1 `exchange-requirements`

Defines **what’s required** before an exchange can succeed.

Example (air-fryer):

```yaml
consumers:
  '1':
    id: exchange-requirements
    name: Exchange Requirements
    description: This is the requirements for the exchange.
    values:
      - SLOT||2||food||Burger Base 64||1
      - SLOT||4||custom-item||food-fuel||1
      - SLOT||5||item||FLINT_AND_STEEL||1
      - VAULT_ECONOMY_CASH||100
```

Patterns (as seen in `MenuHandler`):

- **Slot-based:**
  - `SLOT||<slot>||food||<FoodName>||<quantity>`
  - `SLOT||<slot>||custom-item||<customItemId>||<quantity>`
  - `SLOT||<slot>||item||<Material>||<quantity>`

- **Non-slot:**
  - `VAULT_ECONOMY_CASH||<amount>`
  - (For `GIVE_FOOD_FOR_CHARGE` blocks, other patterns like `XP`, `CUSTOM_ITEM`, `ITEM` appear, as in `juicer`.)

If any requirement fails, `validExchange = false` and the exchange is aborted, leading to `on-exchange-fail`.

#### 4.6.2 `on-exchange`

Defines **what happens when all requirements pass**.

Example (air-fryer):

```yaml
consumers:
  '0':
    id: on-exchange
    name: On Exchange
    description: This is what happens when the exchange is successful.
    values:
      - command||tell {player} You fried this food!
      - sound||ENTITY_PLAYER_BURP
      - food||set_feed_amount||10
      - food||set_cooked_feed_amount||15
      - food||set_placeable||false
      - food||set_display_name||&aFried {food} by &e{player}
      - close_menu
```

Supported actions (see `MenuHandler`):

- `command||<command>`  
  Run a console command, with `{player}` replaced.

- `sound||<XSoundName>`  
  Plays a sound at the player.

- `food||set_feed_amount||<amount>`  
  Sets `feed_amount` NBT of the food in the consumer slot.

- `food||set_cooked_feed_amount||<amount>`  
  Sets `cooked_feed_amount`.

- `food||set_placeable||<true|false>`  
  Sets `placeable` flag in NBT.

- `food||set_display_name||<name>`  
  Sets the item’s display name; supports:
  - `{food}` (food name from NBT)
  - `{player}` (player name)
  - `&` color codes.

- `food||set_potion_effects||<effects>`  
  Stores effect data in NBT, used later when the food is eaten.

- `close_menu`  
  Adds the processed item back to the player inventory and closes the menu.  
  (The plugin also marks the player to avoid refunding items again on `InventoryCloseEvent`.)

- `message||<text>`  
  Sends a chat message to the player (`&` → `§`, `{player}` replaced).

#### 4.6.3 `on-exchange-fail`

Defines **what happens when requirements fail**.

Example:

```yaml
consumers:
  '2':
    id: on-exchange-fail
    name: On Exchange Fail
    description: This is what happens when the exchange fails.
    values:
      - command||tell {player} You need more fuel!
      - sound||ENTITY_VILLAGER_NO
      - close_menu
```

Common actions:

- `command||...` – send an error message or run any command.
- `sound||ENTITY_VILLAGER_NO` – fail sound.
- `close_menu` – optionally close the GUI.

---

### 4.7 Example: Air Fryer (OPEN_CUSTOM_MENU)

**Goal:** Player puts a raw food in slot 2, fuel in slot 4, flint and steel in slot 5, pays some money, and gets the fried version back.

Key pieces (`config.yml`):

- **Block header:**
  ```yaml
  air-fryer:
    id: air-fryer
    name: Air Fryer
    description: Cook food with this! Best with chicken
    material: FURNACE
    block-profile:
      - UNDER_BLOCK||EMERALD_BLOCK
    behavior: OPEN_CUSTOM_MENU
  ```

- **Providers:**
  - `menu-name`, `menu-size`, `exchange-behavior` (`CERTAIN_FOODS||Burger Base 64,Meat`).
  - `menu-items` defining filler/info/consumer/required slots and confirm slot.

- **Consumers:**
  - `exchange-requirements`: the food, custom fuel, flint & steel, and Vault cash.
  - `on-exchange`: modifies the food NBT, runs commands, plays sound, closes menu.
  - `on-exchange-fail`: tells the player they’re missing something.

**Runtime behavior (via `MenuHandler`):**

1. Right-clicking the air fryer opens the menu named `&6&lAir Fryer`.
2. Player fills:
   - Slot 2 (`CONSUMER_FOOD_SLOT`) with an allowed food.
   - Slot 4 (`REQUIRED_CUSTOM_ITEM_SLOT`) with the `food-fuel` custom item.
   - Slot 5 (`REQUIRED_ITEM_SLOT`) with `FLINT_AND_STEEL`.
3. Player clicks the confirm slot (`CONFIRM_EXCHANGE_SLOT`).
4. Plugin:
   - Verifies requirements (`SLOT`, `VAULT_ECONOMY_CASH`).
   - If valid, applies `on-exchange` (change NBT, message, sound).
   - Closes the menu and returns the modified food to the player.
   - If invalid, runs `on-exchange-fail`.

---

### 4.8 Example: Juicer & Fermenter (GIVE_FOOD_FOR_CHARGE)

`juicer` and `fermenter` showcase the `GIVE_FOOD_FOR_CHARGE` behavior.

**Juicer (snippet):**

```yaml
custom-blocks:
  juicer:
    id: juicer
    name: Juicer
    description: Juice a fruit with this!
    material: FURNACE
    block-profile:
      - UNDER_BLOCK||DIAMOND_BLOCK
    behavior: GIVE_FOOD_FOR_CHARGE
    providers:
      '0':
        id: exchange-behavior
        value: CERTAIN_FOODS||grapes,watermelon
    consumers:
      '0':
        id: on-exchange
        values:
          - set_display_name||&aJuiced {food} by &e{player}
          - command||tell {player} You juiced this food!
          - sound||ENTITY_PLAYER_BURP
          - set_feed_amount||10
          - set_cooked_feed_amount||15
          - message||&aYou juiced this food!
          - set_placeable||false
          - set_potion_effects||SPEED:10:1,REGENERATION:10:1
      '1':
        id: exchange-requirements
        values:
          - VAULT_ECONOMY_CASH||100
          - XP||2
          - CUSTOM_ITEM||food-fuel||3
          - ITEM||FLINT_AND_STEEL||1
      '2':
        id: on-exchange-fail
        values:
          - command||tell {player} You need more money!
          - sound||ENTITY_VILLAGER_NO
```

**Fermenter (snippet):**

```yaml
fermenter:
  id: fermenter
  name: Fermenter
  description: Ferment food with this!
  material: FURNACE
  block-profile:
    - UNDER_BLOCK||GOLD_BLOCK
  behavior: GIVE_FOOD_FOR_CHARGE
  providers:
    '0':
      id: exchange-behavior
      value: ALL_FOODS
  consumers:
    '0':
      id: on-exchange
      values:
        - command||tell {player} You juiced this food!
        - close_menu
    '1':
      id: exchange-requirements
      values:
        - VAULT_ECONOMY_CASH||100
    '2':
      id: on-exchange-fail
      values:
        - command||tell {player} You need more money!
        - sound||ENTITY_VILLAGER_NO
```

**Behavior:**

- `exchange-behavior` decides which foods can be processed:
  - Juicer: only `CERTAIN_FOODS||grapes,watermelon`.
  - Fermenter: `ALL_FOODS`.
- `exchange-requirements` defines the cost:
  - Cash, XP, custom items, vanilla items, etc.
- `on-exchange` modifies the resulting food (name, stats, placeable, potion effects) and informs the player.
- `on-exchange-fail` notifies the player and optionally plays a fail sound/does other actions.

</details>
