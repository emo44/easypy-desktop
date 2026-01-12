EasyPyDesktop Plugin SDK - Official Guide

Welcome to the EasyPyDesktop Plugin Development Kit.
This guide explains how to create extensions (.py files) to add new widgets and functionality to the editor.

    📂 LOCATION:
    All plugins must be placed inside the plugins folder of your EasyPyDesktop installation.

1. Anatomy of a Plugin

A valid plugin is a single Python file (.py) containing 5 specific sections:

    IMPORTS: Standard Python libraries and PySide6 widgets.

    METADATA: Information about the author and description.

    GENERATORS: Functions that generate the Python code string.

    SNIPPETS: Configuration for the Wizard dialogs (inputs).

    WIDGET CLASS: The visual element displayed on the canvas.

    INITIALIZATION: Registration function.

2. Section by Section Guide
[SECTION A] Imports

You must import PySide6 widgets for the visual part.
code Python

    
from PySide6.QtWidgets import QPushButton, QLabel, QMessageBox
from PySide6.QtCore import Qt

  

[SECTION B] Metadata

A dictionary named PLUGIN_METADATA. It supports multiple languages (en, es, de, ru, zh).
code Python

    
PLUGIN_METADATA = {
    'name': {
        'en': 'My Tool', 
        'es': 'Mi Herramienta'
    },
    'description': {
        'en': 'Does something cool.', 
        'es': 'Hace algo genial.'
    },
    'version': '1.0',
    'author': 'YourName'
}

  

[SECTION C] Generators (The Logic)

Concept: You are NOT writing code that runs immediately. You are writing a Python function that returns a STRING. That string is the code that will be saved in the final EXE.

Crucial Function: find_widget("ID")
In your generated string, always use find_widget("ID") to reference other objects in the app.

Generator Structure:
code Python

    
def my_generator_func(params):
    # 'params' is a dictionary containing the user inputs from the Wizard.
    target_id = params.get('target_widget', '')
    
    # Return a formatted string (f-string) containing the code.
    return f'''
    # This code runs inside the final App
    w = find_widget("{target_id}")
    if w:
        w.setText("Hello World")
    '''

  

[SECTION D] Snippets (The Wizard UI)

A dictionary named PLUGIN_SNIPPETS. This defines the pop-up windows when a user double-clicks your widget.
code Python

    
PLUGIN_SNIPPETS = {
    "unique_command_id": {
        "labels": {"en": "Wizard Title", "es": "Título del Mago"},
        "params": [
            # List of inputs
            {
                "name": "variable_name", 
                "labels": {"en": "Label Text"}, 
                "type": "text", 
                "default": "value"
            }
        ],
        "generator": my_generator_func # Links to the function in Section C
    }
}

  

Available Input Types (type):

    text: Simple text input field. Returns a string.

    bool: A Checkbox. Returns True or False.

    widget_combo: A dropdown list containing IDs of all widgets in the project. Returns a string (Widget ID).

    code_block: Large text area for Python code.

    hybrid: A tabbed input allowing the user to choose between Fixed Text or Reading from another Widget.

    dynamic_list: A list where users can add multiple rows (useful for databases or mapping).

[SECTION E] The Visual Widget

A class inheriting from a PySide6 Widget (usually QPushButton, QLabel, or QFrame). This is what appears on the Editor Canvas.
code Python

    
class MyVisualButton(QPushButton):
    def __init__(self, text="My Button", parent=None):
        super().__init__(text, parent)
        # Style it to look distinct
        self.setStyleSheet("background-color: orange; color: black; border: 1px solid #333;")

  

[SECTION F] Initialization

The entry point called by EasyPyDesktop to load your plugin.
code Python

    
def initialize_plugin(mw):
    # mw: Reference to the MainWindow
    # register_plugin_tool(Unique_ID, WidgetClass, DefaultWidth, DefaultHeight)
    mw.register_plugin_tool('my_tool_id', MyVisualButton, 120, 40)

  

3. Complete Examples
Example 1: Beginner - "Hello World" Button

Save this as hello_plugin.py
code Python

    
from PySide6.QtWidgets import QPushButton, QMessageBox

PLUGIN_METADATA = {
    'name': {'en': 'Hello Button', 'es': 'Botón Hola'},
    'description': {'en': 'Shows a message.', 'es': 'Muestra un mensaje.'},
    'version': '1.0',
    'author': 'Newbie'
}

# 1. Generator
def gen_show_hello(params):
    msg = params.get('message_text', 'Hello World')
    return f'''
    from PySide6.QtWidgets import QMessageBox
    QMessageBox.information(None, "Message", "{msg}")
    '''

# 2. Wizard
PLUGIN_SNIPPETS = {
    "show_msg_action": {
        "labels": {"en": "Show Message", "es": "Mostrar Mensaje"},
        "params": [
            {"name": "message_text", "labels": {"en": "Text:"}, "type": "text", "default": "Hello!"}
        ],
        "generator": gen_show_hello
    }
}

# 3. Visual Widget
class HelloBtn(QPushButton):
    def __init__(self, text="Hello", parent=None):
        super().__init__(text, parent)
        self.setStyleSheet("background-color: #00AA00; color: white; font-weight: bold;")

# 4. Registration
def initialize_plugin(mw):
    # 'clicked' is the default signal for buttons
    mw.register_plugin_tool('hello_btn', HelloBtn, 100, 30, default_signal='clicked')

  

Example 2: Intermediate - Web Opener

Opens a URL in the browser.
code Python

    
import webbrowser
from PySide6.QtWidgets import QPushButton

PLUGIN_METADATA = {
    'name': {'en': 'Web Link', 'es': 'Enlace Web'},
    'description': {'en': 'Opens URL.', 'es': 'Abre URL.'},
    'version': '1.0', 'author': 'User'
}

# Generator
def gen_open_url(params):
    url = params.get('url', 'https://google.com')
    return f'''
    import webbrowser
    webbrowser.open("{url}")
    '''

# Wizard
PLUGIN_SNIPPETS = {
    "open_url_cmd": {
        "labels": {"en": "Open Website"},
        "params": [
            {"name": "url", "labels": {"en": "URL Address:"}, "type": "text", "default": "https://"}
        ],
        "generator": gen_open_url
    }
}

# Visual Widget
class WebButton(QPushButton):
    def __init__(self, text="Open Web", parent=None):
        super().__init__(text, parent)
        self.setStyleSheet("background: #007acc; color: white;")

# Registration
def initialize_plugin(mw):
    mw.register_plugin_tool('web_opener', WebButton, 120, 30)

  

4. Tips for Developers

    Safe Interactions: Use find_widget("ID") in your generators to interact with other inputs, labels, or elements in the scene.

    Safe Imports: If your plugin requires external libraries (like pandas or requests), import them INSIDE the generator string (f''' ... '''), not at the top of your plugin file. This prevents the editor from crashing if the user doesn't have them installed.

    Mandatory Init: The initialize_plugin function is mandatory. Without it, the plugin will be ignored.

    Styling: You can use CSS in setStyleSheet to make your widget look unique in the editor.
