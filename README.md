from phBot import *
import QtBind
import struct
import json
import os

pName = 'TargetMachine'
pVersion = '1.0.0'
pUrl = 'https://raw.githubusercontent.com/YourRepo/TargetMachinePlugin.py'

# ______________________________ Initializing ______________________________ #

# globals
character_data = None
target_data = []

# Initializing GUI
gui = QtBind.init(__name__, pName)

_x = 6
_y = 9

QtBind.createLabel(gui, '*  Target List', _x, _y)
_y += 20
tbxTargetName = QtBind.createLineEdit(gui, "", _x, _y, 100, 20)
QtBind.createButton(gui, 'btnAddTarget_clicked', "Add Target", _x + 101, _y - 2)
_y += 20
lvwTargets = QtBind.createList(gui, _x, _y, 176, 60)
_y += 60
QtBind.createButton(gui, 'btnRemTarget_clicked', "Remove Target", _x + 49, _y - 2)

# ______________________________ Methods ______________________________ #

# Return folder path
def get_path():
    return get_config_dir() + pName + "\\"

# Return character configs path (JSON)
def get_config():
    return get_path() + character_data['server'] + "_" + character_data['name'] + ".json"

# Check if character is in-game
def is_joined():
    global character_data
    character_data = get_character_data()
    if not (character_data and "name" in character_data and character_data["name"]):
        character_data = None
    return character_data

# Save all config
def save_configs():
    if is_joined():
        data = {}
        data["Targets"] = QtBind.getItems(gui, lvwTargets)
        with open(get_config(), "w") as f:
            f.write(json.dumps(data, indent=4, sort_keys=True))
        log(f"Plugin: {pName} configs have been saved")

# Load saved configs
def load_configs():
    if is_joined():
        if os.path.exists(get_config()):
            with open(get_config(), "r") as f:
                data = json.load(f)
            if "Targets" in data:
                for target in data["Targets"]:
                    QtBind.append(gui, lvwTargets, target)

# Check if target is already in the list
def string_in_list(vString, vList, ModeSensitive=False):
    if not ModeSensitive:
        vString = vString.lower()
    for i in range(len(vList)):
        if not ModeSensitive:
            vList[i] = vList[i].lower()
        if vList[i] == vString:
            return True
    return False

# Add target to the list
def btnAddTarget_clicked():
    if character_data:
        player = QtBind.text(gui, tbxTargetName)
        if player and not string_in_list(player, QtBind.getItems(gui, lvwTargets)):
            QtBind.append(gui, lvwTargets, player)
            save_configs()
            QtBind.setText(gui, tbxTargetName, "")
            log(f'Target added: {player}')
        else:
            log("Invalid or duplicate target")

# Remove target from the list
def btnRemTarget_clicked():
    if character_data:
        selected_item = QtBind.text(gui, lvwTargets)
        if selected_item:
            QtBind.remove(gui, lvwTargets, selected_item)
            save_configs()
            log(f"Target removed: {selected_item}")

# ______________________________ Events ______________________________ #

# Called when the character enters the game world
def joined_game():
    load_configs()

# All packets received from game server will be passed to this function
def handle_joymax(opcode, data):
    # Add logic for handling packets for the target system if needed
    return True

# Plugin loaded
log(f'Plugin: {pName} v{pVersion} successfully loaded')

# Check folder existence
if os.path.exists(get_path()):
    load_configs()
else:
    os.makedirs(get_path())
    log(f'Plugin: {pName} folder has been created')
