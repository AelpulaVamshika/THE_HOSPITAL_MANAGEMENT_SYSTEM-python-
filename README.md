import tkinter as tk 
from tkinter import messagebox, simpledialog
from datetime import datetime
import os

# --- Data Storage (in-memory lists) ---
patients = [] # list of dicts with keys: id, name, age, contact, check_in_time, check_out_time
doctors = [] # list of dicts with keys: id, name, specialization, contact, email
prescriptions = {} # dict patient_id -> list of dicts {doctor_id, medication, dosage, date}
appointments = [] # list of dicts with keys: patient_id, doctor_id, date, time
alloted_doctors = {} # dict patient_id -> doctor_id

# --- Hospital Information ---
HOSPITAL_CONTACT_NUMBER_EMERGENCY = "+91 9876543210"
HOSPITAL_CONTACT_NUMBER_GENERAL = "+91 1234567890"
HOSPITAL_EMAIL = "info@hospital.com"
HOSPITAL_CLOSING_TIME = "10:00 PM" # Example closing time

# --- GUI Styling (Magical Hues) ---
ROOT_BG_COLOR = "#E6E6FA" # Lavender Blush
MAIN_FRAME_BG_COLOR = "#F0F8FF" # Alice Blue
CONTENT_FRAME_BG_COLOR = "#FFFFFF" # Pure White
BUTTON_COLOR = "#8A2BE2" # Blue Violet
BUTTON_TEXT_COLOR = "#FFFFFF" # White
HIGHLIGHT_COLOR = "#9370DB" # Medium Purple
LABEL_COLOR = "#4B0082" # Indigo
FONT_LARGE = ('Arial', 18, 'bold')
FONT_MEDIUM = ('Arial', 14, 'bold')
FONT_SMALL = ('Arial', 11)

# Special Button Colors
CHECK_IN_COLOR = "#32CD32" # Lime Green
CHECK_OUT_COLOR = "#FFD700" # Gold
ALERT_COLOR = "#FF6347" # Tomato
NEUTRAL_COLOR = "#808080" # Gray

# --- Helper Functions ---
def generate_id(data_list, prefix):
    return f"{prefix}{len(data_list) + 1:03d}"

def clear_frames():
    for widget in main_frame.winfo_children():
        widget.destroy()

# Simple file-based persistence without JSON for each data type
DATA_DIR = "./data"

def ensure_data_dir():
    if not os.path.exists(DATA_DIR):
        os.makedirs(DATA_DIR)

def save_patients():
    ensure_data_dir()
    with open(f"{DATA_DIR}/patients.txt", "w", encoding="utf-8") as f:
        for p in patients:
            line = "|".join([
                p['id'], p['name'], str(p['age']), p['contact'],
                p['check_in_time'] if p['check_in_time'] else "",
                p['check_out_time'] if p['check_out_time'] else ""
            ])
            f.write(line + "\n")

def load_patients():
    global patients
    patients.clear()
    path = f"{DATA_DIR}/patients.txt"
    if os.path.exists(path):
        with open(path, "r", encoding="utf-8") as f:
            for line in f:
                line=line.strip()
                if not line:
                    continue
                parts = line.split("|")
                if len(parts) >= 6:
                    patients.append({
                        'id': parts[0],
                        'name': parts[1],
                        'age': int(parts[2]),
                        'contact': parts[3],
                        'check_in_time': parts[4] or None,
                        'check_out_time': parts[5] or None
                    })

def save_doctors():
    ensure_data_dir()
    with open(f"{DATA_DIR}/doctors.txt", "w", encoding="utf-8") as f:
        for d in doctors:
            line = "|".join([
                d['id'], d['name'], d['specialization'], d['contact'], d['email']
            ])
            f.write(line + "\n")

def load_doctors():
    global doctors
    doctors.clear()
    path = f"{DATA_DIR}/doctors.txt"
    if os.path.exists(path):
        with open(path, "r", encoding="utf-8") as f:
            for line in f:
                line=line.strip()
                if not line:
                    continue
                parts = line.split("|")
                if len(parts) >=5:
                    doctors.append({
                        'id': parts[0],
                        'name': parts[1],
                        'specialization': parts[2],
                        'contact': parts[3],
                        'email': parts[4]
                    })

def save_prescriptions():
    ensure_data_dir()
    with open(f"{DATA_DIR}/prescriptions.txt", "w", encoding="utf-8") as f:
        # Each line: patient_id|doctor_id|medication|dosage|date
        for patient_id, pres_list in prescriptions.items():
            for pres in pres_list:
                line = "|".join([
                    patient_id,
                    pres['doctor_id'],
                    pres['medication'],
                    pres['dosage'],
                    pres['date']
                ])
                f.write(line + "\n")

def load_prescriptions():
    global prescriptions
    prescriptions.clear()
    path = f"{DATA_DIR}/prescriptions.txt"
    if os.path.exists(path):
        with open(path, "r", encoding="utf-8") as f:
            for line in f:
                line=line.strip()
                if not line:
                    continue
                parts = line.split("|")
                if len(parts) >=5:
                    pat_id, doc_id, med, dosage, date = parts
                    if pat_id not in prescriptions:
                        prescriptions[pat_id] = []
                    prescriptions[pat_id].append({
                        'doctor_id': doc_id,
                        'medication': med,
                        'dosage': dosage,
                        'date': date
                    })

def save_appointments():
    ensure_data_dir()
    with open(f"{DATA_DIR}/appointments.txt", "w", encoding="utf-8") as f:
        # Each line: patient_id|doctor_id|date|time
        for appt in appointments:
            line = "|".join([
                appt['patient_id'], appt['doctor_id'], appt['date'], appt['time']
            ])
            f.write(line + "\n")

def load_appointments():
    global appointments
    appointments.clear()
    path = f"{DATA_DIR}/appointments.txt"
    if os.path.exists(path):
        with open(path, "r", encoding="utf-8") as f:
            for line in f:
                line=line.strip()
                if not line:
                    continue
                parts = line.split("|")
                if len(parts) >=4:
                    appointments.append({
                        'patient_id': parts[0],
                        'doctor_id': parts[1],
                        'date': parts[2],
                        'time': parts[3]
                    })

def save_alloted_doctors():
    ensure_data_dir()
    with open(f"{DATA_DIR}/alloted_doctors.txt", "w", encoding="utf-8") as f:
        # Each line: patient_id|doctor_id
        for p_id, d_id in alloted_doctors.items():
            f.write(f"{p_id}|{d_id}\n")

def load_alloted_doctors():
    global alloted_doctors
    alloted_doctors.clear()
    path = f"{DATA_DIR}/alloted_doctors.txt"
    if os.path.exists(path):
        with open(path, "r", encoding="utf-8") as f:
            for line in f:
                line=line.strip()
                if not line:
                    continue
                parts = line.split("|")
                if len(parts) >=2:
                    alloted_doctors[parts[0]] = parts[1]

def save_data():
    save_patients()
    save_doctors()
    save_prescriptions()
    save_appointments()
    save_alloted_doctors()

def load_data():
    load_patients()
    load_doctors()
    load_prescriptions()
    load_appointments()
    load_alloted_doctors()
    # Prepopulate doctors if none loaded
    if not doctors:
        prepopulate_doctors()

def prepopulate_doctors():
    default_doctors = [
        {'id':'D001','name':'Alice Smith','specialization':'Cardiology','contact':'+91 1111111111','email':'alice.smith@hospital.com'},
        {'id':'D002','name':'Bob Jones','specialization':'Neurology','contact':'+91 2222222222','email':'bob.jones@hospital.com'},
        {'id':'D003','name':'Charlie White','specialization':'Orthopedics','contact':'+91 3333333333','email':'charlie.white@hospital.com'},
        {'id':'D004','name':'Diana Green','specialization':'Pediatrics','contact':'+91 4444444444','email':'diana.green@hospital.com'},
    ]
    doctors.extend(default_doctors)
    save_doctors()

# ----------------------- Main GUI & Functions Below -----------------------

current_patient = None
current_doctor = None

# -- Patient Functions --

def patient_login():
    global current_patient
    patient_id = simpledialog.askstring("Patient Login", "Enter Patient ID:")
    if not patient_id:
        messagebox.showinfo("Login Cancelled", "Patient login cancelled or ID not entered.")
        return

    found = next((patient for patient in patients if patient['id'] == patient_id), None)
    if found:
        current_patient = found
        messagebox.showinfo("Login Success", f"Welcome, Patient {current_patient['name']}!")
        show_patient_menu()
    else:
        messagebox.showerror("Login Failed", "Invalid Patient ID. Please sign up if you are a new patient.")

def patient_signup():
    name = simpledialog.askstring("Patient Sign Up", "Enter Your Name:")
    if not name: return
    age = simpledialog.askinteger("Patient Sign Up", "Enter Your Age:")
    if age is None or age <= 10: # Age must be more than 10
        messagebox.showerror("Sign Up Failed", "Age must be more than 10.")
        return
    contact = simpledialog.askstring("Patient Sign Up", "Enter Your Contact Number:")
    if not contact: return

    if any(p['contact'] == contact for p in patients):
        messagebox.showerror("Sign Up Failed", "A patient with this contact number already exists. Please try logging in.")
        return

    patient_id = generate_id(patients, "P")
    patients.append({
        'id': patient_id,
        'name': name,
        'age': age,
        'contact': contact,
        'check_in_time': None,
        'check_out_time': None
    })
    save_patients()
    messagebox.showinfo("Patient Sign Up Success", f"You have successfully signed up!\nYour Patient ID is: {patient_id}\nPlease remember this for future logins.")
    global current_patient
    current_patient = patients[-1]
    show_patient_menu()

def show_patient_menu():
    clear_frames()
    patient_menu_frame = tk.Frame(main_frame, bg=CONTENT_FRAME_BG_COLOR)
    patient_menu_frame.pack(pady=20, padx=20, fill="both", expand=True, ipadx=10, ipady=10)

    tk.Label(patient_menu_frame, text=f"Welcome, Patient {current_patient['name']} ({current_patient['id']})", font=FONT_MEDIUM, bg=CONTENT_FRAME_BG_COLOR, fg=LABEL_COLOR).pack(pady=15)

    tk.Button(patient_menu_frame, text="View My Details", command=view_my_patient_details, bg=BUTTON_COLOR, fg=BUTTON_TEXT_COLOR, font=FONT_SMALL, width=32, activebackground=HIGHLIGHT_COLOR, relief="raised", bd=2).pack(pady=5)
    tk.Button(patient_menu_frame, text="Check In", command=patient_check_in, bg=CHECK_IN_COLOR, fg=BUTTON_TEXT_COLOR, font=FONT_SMALL, width=32, activebackground="#28A745", relief="raised", bd=2).pack(pady=5)
    tk.Button(patient_menu_frame, text="Check Out", command=patient_check_out, bg=CHECK_OUT_COLOR, fg=LABEL_COLOR, font=FONT_SMALL, width=32, activebackground="#E0A800", relief="raised", bd=2).pack(pady=5)
    tk.Button(patient_menu_frame, text="View Doctors", command=view_doctors_for_patient, bg=BUTTON_COLOR, fg=BUTTON_TEXT_COLOR, font=FONT_SMALL, width=32, activebackground=HIGHLIGHT_COLOR, relief="raised", bd=2).pack(pady=5)
    tk.Button(patient_menu_frame, text="Book Appointment", command=book_appointment_patient, bg=BUTTON_COLOR, fg=BUTTON_TEXT_COLOR, font=FONT_SMALL, width=32, activebackground=HIGHLIGHT_COLOR, relief="raised", bd=2).pack(pady=5)
    tk.Button(patient_menu_frame, text="View My Prescriptions", command=view_patient_prescriptions, bg=BUTTON_COLOR, fg=BUTTON_TEXT_COLOR, font=FONT_SMALL, width=32, activebackground=HIGHLIGHT_COLOR, relief="raised", bd=2).pack(pady=5)
    tk.Button(patient_menu_frame, text="View Allotted Doctor", command=view_alloted_doctor_for_patient, bg=BUTTON_COLOR, fg=BUTTON_TEXT_COLOR, font=FONT_SMALL, width=32, activebackground=HIGHLIGHT_COLOR, relief="raised", bd=2).pack(pady=5)
    tk.Button(patient_menu_frame, text="Logout", command=patient_logout, bg=NEUTRAL_COLOR, fg=BUTTON_TEXT_COLOR, font=FONT_SMALL, width=32, activebackground="#606060", relief="raised", bd=2).pack(pady=5)
    tk.Button(patient_menu_frame, text="Back to Main Menu", command=show_main_menu, bg=NEUTRAL_COLOR, fg=BUTTON_TEXT_COLOR, font=FONT_SMALL, width=32, activebackground="#606060", relief="raised", bd=2).pack(pady=5)

def view_my_patient_details():
    if current_patient:
        check_in_status = current_patient.get('check_in_time', 'N/A')
        check_out_status = current_patient.get('check_out_time', 'N/A')

        details_lines = [
            f"ID: {current_patient['id']}",
            f"Name: {current_patient['name']}",
            f"Age: {current_patient['age']}",
            f"Contact: {current_patient['contact']}",
            f"Check-in Time: {check_in_status}",
            f"Check-out Time: {check_out_status}"
        ]
        messagebox.showinfo("My Patient Details", "\n".join(details_lines))
    else:
        messagebox.showerror("Error", "No patient logged in.")

def patient_check_in():
    if current_patient:
        if current_patient.get('check_in_time') is None or current_patient.get('check_out_time') is not None:
            current_patient['check_in_time'] = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            current_patient['check_out_time'] = None
            save_patients()
            messagebox.showinfo("Check-in", f"Patient {current_patient['name']} checked in at {current_patient['check_in_time']}.")
        else:
            messagebox.showinfo("Check-in", "You are already checked in.")
    else:
        messagebox.showerror("Error", "No patient logged in.")

def patient_check_out():
    if current_patient:
        if current_patient.get('check_in_time') is not None and current_patient.get('check_out_time') is None:
            current_patient['check_out_time'] = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            save_patients()
            messagebox.showinfo("Check-out", f"Patient {current_patient['name']} checked out at {current_patient['check_out_time']}.")
        else:
            messagebox.showinfo("Check-out", "You are not currently checked in or have already checked out.")
    else:
        messagebox.showerror("Error", "No patient logged in.")

def view_doctors_for_patient():
    if not doctors:
        messagebox.showinfo("View Doctors", "No doctors registered yet.")
        return

    doctor_list_lines = ["Registered Doctors:\n"]
    for doc in doctors:
        doctor_list_lines.append(f"ID: {doc['id']}, Name: {doc['name']}, Specialization: {doc['specialization']}")
    messagebox.showinfo("View Doctors", "\n".join(doctor_list_lines))

def book_appointment_patient():
    if not current_patient:
        messagebox.showerror("Error", "No patient logged in.")
        return

    if not doctors:
        messagebox.showinfo("Book Appointment", "No doctors available to book an appointment with.")
        return

    doctor_id = simpledialog.askstring("Book Appointment", "Enter Doctor ID:")
    if not doctor_id: return

    found_doctor = next((doc for doc in doctors if doc['id'] == doctor_id), None)
    if not found_doctor:
        messagebox.showerror("Error", "Doctor not found.")
        return

    date = simpledialog.askstring("Book Appointment", "Enter Date (YYYY-MM-DD):")
    if not date: return
    time = simpledialog.askstring("Book Appointment", "Enter Time (HH:MM):")
    if not time: return

    # Check if the doctor is already booked for the selected time slot
    if any(appt for appt in appointments if appt['doctor_id'] == doctor_id and appt['date'] == date and appt['time'] == time):
        messagebox.showerror("Error", "The doctor is busy at this time. Please choose another time slot.")
        return

    appointments.append({
        'patient_id': current_patient['id'],
        'doctor_id': found_doctor['id'],
        'date': date,
        'time': time
    })
    save_appointments()
    messagebox.showinfo("Appointment Booked", f"Appointment booked with Dr. {found_doctor['name']} on {date} at {time}.")

def view_patient_prescriptions():
    if not current_patient:
        messagebox.showerror("Error", "No patient logged in.")
        return
    if current_patient['id'] not in prescriptions or not prescriptions[current_patient['id']]:
        messagebox.showinfo("View Prescriptions", "No prescriptions found for you.")
        return
    pres_text_lines = [f"Prescriptions for {current_patient['name']}:\n"]
    for pres in prescriptions[current_patient['id']]:
        doctor_name = "Unknown Doctor"
        for doc in doctors:
            if doc['id'] == pres['doctor_id']:
                doctor_name = doc['name']
                break
        pres_text_lines.append(f"Date: {pres['date']}")
        pres_text_lines.append(f"Doctor: Dr. {doctor_name}")
        pres_text_lines.append(f"Medication: {pres['medication']}")
        pres_text_lines.append(f"Dosage: {pres['dosage']}")
        pres_text_lines.append("-" * 40)
    messagebox.showinfo("View Prescriptions", "\n".join(pres_text_lines))

def view_alloted_doctor_for_patient():
    if not current_patient:
        messagebox.showerror("Error", "No patient logged in.")
        return
    alloted_doc_id = alloted_doctors.get(current_patient['id'])
    if alloted_doc_id:
        found_doctor = next((doc for doc in doctors if doc['id'] == alloted_doc_id), None)
        if found_doctor:
            messagebox.showinfo("Allotted Doctor", f"Your allotted doctor is: Dr. {found_doctor['name']} ({found_doctor['specialization']})")
        else:
            messagebox.showinfo("Allotted Doctor", "Your allotted doctor's details could not be found.")
    else:
        messagebox.showinfo("Allotted Doctor", "You have not been allotted a doctor yet.")

def patient_logout():
    global current_patient
    current_patient = None
    save_data()
    messagebox.showinfo("Logout", "Logged out successfully.")
    show_main_menu()

def show_patient_login():
    clear_frames()
    login_frame = tk.Frame(main_frame, bg=CONTENT_FRAME_BG_COLOR)
    login_frame.pack(pady=20, padx=20, fill="both", expand=True, ipadx=10, ipady=10)
    tk.Label(login_frame, text="Patient Login", font=FONT_MEDIUM, bg=CONTENT_FRAME_BG_COLOR, fg=LABEL_COLOR).pack(pady=15)
    tk.Button(login_frame, text="Login as Existing Patient", command=patient_login, bg=BUTTON_COLOR, fg=BUTTON_TEXT_COLOR, font=FONT_SMALL, width=32, activebackground=HIGHLIGHT_COLOR, relief="raised", bd=2).pack(pady=5)
    tk.Button(login_frame, text="Sign Up as New Patient", command=patient_signup, bg=BUTTON_COLOR, fg=BUTTON_TEXT_COLOR, font=FONT_SMALL, width=32, activebackground=HIGHLIGHT_COLOR, relief="raised", bd=2).pack(pady=5)
    tk.Button(login_frame, text="Back to Main Menu", command=show_main_menu, bg=NEUTRAL_COLOR, fg=BUTTON_TEXT_COLOR, font=FONT_SMALL, width=32, activebackground="#606060", relief="raised", bd=2).pack(pady=5)

# -- Main Menu --

def show_main_menu():
    clear_frames()
    menu_frame = tk.Frame(main_frame, bg=MAIN_FRAME_BG_COLOR)
    menu_frame.pack(fill="both", expand=True, padx=30, pady=30)
    tk.Label(menu_frame, text="Welcome to Hospital Management System", font=FONT_LARGE, bg=MAIN_FRAME_BG_COLOR, fg=LABEL_COLOR).pack(pady=20)
    tk.Button(menu_frame, text="Patient Login / Signup", command=show_patient_login, bg=BUTTON_COLOR, fg=BUTTON_TEXT_COLOR, font=FONT_MEDIUM, width=30, activebackground=HIGHLIGHT_COLOR, relief="raised", bd=2).pack(pady=10)
    tk.Button(menu_frame, text="Exit", command=root.destroy, bg=ALERT_COLOR, fg=BUTTON_TEXT_COLOR, font=FONT_MEDIUM, width=30, activebackground="#e55347", relief="raised", bd=2).pack(pady=10)

# --- Initialize and run the app ---

root = tk.Tk()
root.title("Hospital Management System")
root.configure(bg=ROOT_BG_COLOR)

# Set fullscreen
root.state('zoomed') # Works on Windows, on Linux/Mac you might adjust accordingly

main_frame = tk.Frame(root, bg=MAIN_FRAME_BG_COLOR)
main_frame.pack(fill="both", expand=True)

load_data()
show_main_menu()

root.mainloop()
