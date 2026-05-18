# 3D Controller Overlay (FSV Update) - Development Guide

This document provides critical context and architectural standards for maintaining and extending this project.

## Project Overview
A 3D controller overlay application built with **C++**, **OpenGL**, **GLFW**, **ImGui**, and **SDL3**. It specializes in high-fidelity 3D visualization of controller inputs, including motion sensors (Gyro/Accel).

## Key Dependencies & Versions
- **SDL3 (3.2.0+):** Core library for gamepad input and motion sensors. 
    - *Note:* Migrated from SDL2. Use `SDL_Gamepad` instead of `SDL_GameController`.
- **GLFW (3.3+):** Window management and OpenGL context creation.
- **ImGui:** Integrated with docking and multiple viewports (in settings).
- **GLM:** Mathematics library for 3D transformations.
- **MinGW-w64 (GCC 15.2.0+):** Recommended build toolchain.

## Architectural Standards

### 1. SDL3 Integration
- **Enumeration:** Always use `SDL_GetGamepads()` for finding controllers. It returns an array of `SDL_JoystickID`.
- **Memory Management:** Any `char*` returned by `SDL_GetGamepadMapping` or arrays from `SDL_GetGamepads` **must** be freed using `SDL_free()`.
- **Sensors:** Use `SDL_SetGamepadSensorEnabled(controller, SDL_SENSOR_GYRO, true)` to activate motion data.
- **Initialization:** `SDL_Init` returns `bool` in SDL3. `true` means success.

### 2. Motion (Gyro) Logic
- **Precision:** Gyro data is processed in `controller_window.cpp` using `SDL_GetGamepadSensorData`.
- **Timestamps:** Since SDL3 removed polling-with-timestamp, we use `SDL_GetTicksNS() / 1000` to get microsecond-precision timing.
- **Initialization:** `gyro_toggled` must be `true` when a controller is first opened or gyro is enabled to baseline the timestamp and avoid massive rotation jumps.

### 3. Controller Mappings
- **Format:** Mapping strings follow the SDL3 standard: `GUID,Name,bindings...`.
- **Parsing Safety:** When parsing mapping strings (in `settings_window.cpp`), always check `binding.size() >= 2` before accessing elements to avoid crashes on GUID/Name fields.
- **GUID Handling:** Use `SDL_GetJoystickGUID` and `SDL_GUIDToString`.

### 4. Build System
- **Batch Script:** `windows_build.bat` is the primary entry point for Windows builds.
- **DLLs:** Ensure `SDL3.dll`, `libstdc++-6.dll`, `libgcc_s_seh-1.dll`, and `libwinpthread-1.dll` are in the project root to match the compiler version.
- **Includes:** Use `<SDL3/SDL.h>`.

## Common Pitfalls
- **Iterator Erasure:** Never use `vec.erase(vec.end())`. Use `vec.pop_back()` with a size check.
- **Polling Loop:** The `SDL_PollEvent` loop in `controller_window_input()` is essential for hot-plugging and sensor updates. Do not remove it.
- **Const Correctness:** `SDL_GetBasePath()` returns `const char*` in SDL3.

## Adding New Models
- Models are located in `3d-controller-overlay-fsv/models/`.
- Each model folder should contain `.obj` files matching the indices defined in `settings_window.cpp` (`mesh_filenames`).
