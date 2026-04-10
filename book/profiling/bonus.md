---
title: "Bonus"
excerpt: ""
permalink: "/book/unrealengine/bonus/"
toc: true
toc_sticky: true
---

## Cook and Build .bat

A simple `.bat` file that runs both the build and the cook in one go — no need to trigger them separately from the editor.

{% capture tutorialvideo %}FygowRHfgkc?showinfo=1{% endcapture %}
{% include video id=tutorialvideo provider="youtube" %}

<div class="notice--info" markdown="1">
**cook_and_build.bat**

```bat
@echo off
setlocal

:: ---- CONFIGURATION ----
set UE_PATH=C:\Program Files\Epic Games\UE_5.3
set PROJECT_PATH=%~dp0MyProject.uproject
set PLATFORM=Win64
set CONFIG=Development
:: ------------------------

set UAT=%UE_PATH%\Engine\Build\BatchFiles\RunUAT.bat

echo [1/2] Building...
call "%UE_PATH%\Engine\Build\BatchFiles\Build.bat" ^
    MyProjectEditor %PLATFORM% %CONFIG% "%PROJECT_PATH%"

echo [2/2] Cooking...
call "%UAT%" BuildCookRun ^
    -project="%PROJECT_PATH%" ^
    -noP4 ^
    -platform=%PLATFORM% ^
    -clientconfig=%CONFIG% ^
    -cook ^
    -compressed ^
    -pak ^
    -stage

echo Done!
pause
```
</div>

Replace `UE_PATH`, `PROJECT_PATH`, and the project name (`MyProjectEditor`) to match your setup. The script builds the editor first, then immediately kicks off the cook — no manual steps in between.
