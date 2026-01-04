EasyPy Desktop SDK - Plugin Development Guide v1.0

This document explains how to create, register, and integrate dynamic plugins for EasyPy Desktop.
1. Overview

EasyPy Desktop plugins are standalone .py files located in the /plugins directory. The IDE scans this folder at startup, imports each module, and calls its initialization function.
2. Plugin Structure

Every plugin must contain three core elements:

    PLUGIN_METADATA: Information about the plugin.

    PLUGIN_SNIPPETS: Logic assistants (Wizards) for the Action Editor and AI.

    initialize_plugin(): The entry point function.

A. Metadata Example
    
PLUGIN_METADATA = {
    'name': {
        'en': 'Database Manager',
        'es': 'Gestor de Base de Datos'
    },
    'description': {
        'en': 'Simple SQLite integration for your projects.',
        'es': 'Integración sencilla de SQLite para tus proyectos.'
    },
    'version': '1.0',
    'author': 'Your Name'
}

  

B. Snippets (Action Wizards)

Snippets define the commands that will appear in the Action Editor and what the AI Assistant will recommend.

Types of parameters supported:

    'text': Simple text input.

    'widget_combo': A dropdown to select any widget from the current page.

    'hybrid': A tabbed input (Fixed text OR Value from another widget).

Example:
    
PLUGIN_SNIPPETS = {
    'db_save': {
        'labels': {'en': 'DB: Save Record', 'es': 'BD: Guardar Registro'},
        'params': [
            {'name': 'table', 'label': 'Table Name', 'type': 'text', 'default': 'users'},
            {'name': 'val', 'label': 'Value', 'type': 'hybrid'}
        ],
        'generator': lambda v: f'find_widget("Sqlite_1").sqlite_insert("{v["table"]}", {v["val"]})'
    }
}

  

3. The Custom Widget Class

If your plugin adds a new visual component to the Canvas, you must define a class that inherits from a PySide6 widget.

    
from PySide6.QtWidgets import QPushButton

class MyPluginWidget(QPushButton):
    def __init__(self, parent=None):
        super().__init__(parent)
        self.setText("Plugin Widget")
        self.setStyleSheet("background-color: blue; color: white;")

    def my_custom_method(self, data):
        print(f"Data received: {data}")

  

4. Initialization

The initialize_plugin(mw) function is called by the IDE. Here you register your widget in the Toolbox.

    
def initialize_plugin(mw):
    # Register the tool
    # mw.register_plugin_tool(id, class, width, height, default_signal)
    mw.register_plugin_tool('my_custom_tool', MyPluginWidget, 120, 40, 'clicked')

  

5. IDE Integration Summary

    Create a file: plugins/my_plugin.py.

    Define your Logic/Widget class.

    Define PLUGIN_METADATA and PLUGIN_SNIPPETS.

    Call mw.register_plugin_tool inside initialize_plugin(mw).

    Restart EasyPy Desktop.

6. Accessing Plugins via Script

In the EasyPy Action Editor, users will access your plugin using:

    
# Accessing by ID
obj = find_widget("MyPlugin_1")
obj.my_custom_method("Hello World")

  

7. Global Context (ctx)

All scripts in EasyPy run within a context (ctx) that includes:

    find_widget(id): Find any widget on the page.

    goto_page(name): Navigation.

    close_application(): Exit the app.

    QMessageBox: System alerts.

    requests: HTTP capability (if installed).

    sqlite3: Database capability.
