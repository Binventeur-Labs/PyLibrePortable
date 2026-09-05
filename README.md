# PyLibrePortable

**Create portable Python applications for Windows — from a standalone Python environment to a professional native `.exe` launcher — while keeping your application files fully accessible and editable.**

PyLibrePortable is a practical guide for building a **fully portable Python application on Windows**, based on the official **Python Windows Embeddable Package**.

The goal is not only to run Python without installing it on the target machine, but to provide everything needed to turn a Python project into a **portable, self-contained application with an optional native Windows executable launcher**.

Your Python code, resources, and portable environment remain accessible and modifiable.

## ✨ What PyLibrePortable provides

### 🐍 Portable Python environment

The foundation of the project is a standalone Python environment that can be copied and moved without installing Python on Windows.

The resulting application can be:

* copied to another folder;
* stored on a USB drive or external disk;
* distributed as an archive;
* used without modifying the Windows registry;
* used without adding Python to the system `PATH`;
* kept independent from other Python installations.

The portable environment is based on the official **Python Windows Embeddable Package**.

### 🧩 Optional Python components

The base Python environment is intentionally minimal.

Depending on the needs of your application, the guide explains how to add and configure optional components:

* **External Python modules** — enable third-party packages in the embedded environment.
* **Pip** — install Python libraries directly into the portable environment.
* **External libraries** — such as Pillow, Requests, CustomTkinter, and other packages.
* **Tkinter** — add the files required for native Python GUI applications.
* **Visual Studio Code** — use the portable Python interpreter directly for development and debugging.

Python Portable is the only required foundation. Everything else is added according to the needs of the project.

### ▶️ Simple batch launchers

If a native executable is not necessary, the project can remain extremely simple.

The guide provides batch scripts for:

* launching the application without a console;
* launching in debug mode with a console;
* installing Python libraries with Pip.

For example:

```text
Lancer_App.bat
      ↓
Python Portable
      ↓
Your Python Application
```

This keeps the project completely transparent and requires no native launcher.

### 🖥️ Professional native Windows launcher

For applications intended to look and behave more like a traditional Windows application, PyLibrePortable provides a method for creating a **native `.exe` launcher in C#**.

The launcher sits in front of the portable Python environment:

```text
YourApplication.exe
        │
        ▼
 Native Windows Launcher
        │
        ▼
 Portable Python
        │
        ▼
 Your Python Application
```

The Python application itself remains outside the executable as normal Python files.

The launcher can provide a more professional Windows experience with features including:

* application icon and Windows executable metadata;
* native application identity through `AppUserModelID`;
* proper taskbar grouping;
* optional splash screen;
* splash screen synchronization with the Python application;
* console or GUI-only execution modes;
* management of single or multiple instances;
* support for Python `multiprocessing`;
* forwarding command-line arguments to Python through `sys.argv`;
* communication between the native launcher and Python through environment variables;
* launch and crash logging;
* automatic log history cleanup;
* portable relative paths;
* execution directly from the included Python environment;
* no Python installation or system configuration required on the target machine.

The launcher is designed as a **native entry point for the portable Python application**, rather than as a replacement for the Python source itself.

### 🖼️ Optional visual customization

The professional launcher can optionally use:

* a custom Windows `.ico` application icon;
* a custom PNG splash screen.

The splash screen is handled by the native launcher and can remain visible while Python initializes, then be closed by the application when initialization is complete.

These features are optional: a project can use the portable Python environment alone, batch launchers, or the full native launcher experience.

## 🔓 Open, accessible and editable

PyLibrePortable is designed so that the application remains understandable and modifiable.

A possible project structure is:

```text
MyProject/
├── python_env/
│   ├── python.exe
│   ├── pythonw.exe
│   ├── python3xx.dll
│   ├── python3xx.zip
│   ├── Lib/
│   ├── Scripts/
│   └── ...
│
├── logs/
│
├── src/
│   ├── main.py
│   └── ...
│
├── assets/
│   ├── app.ico
│   └── splash.png
│
├── Lancer_App.(bat ou exe)
├── Debug.(bat ou exe)
├── Installer_Libs.bat
│
├── Launcher.cs
└── Build_launcher.bat
```

Depending on your project and the features you choose, some files and folders may not be present.

The Python source remains directly accessible.

The portable Python runtime remains directly accessible.

The native launcher is also built from its own source code and can be rebuilt independently.

## ⚙️ Optional components, according to your project

PyLibrePortable does not require every component for every application.

The basic progression is:

```text
Python Portable
      │
      ├── Optional: Pip
      │       └── Optional: External Libraries
      │
      ├── Optional: External Modules
      │
      ├── Optional: Tkinter
      │
      ├── Optional: VS Code
      │
      ├── Optional: Batch Launchers
      │
      └── Optional: Native Windows Launcher
              ├── Icon
              ├── Splash Screen
              ├── Single Instance
              ├── Multiprocessing Support
              ├── Arguments
              ├── Logs
              └── Windows Integration
```

**Python Portable is the only mandatory foundation.**

Everything else depends on what your application requires and how you want to use or distribute it.

## 📚 Documentation

The complete V1.1 guide explains the process step by step, including:

1. Python Portable with the Windows Embeddable Package
2. Activation of external modules
3. Pip installation
4. External Python libraries
5. Tkinter integration
6. Visual Studio Code configuration
7. Batch launch and maintenance scripts
8. Native Windows `.exe` launcher
9. Splash screen integration
10. Final project structure and distribution

**[Installation and Usage Guide — Version 1.1 (FR)](docs/Guide_Python_Portable_v1.1_FR.md)**

## 📄 License

This project, its guides, and its source codes are distributed under the **GNU General Public License v3.0 (GPLv3)**. 
For more details, please check the [LICENSE](./LICENSE) file at the root of the repository.
