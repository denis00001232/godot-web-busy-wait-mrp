# Minimal reproduction for godotengine/godot#93652

A Godot **web** export with `application/run/max_fps` set keeps a CPU core busy at ~100 %
even on an empty scene. Happens in single-threaded builds (**Thread Support off**), on
Linux and Android alike (`nanosleep` on the browser's main thread is a busy loop).

## Run it

1. Open this folder in Godot 4.x (any renderer; `gl_compatibility` is set here).
2. `project.godot` already sets `application/run/max_fps = 30`; `main.tscn` is an empty scene.
3. Project → Export → Web, with **Thread Support off**, export.
4. Open the exported `index.html` in Chrome, then open Chrome's Task Manager (Shift+Esc):
   the tab sits at ~100 % CPU on an empty scene. In DevTools → Performance every
   `requestAnimationFrame` callback takes ~33 ms (~1/max_fps) while the scene does ~0.1 ms.

## After the fix

With `OS_Web::add_frame_delay()` made a no-op in `platform/web/os_web.cpp` (the frame cap is
already applied by `main_loop_callback()` in `web_main.cpp`), the same tab idles at a few
percent: skipped frames take ~0.1 ms, CPU clocks and power draw drop.

Measured on a Pixel 8 Pro (Chrome, `max_fps = 30`): main thread ~100 % → ~60 %, big cores
2.7–2.9 GHz → 1.1–1.9 GHz, battery draw in game 4.0 W → 2.9 W at the same frame rate. A
60 FPS cap then costs about the same as 30.

Links: [issue #93652](https://github.com/godotengine/godot/issues/93652),
[pull request #125382](https://github.com/godotengine/godot/pull/125382).
