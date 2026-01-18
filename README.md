# dhruv
code
# -*- coding: utf-8 -*-
"""
Created on Fri Jan 16 21:26:39 2026

@author: HP
"""

import tkinter as tk
from tkinter import ttk, messagebox, simpledialog
import csv
import os

USER_FILE = "users.csv"
EMP_DETAILS = "employee_details.csv"
EMP_SKILLS = "employee_skills.csv"

current_theme = "dark"

themes = {
    "dark": {
        "bg": "#1e1e1e",
        "card": "#2a2a2a",
        "text": "white",
        "btn": "#4caf50",
        "entry": "#3a3a3a"
    },
    "light": {
        "bg": "#f5f5f5",
        "card": "#ffffff",
        "text": "black",
        "btn": "#1976d2",
        "entry": "#ffffff"
    }
}

skills = [
    "Python","Java","C","C++","JavaScript","HTML","CSS","React",
    "Node.js","Django","Flask","SQL","MongoDB","Machine Learning",
    "Data Analysis","AI","Cyber Security","Networking","Cloud",
    "AWS","Docker","Kubernetes","UI/UX","Figma","Photoshop"
]

def apply_theme(widget):
    t = themes[current_theme]
    widget.configure(bg=t["bg"])
    for w in widget.winfo_children():
        try:
            if isinstance(w, tk.Frame):
                w.configure(bg=t["bg"])
            elif isinstance(w, tk.Label):
                w.configure(bg=t["bg"], fg=t["text"])
            elif isinstance(w, tk.Entry):
                w.configure(bg=t["entry"], fg=t["text"], insertbackground=t["text"])
            elif isinstance(w, tk.Button):
                w.configure(bg=t["btn"], fg="white")
        except:
            pass
        apply_theme(w)

def toggle_theme():
    global current_theme
    current_theme = "light" if current_theme == "dark" else "dark"
    apply_theme(root)

def login():
    u = username.get().strip()
    p = password.get().strip()

    if not u or not p:
        messagebox.showerror("Error", "Enter username & password")
        return

    if not os.path.exists(USER_FILE):
        with open(USER_FILE, "w", newline="") as f:
            csv.writer(f).writerow(["Username","Password","Role"])

    with open(USER_FILE, "r") as f:
        users = list(csv.DictReader(f))

    for row in users:
        if row["Username"] == u:
            if row["Password"] == p:
                if row["Role"] == "employee":
                    employee_form(u)
                else:
                    recruiter_form()
                return
            else:
                messagebox.showerror("Error", "Wrong password")
                return

    role = simpledialog.askstring("New User", "Enter role (employee/recruiter):")
    if role not in ("employee","recruiter"):
        messagebox.showerror("Error", "Invalid role")
        return

    with open(USER_FILE, "a", newline="") as f:
        csv.writer(f).writerow([u,p,role])

    messagebox.showinfo("Registered", "Account created")

    if role == "employee":
        employee_form(u)
    else:
        recruiter_form()

def employee_form(user):
    clear()

    tk.Label(root, text="EMPLOYEE DETAILS", font=("Segoe UI",22,"bold")).pack(pady=10)

    fields = {}
    for lbl in ["Name","Email","Age","Gender","Address"]:
        tk.Label(root, text=lbl).pack()
        e = tk.Entry(root, width=40)
        e.pack(pady=3)
        fields[lbl] = e

    def next_step():
        if not os.path.exists(EMP_DETAILS):
            with open(EMP_DETAILS,"w",newline="") as f:
                csv.writer(f).writerow(["Username","Name","Email","Age","Gender","Address"])

        with open(EMP_DETAILS,"a",newline="") as f:
            csv.writer(f).writerow([
                user,
                fields["Name"].get(),
                fields["Email"].get(),
                fields["Age"].get(),
                fields["Gender"].get(),
                fields["Address"].get()
            ])
        skill_selector(user)

    tk.Button(root,text="Next (Skills)",command=next_step).pack(pady=10)
    back_button()
    apply_theme(root)

def skill_selector(user):
    clear()

    tk.Label(root,text="SELECT SKILLS",font=("Segoe UI",20,"bold")).pack(pady=10)

    search = tk.Entry(root,width=40)
    search.pack()

    canvas = tk.Canvas(root)
    frame = tk.Frame(canvas)
    scroll = tk.Scrollbar(root,command=canvas.yview)
    canvas.configure(yscrollcommand=scroll.set)

    canvas.pack(side="left",fill="both",expand=True)
    scroll.pack(side="right",fill="y")
    canvas.create_window((0,0),window=frame,anchor="nw")

    vars = {}

    def render():
        for w in frame.winfo_children():
            w.destroy()
        for s in skills:
            if search.get().lower() in s.lower():
                v = tk.BooleanVar()
                vars[s] = v
                tk.Checkbutton(frame,text=s,variable=v).pack(anchor="w")

    render()
    search.bind("<KeyRelease>",lambda e: render())

    def save():
        if not os.path.exists(EMP_SKILLS):
            with open(EMP_SKILLS,"w",newline="") as f:
                csv.writer(f).writerow(["Username","Skills"])

        selected = [s for s,v in vars.items() if v.get()]
        with open(EMP_SKILLS,"a",newline="") as f:
            csv.writer(f).writerow([user,", ".join(selected)])

        messagebox.showinfo("Done","Profile saved")

    tk.Button(root,text="Save",command=save).pack(pady=10)
    back_button()
    frame.update_idletasks()
    canvas.config(scrollregion=canvas.bbox("all"))
    apply_theme(root)

def recruiter_form():
    clear()

    tk.Label(root,text="RECRUITER SEARCH",font=("Segoe UI",22,"bold")).pack(pady=10)
    entry = tk.Entry(root,width=40)
    entry.pack()

    result = tk.Text(root,height=15,width=70)
    result.pack(pady=10)

    def search():
        result.delete("1.0","end")
        if not os.path.exists(EMP_SKILLS):
            result.insert("end","No employees yet")
            return

        with open(EMP_SKILLS,"r") as f:
            reader = csv.DictReader(f)
            for row in reader:
                if entry.get().lower() in row["Skills"].lower():
                    result.insert("end",f"{row['Username']} → {row['Skills']}\n")

    tk.Button(root,text="Search",command=search).pack()
    back_button()
    apply_theme(root)

def clear():
    for w in root.winfo_children():
        w.destroy()

def back_button():
    tk.Button(root,text="← Back",command=main_screen).pack(pady=5)

def main_screen():
    clear()
    tk.Label(root,text="JOB APPLICATION TRACKER",font=("Segoe UI",26,"bold")).pack(pady=20)

    tk.Label(root,text="Username").pack()
    global username
    username = tk.Entry(root,width=30)
    username.pack()

    tk.Label(root,text="Password").pack()
    global password
    password = tk.Entry(root,width=30,show="*")
    password.pack()

    tk.Button(root,text="Login / Register",command=login).pack(pady=10)
    tk.Button(root,text="Toggle Theme",command=toggle_theme).pack()

    apply_theme(root)

root = tk.Tk()
root.title("JobSphere")
root.geometry("900x600")

main_screen()
root.mainloop()













