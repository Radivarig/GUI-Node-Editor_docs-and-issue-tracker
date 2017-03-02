# GUI Node Editor

## Introduction
- GUI Node Editor is an API for creating custom GUI/GUILayout node based editors that work both in editor window and exported in builds.
- Intended usage is to derrive from `Node` class that holds the data and `Node_Window` class that displays that data.
- Special node role is the menu node whose window derrives from `NodeWindow_Menu`. It lists all nodes intended to be used in the node editor and is spawned on right click.
- Each node holds information about its docks (left input and right output boxes) and each dock holds information about other docks it is connected to, its type etc.
- Role of each node is meant to be implemented however you like.
- You can check the demo scene which is the one from WebGL demo, then start from scratch by following the **Getting Started** instructions below.

## Features

##### Menu
- <Right Click> on background spawns a menu that creates your node instances.
- <Right Click> on node title fills `clickedWindow` field which you can use for node specific menus.

##### Selecting
- <Left Click Drag> draws a selection box that adds nodes it overlaps to `nodeEditor.selectedWindows`.
- Selected nodes have a highlighted border, `onNormal` texture is used.
- <Shift + Click> on node toggles selection for that node.
- <Click> on background clears selection.

##### Dragging
- <Left Click Drag> of node title will drag all selected windows.

##### Panning
- <Right Click Drag> will pane all windows and the grid.
- Deconnection and selection are not canceled while panning.

##### Deleting
- <Del> deletes all selected nodes.

##### Connecting
- Connection is shown as a *bezier curve* that is colored green if `node.isTriggered` is set to `true`.
- Only docks whose types match can be connected.
- While connecting, these docks be highlighted **green**.
- If the dock does not match the type but has a sibling dock that does, it will be highlighted **yellow** and connecting will redirect the connection to that sibling dock.
- <Right Click> or <Left Double Click> on background cancels connecting.

##### Deconnecting
- <Right Click> on a dock starts deconnecting.
- Continuous <Right Clicking> on the same dock will toggle trough deconnecting endpoints.

##### Sizing
- Window height is automatically calculated by default.
- Both width and height can be set manually (see API).

##### Renaming
- <Double Left Click> on node title switches the title to a textField for renaming.

##### Minimap
- Shown while dragging or panning.
- It draws nodes position, connections and title in reference to the screen.

##### Grid
- Moves with panning.

#### Snapping
- When dragging ends, windows position is snapped to grid.

##### Styling
- Visuals can be set from the NodeEditor inspector config like `GUISkin`, background, grid and dock textures, connection colors etc.
- Window color can be set by `nodeWindow.backgroundColor`.

##### Popups
- GUI Dropdown used to switch enums.
- Can be drawn outside of the parent area where its button is rendered.

##### Tooltip
- Calling `DrawTooltip ("string")` right after a GUI element detects last rect and shows a tooltip when that rect is hovered.
- All docks have a tooltip showing their type.

##### Runtime
- Attaching **RuntimeNodeEditor.cs** on the gameObject holding **NodeEditor.cs** will render the editor in play mode.

##### Serialization
- Save files are located in **/Resources/${editorName}Saves/**.
- Third party MIT licenced **FullSerializer** https://github.com/jacobdufault/fullserializer is used to serialize the editor to string.
- Example of save/load using `System.IO` is located in **EditorWindowNodeEditor.cs**.
- In cases where `System.IO` is not supported like for WebGL, you can still read from `Resources` folder and use any other approach to serialize the generated string.

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
        Init (position, nodeWindow: new NodeWindow_Menu_Example ());
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
        Init (position, nodeWindow: new NodeWindow_Example (), title: "Example");

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

## Contact
Email: `reslav.hollos@gmail.com`.

## Feedback
- For bug reports please check the issue tracker at https://github.com/Radivarig/GUI-Node-Editor_docs-and-issue-tracker/issues, you can open a new issue there or contact me directly via email.
- All suggestions are appreciated and will improve the quality of this asset.

## API

API is generated with **Doxygen** and is located in a separate file `GUINodeEditor_${version}_API.pdf`.

> Keep in mind that this is the first release and some things might be missing. If you'd like to have something added just drop me an email with a suggestion.

---
Cheers, Reslav
