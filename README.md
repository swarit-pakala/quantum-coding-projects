# quantum-coding-projects
import numpy as np
import matplotlib.pyplot as plt
from qutip import Bloch
from tkinter import *

# Initialize parameters
qubit_state = [1, 0]  # Initial qubit state (|0⟩)
phase = 0  # Initial phase
constructive_interference = False  # Initial interference state

# Create a Bloch sphere plot
bloch = Bloch()
bloch.vector_color = ['r']

# Create a tkinter window
root = Tk()
root.title("Interactive Bloch Sphere")

# Create a frame for controls
control_frame = Frame(root)
control_frame.pack(side="right")

# Create a Scale widget for phase
phase_scale = Scale(control_frame, label="Phase", from_=-np.pi, to=np.pi, orient="horizontal", resolution=0.01)


# Function to update the Bloch sphere and phase
def update():
    global phase, constructive_interference
    phase = phase_scale.get()

    # Update the Bloch sphere
    bloch.clear()
    bloch.add_vectors([np.sin(phase), 0, np.cos(phase)])
    bloch.show()

    # Determine interference (constructive or destructive)
    if np.cos(phase) > 0:
        interference_label.config(text="Interference: Constructive")
        constructive_interference = True
    else:
        interference_label.config(text="Interference: Destructive")
        constructive_interference = False


# Pack widgets and set the update function
phase_scale.pack()
phase_scale.set(phase)
phase_scale.bind("<Motion>", lambda event: update())
update_button = Button(control_frame, text="Update", command=update)
update_button.pack()

# Create a label for interference
interference_label = Label(control_frame, text="Interference: N/A")
interference_label.pack()

# Update the initial Bloch sphere
update()

# Create an interactive 3D plot using Matplotlib
fig = plt.figure(figsize=(6, 6))
ax = fig.add_subplot(111, projection='3d')
ax.set_box_aspect([1, 1, 1])

# Create a Bloch sphere grid
u = np.linspace(0, 2 * np.pi, 100)
v = np.linspace(0, np.pi, 100)
x = 1 * np.outer(np.cos(u), np.sin(v))
y = 1 * np.outer(np.sin(u), np.sin(v))
z = 1 * np.outer(np.ones(np.size(u)), np.cos(v))

# Plot Bloch sphere grid
ax.plot_surface(x, y, z, color='c', alpha=0.1, edgecolor='k', rstride=4, cstride=4)

# Create a label to display qubit state
state_label = Label(control_frame, text="Qubit State: |0⟩")
state_label.pack()


# Define a function to handle qubit state movement
def on_click(event):
    global qubit_state
    if event.inaxes == ax:
        # Get the normalized coordinates of the click
        x, y, z = event.xdata, event.ydata, event.zdata
        norm = np.sqrt(x ** 2 + y ** 2 + z ** 2)
        qubit_state = [x / norm, y / norm, z / norm]
        bloch.clear()
        bloch.add_vectors(qubit_state)
        bloch.show()
        update()


# Connect the click event to the function
fig.canvas.mpl_connect('button_press_event', on_click)


# Function to update qubit state and label
def toggle_state():
    global qubit_state
    if qubit_state == [1, 0, 0]:
        qubit_state = [0, 0, 1]
        state_label.config(text="Qubit State: |1⟩")
    else:
        qubit_state = [1, 0, 0]
        state_label.config(text="Qubit State: |0⟩")
    update()


toggle_button = Button(control_frame, text="Toggle Qubit State", command=toggle_state)
toggle_button.pack()

# Start the tkinter main loop
root.mainloop()
