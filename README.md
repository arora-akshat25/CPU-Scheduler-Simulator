# CPU-Scheduler-Simulator

import tkinter as tk
from tkinter import ttk, messagebox
import matplotlib.pyplot as plt
from tkinter import simpledialog

# Application Window 
app = tk.Tk()
app.title("CPU Scheduler Simulator")
app.geometry("800x500")
app.configure(bg="#2C3E50")

# Styling
style = ttk.Style()
style.configure("TButton", font=("Arial", 12), padding=10)
style.configure("TEntry", font=("Arial", 12), padding=5)
style.configure("TLabel", font=("Arial", 12), background="#2C3E50", foreground="white")

# Title
title = tk.Label(app, text="CPU Scheduling Simulator", font=("Arial", 16, "bold"), bg="#2C3E50", fg="white")
title.pack(pady=10)

# Dropdown Menu
tk.Label(app, text="Choose a Scheduling Algorithm:", bg="#2C3E50", fg="white").pack(pady=5)
algorithm_choice = ttk.Combobox(app, values=["First Come First Serve", "Shortest Job First", "Round Robin", "Priority Scheduling"], state="readonly")
algorithm_choice.pack(pady=5)
algorithm_choice.set("First Come First Serve")

# Frame for entering process details 
input_section = tk.Frame(app, bg="#D3D3D3", padx=20, pady=20, bd=2, relief="ridge")
input_section.pack(pady=10, fill="x")

tk.Label(input_section, text="Enter Process Details", font=("Arial", 14, "bold"), bg="#D3D3D3", fg="black").pack()

# Input Fields
tk.Label(input_section, text="Process ID:", bg="#D3D3D3", fg="black").pack(pady=2)
pid_entry = ttk.Entry(input_section)
pid_entry.pack(pady=2)

tk.Label(input_section, text="Arrival Time:", bg="#D3D3D3", fg="black").pack(pady=2)
arrival_entry = ttk.Entry(input_section)
arrival_entry.pack(pady=2)

tk.Label(input_section, text="Burst Time:", bg="#D3D3D3", fg="black").pack(pady=2)
burst_entry = ttk.Entry(input_section)
burst_entry.pack(pady=2)

tk.Label(input_section, text="Priority:", bg="#D3D3D3", fg="black").pack(pady=2)
priority_entry = ttk.Entry(input_section)
priority_entry.pack(pady=2)

# Store processes
process_list = []

# Function to add process
def add_process():
    pid = pid_entry.get()
    arrival = arrival_entry.get()
    burst = burst_entry.get()
    priority = priority_entry.get()

    if not pid or not arrival or not burst or not priority:
        messagebox.showerror("Error", "Please enter all values!")
        return

    try:
        arrival = int(arrival)
        burst = int(burst)
        priority = int(priority)  
        process_list.append((pid, arrival, burst, priority))
        process_table.insert("", "end", values=(pid, arrival, burst, priority))  
    except ValueError:
        messagebox.showerror("Error", "Arrival, Burst Time, and Priority must be integers!")
        return

    pid_entry.delete(0, tk.END)
    arrival_entry.delete(0, tk.END)
    burst_entry.delete(0, tk.END)
    priority_entry.delete(0, tk.END)

# Button for "Add Process"
add_button = tk.Button(input_section, text="Add Process", command=add_process, 
                       bg="red", fg="white", font=("Arial", 12, "bold"), 
                       relief="flat", bd=5, padx=10, pady=5)
add_button.pack(pady=5)

# Table to display processes
columns = ("PID", "Arrival Time", "Burst Time", "Priority")  
process_table = ttk.Treeview(app, columns=columns, show="headings", height=5)

for col in columns:
    process_table.heading(col, text=col)
    process_table.column(col, width=120)  

process_table.pack(pady=10)

# Function to execute FCFS scheduling
def run_fcfs():
    if not process_list:
        messagebox.showerror("Error", "No processes added!")
        return

    sorted_processes = sorted(process_list, key=lambda x: x[1])
    completion_time = []
    waiting_time = []
    turnaround_time = []

    time = 0
    for pid, arrival, burst, _ in sorted_processes:
        if time < arrival:
            time = arrival

        start_time = time
        end_time = start_time + burst
        completion_time.append((pid, start_time, end_time))

        tat = end_time - arrival
        wt = tat - burst
        turnaround_time.append((pid, tat))
        waiting_time.append((pid, wt))

        time = end_time  

# Pop up for FCFS
    result_window = tk.Toplevel(app)
    result_window.title("FCFS Results")
    result_window.geometry("400x300")
    result_window.configure(bg="black")  

    result_text = "Process\tWaiting Time\tTurnaround Time\n"
    total_wt, total_tat = 0, 0
    for pid, wt in waiting_time:
        tat = dict(turnaround_time)[pid]
        result_text += f"{pid}\t{wt}\t\t{tat}\n"
        total_wt += wt
        total_tat += tat

    avg_wt = total_wt / len(process_list)
    avg_tat = total_tat / len(process_list)
    result_text += f"\nAverage Waiting Time: {avg_wt:.2f}\nAverage Turnaround Time: {avg_tat:.2f}"

    result_label = tk.Label(result_window, text=result_text, font=("Arial", 12), bg="black", fg="white")
    result_label.pack(pady=10, padx=10)

    draw_gantt_chart(completion_time)

# Pop up for SJF
def show_results(waiting_time, turnaround_time):
    result_window = tk.Toplevel(app)
    result_window.title("Scheduling Results")
    result_window.geometry("400x200")
    result_window.configure(bg="black")

    avg_wt = sum(wt for _, wt in waiting_time) / len(waiting_time)
    avg_tt = sum(tt for _, tt in turnaround_time) / len(turnaround_time)

    tk.Label(result_window, text=f"Average Waiting Time: {avg_wt:.2f}",
             font=("Helvetica", 12), bg="black", fg="white").pack(pady=10)
    tk.Label(result_window, text=f"Average Turnaround Time: {avg_tt:.2f}",
             font=("Helvetica", 12), bg="black", fg="white").pack(pady=10)
 
def run_sjf():
    global processes
    processes.sort(key=lambda x: (x[1], x[2]))  

    current_time = 0
    waiting_time = []
    turnaround_time = []
    completed_processes = []
    
    ready_queue = []
    remaining_processes = processes.copy()

    while remaining_processes or ready_queue:
        while remaining_processes and remaining_processes[0][1] <= current_time:
            ready_queue.append(remaining_processes.pop(0))

        if ready_queue:
            ready_queue.sort(key=lambda x: x[2])  
            process = ready_queue.pop(0)

            pid, arrival, burst, priority = process
            start_time = current_time
            end_time = start_time + burst

            waiting_time.append((pid, start_time - arrival))
            turnaround_time.append((pid, end_time - arrival))

            completed_processes.append((pid, start_time, end_time))
            current_time = end_time
        else:
            current_time += 1 

    show_results(waiting_time, turnaround_time)
    draw_gantt_chart(completed_processes)

# Function to execute RR scheduling
def run_round_robin():
    if not process_list:
        messagebox.showerror("Error", "No processes added!")
        return

    try:
        quantum = int(simpledialog.askstring("Round Robin", "Enter Time Quantum:"))
    except ValueError:
        messagebox.showerror("Error", "Invalid quantum value!")
        return

    queue = process_list[:]
    time = 0
    completion_time = []
    remaining_burst = {pid: burst for pid, arrival, burst, _ in queue}

    while queue:
        data = queue.pop(0)

        if len(data) == 4:  
            pid, arrival, burst, _ = data  
        else:  
            pid, arrival, burst = data
        if remaining_burst[pid] > quantum:
            start = time
            time += quantum
            end = time
            remaining_burst[pid] -= quantum
            queue.append((pid, arrival, remaining_burst[pid]))
        else:
            start = time
            time += remaining_burst[pid]
            end = time
            remaining_burst[pid] = 0
        completion_time.append((pid, start, end))

    draw_gantt_chart(completion_time)

# Function to execute Priority scheduling
def run_priority_scheduling():
    if not process_list:
        messagebox.showerror("Error", "No processes added!")
        return

    sorted_processes = sorted(process_list, key=lambda x: x[3])  
    completion_time = []
    waiting_time = []
    turnaround_time = []

    time = 0
    for pid, arrival, burst, priority in sorted_processes:
        if time < arrival:
            time = arrival

        start_time = time
        end_time = start_time + burst
        completion_time.append((pid, start_time, end_time))

        tat = end_time - arrival
        wt = tat - burst
        turnaround_time.append((pid, tat))
        waiting_time.append((pid, wt))

        time = end_time  

    show_results(waiting_time, turnaround_time)
    draw_gantt_chart(completion_time)

# Function to draw Gantt Chart
import matplotlib.pyplot as plt
import numpy as np

def draw_gantt_chart(completion_time):
    fig, ax = plt.subplots(figsize=(10, 3))  
    
    y_labels = []
    start_points = []
    burst_durations = []
    colors = ["#FF5733", "#33FF57", "#3357FF", "#F39C12", "#8E44AD", "#E74C3C"]  
    
    for i, (pid, start, end) in enumerate(completion_time):
        y_labels.append(pid)
        start_points.append(start)
        burst_durations.append(end - start)

    bars = ax.barh(y_labels, burst_durations, left=start_points, color=colors[:len(y_labels)], edgecolor='black', height=0.6)

    for bar, pid, start, end in zip(bars, y_labels, start_points, burst_durations):
        ax.text(start + (end / 2), bar.get_y() + 0.25, f"{pid}", ha='center', va='center', fontsize=12, color='white', fontweight='bold')

    for i, (pid, start, end) in enumerate(completion_time):
        ax.text(start, i, f"{start}", verticalalignment='bottom', fontsize=10, fontweight='bold')
        ax.text(start + (end - start), i, f"{start + (end - start)}", verticalalignment='bottom', fontsize=10, fontweight='bold')

    ax.set_xlabel("Time")
    ax.set_title("Enhanced Gantt Chart - CPU Scheduling")
    ax.grid(True, linestyle="--", alpha=0.5)
    ax.set_yticks(range(len(y_labels)))
    ax.set_yticklabels(y_labels, fontweight="bold")

    plt.show()

# delete
def delete_selected_process():
    selected_item = process_table.selection()  
    if not selected_item:
        messagebox.showerror("Error", "Please select a process to delete!")
        return

    for item in selected_item:
        values = process_table.item(item, "values")  
        process_list[:] = [p for p in process_list if p[0] != values[0]]  
        process_table.delete(item) 

delete_button = ttk.Button(app, text="Delete Selected Process", command=delete_selected_process)
delete_button.pack(pady=5)

# Run Simulation Button
def run_simulation():
    selected_algo = algorithm_choice.get()
    if selected_algo == "First Come First Serve":
        run_fcfs()
    elif selected_algo == "Shortest Job First":
        run_sjf()
    elif selected_algo == "Round Robin":
        run_round_robin()
    elif selected_algo == "Priority Scheduling":
        run_priority_scheduling()
    else:
        messagebox.showerror("Error", "Please select a valid algorithm!")
simulate_button = ttk.Button(app, text="Run Simulation", command=run_simulation)
simulate_button.pack(pady=15)

# Start the Application
app.mainloop()
