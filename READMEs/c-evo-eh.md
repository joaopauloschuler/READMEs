
# C-evo-eh-code Project Documentation

## Project Overview

C-evo is a turn-based strategy game engine inspired by Civilization. This codebase represents the core engine and a reference graphical user interface (GUI) implementation. It simulates the development of a civilization from ancient times to the space age, including city management, unit operations, technology research, diplomacy, and combat on a procedurally generated world map.

The architecture is structured around a central game server (`GameServer.pas`) which uses the game state managed by `Database.pas`. The rules, data formats, and communication commands are defined in `Protocol.pas`, which acts as the common language. Player actions and game events are logged and managed by `CmdList.pas`. Modules like `CityProcessing.pas` and `UnitProcessing.pas` contain the core logic for handling city and unit actions during a turn, interacting directly with the game state via `Database.pas` and using the definitions in `Protocol.pas`. Clients (like the `LocalPlayer` GUI or the `NoTerm` console client) communicate with the `GameServer` by sending commands defined in `Protocol.pas` and receiving updated game state or notifications. The `LocalPlayer`'s interaction is often mediated by `ClientTools.pas`, which provides a simplified interface to query game data and send commands to the server.

The overall system architecture follows a client-server model. The core server components (`GameServer.pas`, `Database.pas`, `Protocol.pas`, `CmdList.pas`) manage the game state and logic. Clients (like the graphical `LocalPlayer` or the headless `NoTerm`) communicate with the server using the command structure and data formats defined in `Protocol.pas`. The graphical client utilizes `ClientTools.pas` as an intermediary to interact with the server and manage the various UI modules.

Conceptually:
Server Core <--> Client Tools <--> GUI/UI Modules

## Architecture and Core Modules

*   **Protocol (`Protocol.pas`)**: This fundamental unit defines the data structures (`TUn`, `TCity`, `TModel`, `TPlayerContext`, `TOffer`, etc.) and constants used throughout the engine. It specifies the format for game state data, unit and city properties, technology details, improvement types, terrain characteristics, diplomatic states, and the commands used for communication between the server and clients. It acts as the common language of the system. Server commands are defined as constants (e.g., `sMessage`, `sSetDebugMap`) and often include information encoded in the constant value itself, such as command type (`sctMask`) or whether execution is requested (`sExecute`). Client commands (e.g., `cInitModule`, `cLoadGame`) follow a similar pattern. Communication typically involves calling procedures like `TServerCall` (client to server) or `TClientCall` (server to client) with the command and a variable `Data` parameter whose structure and size depend on the specific command, facilitating the transfer of complex data.

*   **Database (`Database.pas`)**: This module manages the global game state in memory. Key global variables include `RealMap` (an array storing tile terrain types, improvements, and other flags), `Continent` (tile continent IDs), `Occupant` (tile ownership/occupancy), and `RW` (an array of pointers, `^TPlayerContext`, representing the state of each potential player slot). It provides core procedures like `InitRandomGame` for starting a new game world, `CreateUnit` for adding entities to the world, and `GetTileInfo` for querying map details based on a given location (`Loc`). The `Loc` parameter, used extensively for map interaction, is typically an integer encoding 2D coordinates (likely `y * lx + x`), as functions like `dLoc` and `dxdy` manipulate locations based on `lx` (map width) and relative offsets. It relies heavily on the definitions in `Protocol.pas`.

*   **Command List (`CmdList.pas`)**: This unit implements a system for logging and managing game commands (`TCmdList`). It appears to be used for saving, loading, and potentially replaying game states or sequences of player actions by storing commands and associated data changes in the `LogData` array. Functions allow putting new commands (`Put`, `PutDataChanges`), retrieving commands (`Get`, `GetDataChanges`), and managing the command list via file streams (`LoadFromFile`, `SaveToFile`, `AppendToFile`). This structure is central to the game's turn processing and save/load functionality.

*   **Game Server (`GameServer.pas`)**: This is the main server-side execution unit. It orchestrates the game flow, handles game initialization (`StartNewGame`, `LoadGame`, `EditMap`), processes turn transitions (`NextPlayer`), and manages connected AI clients by storing their information in the `Brain` array (of type `TBrainInfo`). It defines constants related to versioning, notifications (`nt...`), and AI brain management (`maxBrain`, `bix...`). It uses the `Protocol.pas` and `Database.pas` units to access and modify the game state, and communicates with clients by invoking their `TClientCall` procedures.

*   **City Processing (`CityProcessing.pas`)**: This module encapsulates the logic related to city operations. It handles processes like `CityTurn` for processing a city's actions each turn, `CityGrowth` for managing population changes, and `GetCityReportNew` for generating detailed status reports based on the `TCityReportNew` structure. Procedures like `SetCityTiles` manage the tiles being exploited by a city, and it interacts directly with game data structures defined in `Protocol.pas` and managed by `Database.pas`.

*   **Unit Processing (`UnitProcessing.pas`)**: This module contains the logic for unit operations. Key procedures here include `CalculateMove` for determining movement paths and costs based on terrain and unit properties, `GetBattleForecast` for simulating combat outcomes using structures like `TBattleForecastEx`, and `StartJob` for initiating terrain improvement tasks by settler units. It interacts with the game state via `Database.pas` and uses data structures from `Protocol.pas`.

## User Interface (LocalPlayer) Modules

The `LocalPlayer` directory contains units responsible for the graphical interface used by a human player on the local machine.

*   **Base Windows (`BaseWin.pas`)**: Provides base classes for graphical dialogs (`TDrawDlg`, `TBufferedDrawDlg`, `TFramedDlg`), implementing fundamental drawing, buffering, and window management features like staying on top and handling resize. `TBufferedDrawDlg` uses an `Offscreen` bitmap for smoother rendering.

*   **Main Screen (`Term.pas`)**: This is the central game screen (`TMainScreen`) for the local player. It displays the game map using the `IsoEngine`, handles user input (mouse clicks, keyboard shortcuts), manages the main menu and various sub-dialogs (city screen, unit info, reports), controls unit movement and actions, and provides the End-of-Turn (`EOT`) button (`TEOTButton`). It coordinates interactions between the user and the game state by calling functions in `ClientTools.pas`, interpreting server notifications (`Client` procedure), and invoking server commands (`DipCall`, `OfferCall`). Entities like `TUn` and `TCity` have `Status` fields that the client can use (as indicated by comments in `Protocol.pas`). The `Flags` field in `TUn` and `TCity` uses constants like `us...` and `ch...` respectively (defined in `Protocol.pas`) to indicate specific states or recent events, which the UI uses for display (e.g., `unFortified`, `chDisorder`).

*   **Client Tools (`ClientTools.pas`)**: Provides helper functions for the local client to interact with the server via the `Server: TServerCall` interface. It offers simplified access to player-specific game data through pointers like `MyRO` (`^TPlayerContext`), `MyMap` (`^TTileList`), `MyUn` (`^TUnList`), `MyCity` (`^TCityList`), and `MyModel` (`^TModelList`). It includes utility functions for calculating movement advice (`GetMoveAdvice`), checking unit status (`UnitExhausted`), processing enhancements (`ProcessEnhancement`), and managing city automation (`AutoBuild`). It acts as an intermediary layer between the GUI logic and the core game server commands/data.

*   **Isometric Engine (`IsoEngine.pas`)**: This module handles the drawing and rendering of the game world in an isometric perspective onto a `TBitmap`. It paints terrain tiles, units, and cities using data from `ClientTools.pas`. It manages map options (`Options` integer, using `mo...` constants), visualizes battle effects (`AttackBegin`, `AttackEffect`, `AttackEnd`), and uses graphic resources managed by `ScreenTools.pas` and player-specific graphics from `Tribes.pas`.

*   **Dialogs**: Several specialized dialog units (`Messg.pas`, `MessgEx.pas`, `Inp.pas`, `CityScreen.pas`, `CityType.pas`, `Draft.pas`, `Enhance.pas`, `Help.pas`, `NatStat.pas`, `Nego.pas`, `Rates.pas`, `Select.pas`, `TechTree.pas`, `UnitStat.pas`, `Wonders.pas`) provide interfaces for specific game interactions like showing messages (`TMessgDlg`, `TMessgExDlg`), getting input (`TInputDlg`), managing cities (`TCityDlg`, `TCityTypeDlg`), configuring units (`TDraftDlg`, `TEnhanceDlg`), viewing statistics and reports (`TNatStatDlg` for `TEnemyReport`, `TUnitStatDlg` for `TUnitInfo`/`TModelInfo`), handling diplomacy (`TNegoDlg` using `TOffer`), and browsing the help system (`THelpDlg`) or tech tree (`TTechTreeDlg`). These dialogs interact with the game state through `ClientTools.pas`.

*   **Controls**: Units like `ButtonBase.pas`, `ButtonA.pas`, `ButtonB.pas`, `ButtonC.pas`, `ButtonN.pas`, and `EOTButton.pas` define custom graphical buttons used throughout the UI, often extending `TGraphicControl` and implementing custom drawing (`Paint` procedures). `Area.pas` defines clickable regions without visual representation, used in conjunction with custom drawing surfaces. `PVSB.pas` provides a custom implementation for scrollbars (`TPVScrollbar`) that interfaces with Windows messages (`WM_VSCROLL`, `WM_MOUSEWHEEL`).

*   **Tribes (`Tribes.pas`)**: This unit manages player-specific visual assets and naming conventions. It loads and provides access to unit pictures (`ModelPicture`), city pictures (`CityPicture`), player colors (`Color`), city names (`CityName`), and model names (`ModelName`) from files. These are used by the `IsoEngine.pas` and various dialogs. It interacts with the game data via structures and constants defined in `Protocol.pas` and uses graphic loading functions from `ScreenTools.pas`.

*   **Screen Tools (`ScreenTools.pas`)**: A collection of low-level graphical and utility functions. This includes changing screen resolution (`ChangeResolution`), playing sounds (`Play`), loading graphics (`LoadGraphicFile`, `LoadGraphicSet`), drawing various graphical elements (frames, gradients, text with icons, progress bars), and converting game turns to historical years (`turntoyear`). It depends on `Directories.pas` and `Sound.pas`. Graphics are often managed via handles (`HGr...`) obtained from `LoadGraphicSet`, referencing structures like `TGrExtDescr`.

*   **Directories (`Directories.pas`)**: A simple unit for managing the game's home (`HomeDir`) and data (`DataDir`) directory paths, including localized file paths (`LocalizedFilePath`).

*   **Sound (`Sound.pas`)**: Provides basic sound playing functionality (`PlaySound`, `PrepareSound`) interacting with Windows multimedia APIs (e.g., `MMSystem`).

## Other Modules

*   **NoTerm (`NoTerm.pas`)**: An alternative client implementation, possibly used for running games without the full graphical interface (e.g., for AI vs AI simulations or testing). It provides a command-line-like interaction or simplified display, implementing the `Client` procedure to receive server updates.

*   **Log (`Log.pas`)**: A dialog (`TLogDlg`) for displaying game log messages, useful for debugging and monitoring game events. It uses a `TMemo` to display text.

*   **Direct (`Direct.pas`)**: A dialog (`TDirectDlg`) that appears to handle direct game flow control messages (`WM_GO`, `WM_CHANGECLIENT`, `WM_NEXTPLAYER`), possibly used for debugging or specific game modes like hand-over control in multiplayer.

*   **IPQ (`IPQ.pas`)**: An Integer Priority Queue implementation (`TIPQ`), likely a general-purpose data structure used within the engine, possibly for pathfinding or managing events/tasks based on priority. It uses pointer arrays (`bh`, `Ix`) to manage the heap structure.

*   **String Tables (`StringTables.pas`)**: A utility class (`TStringTable`) for loading and looking up strings from text files (like `.txt` language files). Stores lines of text in arrays of pointers (`Lines`) pointing into a large character buffer (`Data`). Provides the `Lookup` function to retrieve strings by item name and optional index.

## Key Components (Classes and Structures)

Based on the Pascal interface definitions, here are some of the critical data structures and classes that form the backbone of the game state and logic, with expanded descriptions:

*   **`TPlayerContext` (`Protocol.pas`)**: Represents the complete state of a single player. This record holds pointers to lists of the player's units (`Un: ^TUnList`), cities (`City: ^TCityList`), and developed unit models (`Model: ^TModelList`). It also includes dynamic arrays/lists holding the player's knowledge of other players' units (`EnemyUn: ^TEnemyUnList`), cities (`EnemyCity: ^TEnemyCityList`), and models (`EnemyModel: ^TEnemyModelList`), as well as diplomatic reports (`EnemyReport: array[0..nPl-1] of ^TEnemyReport`). Essential fields include `Turn` (current turn for this player), `Alive` (bitset indicating which players are still in the game), `Happened` (flags using `ph...` constants indicating events during the last turn), `Government`, `Money`, `TaxRate`, `LuxRate`, `Research`, `ResearchTech`, an array of technology status indicators (`Tech: array of ShortInt` using `ts...` constants), diplomatic relations (`Attitude`, `Treaty` using `at...` and `tr...` constants), `Wonder` (array of `TWonderInfo`), `Ship` (array of `TShipInfo`), `NatBuilt` (array indicating built national improvements), and `BattleHistory` (`^TBattleList`). It's the central repository for all player-specific dynamic information.

*   **`TUn` (`Protocol.pas`)**: Represents a single unit in the game world. Key fields include its map `Loc` (an integer location code), `Status` (a `LongInt` often used for AI or client-specific state), `ID` (a word, unique per nation), `mix` (index of the unit's model), `Home` (index of home city, -1 if none), `Master` (index of transporting unit, -1 if none), `Movement` (remaining movement points, SmallInt), `Health` (ShortInt, 100-Damage), `Fuel` (ShortInt), `Job` (current terrain improvement job using `j...` constants), `Exp` (micro experience), `TroopLoad` (ground units transported), `AirLoad` (air units transported), and `Flags` (Cardinal, using `un...` constants like `unFortified`, `unConscripts`, `unWithdrawn`).

*   **`TCity` (`Protocol.pas`)**: Represents a single city. Key fields include its map `Loc` (integer location code), `Status` (LongInt for client/AI use), `ID` (word, unique game-wide, encoding founder player and city number), `Size` (word), `Project` (current production project, encoding model/improvement index and flags like `cpConscripts`, `cpImp`), `Food`, `Pollution`, `Prod`, `Prod0` (collected resources and production points, SmallInt), `Flags` (Cardinal, using `ch...` constants for events like `chDisorder`, `chProduction`, `chPollution`, `chFounded`), `Tiles` (Cardinal bitset indicating exploited tiles relative to the city center), and `Built` (array of ShortInt indicating built improvements using `im...` constants).

*   **`TModel` (`Protocol.pas`)**: Represents a unit model (design) developed by a player. Key fields include `Status` (LongInt for client/AI use), `ID` (word, unique game-wide, encoding developing player and model number), `IntroTurn`, `Built` (units built), `Lost` (units lost in combat), `Kind` (Byte, e.g., `mkSelfDeveloped`, `mkSpecial_TownGuard`), `Domain` (Byte, `dGround`, `dSea`, `dAir`), combat stats (`Attack`, `Defense`, Speed, Cost, word), production multipliers (`MStrength`, `MTrans`, `MCost`, word/Byte), `Weight`, `MaxWeight` (Byte, related to construction time), `Upgrades` (Cardinal bitarray), `Flags` (Cardinal, using `ms...` constants like `msObsolete`), and `Cap` (array of Byte, representing special features using `mc...` constants like `mcWeapons`, `mcArmor`, `mcSeaTrans`).

*   **`TOffer` (`Protocol.pas`)**: A packed record used in diplomacy to represent an offer or demand between players. It contains `nDeliver` and `nCost` (integers specifying the number of items offered and demanded) and a `Price` array (array of Cardinal) where each element uses `opMask` and `op...` constants (e.g., `opCivilReport`, `opMap`, `opTech`, `opMoney`, `opTreaty`) to define the type and value of the item being exchanged.

*   **`TCmdList` (`CmdList.pas`)**: Manages a sequence of game commands for save/load/replay. Stores commands and associated data in the `LogData` array (of type `^TLogData`). `State` records the current position in the log.

*   **`TMainScreen` (`Term.pas`)**: The main form class (`TDrawDlg` descendant) for the local player GUI. Manages the display of the game map and UI elements, handles user input events (mouse clicks, keyboard shortcuts), manages the main menu (`GamePopup`, `UnitPopup`, `EditPopup`, `StatPopup`, `TerrainPopup`), various sub-dialogs (opened via menu clicks or button events), controls unit movement and actions (`MoveUnit`, `FocusOnLoc`, `SetUnFocus`, `EndTurn`), and provides the End-of-Turn (`EOT`) button (`TEOTButton`). It uses several helper buttons (`TButtonB`, `TButtonC`) and clickable areas (`TArea`) for UI interaction.

*   **`TIsoMap` (`IsoEngine.pas`)**: A class responsible for drawing the isometric game map and entities onto a `TBitmap` (`FOutput`). Contains methods for rendering tiles (`Paint`), units (`PaintUnit`), and cities (`PaintCity`). Manages map options (`Options`). Visualizes attack animations (`AttackBegin`, `AttackEffect`, `AttackEnd`) and uses graphic resources from `ScreenTools.pas` and `Tribes.pas`.

*   **`TTribe` (`Tribes.pas`)**: A class representing a player's civilization's visual and naming style, loaded from external files. Holds graphic handles (`symHGr`, `faceHGr`, `cHGr`, `HGrStdUnits`) and picture information (`CityPicture`, `ModelPicture`) for symbols, leaders, cities, and units. Stores city names (`CityName`) and model names (`ModelName`). Provides methods to get names (`GetCityName`, `CityName`) and set/choose model pictures (`SetModelPicture`, `ChooseModelPicture`). Uses `TStringList` internally to manage script content.

*   **`TStringTable` (`StringTables.pas`)**: A utility class for loading and looking up strings from text files (like `.txt` language files). Stores lines of text in arrays of pointers (`Lines`) pointing into a large character buffer (`Data`). Provides the `Lookup` function to retrieve strings by item name and optional index.

## Core Functionality

The interfaces reveal the core operations of the game engine:

*   **Game Initialization and Setup**:
    *   `InitRandomGame`, `InitMapGame`, `EditMap` (`Database.pas`): Procedures to create a new game world, either randomly generated or based on a specific map, setting up the initial state in `Database.pas`.
    *   `Init`, `Done` (`GameServer.pas`, `Tribes.pas`, `IsoEngine.pas`, `ScreenTools.pas`, etc.): Initialization and cleanup procedures for various modules, often involving loading resources or setting up data structures.
    *   `Client` (`LocalPlayer.pas`, `NoTerm.pas`): The main entry point for client communication, receiving commands and data from the server via the `TClientCall` procedure signature.
    *   `StartNewGame`, `LoadGame`, `EditMap` (`GameServer.pas`): Server-side procedures to initiate or load game sessions.

*   **Turn Processing**:
    *   `CityTurn` (`CityProcessing.pas`): Processes one turn for a specific city, handling production, growth, pollution, etc.
    *   `UnitTurn` (Implied, logic distributed across `UnitProcessing.pas` functions like `CalculateMove`, `Work`, `Recover`): Processes actions and state changes for units during a turn.
    *   `NextPlayer` (`GameServer.pas`): Advances the game to the next player's turn, triggering AI processing or waiting for human client input.
    *   `sTurn`, `cTurn` (`Protocol.pas`): Commands for players to end their turn and notify the server.

*   **Unit Operations**:
    *   `CreateUnit`, `FreeUnit`, `PlaceUnit`, `RemoveUnit` (`Database.pas`): Manage the lifecycle and placement of units in the game world, updating the map and relevant lists.
    *   `CalculateMove` (`UnitProcessing.pas`): Determines the feasibility, cost (in movement points), and potential outcomes (`TMoveInfo` record) of a unit movement to a target location.
    *   `MoveUnit` (`Protocol.pas`): Server command (often sent by the client) to execute a unit movement, taking parameters for direction/location.
    *   `LoadUnit`, `UnloadUnit` (`UnitProcessing.pas`, `Protocol.pas`): Handle unit embarkation and disembarkation from transports, checking capacity and validity.
    *   `StartJob`, `Work` (`UnitProcessing.pas`): Initiate and progress terrain improvement jobs (using `j...` constants) by settler units, consuming movement points and potentially modifying the tile in `Database.pas`.
    *   `GetMoveAdvice` (`UnitProcessing.pas`, `ClientTools.pas`): Provides information about possible unit destinations, movement costs, and potential outcomes using the `TMoveAdviceData` structure.
    *   Combat logic is distributed, with `GetBattleForecast` (`UnitProcessing.pas`, `Protocol.pas`) calculating outcomes (`TBattleForecastEx`) and the `IsoEngine.pas` visualizing attacks (`AttackBegin`, `AttackEffect`, `AttackEnd`).

*   **City Management**:
    *   `FoundCity`, `DestroyCity`, `ChangeCityOwner` (`Database.pas`): Manage city creation at a `Loc`, destruction, and capture/transfer of ownership between players, updating the map and player contexts.
    *   `SetCityProject`, `BuyCityProject`, `SellCityImprovement`, `RebuildCityImprovement` (`Protocol.pas`): Server commands for managing city production queues and built improvements. `sSetCityProject` uses flags like `cpImp` and `cpConscripts`.
    *   `SetCityTiles` (`CityProcessing.pas`, `Protocol.pas`): Manage the set of tiles being exploited by a city for resource collection, updating the city's `Tiles` bitset.
    *   `AutoBuild` (`ClientTools.pas`): Client-side automation logic for managing a city's production queue based on a desired build order (`TImpOrder`).
    *   `GetCityReport`, `GetCityReportNew`, `GetCityAreaInfo` (`CityProcessing.pas`, `Protocol.pas`): Functions to retrieve detailed information about a city's status, resource yields, happiness, corruption, and the availability of surrounding tiles for exploitation (`TCityAreaInfo`).

*   **Technology and Models**:
    *   `DiscoverTech`, `SeeTech` (`Database.pas`): Manage technology discovery and visibility for players, updating the `Tech` array in `TPlayerContext`.
    *   `TechCost` (`Database.pas`): Calculates the research cost for a technology (`ad...` constants) based on game parameters and difficulty.
    *   `SetResearch` (`Protocol.pas`): Server command for a player to choose a technology to research (`ResearchTech` in `TPlayerContext`).
    *   `CreateDevModel`, `SetDevModelCap` (`Protocol.pas`): Server commands for players to design new unit models (`TModel`), setting their base type and special capabilities (`Cap` using `mc...` constants).
    *   `CalculateModel` (`Database.pas`): Calculates the final stats of a developed unit model based on its base type, capabilities, and available technologies (`Upgrades` field in `TModel`).

*   **Diplomacy**:
    *   `IntroduceEnemy`, `CancelTreaty` (`Database.pas`): Server-side management of diplomatic relations between players, updating the `Attitude` and `Treaty` arrays in relevant `TPlayerContext` records.
    *   `scContact`, `scDipStart`, `scDipOffer` (`scDipOffer` uses the `TOffer` structure), `scDipAccept`, etc. (`Protocol.pas`): Protocol commands for initiating and conducting diplomacy negotiations between clients/AIs and the server.
    *   `DipCall`, `OfferCall` (`Term.pas`): Client-side functions to initiate diplomacy dialogs (`TNegoDlg`) and handle sending/receiving offers.
    *   `DoSpyMission` (`Database.pas`): Executes a spy mission (using `sm...` constants) at a target location, potentially revealing information or causing sabotage.

*   **Reporting and Information Retrieval**:
    *   Numerous `Get...Info`, `Get...Report`, `Can...` functions in `Database.pas`, `Protocol.pas`, `ClientTools.pas`: Provide ways to query the game state for information about tiles, units, cities, models, reports, etc.
    *   `HelpDlg` (`THelpDlg`, `Help.pas`), `NatStatDlg` (`TNatStatDlg`, `NatStat.pas`), `UnitStatDlg` (`TUnitStatDlg`, `UnitStat.pas`), `SelectDlg` (`TListDlg`, `Select.pas`), `TechTreeDlg` (`TTechTreeDlg`, `TechTree.pas`), `WondersDlg` (`TWondersDlg`, `Wonders.pas`): GUI dialogs for presenting various types of game information to the player in a structured format, often querying data via `ClientTools.pas`.

## Extending the Project

Developers looking to extend or modify this project should consider the following:

1.  **Understanding the Protocol (`Protocol.pas`)**: The `Protocol.pas` unit is central. Any changes involving new game entities, properties, commands, or information structures must start here by defining new constants, record fields, or message structures. Understanding the existing command structure (`sctMask`, `sExecute` flags, data sizes) and return codes (e.g., `eOK`, `eInvalid`, `eNoTime_...`) is essential for interacting correctly with the server.

2.  **Modifying Game Logic**: Core game rules, calculations (combat, movement cost, city production, research cost, etc.), and entity processing (`CityProcessing.pas`, `UnitProcessing.pas`, `Database.pas`) are implemented server-side. Modifications to game mechanics require changes in these units. Pay close attention to how game state data is accessed and modified via the pointers and arrays managed by `Database.pas` (e.g., `RW`, `RealMap`, `Occupant`, `Continent`).

3.  **Developing a New Client**: The separation of server and client logic allows for developing entirely new client interfaces (e.g., a web client, a different GUI toolkit, or a headless client for simulations). Such a client would need to implement the `TClientCall` procedure to receive updates and notifications from the server and communicate with the server using the commands defined in `Protocol.pas` via the `TServerCall` interface. The `NoTerm.pas` unit can serve as a simpler example of a client implementation compared to the full `LocalPlayer` GUI.

4.  **Developing a New AI**: AI players interact with the server via the `TServerCall` interface (provided to the AI module via `TInitModuleData`) and implement the `TClientCall` procedure. AI logic should be implemented in separate units (like those found in the original c-evo AI projects) and interact with the game state by sending commands to the server and processing the data received through the client procedure (`TBrainInfo` holds the `Client: TClientCall`). The `GameServer.pas` manages the loading and execution of these AI modules. AI often needs to react to server notifications (`nt...`) to know when to act.

5.  **Adding New Content (Techs, Improvements, Units, Terrain)**: Adding new technologies (`ad...` constants), city improvements (`im...` constants, entries in the `Imp` array), unit model features (`mc...` constants, entries in the `Feature` array), unit types (new entries or modifications to `SpecialModel`, `upgrade` arrays), or terrain types would primarily involve modifications to the constants and data structures defined in `Protocol.pas`. Implementing the *gameplay effects* of this new content (e.g., how a new tech unlocks other content or abilities, how a new terrain affects movement or production) requires modifying the relevant logic in `Database.pas`, `CityProcessing.pas`, and `UnitProcessing.pas` that utilize this new content. Adding new graphical assets would involve modifying the resource loading and drawing logic, likely in `ScreenTools.pas`, `Tribes.pas`, and `IsoEngine.pas`.

6.  **Modifying the Existing GUI (`LocalPlayer`)**: Changes to the graphical interface involve modifying the forms and controls within the `LocalPlayer` directory. This includes altering layouts, adding new dialogs, or changing how game data (accessed via `ClientTools.pas`) is visualized. Custom controls and drawing logic are common (`ButtonBase.pas`, `Area.pas`, `ScreenTools.pas`).

7.  **Localization and Assets**: Text strings are managed via `StringTables.pas`, loading from text files (`Language.txt`, `Language2.txt`). Graphics are handled by `ScreenTools.pas` and loaded from the `Graphics` directory using functions like `LoadGraphicFile` and `LoadGraphicSet`. Sounds are handled by `Sound.pas` and `ScreenTools.pas` (`Play` procedure) and loaded from the `Sounds` directory based on names looked up in the `Sounds` string table. Adding new languages or replacing assets involves providing appropriately named files in the relevant directories and potentially updating the string table or graphic loading code.

## Potential Real-World Applications and Problem Solving Ideas

While primarily a game engine, the architecture and concepts within c-evo-eh-code could be adapted or provide insights for simulating and solving certain real-world problems. The modular design, state management, and processing loops offer a foundation for various simulation and strategic modeling tasks:

*   **Resource Management and Logistics Simulation**: The city logic in `CityProcessing.pas`, managing `TCity` structures with fields like `Food`, `Pollution`, `Prod`, `Tiles` (exploited resources), and the rules in `Protocol.pas` governing resource yields (`Terrain` array) and production (`Imp` array), provides a framework for simulating complex resource management and logistics networks. This could be applied to optimize supply chains, manage resource allocation in a decentralized system, or model the flow of goods and personnel in urban or regional planning scenarios by adapting the city processing and map interaction (`Database.pas`) logic.

*   **Agent-Based Modeling**: The player (`TPlayerContext`) and unit (`TUn`) entities, with their individual states (`Status`, `Health`, `Movement`, `Flags`), goals (implicit in game mechanics or explicit in AI behavior), and interactions with the environment (`Loc` on `RealMap`) and each other (`UnitProcessing.pas` for combat, movement), form a basis for agent-based modeling. This approach could be used to simulate social systems, economic interactions, or the spread of phenomena (like information or disease) where individual agent behavior significantly influences the system as a whole. Modifying `UnitProcessing.pas` and the AI logic (`GameServer.pas`, external AI modules) would be key.

*   **Strategic Planning and Optimization**: The game's core loop involves players making strategic decisions (where to move units, what to produce, what to research, whom to ally with) based on limited information (`TPlayerContext.Map`, `Enemy...` lists) and environmental factors. The engine's game turn processing (`GameServer.pas`) and diplomatic system (`Database.pas`, `Protocol.pas` `scDip...` commands) could be adapted to model strategic interactions in competitive environments, providing a testbed for evaluating different strategies or optimizing decision-making under uncertainty. Examples include simulating market competition, political campaign strategies, or even military planning scenarios.

*   **Procedural Content Generation**: The map generation code (`Database.pas`, `GameServer.pas`, functions like `CreateElevation`, `CreateMap`) demonstrates techniques for creating complex and varied environments programmatically by manipulating the `RealMap` array. These techniques could be applied to generate diverse synthetic environments for testing autonomous systems, creating complex virtual worlds for simulations, or even assisting in real-world design processes by generating variations of potential layouts (e.g., urban planning, network design).

*   **System Dynamics Simulation**: Aspects of the game, such as population growth (`CityGrowth` in `CityProcessing.pas`), pollution spread (`Pollute` in `CityProcessing.pas`, `fPoll` flag in `Protocol.pas`), resource depletion/generation (`Terrain` array in `Protocol.pas`), and the impact of technology (`AdvPreq`, `Imp`, `upgrade` arrays in `Protocol.pas`, `Database.pas`), represent classic elements of system dynamics. The engine's turn-based processing (`GameServer.pas`) and state update logic could be refactored or used as a base to build simulations of interconnected systems, allowing for the study of feedback loops and emergent behavior in ecological, economic, or social contexts by modifying the core processing modules.

*   **Educational Tool**: The transparent nature of the Pascal code could make this engine a valuable educational tool for teaching concepts in programming, game development, simulation modeling, or even introductory topics in strategy and systems thinking. Developers could modify or simplify parts of the code (e.g., the logic in `UnitProcessing.pas` or `CityProcessing.pas`) to illustrate specific principles.

Adapting the engine for these purposes would require significant effort, likely involving modifying the core data structures (`Protocol.pas`), game logic (`Database.pas`, `CityProcessing.pas`, `UnitProcessing.pas`), and potentially replacing the game-specific AI with goal-oriented agents relevant to the real-world problem. However, the existing framework provides a robust starting point with established patterns for managing complex state and interactions within a turn-based simulation.

**Important Note on Adaptation**: It is crucial to understand that adapting this game engine for serious real-world simulation or optimization tasks would require significant effort. It would likely involve substantial refactoring of core game logic, data structures, and the implementation of domain-specific models and algorithms relevant to the target problem, going far beyond the existing game mechanics.

## Documentation Verification

When contributing to this documentation, especially when mentioning specific files, procedures, or data structures, developers should verify their existence and correct naming by cross-referencing with the Pascal source code files provided. This helps prevent broken links and ensures the documentation accurately reflects the current state of the codebase.
