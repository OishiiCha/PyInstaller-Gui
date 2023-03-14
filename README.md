# PyInstaller-Gui

```
import PySimpleGUI as sg
import subprocess
import os
from PIL import Image

sg.theme('DarkAmber')
sg.set_options(font=('Arial', 14))

layout = [
    [sg.Text('Python File:'), sg.Input(key='-FILE-', size=(40, 1)), sg.FileBrowse()],
    [sg.Text('Icon File (optional):'), sg.Input(key='-ICON-', size=(40, 1)), sg.FileBrowse()],
    [sg.Text('Include Libraries (optional):'), sg.Input(key='-LIB-', size=(40, 1)), sg.FileBrowse()],
    [sg.Text('Output Folder:'), sg.Input(key='-FOLDER-', size=(40, 1)), sg.FolderBrowse()],
    [sg.Button('Create EXE'), sg.Button('Cancel')],
    [sg.Text('Debug Console:')],
    [sg.Output(size=(80, 10), key='-OUTPUT-')],
]

window = sg.Window('Python to EXE', layout, resizable=False)

while True:
    event, values = window.read()

    if event == sg.WINDOW_CLOSED or event == 'Cancel':
        break

    if event == 'Create EXE':
        pyfile = values['-FILE-']
        icon = values['-ICON-']
        libs = values['-LIB-']
        output_folder = values['-FOLDER-']
        
        if not pyfile:
            sg.popup_error('Please select a Python file')
            continue

        if not output_folder:
            sg.popup_error('Please select an output folder')
            continue

        if icon:
            if not os.path.isfile(icon):
                sg.popup_error('Invalid icon file')
                continue
            
            # Convert the icon file to .ico format
            try:
                img = Image.open(icon)
                img = img.resize((min(img.size[0], 100), min(img.size[1], 100)), Image.ANTIALIAS)
                img.save(os.path.join(output_folder, 'icon.ico'))
                icon = os.path.join(output_folder, 'icon.ico')
            except Exception as e:
                sg.popup_error(f'Error converting icon file: {str(e)}')
                continue

        command = ['pyinstaller', '--onefile', '--distpath', output_folder]
        
        if icon:
            command += ['-i', icon]
        
        if libs:
            if not os.path.isfile(libs):
                sg.popup_error('Invalid libraries file')
                continue
            command += ['--add-data', f'{libs};.']
            
        command.append(pyfile)

        try:
            process = subprocess.Popen(command, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
            stdout, stderr = process.communicate()
            if process.returncode != 0:
                raise subprocess.CalledProcessError(process.returncode, command, stderr)
            else:
                sg.popup('EXE file created successfully!', f'Output: {os.path.join(output_folder, os.path.splitext(os.path.basename(pyfile))[0] + ".exe")}')
        except subprocess.CalledProcessError as e:
            sg.popup_error(f'Error creating EXE file: {e.stderr.decode()}')
        
    window['-OUTPUT-'].update('') # Clear debug console after each run
    window['-OUTPUT-'].print(stdout.decode())
    window['-OUTPUT-'].print(stderr.decode())

window.close()

```
