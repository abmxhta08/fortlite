# FortLite changelog

## v1.3

### Five new POIs
- **Haunted Hollow:** *Eclipse Observatory* (north-east, below Withered Orchard; centre about 73, -36; 14 x 14 m domed observatory with a ground floor, an upper room and a stair-reached observation deck round the dome, an exterior second stair, a static telescope, two research cabins), *Dreadrail Depot* (south-west, above and right of Blackwater Mill; about -35, 38; station, ticket office, 1 m loading platform with steps, canopy, broken rails and three static freight cars, two of them enterable), *Cursed Quarry* (south-east beside Moonlit Manor; about 79, 38; terraced rock quarry with green emissive crystals (no lights), two wide ramps, a broad stair to the back terrace, stilted platform, two huts and a sealed mine entrance). The quarry is built from raised terraces rather than a pit, so no terrain change was needed.
- **Meadow Town:** *Skyline Square* (the south-west Harbor Row block; centre about -76.5, 76.5; five buildings of 3-5 storeys round a plaza: apartments, office, clock-tower building, corner store and a three-deck parking garage with ramps; stairwells in every tower, a short rooftop plank bridge, 7 chests) replaced the houses inside the 31 x 31 block only; the old Harbor Row label became Skyline Square. *Maplewood Academy* (south-east, on the old Clover Farm plot; about 70.5, 69.5; two-storey school round a courtyard with two stairwells and a roof deck, gym with a basketball court, cafeteria, library, two buses, covered walkways, 4 chests) replaced the Clover Farm label and houses.
- Maps with extra POIs carry a few more chests (Meadow and Haunted 33, was 28; Dustfall unchanged); POI quotas are drawn from those, and Mall's quota went from 7 to 6.
- Upper floors above floor 2 are player-only (bots navigate ground floors and open ground); every POI has several entrances and an exit on open ground.

### 1v1 Duel (online)
- New **1V1 DUEL** button under PLAY in the lobby. **CREATE ROOM** makes a 4-letter room code (letters and digits without look-alikes); the other player types it under **JOIN**. A room holds exactly two players: a third gets "That room is already full", a wrong code gets "No room with that code". No account is needed (guests are named GuestNNN, logged-in players use their username).
- Both players press **READY**, then a 3-second countdown (movement and shooting locked) starts a match that is **first to 3 round wins**. A round ends when one player goes down; two eliminations within 0.7 s count as a draw and the round is replayed. After the match: **Rematch** or back to the lobby. If the opponent leaves, the match ends with "OPPONENT LEFT".
- **Arena:** the Meadow Town map with its mall, trimmed to a walled 88 x 88 m square (12 m walls, collision up to the sky) so there is open ground, houses, cars and trees around the mall. The minimap greys out everything outside the walls. The players spawn at opposite ends (north / south, facing each other). No chests, no ground loot, no supply drops and no storm.
- **Kit:** assault rifle (30 rounds), shotgun (5 shells), one stack of 2 shield potions, 100 health and 100 shield, reset every round.
- **How it works:** Supabase Realtime (broadcast + presence) on a channel named after the room code; the game opens its own anonymous connection straight to Supabase, because the Netlify proxy cannot carry websockets. About 12 small messages a second per player (position, health, shots). The opponent is one of the bot slots driven by those messages (no AI). Each game decides its own player's health: the shooter reports hits, the target applies them and reports its own death.
- **School-friendly rooms:** if the live (websocket) connection is blocked, CREATE ROOM switches by itself to a slower connection made of ordinary requests (through the same `/sb` proxy that logins use), and there is also a manual **SCHOOL-FRIENDLY ROOM** button. Those room codes start with a digit; a code starting with a letter is a live room. It needs a `duel_msgs` table in Supabase (SQL below in the setup notes); about 7 updates a second instead of ~12, so it feels laggier.
- Duels do not touch coins, battle pass XP, ranked points or career stats. Pausing (Esc) does not pause the duel and a paused player can still be hit.
- Known limits: a modified client could cheat (it is peer to peer; a server would be needed to prevent that), the opponent is shown as a coloured bot rather than their skin, and networks that block websockets to supabase.co cannot play.

### Accounts, cloud saves and leaderboard
- Optional email + password accounts (Supabase Auth) with a username shown on the leaderboard. A login popup appears after the news popup with **Play as guest**; Settings has an ACCOUNT section with LOG OUT, and the nav shows LOG IN / your name.
- Coins, owned skins, equipped skin, battle pass (XP, claimed tiers, equipped items), career stats and ranked points are saved to the account a couple of seconds after every change (and when the tab closes), and loaded on login. If the connection drops, changes stay on the device and sync later. The first login adds the guest progress on this device to the account once; logging out returns the device to a clean guest.
- New **LEADERBOARD** tab: top 50 players by ranked points, showing each player's rank division and points, plus your own position. It needs a `rank_points` column on the Supabase `profiles` table; scores are reported by the browser, so a determined player can edit them (no server-side verification yet).
- Network friendliness: the sign-in library ships with the game (`supabase.min.js`, CDN fallback) and on a `*.netlify.app` address all account traffic goes through the same site at `/sb/*` (the `_redirects` file forwards it to Supabase), so school filters that block third-party hosts still allow login. The LOG IN button is always shown, with a clear message when the service cannot be reached.
- Passwords are handled by Supabase (hashed, never stored by the game); the page only contains the public anon key and row-level security limits each player to their own row. The game still works offline and with the sign-in library blocked.

### Locker
- New **LOCKER** tab (a Fortnite-inspired blue layout): five slots (outfit, back bling, glider, contrail, weapon wrap), an owned-item grid with search, a live 3D preview on the lobby character, EQUIP / EQUIPPED, NONE / DEFAULT options, drag to turn and a reset button, a rotatable weapon preview with a gun selector for wraps, and a Back-to-PLAY button. Previews never change the saved look until EQUIP is pressed; nothing costs coins. Retired skins you already own appear here.

### Skins and Item Shop arrangement
- **Raven Developer** (new, 25 coins): tan skin, curly black hair, purple glowing eyes (an unlit emissive-colour material), purple scarf, layered black armour, straps and pouches, a ring of raven-feather spikes over the shoulders, ragged coat panels, deep purple trousers, muted gold buckle, knee pads and strapped tall boots.
- **Sergeant Trubin** replaces the tracksuit design in place (same id, so owners keep it): camouflage fatigues from one small shared texture, olive tactical vest with pouches, thigh straps, knee pads, tall laced black boots, and a charcoal ushanka with a white-blue-red flag patch.
- Only Raven Developer and Sergeant Trubin are featured (two cards, stacking on narrow screens); Samurai Riley, Pablo and Luke are regular catalog items; Raven Developer costs 25 coins and the other four 10 coins each.

### little pablo
- New skin **little pablo** (1 coin, regular Item Shop catalog; Raven Developer and Sergeant Trubin stay the only featured skins): a thick pepperoni-pizza-slice head (extruded triangle, wide crust bar, pepperoni on both faces, cheese drips, drawn eyes, brows and smile), cheese-coloured arms, rainbow shirt and trousers from one small gradient texture, and white sneakers with laces. Same animated rig, Locker/shop previews and cosmetics as every skin. Simplified: no toasted-spot texture on the arms, and the rainbow repeats on each leg segment.

### Startup news popup
- The UPDATE / NEWS popup opens on every page load (nothing is stored) with a single page announcing the new **1v1 multiplayer** mode, with two snapshots (the walled Meadow Mall arena from above, and a first-person duel view). A line above the title tells players to use the new link, https://fortlite.netlify.app/. Buttons: **TRY 1V1 DUEL** (opens the duel screen, and skips the login popup for this visit) and **LET'S PLAY**. The earlier summary, featured-skins and follow pages were removed.

### Balance tweaks
- SMG bullet damage 9 -> 19, medkit heal 40 -> 50 HP, shockwave launches are higher (vertical speed up to 11.5 m/s, was 10). Nothing else about those items changed.

### Six corner POIs and a relocated graveyard
- **Meadow Town:** *Camp Evergreen* (NW corner, centre about -79.5, -79.5): shallow pond with a dock, ranger station and two cabins, six tents, campfire ring with log seats, picnic tables, fallen logs / boulders / clustered trees, and a 3.9 m ranger lookout with two ordinary stair flights (north and south), a dirt path east to the x = -55 road. *Sunny Acres* (NE corner, about 79.5, -79.5): red barn with three entrances and a stair-reached hayloft, farmhouse, greenhouse, static tractor, hay bales and carts, short crop rows with crossing gaps, fences with openings, paths west to the road and south toward Meadow Pitch.
- **Haunted Hollow:** *Withered Orchard* (NE corner, about 79.5, -82): crooked farmhouse with glowing windows, cider shed, roofed wrecked delivery truck, loose rows of twisted bare trees with crossing lanes, pumpkin patches, broken walls, carts, barrels, lantern posts (glowing boxes, no dynamic lights) and an 11 m scarecrow visible while gliding. *Blackwater Mill* (SW corner, about -79.5, 82): mossy timber warehouse with boarded windows, mill office, broken waterwheel, a lean 14 m chimney with a faint green band, a shallow walkable creek (plank bridge, stepping stones, log), loading platforms with steps, log stacks, racks and an abandoned logging truck.
- **Graveyard:** the Hollow graveyard (Crypt Hollow label, crypt, fence, gravestones, chests) moved from (-30, 30) to (-27.5, -78) - 34 units of open ground north of the Hollow Manor grounds (manor z -26 to graveyard z -60) - with a winding path, bare trees, boulders and broken wall pieces in between. Its objects, collision, chest spots, minimap label and location trigger moved together; the old spot is ordinary Hollow terrain again. Because the generic house scatter avoids the graveyard, Hollow's random houses were re-rolled.
- **Dustfall Canyon:** *Dead-End Drive-In* (NW corner, about -79.5, -79.5): a 20 x 13 m solid screen with a faded flat graphic, projector building, snack shack, ticket booth, three staggered rows of static wrecked cars (hoods and roofs can be stood on), low walls, fence openings and a way round either end of the screen. *Sunburn Solar* (south edge, about -10, 79.5): rows of matte solid stepped solar panels with open crossing lanes, control building, two sheds, a fenced comms mast with a slim pole (no climbable deck), parked wrecks and boulder cover, paths toward Scrap Valley and the mine. Dustfall has no free corner left (the other three hold Scrap Valley, Redrock Outpost and Oasis Springs), so Sunburn Solar sits on the empty south edge.
- Integration: each POI has 4-5 chest candidates of which 3 spawn per match (the per-map chest total is unchanged, so the rest of the map gives up the same number), landing targets and minimap labels, supply-drop exclusions, map-card art, nav-grid collision from the same solids you see, and no grapple, jetpack or shockwave is needed to reach any chest that bots can use (tower and loft chests are player-only, like the mall roofs).

### Pumpkin RPG
- New chest weapon on all three maps: **15% of chest openings** award it (rolled once when the chest opens, independently per chest); the other 85% is split equally between the assault rifle, shotgun and SMG exactly as before (28.3% each). Still one weapon plus one collectible per chest.
- Orange-and-black compact launcher with a jack-o'-lantern warhead that slides in on reload and disappears after firing; the rocket is a glowing jack-o'-lantern with a short spark trail. Works in first and third person with every skin and wrap, has a hotbar icon, `1/2` ammo display, its own launch sound, and kill-feed icon.
- Numbers: 1 rocket per magazine + 3 spare (4 in total), 3.2 s reload, rocket speed 24 m/s (dodgeable), 3.2 s lifetime, 5.5 m blast radius, a flat 120 damage to everyone inside it (centre or edge), 1 s between shots. Explodes once on ground, any solid or a contestant; solid cover blocks splash; you take damage from your own blast. Brief blast flash, particles and a small camera shake; buildings are not damaged.
- Bots can use it through the normal weapon system (rating equal to the SMG / shotgun): they never fire it closer than 5 m, switch away when it is empty, and aim with the same error as for bullets. Dropped RPGs keep their loaded rocket. Everything is cleared on match end.

## v1.2

### New skins (Item Shop)
- Four new playable skins at 10 coins each, featured as four cards at the top of the Item Shop (2x2 on narrow screens): **Samurai Riley** (red/gold lacquered armour over navy: cuirass with gold dragon detail, layered shoulder plates, forearm and shin guards, waist armour, dark scarf, red belt ties, messy black hair, cosmetic sheathed sword that hides when a back bling is equipped), **Pablo** (sky-blue and white striped jersey with 10 on the front and PABLO / 10 on the back, navy shorts and socks with light-blue bands, red/pink cleats, white left wrist wrap, bleached hair with dark roots), **Trubin** (black track jacket and pants with white triple stripes, zip, small logo, black/white sneakers, big ushanka with tan fur, ear flaps and a small flag patch) and **Luke** (all-black tactical: high collar, crossing chest straps and rig, utility belt, thigh pouches, knee pads, gloves, boots, curly light-brown hair).
- They are the same animated model as every other skin (all poses, reloads, gliding, first-person arms), with outfit pieces parented to the existing joints and merged to one mesh per colour. Hitboxes, speed and abilities are identical. Only the player's own model carries the extra pieces (about +5 to +21 draw calls and +1-2k triangles in third person).
- Crimson, Arctic, Jungle, Midnight and Golden are retired: not shown in any shop list unless you own them (then they appear as OWNED and can be equipped), and the purchase handler rejects them. Existing coins, owned skins and equipped skins are untouched.

### Item Shop and Battle Pass (lobby navigation)
- The lobby tabs are now PLAY, ITEM SHOP, BATTLE PASS, HOW TO PLAY and SETTINGS. The Loadout tab/page was removed and SKINS became the **ITEM SHOP**, which sells skins only: three large FEATURED skins (Midnight 5, Golden 10, Arctic 2), a skin catalog (ALL SKINS / OWNED ONLY), free previews on the lobby character and a sticky BUY / EQUIP / EQUIPPED bar. Skin prices, ownership and saves are unchanged.
- The Battle Pass grows from 20 to **30 tiers** (existing tier ids, XP and claims are kept; 150 XP per tier). Player banners were removed and replaced by 12 new cosmetics: weapon wraps Crimson Stripe (tier 1), Arctic Blue (21), Golden Touch (27); gliders Sunset Wing (22), Desert Patrol (25), Batwing (28); contrails Sky Streak (8), Ember Drop (23), Phantom Trail (26); back blings Trail Pack (16), Patch Pumpkin Pack (24), Mini Bot (29); tier 30 is the Haunted Legend title. Claim, equip and "use default" work as for other pass rewards; one item per category.
- Contrails are a new pass category; gliders and contrails preview as a descent view on the Battle Pass screen. In matches wraps apply to held, third-person and dropped player weapons, gliders replace the descent look (hands on the handles), contrails are a bounded 26-puff trail emitted only while gliding (gone about a second after landing), and blings attach to the back.
- Migration: coins spent on the short-lived coin-bought wraps / gliders / contrails / blings are refunded once on load (shown as a toast); everything else in `br_progress` and `br_pass` is kept.

### Character models
- Players and bots share a detailed, rounded humanoid (head with jaw, nose, eyes, brows and hair, shaped torso, hips and belt, two-segment limbs with elbows, knees, wrists and ankles, hands with thumbs, boots with soles, pockets and knee pads). Skeleton and Pumpkin King skins, wraps, back blings and gliders attach to it. Distant bots swap to a lighter model; hitboxes, speed and weapon stats are unchanged. Animation is the existing procedural pose / IK system (no imported animation clips).

### Graphics refresh (added to v1.2)
- Brighter, more saturated colours (Meadow Town grass, sky, trees and house palette; clearer Dustfall sky) and warmer sunlight, with no change to geometry, lighting model, shadows or draw calls.
- Characters: slightly bigger head, hands, shoulders and boots, plus shoulder armour, a chest strap and knee pads (bots draw them in the same single mesh; no extra bot draw calls).

### New points of interest (added to v1.2)
- **Meadow Mall** (Meadow Town, map centre): two single-storey wings round an open courtyard with four stores (Market, Style, Sports, Café), awnings, signs, planters, kiosks, benches, a parking lot and pylon signs. Roof stairs on each wing, 8 chest spots (6 normal + 2 on the roofs; 7 are used per match), four approaches. Replaces the random buildings that stood in the middle; the main roads end at the mall.
- **Meadow Pitch** (Meadow Town, east of the mall): a 40 x 24 soccer field (lines, centre circle, penalty areas), two goals with see-through nets, bleachers, team benches with roofs, a scoreboard, a decorative ball, a perimeter fence with several openings and an equipment shed with a chest. Labelled on the minimap.
- **Hollow Manor** (Haunted Hollow, centre-east, next to the graveyard): a two-floor mansion with a front porch and balcony, a grand staircase, outside balcony stairs, kitchen wing, a cellar under the rear garden (two stair-ramps, so it has an entrance and an exit), pitched roofs, chimneys, a fenced courtyard and garden with gates, pumpkins, gravestones and bare trees. 10 chest spots (7 used per match). Bots use the stairs to loot upstairs.
- **Dustfall Speedway** (Dustfall Canyon, north of the mine's shaft): an irregular dirt circuit with a chequered start line, sign gantry, tire barriers, an open pit garage with race cars (props only), bleachers, a timing board, a low dirt jump and 3 chests. No vehicles or racing mode.
- The final safe zone on Meadow Town and Haunted Hollow now drifts to a random spot instead of always ending in the centre. Landings favour the new centres (some bots land there, others spread out). The total number of chests per match is unchanged (the new POIs take their chests from the rest of the map).

### Victory Royale (added to v1.2)
- Winning now plays a Victory Royale sequence: a blue/cyan "#1 VICTORY ROYALE" banner slides in over the frozen final scene with a short fanfare, a brief golden glow and a 36-piece confetti burst. After about two seconds a results panel fades in (placement of the real starting field, eliminations, damage dealt, match time, coins, Battle Pass XP with breakdown, and ranked change in Ranked), then PLAY AGAIN and RETURN TO LOBBY appear. Reduced-motion users get a simple fade and no confetti.
- Damage dealt is now tracked: shield plus health actually removed from contestants (no overkill, no storm / environment).
- A match can only end once, so repeated completion calls never repeat rewards, stats or the sequence. Rewards are still saved at the moment of the win; if browser storage is unavailable the panel says so. Defeat and draw screens are unchanged.

### Other v1.2 changes
- Ranked entry fees: Champion 25 coins, Legendary 30 coins.
- Meadow Mall: the red "MEADOW MALL" roof signs and the two parking-entrance pylon signs (with their pylons and posts) were removed; the store signs stay.
- Chest / loot rebalance and a fix for swapping the held item when the inventory is full.
- Losing now shows its own results sequence (YOU PLACED #X in a restrained defeat style).

## v1.1

### New map: Dustfall Canyon
- A third battle-royale map: a desert town built around Redrock Mine, a shallow (3 m) quarry with an abandoned quarry with a real lower floor, stair-ramps, a road bridge and cliff walls. Built in the existing low-poly building, tree, prop and weapon style.
- Five named locations: **Dusty Junction** (central town: motel, gas station with a roadside dinosaur, mechanic's garage, houses with roof stairs, water tower), **Scrap Valley** (rusty car rows, enterable shipping containers, tyre stacks, crusher), **Redrock Outpost** (fenced compound, warehouse, barracks, a modest watchtower), **Oasis Springs** (shallow walkable pond, palms, colourful courtyards) and **Redrock Mine** (an irregular 3 m deep quarry under the road bridge: four stair-ramps, two rock benches, rock formations, carts on short rails, and a timber-framed mine entrance alcove with loot; an abandoned bus hangs over its edge).
- Other landmarks: crashed supply plane near the east edge, signs, distant decorative mesas outside the playable boundary.
- Maps now rotate Meadow Town → Haunted Hollow → Dustfall Canyon (existing saves keep working).
- Bots navigate ramps, the bridge deck (a second walking layer) and the canyon floor; supply drops, chests and bot destinations are only placed on reachable ground.

### Ranked bots rebalanced (about 30% less combat effectiveness in every division)
- Aim error retuned per division (measured, not randomly discarded hits). Champion and Legendary no longer have perfect aim; their reaction times and aim-tracking speed were also eased. The difficulty order Bronze < Silver < Gold < Platinum < Diamond < Champion < Legendary is preserved.
- No change to weapon damage, bot health, ranked points, thresholds, entry costs or rewards.

### v1.1 changes
- Movement: jump height 1.7 m, standing on cars / crates / roofs, mantling ledges up to ~2 m, crouch (C).
- Third-person camera (V / Settings) with obstruction pull-in.
- Sniper scope toggle (right-click or Q).
- Minigun buff (+15% damage, -30% spin-up, -20% spread).
- Grapple gun and Shockwave grenade (rare chest finds).
- Ranked: Legendary division at 5,000 points (entry fee 30), Champion entry fee 25, per-division bot profiles.
- Incoming-damage feedback: blue / red edge pulses, hit-direction wedges, trailing bars, shield-break cue.

### Third-person gliding and shotgun reload (added to v1.1)
- Gliding works in both camera views and respects the saved preference; V switches views while gliding. In third person the character hangs below the equipped glider (a striped canopy, or the Bat Wings) with both hands on its grips, and the camera sits behind and above so the landing area stays visible.
- The shotgun reloads one shell at a time: the empty-to-full time is unchanged (2.4 s, split over the 5 shells), each shell is added to the magazine the moment it is inserted, and the weapon can fire as soon as one shell is loaded (firing cancels the rest of the reload). Switching, dropping, dying or leaving a match cancels cleanly and keeps the shells already loaded. Bots reload the same way.

