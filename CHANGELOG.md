# CHANGELOG

## v2.1.0

### Addon Changes
- [[addon/#26]](https://github.com/OCAP2/addon/pull/26) Optimize fn_getUnitType
- [[addon/#27]](https://github.com/OCAP2/addon/pull/27) Add option for manual capture start
- [[addon/#28]](https://github.com/OCAP2/addon/pull/28) Support onLoadName for mission names
- [[addon/#33]](https://github.com/OCAP2/addon/pull/33) OCAP 2.0: Modular architecture with database export
- [[addon/#34]](https://github.com/OCAP2/addon/pull/34) Fix MarkerUpdated exclusion check always excluding all markers
- [[addon/#35]](https://github.com/OCAP2/addon/pull/35) Remove erroneous PREP(sendData) from database addon
- [[addon/#36]](https://github.com/OCAP2/addon/pull/36) Fix callExtension error by passing empty array instead of nil
- [[addon/#37]](https://github.com/OCAP2/addon/pull/37) Fix marker side format to use string instead of numeric ID
- [[addon/#38]](https://github.com/OCAP2/addon/pull/38) Fix HEMTT lint warnings
- [[addon/#39]](https://github.com/OCAP2/addon/pull/39) Add GitHub Actions workflow for build validation
- [[addon/#42]](https://github.com/OCAP2/addon/pull/42) Refactor ACE explosives to use Explode event handler
- [[addon/#43]](https://github.com/OCAP2/addon/pull/43) Refactor projectile tracking to use lightweight array format
- [[addon/#44]](https://github.com/OCAP2/addon/pull/44) Compatibility with Restrict Markers
- [[addon/#45]](https://github.com/OCAP2/addon/pull/45) Add magazine icon path to projectile data
- [[addon/#46]](https://github.com/OCAP2/addon/pull/46) Rename licence.txt to LICENSE for GitHub detection
- [[addon/#47]](https://github.com/OCAP2/addon/pull/47) Fix re-recording after export losing data (missing NEW:MISSION)
- [[addon/#48]](https://github.com/OCAP2/addon/pull/48) Fix vehicle tracking: ejection seats, disappeared vehicles, parachute lifetime
- [[addon/#49]](https://github.com/OCAP2/addon/pull/49) Filter parachute/ejection seat kill events and fix dead parachute tracking
- [[addon/#50]](https://github.com/OCAP2/addon/pull/50) Remove duplicate mission_data/time metric
- [[addon/#51]](https://github.com/OCAP2/addon/pull/51) Fix extension VERSION parsing for new 3-element response
- [[addon/#52]](https://github.com/OCAP2/addon/pull/52) Grant admin diary controls to already-connected players
- [[addon/#53]](https://github.com/OCAP2/addon/pull/53) Resume tracking units that become player-controlled after disconnect
- [[addon/#54]](https://github.com/OCAP2/addon/pull/54) Skip kill event for disconnected players
- [[addon/#55]](https://github.com/OCAP2/addon/pull/55) Track groupID and side dynamically per unit state update
- [[addon/#56]](https://github.com/OCAP2/addon/pull/56) Convert owner ID to string for getUserInfo in admin UI init
- [[addon/#57]](https://github.com/OCAP2/addon/pull/57) Vehicle weapon tracking, projectile system overhaul
- [[addon/#58]](https://github.com/OCAP2/addon/pull/58) Remove global toFixed calls that interfere with TFAR radios
- [[addon/#59]](https://github.com/OCAP2/addon/pull/59) Remove undefined _hitThings reference in HitPart EH
- [[addon/#60]](https://github.com/OCAP2/addon/pull/60) Unify :FPS: and :METRIC: into single :TELEMETRY: command
- [[addon/#61]](https://github.com/OCAP2/addon/pull/61) Broadcast OCAP_id to clients for dedicated server projectile tracking
- [[addon/#62]](https://github.com/OCAP2/addon/pull/62) Stop telemetry loop when recording is inactive
- [[addon/#63]](https://github.com/OCAP2/addon/pull/63) Gate extension call logging behind debug mode
- [[addon/#64]](https://github.com/OCAP2/addon/pull/64) Resolve adminUIcontrol race condition during postInit
- [[addon/#65]](https://github.com/OCAP2/addon/pull/65) Fix diary status never updating past initial values
- [[addon/#66]](https://github.com/OCAP2/addon/pull/66) Broadcast lastFired to server for correct kill weapon attribution
- [[addon/#67]](https://github.com/OCAP2/addon/pull/67) Fix PFH interval variables not accessible due to private scoping
- [[addon/#68]](https://github.com/OCAP2/addon/pull/68) Use HandleDamage for explosive kill weapon attribution

### Extension Changes (v5.0.0)
Major rewrite from C++ to Go with significant architectural changes:
- Complete migration to Go-based recorder with database storage
- Refactored architecture with pluggable storage backends
- Replaced channel-based architecture with event dispatcher
- Migrated from zerolog to slog with OpenTelemetry
- Added WebSocket streaming storage backend
- SQLite storage backend implementation
- Added POLYLINE marker support
- Projectile marker support for grenades and smokes
- Structured weapon arrays and derived hit events
- Dynamic groupID and side tracking in SoldierState
- JSON export format aligned with OCAP2 web expectations
- Comprehensive test coverage improvements
- Multi-platform build support (Windows x64, Linux x64)

### Web Changes
- Large recording playback with chunked streaming
- Refactored to standard Go project layout
- Added admin UI with auth, CRUD operations, and upload
- Steam profile display and role-based authentication
- Canvas-based entity renderer for improved performance
- Render all firelines per frame instead of only the first
- Redesigned player transport controls
- Activity heatmap timeline with vertical playhead
- Admin-defined focus range for recordings
- Vehicle kills column in scoreboard leaderboard
- Migrated from Echo v4 to Fuego with OpenAPI/Swagger support
- Render projectiles on canvas instead of Leaflet DOM markers
- Apply marker type/color/brush changes during playback
- Match Arma 3 coordinate grid pattern
- Eliminate grid wobble during zoom animation
- Add Ukrainian (українська) UI translation
- Add Finnish (Suomi) UI translation
- Add Italian translation
- Add Czech localization
- Internationalize map manager UI
- Multiple dependency updates and CI/CD improvements

## v1.1.0

### Fixes

#### Addon
- [[addon/#25]](https://github.com/OCAP2/addon/pull/25) Fixes incorrect color for "ColorUNKNOWN" in playback
- [[OCAP/#32]](https://github.com/OCAP2/OCAP/issues/32) Adds submunition ammo type handling for projectile tracking
- [[OCAP/#33]](https://github.com/OCAP2/OCAP/issues/33) Marker creation time was displayed ~2sec later during playback
- [[OCAP/#36]](https://github.com/OCAP2/OCAP/issues/36) Grenades, any other thrown object and fire lines were not shown after unit respawn

#### Web
- [[OCAP/#14]](https://github.com/OCAP2/OCAP/issues/14) Prevent markers from appearing if they should not be visible
- [[web/#21]](https://github.com/OCAP2/web/pull/21) Fixed group list is not filtered on first load

### Changes

#### Addon
- [[addon/#25]](https://github.com/OCAP2/addon/pull/25)
	- Adds support for "isKindOf" object recording exclusion
	- Checks for ACE modules before running monitors that depend on them
	- Positions now tracked in ASL for future surprises :)
	- Addon and extension versions now recorded in JSON
	- Adds descriptions to userconfig/config.hpp settings
- [[OCAP/#45]](https://github.com/OCAP2/OCAP/issues/45) Adds diary entry in-game showing about and status of addon, plus versions
- [[OCAP/#34]](https://github.com/OCAP2/OCAP/issues/34) Adds support for ArmA3 unconscious state
- [[OCAP/#38]](https://github.com/OCAP2/OCAP/issues/38) Disconnect now hides the controlled unit
- [[OCAP/#39]](https://github.com/OCAP2/OCAP/issues/39) Hit/Killed events now brought into parity & sharing more accurate tracking
- [[addon/#17]](https://github.com/OCAP2/addon/pull/17) Reduce network traffic by checking configured marker exclusions clientside
- [[addon/#22]](https://github.com/OCAP2/addon/pull/22) Projectile markers now reflect the direction of travel
- [[web/#29]](https://github.com/OCAP2/web/pull/29) Show kill reason as disconnect, when disconnected at the same time

#### Web
- [[OCAP/#42]](https://github.com/OCAP2/OCAP/issues/42) Adds CUP Maps markers
- [[OCAP/#41]](https://github.com/OCAP2/OCAP/issues/41) Adds 3CB Factions markers
- [[OCAP/#46]](https://github.com/OCAP2/OCAP/issues/46) Adds ability to hide killcount info during playback
- [[OCAP/#28]](https://github.com/OCAP2/OCAP/issues/25) Updates 'share' button icon
- [[OCAP/#8]](https://github.com/OCAP2/OCAP/issues/8) Adds ability to switch between time elapsed, in-world time, and system time, if available
- [[OCAP/#7]](https://github.com/OCAP2/OCAP/issues/7) Adds support for new custom event [`capture`](https://github.com/OCAP2/OCAP/wiki/Custom-Game-Events#captured-captured)
- [[OCAP/#28]](https://github.com/OCAP2/OCAP/issues/27) Re adds customizing for own logo and link, like as it existed in the old OCAP
- [[OCAP/#12]](https://github.com/OCAP2/OCAP/issues/12) Killing units of the same side will now decrease kill count
- [[web/#18]](https://github.com/OCAP2/web/pull/18) Increases left panel width for recent role addition
- [[web/#33]](https://github.com/OCAP2/web/pull/33) Enables caching for static marker icons & map tiles

## [v1.0.0](https://github.com/OCAP2/OCAP/releases/tag/v1.0.0)

### Fixes
- [[addon/#3]](https://github.com/OCAP2/addon/pull/3) respawned units no longer 'flicker' after respawning when corpse + new unit have same ocapID
- [[addon/#1]](https://github.com/OCAP2/addon/pull/1) changes 'side' tracking methodology to maintain tracked side of a player post-respawn
- [[OCAP/#17]](https://github.com/OCAP2/OCAP/issues/17) Fixed zoom parameter were ignored in share link
- adds http:// prefix to links generated using Share button

### Changes

#### Platform
- improvements to both extension and webserver builds
- now supports Windows x64 & Linux x64
- adds tag param in SQF export function allowing you to define the recording's 'tag' at mission end
- removes hardcoded 'tag' definitions in web/option.json, allowing you to use any tag for missions and use them to filter by in playback selection

#### Markers
- adapted for use with A3 vanilla marker system -- SWT will be reintegrated in future [[addon/#5]](https://github.com/OCAP2/addon/issues/5)
- now tracks RECTANGLE, ELLIPSE, and POLYLINE markers correctly, including those made via scripted commands
- now tracks direction, size, alpha (opacity), and markerBrush (for area markers) for more accurate display
- adds base game, RHS, ACE, IFA map marker icons
	- future plans to integrate VK marker icons
- integrated support for BIS_fnc_moduleCoverMap usage
- backwards compatibility maintained for older recordings

#### Projectile Tracking
- now tracks non-bullet projectiles, ACE throwing, and ACE mine (place > arm > detonate)
- adds base game, CUP Weapons, RHS, ACE, IFA projectile images in both default & IFA/FOW compatible colors

#### Maps
- base maps removed, available in external GDrive
- terrain generation automation improved -- looking to change render method which will make this obsolete

#### Web
- adds 'type' tag column in playback selection
- adds total kill count to players in ORBAT column at left
- adds tracking of selected-weapon of killer when in a vehicle (events column)
- [[OCAP/#4]](https://github.com/OCAP2/OCAP/issues/4) show role information as prefix before the players name
