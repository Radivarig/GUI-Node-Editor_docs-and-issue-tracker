# GUI Node Editor

## Overview
Unity package that provides API for creating custom GUI node based editors that work both in editor window and exported in builds.

## Brief introduction
- Class `Node` holds the data and `NodeWindow` displays it.
- Each node has lists `inputs` and `outputs` which hold left and right connection docks.
- `Dock` has a type field to define which inputs can be connected to which outputs.
- `NodeWindow` has `OnGUI` and `Update` overrides, one for drawing node content and other for node logic.

## Features

##### Menu
- Right click on background spawns a menu that can create your node instances.
- Right click on a node title fills `clickedWindow` field for node specific menus.

##### Selecting
- Left click drag draws a selection box that adds overlapping windows to `nodeEditor.selectedWindows`.
- Selected windows have a highlighted border, `onNormal` texture is used.
- Shift and click toggles selection.
- Background click clears selection.

##### Deleting
- Del deletes all selected nodes.

##### Dragging
- Left click drag of node title will drag all selected windows.

##### Panning
- Right click drag will pane all windows and the grid.
- Deconnection and selection are not canceled while panning.

##### Connecting
- Allowed docks are of the matching type and are highlighted **green**.
- If the dock is not allowed but has a sibling dock that is, it will be highlighted **yellow** and the connection will be redirected to that sibling.
- Connecting is canceled on right click on background.
- Connection is shown as a *bezier curve* that is colored green if `node.isTriggered` is set to true.

##### Deconnecting
- Right click on a dock starts deconnecting.
- Continuing to right click on the same dock will toggle deconnecting endpoints.

##### Sizing
- Window height is automatically calculated by default.
- Both width and height can be set manually.

##### Renaming
- Double left click on title turns it into textField for renaming.

##### Minimap
- Shown while dragging or panning.
- Draws nodes position, connections and title in reference to the screen.

##### Grid
- Moves with panning.

#### Snapping
- When dragging ends, windows position is snapped to grid.

##### Styling
- Custom `GUISkin` can be specified.
- Background, grid and dock textures can be specified.
- Window color is set by `nodeWindow.backgroundColor`.

##### Popups
- Dropdown used to switch enums.
- Can be drawn outside of the parent area where its button is rendered.

##### Tooltip
- Function `Tooltip ("string")` called after GUI element automatically detects last rect.
- All docks have a tooltip showing their type.

##### Runtime
- Attach **RuntimeNodeEditor.cs** on the gameObject holding **NodeEditor.cs**.

##### Serialization
- Third party MIT licenced [FullSerializer](https://github.com/jacobdufault/fullserializer) is used to serialize the editor to string.
- Example of save/load using `System.IO` is used in **EditorWindowNodeEditor.cs**.
- Where `System.IO` is not supported like for WebGL, you can still read from resources and use any approach to serialize the generated string.
- Save files are located in **Resources/${editorName}Saves/**.

## Getting started

We will create three scripts of types:
1. `EditorWindowNodeEditor` (creates a `NodeEditor: MonoBehaviour` that does not depend on `UnityEditor`).
2. `Node` menu node with `NodeWindow_Menu` window.
3. `Node` custom node with `NodeWindow` window.

### 1. Creating a custom node editor

- In this file everything except the *NODE_EDITOR* region is standard code for drawing an `EditorWindow`.
- Save this as **ExampleEditor.cs**:
```cs
// in runtime UnityEditor does not exist so we wrap it in #if
#if UNITY_EDITOR
using UnityEditor;
using UnityEngine;
using Type = System.Type;
using GUINodeEditor;

// EditorWindowNodeEditor inherits from EditorWindow
public class ExampleEditorWindow: EditorWindowNodeEditor {
    // static reference to EditorWindow
    static ExampleEditorWindow editor;

    // name of the menu item (NOTE: has to be hard coded string)
    [MenuItem("Window/Node Editor Demos/Example Node Editor")]
    // called once after opening the window
    static void Init() {
        // setting the window reference
        editor = (ExampleEditorWindow) EditorWindow.GetWindow (typeof (ExampleEditorWindow));
        // title of the window/tab (NOTE: has to be hard coded string)
        editor.titleContent = new GUIContent ("Example");
    }

    #region NODE_EDITOR
    // node editor name to be used to get nodeEditor gameObject
    public override string GetNodeEditorName () { return "ExampleEditor"; }
    // Node menu type as string
    public override Type GetMenuNodeType () { return typeof(Node_Menu_Example); }
    #endregion
}
#endif

```
- It will appear in the menu `Window > NewNodeEditor`, open it and nest as a tab in Unity.

Next, `Node` menu type is needed to be spawned on right click.

### 2. Creating a custom menu

- Save this as **Node_Menu_Example.cs**:
```cs
using UnityEngine;
using GUINodeEditor;

public class Node_Menu_Example: Node {
    public override void Init (Vector2 position) {
        // init with custom nodeWindow, set node reference
        Init (position, nodeWindow: new NodeWindow_Menu_Example (), node: this);
    }
}
public class NodeWindow_Menu_Example: NodeWindow_Menu {
    public override void OnGUI () {
        // clicked on a window title
        if (clickedWindow != null) {
            title = clickedWindow.node.GetType ().ToString ();
            GUILayout.Box ("Specific node menu");
        }
        // clicked on background
        else {
            title = "Add:";
            // creates a new node window at menu location (closes the menu)
            if (GUILayout.Button ("Node Example")) nodeEditor.CreateNewWindow <Node_Example> ();
        }
    }
}
```
- Set `Node_Menu_Example` as return value of `GetMenuNodeType` in **ExampleNodeEditor.cs**.

Next, create custom nodes to add to the menu.

### 3. Creating custom nodes

- Save this as **Node_Example.cs**:
```cs
using UnityEngine;
using GUINodeEditor;

public class Node_Example: Node {
    public override void Init (Vector2 position) {
        // if you do not plan to change the title from OnGUI you can set it here
        Init (position, nodeWindow: new NodeWindow_Example (), node: this, title: "Example");

        // these will create input and output dock
        AddInput (typeof(string), "input_name");
        AddOutput (typeof(string), "output_name");
    }
}
public class NodeWindow_Example: NodeWindow {
    public override void OnGUI () {
        // cast node to the right type
        Node_Example n = (Node_Example)node;
        // change background color
        backgroundColor = Color.blue;

        // get dock references
        DockInput dockInput = n.GetDockInputByName ("input_name");
        DockOutput dockOutput = n.GetDockOutputByName ("output_name");

        // draw first row with dock input and its targets value
        GUILayout.BeginHorizontal ();
        DrawDock (dockInput);
        GUILayout.Label (n.GetFirstTargetValue<string> (dockInput));
        GUILayout.EndHorizontal ();
        
        // draw a second row with the output dock and text field that modify the output value
        GUILayout.BeginHorizontal ();
        dockOutput.value = GUILayout.TextField ((string) dockOutput.value);
        DrawDock (dockOutput);
        GUILayout.EndHorizontal ();
    }
}
```

## Feedback
All bug reports and suggestions are appreciated.
Please check the [issue tracker](https://github.com/Radivarig/GUINodeEditorDocs/issues), either open a new issue or contact me directly via `reslav.hollos@gmail.com`.

## API
todo
