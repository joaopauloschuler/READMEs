
# Cevo.Map.Gen

## Introduction
CevoMapGen is a specialized tool designed to generate map files for the Cevo game. Developed primarily in **Free Pascal**, it provides advanced control over terrain, climate, and resource distribution compared to the in-game generator, enabling the creation of more challenging and diverse game worlds. This tool is particularly useful for generating novel or extreme world types for gameplay or analysis. This documentation aims to provide a detailed technical overview and usage guide for both users and potential developers running on systems like **Ubuntu**.


## Features

*   Fast generation of random Cevo map files.
*   Increased range and control over input parameters.
*   More extreme terrain and climate possible.
*   Sample world templates for instant use.
*   Optional thumbnail view of map files.
*   Estimate of city sites to indicate game length.
*   City site count and thumbnail view possible with exiting maps.
*   Support of larger maps for 'Distant Horizon' version of Cevo.
## Technical Architecture Overview

The Cevo.Map.Gen project is structured into several Free Pascal units (`.pas` files) and include files (`.pp` files), each responsible for specific aspects of map generation, data handling, or the user interface. Understanding these components is key to modifying or extending the generator.

*   **`CevoMap.pas`**: This is the central unit defining the `tMap` class, which represents the map data structure (`tMapSquare` records) and orchestrates the main generation processes. It holds the terrain, temperature, rainfall, resource, and other per-tile data. It includes procedures for generating fractal noise (`GenMap`), sorting tiles by altitude (`AltitudeSort`), adjusting sea and mountain levels, placing start positions, and building the final terrain based on noise data. It uses units like `MapTiles`, `Textures`, `Utils`, `SortTiles`, `Message`, `Rivers` and includes files like `StartPositions.pp`, `DeadLands.pp`, `TileResource.pp`, and `Coastal.pp`.
*   **`MapTiles.pas`**: Defines the basic `tTerrain` enum for different terrain types (Ocean, Coast, Grass, Desert, etc.) and the `tTile` packed record which represents the 4-byte structure of a tile in the C-evo map file format. It also contains simple functions for determining terrain based on rain/temperature for different altitude bands (`Lowlands`, `Steppe`, `Highlands`) and a utility for getting a character representation of a terrain type (`TerrainChar`).
*   **`Fractal.pas`**: Implements the `TFBm` class for generating Fractional Brownian Motion (fBm), a type of fractal noise used as a basis for generating terrain altitude, rainfall, and temperature variations across the map. It utilizes the `Noise` unit.
*   **`Noise.pas`**: Provides the fundamental Perlin noise implementation via the `TNoise3` class. This unit is a core building block for the fractal noise generated in `Fractal.pas`.
*   **`Noises.pas`**: Extends the basic Perlin noise (`TNoise3`) with the `tNoises` class, specifically implementing "Variable Rate Noise" (`VRNoise3`), a technique used for distorting the noise domain to create more organic and varied patterns.
*   **`Rivers.pas`**: Manages the generation of rivers. It uses a diffusion-like process, starting from potential estuary locations (`BestStart`) and flowing downhill based on the altitude data provided by the `CevoMap` unit (`RunRiver`, `NextRiverTile`). This unit is *used by* `CevoMap.pas`.
*   **`StartPositions.pp`** (Included in `CevoMap.pas`): Contains routines for calculating city site scores (`StartScore`) based on surrounding terrain and resources, adding potential sites to a list (`AddCity`), culling overlapping sites (`CullCitySites`), and finally placing the selected starting positions on the map (`PlaceStartPositions`).
*   **`TileResource.pp`** (Included in `CevoMap.pas`): Provides the logic for determining the food, production, and trade output of a specific tile based on its terrain and the presence of bonus resources or rivers (`TileResource`). It also identifies tiles suitable for potential Dead Lands placement (`PosDeadLands`).
*   **`DeadLands.pp`** (Included in `CevoMap.pas`): Handles the allocation and placement of Dead Lands (`AllocateDeadLands`, `SetDeadLand`) (which can include special resources like Cobalt, Uranium, Mercury) near viable city sites (`CheckDeadLand`) after the starting positions are determined.
*   **`Coastal.pp`** (Included in `CevoMap.pas`): Contains procedures for identifying and marking coastal tiles (`DoInnerCoast`, `DoOuterCoast`) based on their proximity to land and ocean (`CheckNearest`, `CoastTarget`). It also includes a function to remove isolated single-tile islands (`CullSingleTileIslands`).
*   **`Resources.pas`** (Included in `CevoMap.pas`): Contains routines for the allocation of terrain resources, including the `SpecialTile` function based on the in-game algorithm and the logic for random distribution. This file is *included by* `CevoMap.pas`.
*   **`CevoMapFile.pas`**: Implements the `TmapFile` class, responsible for reading and writing C-evo map files (`.cevomap`) according to their binary format. It handles the file header (`THeader`) and tile body data (`TmapBody`). It also manages saving/loading the GUI's INI settings.
*   **`Settings.pas`**: Defines constants and a lookup function (`UpdateSetting`) to map the user-friendly radio group indices in the GUI (and implicitly, the template settings) to specific integer values used by the map generation algorithms for parameters like land mass percentage, mountain percentage, etc.
*   **`Utils.pas`**: A collection of general utility functions, including mathematical helpers like `JumpStep` (for scaling), `Bias`, `Gain`, `BlendRound`, `MyRound`, and `GainBias` (for manipulating noise values).
*   **`SortTiles.pas`**: Provides a simple procedure (`TileSort`) for sorting an array of `tTileData` records (used in city site scoring) based on their food, production, and trade values.
*   **`Message.pas`**: A simple unit providing a `ShowMessage` procedure that abstracts between using `Dialogs.ShowMessage` (for GUI) and `Writeln` (for CLI).
*   **`CevoMapGen.lpr`**: The main program file for the GUI application, using the Lazarus IDE.
*   **`CevoMapMain.pas`**: The unit for the main GUI form (`TFrmInterface`), handling user input, calling the map generation logic, displaying statistics, and managing templates and file operations. It includes the `TemplateDefaults.pp` and `TemplateFiles.pp` includes.
*   **`ImageForm.pas`**: The unit for the map thumbnail preview window (`TFrmImage`).
*   **`CevoMapCheck.pas`**: A separate CLI program to check map file validity and output statistics (e.g., terrain percentages, resource counts, start position details).
*   **`CevoMapGenCLI.pas`**: The main program file for the command-line application, parsing command-line arguments, loading templates, building the map, and saving the output file without a GUI.


## Installation

CevoMapGen is primarily developed using Free Pascal and the Lazarus IDE. To install it, you will typically need to compile the source code yourself.

1.  **Prerequisites:**
    *   Install the Free Pascal Compiler (FPC) suitable for your operating system. On Ubuntu, this can often be done via your package manager (e.g., `sudo apt update && sudo apt install fpc`).
    *   If you plan to use the Graphical User Interface (GUI) version, you will also need the Lazarus IDE. On Ubuntu, you can install it via the package manager (e.g., `sudo apt install lazarus`).
    *   Ensure you have a standard build environment (like `make` and a C compiler) as sometimes required by FPC/Lazarus for linking libraries. These are usually installed as part of essential build tools (e.g., `sudo apt install build-essential`).

2.  **Obtaining the Source Code:**
    *   Get the project source code. In the context of this task, the source code is already provided to you. In a typical scenario, this would involve cloning a Git repository or downloading a source archive.

3.  **Compiling the Command-Line Interface (CLI):**
    *   Navigate to the directory containing the source files provided.
    *   The main source file for the CLI is `CevoMapGenCLI.pas`.
    *   Compile it using the Free Pascal Compiler. If FPC is in your PATH, the command is straightforward:
        ```bash
        fpc CevoMapGenCLI.pas
        ```
    *   This command should produce an executable file (e.g., `cevomapgencli` on Linux, `cevomapgencli.exe` on Windows) in the same directory.

4.  **Compiling the Graphical User Interface (GUI):**
    *   Open the Lazarus IDE.
    *   Go to `Project -> Open Project` and select the project file (`CevoMapGen.lpr`).
    *   Use the Lazarus IDE's build function, typically found under the `Run` menu (`Run -> Build Project` or `Run -> Build All Project Files`).
    *   Lazarus will compile the necessary units and the main program file, producing the GUI executable (e.g., `cevomapgen` on Linux, `cevomapgen.exe` on Windows) in the project directory (often within a `lib/<architecture>-<os>` subdirectory).

5.  **Placement:**
    *   Place the compiled executable(s) in a directory included in your system's PATH, or in a convenient location from which you can run them.
    *   Ensure the necessary template files (`Templates/*.INI`) are accessible. The GUI version on Linux automatically looks for and copies default templates to `~/.config/cevomapgen/Templates` if they are missing or outdated, as implemented in the `CopyTemplates` procedure in `CevoMapMain.pas`. The CLI tool, by default, expects the template INI file to be in the same directory from which it is run.



## Usage (GUI)

The Graphical User Interface provides an interactive environment for generating and previewing Cevo maps.

1.  **Starting the GUI:**
    *   Run the compiled `cevomapgen` executable. A window titled "C-evo Map Generator" will appear.

2.  **Main Tab:**
    *   **Map Size:** Select a predefined map size (e.g., 100%, 230%) using the radio buttons. This sets the overall `Width` and `Height` (`SpinHeight`, `SpinWidth`). Note that maps larger than 230% (9,600 tiles) are generally only compatible with the 'Distant Horizon' variant of C-evo.
    *   **Customise Map Size:** Use the spin edits for Height (`SpinHeight`) and Width (`SpinWidth`) to set custom map dimensions. The maximum allowed map size is 126 Width x 132 Height (16,632 tiles). The Area panel shows the total number of tiles. Colors indicate compatibility with older Cevo versions (Yellow if exceeding 100x96 or 9600 tiles).
    *   **Map Files:**
        *   `[SAVE]` button (`BtnSave`): Saves the currently generated map to a `.cevomap` file. A dialog will open to choose the filename and location. The dialog title may indicate suitability for 'Distant Horizon' only if the map is large or uses non-standard resources.
        *   `[LOAD]` button (`BtnLoad`): Loads an existing `.cevomap` file. The terrain analysis and city estimates for the loaded map will be displayed, and a thumbnail preview will be generated if enabled.
    *   **Thumbnail View:**
        *   `SpinEditImageSize`: Adjusts the size of each map tile in the preview (0 to 4 pixels per square). Setting the size to 0 hides the preview image display.
        *   `UD_Scroll`: Scrolls the thumbnail view horizontally, useful for viewing maps that wrap around. This is implemented in `ImageForm.pas`.
        *   Right-click on the thumbnail preview to save it as a JPEG image.
    *   **Terrain Analysis:** Displays the percentage of each terrain type on the current map (Ocean, Mountain, Grass, Desert, etc.). The River count is a percentage of total tiles, not exclusive of other terrain.
    *   **No. Cities:** Provides a rough estimate of the number of viable city sites available on the map. The background color of the display panel (`OP_Citysquares`) indicates the estimate range (Orange: <12, Yellow: 12-27, Gray: >27).
    *   **Generate New Map:**
        *   `[New Land]` button (`MultiLineBtn1`): Generates a completely new map. This includes a new random land shape (unless "Random" is unchecked on the Land tab) and new rainfall and temperature patterns (unless their "Random" checkboxes are unchecked). `ResetNoise` is set to True in the code.
        *   `[Same Land]` button (`Button1`): Regenerates the map using the *same* land shape as the previous generation but applies new rainfall and temperature patterns. This is useful for fine-tuning terrain and climate on a preferred landmass. `ResetNoise` is set to False.
    *   `ProgressBar1`: Shows the progress of the map generation process.
    *   `LblStatus`: Displays the current stage of the generation process (e.g., Altitude, Rainfall, Temperature, Rivers, Terrain, OK).

## Usage (CLI)

The Command Line Interface (`cevomapgencli`) allows you to generate maps programmatically, which is useful for scripting or batch processing without the graphical interface.

The basic syntax for running `cevomapgencli` is:

```bash
cevomapgencli <Map Size Index> <Seed> <Template Filename> [Output Directory]
```

*   `<Map Size Index>`: An integer from 0 to 8, corresponding to predefined map sizes. These indices map to specific width and height values defined in the `Settings.pas` unit.
    *   0: 35% (30x46)
    *   1: 50% (40x52)
    *   2: 70% (50x60)
    *   3: 100% (60x70)
    *   4: 150% (75x82)
    *   5: 180% (84x90)
    *   6: 230% (100x96)
    *   7: 300% (105x120)
    *   8: 400% (126x132)
*   `<Seed>`: An integer used to initialize the pseudorandom number generator (PRNG) via `RandSeed`. Using the same seed (and same template/size) will produce the exact same map. A value of 0 uses a random seed based on the system time.
*   `<Template Filename>`: The base name of the INI template file to load (without the `.INI` extension). The CLI tool (`CevoMapGenCLI.pas`) expects this file to be located in the directory from which you run the command.
*   `[OutputDirectory]`: An optional path where the generated `.cevomap` file will be saved. If not provided, the map file will be saved in the current working directory. The output filename will be derived from the template name (e.g., `Standard.cevomap`).

**Example:**

To generate a 100% map (index 3) using seed 54321 and the template `Lakes` (assuming `Lakes.INI` is in the current directory), saving the output to `/home/user/cevo_maps`:

```bash
./cevomapgencli 3 54321 Lakes /home/user/cevo_maps
```

This will create `/home/user/cevo_maps/Lakes.cevomap`.

The CLI program will output the name of the generated map file upon successful completion.

## Configuration Templates (.INI Files)

Cevo.Map.Gen uses INI files to store configuration templates, allowing users to define and reuse specific map generation settings. When using the `CevoMapGenCLI` or the GUI's load/save template features, a template file is used to configure the map generation process.

Template files for the GUI are typically stored in the application's configuration directory, in a `Templates` subdirectory (e.g., `~/.config/cevomapgen/Templates/Standard.INI` on Linux). The CLI loads templates from the current working directory.

A template file contains several sections, each corresponding to a tab or group of settings in the GUI:

### `[LAND]` Section

Controls global landmass, mountain levels, and the "age" and "size" of geographical features.

*   `RG_LandMass` (Integer): Corresponds to the "Land Mass" radio group index (0-5 in the GUI).
*   `SE_LandMass` (Integer): Direct value for the land mass percentage (e.g., 8, 20, 32, 44, 60, 80). This is the value used internally.
*   `RG_Resources` (Integer): Corresponds to the "Resources" radio group index (0-3 in the GUI).
*   `SE_Resources` (Integer): Direct value for resource percentage (-1 for Standard, 4, 6, 8). Note that random resource distribution other than Standard (-1) is only supported by Cevo Distant Horizon. This value is used directly by the `AllocateResources` procedure in `Resources.pas`.
*   `RG_Mountains` (Integer): Corresponds to the "Mountains" radio group index (0-4 in the GUI).
*   `SE_Mountains` (Integer): Direct value for mountain percentage (0, 5, 10, 15, 20). Used by `AdjustMountainLevel` in `CevoMap.pas`.
*   `RG_Size` (Integer): Corresponds to the "Land Form" radio group index (0-4 in the GUI).
*   `SE_Size` (Integer): Direct value for the feature wavelength parameter (8, 12, 16, 22, 28). Used as the `Wave` parameter in `Ttexture.GenData` for Altitude noise.
*   `RG_Age` (Integer): Corresponds to the "Age (Years)" radio group index (0-3 in the GUI).
*   `SE_Age` (Integer): Direct integer value (2, 3, 4, 5). This value is used as the `Dim` parameter in `Ttexture.GenData` for Altitude noise *after being divided by 10* in `CevoMapGenCLI.pas` and `CevoMapMain.pas`. When creating templates manually, use the integer value (e.g., 3 for 3 Billion years).

### `[RAIN]` Section

Configures rainfall distribution and river generation.

*   `RG_RainPoles` (Integer): Corresponds to the "Poles" rainfall radio group index (0-4 in the GUI).
*   `SE_RainPoles` (Integer): Direct value for rainfall intensity at the poles (30, 45, 60, 75, 90). Used as `BiasP` in `CevoMap.GenMap` for Rainfall noise.
*   `RG_RainEquat` (Integer): Corresponds to the "Equator" rainfall radio group index (0-4 in the GUI).
*   `SE_RainEquat` (Integer): Direct value for rainfall intensity at the equator (30, 45, 60, 75, 90). Used as `BiasE` in `CevoMap.GenMap` for Rainfall noise.
*   `RG_River` (Integer): Corresponds to the "Rivers" radio group index (0-2 in the GUI) which maps to river percentages 2 (Fewer), 4 (Normal), 6 (More) in `Settings.pas`.
*   `SE_Water` (Integer): This value represents the river percentage. The GUI saves the RadioGroup index value multiplied by 15 (e.g., 2*15=30, 4*15=60, 6*15=90). The CLI reads this value and divides it by 15 to get the actual percentage used by the `tRivers` object in `Rivers.pas`. When creating templates manually, it is simpler to use the actual percentage (2, 4, or 6).
*   `SE_RainLen` (Integer): Direct value for the rainfall pattern wavelength (8-32). Used as `Wave` in `CevoMap.GenMap` for Rainfall noise.
*   `SE_RainGain` (Integer): Direct value for rainfall variability (20-70). Used as `Gain` in `CevoMap.GenMap` for Rainfall noise.

### `[TEMP]` Section

Configures temperature distribution.

*   `RG_TempEquat` (Integer): Corresponds to the "Equator" temperature radio group index (0-4 in the GUI).
*   `SE_TempEquat` (Integer): Direct value for temperature at the equator (85, 70, 55, 40, 25). Used as `BiasE` in `CevoMap.GenMap` for Temperature noise.
*   `RG_TempPoles` (Integer): Corresponds to the "Poles" temperature radio group index (0-4 in the GUI).
*   `SE_TempPoles` (Integer): Direct value for temperature at the poles (75, 60, 45, 30, 15). Used as `BiasP` in `CevoMap.GenMap` for Temperature noise.
*   `SE_TempLen` (Integer): Direct value for the temperature pattern wavelength (6-40). Used as `Wave` in `CevoMap.GenMap` for Temperature noise.
*   `SE_TempGain` (Integer): Direct value for temperature variability (20-70). Used as `Gain` in `CevoMap.GenMap` for Temperature noise.

Understanding these parameters and their corresponding values is essential for creating custom templates for the CLI tool or modifying existing ones.



## Leveraging the Generator

While primarily designed for the C-evo game, the Cevo.Map.Gen tool, particularly its command-line interface (`CevoMapGenCLI`), can be leveraged for various purposes related to procedural world generation and scenario testing.

*   **Geographical Scenario Simulation:** By adjusting parameters in the template files, users can simulate different planetary or geographical conditions. For example:
    *   Generating maps with high land mass and low rainfall (arid worlds).
    *   Creating archipelagos with high rainfall (tropical island chains).
    *   Designing worlds with extensive mountain ranges and low temperatures (harsh, mountainous planets).
    *   Exploring the impact of different "ages" on terrain complexity.
    This allows for the creation of diverse environmental backdrops for simulations, game design prototypes, or educational purposes illustrating geographical concepts. The `GenMap` procedure in `CevoMap.pas`, using the noise generation units (`Fractal.pas`, `Noise.pas`, `Noises.pas`) with configurable `Wave` and `Dim` parameters, is key to generating these varied landscapes.
*   **Consistent Test Environments:** The ability to specify a random seed when running `CevoMapGenCLI` means that a map generated with a specific seed, template, and size will always be the same. This is invaluable for creating consistent test environments for game AI development, multiplayer balance testing, or comparing strategies on identical terrain. The CLI tool is ideal for automating the generation of such consistent maps for test suites.
*   **Generating Unique Challenges:** Game designers or players seeking novel gameplay experiences can use the tool to generate maps with parameters outside the 'standard' range. This can lead to worlds with extremely limited city sites (influenced by `MinCityScore` and `MinCityDist` in `StartPositions.pp`), vast oceans (controlled by `SE_LandMass`), dominant mountain ranges (`SE_Mountains`), or unusual resource distributions (`SE_Resources`), providing unique strategic challenges.
*   **Batch Generation:** The `CevoMapGenCLI` can be integrated into shell scripts or other automation workflows to generate large batches of maps with varying parameters or seeds. This is useful for generating a large pool of diverse maps for competitive leagues, randomized gameplay sessions, or data analysis.

By understanding and manipulating the parameters exposed through the template files, users can go beyond simple random map generation to actively design the kind of world they want to explore or test within the C-evo framework or potentially adapt for other procedural generation tasks.

## Extending the Generator

CevoMapGen is open-source and written in Free Pascal, making it possible for developers to modify and extend its functionality. Here are some areas where contributions could be made:

*   **New Terrain Types:** The `MapTiles.pas` unit defines the existing terrain types and the logic for assigning them based on temperature, rainfall, and altitude (`Lowlands`, `Steppe`, `Highlands`). Developers could add new terrain types or modify the assignment logic to create novel environments.
*   **Alternative Noise Functions:** The `Noise.pas` and `Noises.pas` units implement Perlin noise and its variations. Experimenting with other noise algorithms (e.g., Simplex noise, Worley noise) could lead to different terrain aesthetics and features. This would involve modifying the `Ttexture.GenData` procedure in `Textures.pas` and potentially the `TfBm.Compute` function in `Fractal.pas`.
*   **Refined Resource Placement:** The `Resources.pas` include file handles resource allocation (`AllocateResources`, `SetResource`). More sophisticated algorithms could be implemented for placing resources based on terrain types, proximity to other features, or historical/geological simulations.
*   **Advanced River Systems:** The diffusion-based river generation in `Rivers.pas` could be enhanced to create more complex or realistic river networks, perhaps incorporating concepts like watersheds or erosion simulation.
*   **City Site Logic:** The criteria and scoring for city sites in `StartPositions.pp` (`StartScore`) could be adjusted to favor different types of locations or to implement alternative starting position distribution strategies. The `MinCityDist` and `MinCityScore` constants in `CevoMap.pas` are key parameters here.
*   **Integration with External Data:** The generator could be extended to incorporate external data, such as real-world elevation data or climate models, to generate maps based on geographical data. This would likely involve adding new data loading and processing routines, potentially requiring modifications across multiple units like `CevoMap.pas` and `CevoMapFile.pas`.
*   **Support for New Cevo Features:** As Cevo or its variants evolve, the map file format might change, or new map-related features might be introduced. The `CevoMapFile.pas` unit would be the primary place to update to support such changes.

Before making significant changes, it is recommended to study the existing codebase, particularly the interaction between the core units (`CevoMap.pas`, `CevoMapFile.pas`, `Textures.pas`, `Rivers.pas`) and the include files (`Coastal.pp`, `DeadLands.pp`, `Resources.pas`, `TileResource.pp`, `StartPositions.pp`).

<h2>License</h2>
Cevo.Map.Gen is free software. It is licensed under the GPLv3 or any later version. See the included source files for copyright and license details.

<h2>Contact Information</h2>
For any issues or questions regarding Cevo.Map.Gen, please refer to the contact information provided in the original `Docs/Readme.md` file:

Peter at PBlackman dot plus dot com

When reporting issues, please specify the operating system and version number you are using.
