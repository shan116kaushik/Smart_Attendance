# Smart_Attendance Tracker
import tkinter as tk
from tkinter import ttk
import pandas as pd
from datetime import datetime
import os

FILE = "attendance.csv"

# create file if not exists
if not os.path.exists(FILE):
    df = pd.DataFrame(columns=["RollNo", "Date", "Time", "Status"])
    df.to_csv(FILE, index=False)


# -------- Attendance Function --------

def mark_attendance():
    roll = entry.get().strip()

    if roll == "":
        status_label.config(text="Enter Roll Number ⚠️")
        return

    now = datetime.now()
    today = now.strftime("%Y-%m-%d")

    df = pd.read_csv(FILE)

    if ((df["RollNo"] == roll) & (df["Date"] == today)).any():
        status_label.config(text="Already Marked Today")
        return

    new_row = {
        "RollNo": roll,
        "Date": today,
        "Time": now.strftime("%H:%M:%S"),
        "Status": "Present"
    }

    df = pd.concat([df, pd.DataFrame([new_row])])
    df.to_csv(FILE, index=False)

    status_label.config(text="Attendance Marked ✅")
    entry.delete(0, tk.END)


# -------- Report Window --------

def show_report():
    df = pd.read_csv(FILE)

    report_win = tk.Toplevel(root)
    report_win.title("Attendance Report")
    report_win.geometry("500x300")

    tree = ttk.Treeview(report_win)
    tree["columns"] = list(df.columns)
    tree["show"] = "headings"

    for col in df.columns:
        tree.heading(col, text=col)
        tree.column(col, width=100)

    for _, row in df.iterrows():
        tree.insert("", "end", values=list(row))

    tree.pack(expand=True, fill="both")

    # summary
    count = df["RollNo"].value_counts()
    summary = tk.Label(report_win,
                       text=f"Attendance Count:\n{count.to_string()}",
                       justify="left")
    summary.pack(pady=10)


# -------- GUI --------

root = tk.Tk()
root.title("Smart Attendance System")
root.geometry("360x260")

title = tk.Label(root, text="Smart Attendance", font=("Arial", 16))
title.pack(pady=10)

entry = tk.Entry(root, font=("Arial", 14))
entry.pack(pady=10)

btn = tk.Button(root, text="Mark Attendance",
                command=mark_attendance,
                font=("Arial", 12))
btn.pack(pady=5)

report_btn = tk.Button(root, text="Show Report",
                       command=show_report,
                       font=("Arial", 12))
report_btn.pack(pady=5)

status_label = tk.Label(root, text="", font=("Arial", 12))
status_label.pack(pady=10)

root.mainloop()
