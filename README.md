# nuke_python_snippets

### Get selected node type
```
print(nuke.selectedNode().Class())
```

### Reload Scripts
```
importlib.reload(script)
```

### Performance Timers
Useful for determining where things are slowing down
```
nuke.startPerformanceTimers()
nuke.stopPerformanceTimers()
```

### Add Multi-Line string knob
```
x = nuke.toNode("NodeName")
k = nuke.Multiline_Eval_String_Knob('knobName','knobLabel')
x.addKnob(k)
```

### Get node dependencies
```
def get_dependencies(node):
    dependencies = []
    nodes_to_process = node.dependencies()
    while nodes_to_process:
        node = nodes_to_process.pop(0)
        if node not in dependencies:
            dependencies.append(node)
            nodes_to_process.extend(node.dependencies())
    return dependencies

all_dependencies = get_dependencies(n)

for n in all_dependencies:
    print(n.name())
```

### Get Node Graph Position
```
def getNodeGraphPosition():
    dd = nuke.createNode('NoOp')
    x = dd.xpos()
    y = dd.ypos()
    nuke.delete(dd)
    return(x,y)
```

### Delete all Viewers including within Groups
```
def nuke_viewers_delete_all():
    allViewers = [x for x in nuke.allNodes(recurseGroups = True) if x.Class()=="Viewer"]
    for viewer in allViewers:
        nuke.delete(viewer)
```

### Open with RV (assuming RV is installed)
```
def open_with_rv():
    import subprocess

    selected = nuke.selectedNodes()
    for i in selected:
        if i.knob('file'):
            path = i.knob('file').value()
            path = path.replace('%d',"#")
            cmd = 'rv '+path
            subprocess.Popen(cmd, shell=True)
```

### Set what Read nodes should when there are missing frames
```
def set_missing_frames():
    selected_nodes = nuke.selectedNodes()
    options = ['error', 'black', 'checkerboard', 'nearest frame']
    dialogChoice = nuke.choice("Missing Frames", "", options, default=0)

    if selected_nodes:
        for n in selected_nodes:
            if n.Class()=="Read" or n.Class()=="DeepRead":
                n['on_error'].setValue(dialogChoice)
    else:
        all_nodes =  [x for x in nuke.allNodes(recurseGroups = False)]
        for n in all_nodes:
            if n.Class()=="Read" or n.Class()=="DeepRead":
                n['on_error'].setValue(dialogChoice)
```

### Display Nuke icons
```
from PySide2 import QtCore, QtGui, QtWidgets

class AllIcons(QtWidgets.QWidget):
    def __init__(self, resource):
        super(AllIcons, self).__init__()
        self.setLayout(QtWidgets.QVBoxLayout())
        # put a scrollbar first
        body_container = QtWidgets.QWidget()
        self.body = QtWidgets.QVBoxLayout()
        # self.body.addStretch(1)
        body_container.setLayout(self.body)

        # Scroll Area Properties
        scroll = QtWidgets.QScrollArea()
        scroll.setWidgetResizable(True)
        scroll.setWidget(body_container)
        for icon in QtCore.QDir(resource).entryList():
            image_label = QtWidgets.QLabel()
            image_label.setPixmap(QtGui.QPixmap("{}/{}".format(resource, icon)))
            text_label = QtWidgets.QLabel(icon)
            self.body.addWidget(image_label)
            self.body.addWidget(text_label)
            
        self.layout().addWidget(scroll)
print("loading nuke icons")
w = AllIcons(u':/qrc/images')
w.show()
```
