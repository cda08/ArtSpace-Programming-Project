# ArtSpace-Programming-Project
import sqlite3
import hashlib
import string
import tkinter as tk
from tkinter import messagebox as mb


# ---------------- Database ----------------
conn = sqlite3.connect("task.db")
cursor = conn.cursor()

cursor.execute("""
    CREATE TABLE IF NOT EXISTS task(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        task TEXT NOT NULL
    )
""")
cursor.execute("""
    CREATE TABLE IF NOT EXISTS users(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        username TEXT UNIQUE NOT NULL,
        email TEXT NOT NULL,
        password TEXT NOT NULL
    )
""")
cursor.execute("""
    CREATE TABLE IF NOT EXISTS seller(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        username TEXT UNIQUE NOT NULL,
        email TEXT NOT NULL,
        password TEXT NOT NULL
    )
""")
conn.commit()

# Secret key required to create a seller account
SELLER_SECRET_KEY = "SELLER-ART-SPACE"   # change this!


def hash_password(password: str) -> str:
    return hashlib.sha256(password.encode("utf-8")).hexdigest()


def validate_password(password: str):
    """Checks password against 5 specific security requirements."""
    if len(password) < 6:
        return "Password must be at least 6 characters long."
    if not any(c.isupper() for c in password):
        return "Password must contain at least 1 capital letter."
    if not any(c.islower() for c in password):
        return "Password must contain at least 1 small letter."
    if not any(c.isdigit() for c in password):
        return "Password must contain at least 1 number."
    if not any(c in string.punctuation for c in password):
        return "Password must contain at least 1 special symbol (e.g. !@#$%)."
    return None


def toggle_passwords(var, *entries):
    """Toggles password visibility for given entry widgets based on checkbox state."""
    show_char = "" if var.get() else "*"
    for entry in entries:
        entry.config(show=show_char)


# ---------------- App ----------------
app = tk.Tk()
app.title("ArtSpace")
app.geometry("1000x700")
app.resizable(True, True)

# Note: Update or remove the bg_image line if the file path on your PC causes an error
try:
    bg_image = tk.PhotoImage(file=r"c:\Users\User\Pictures\ArtSpace.png")
    bg_label = tk.Label(app, image=bg_image)
    bg_label.place(x=0, y=0, relwidth=1, relheight=1)
except Exception:
    pass  # Fallback if background image file is missing

# ---------------- Page switching ----------------
def hide_all():
    for f in (login_frame, register_frame, seller_login_frame,
              seller_register_frame, main_frame, seller_frame):
        f.place_forget()

def show(frame):
    hide_all()
    frame.place(relx=0.5, rely=0.5, anchor="center")
    frame.lift()  # Crucial: lifts frame above background image so entry boxes are clickable

def show_login():         show(login_frame)
def show_register():        show(register_frame)
def show_seller_login():    show(seller_login_frame)
def show_seller_register(): show(seller_register_frame)
def show_main():            show(main_frame)
def show_seller_panel():    show(seller_frame)


# ============================================================
# USER LOGIN
# ============================================================
login_frame = tk.Frame(app, bg="white", bd=2, relief="groove")

tk.Label(login_frame, text="Login", font=("Garamond", 22, "bold"),
         bg="white").pack(pady=10)

tk.Label(login_frame, text="Username:", font=("Arial", 10), bg="white", anchor="w").pack(fill="x", padx=30)
login_user = tk.Entry(login_frame, width=30, font=("Arial", 12))
login_user.pack(pady=2, ipady=3)

tk.Label(login_frame, text="Password:", font=("Arial", 10), bg="white", anchor="w").pack(fill="x", padx=30)
login_pass = tk.Entry(login_frame, width=30, font=("Arial", 12), show="*")
login_pass.pack(pady=2, ipady=3)

# Show Password Checkbox for User Login
login_show_var = tk.BooleanVar()
tk.Checkbutton(login_frame, text="Show Password", variable=login_show_var, bg="white",
               command=lambda: toggle_passwords(login_show_var, login_pass)).pack(anchor="w", padx=30, pady=2)


def do_login():
    username = login_user.get().strip()
    password = login_pass.get()

    if not username or not password:
        mb.showerror("Error", "Please enter username and password.")
        return

    cursor.execute("SELECT password FROM users WHERE username = ?", (username,))
    row = cursor.fetchone()

    if row is None:
        mb.showerror("Error", "User not found.")
        return
    if row[0] != hash_password(password):
        mb.showerror("Error", "Wrong password.")
        return

    mb.showinfo("Success", f"Welcome, {username}!")
    login_user.delete(0, "end")
    login_pass.delete(0, "end")
    login_show_var.set(False)
    toggle_passwords(login_show_var, login_pass)
    show_main()


tk.Button(login_frame, text="Login", width=25, command=do_login).pack(pady=10)
tk.Button(login_frame, text="No account? Register",
          fg="blue", bg="white", bd=0, cursor="hand2",
          command=show_register).pack()
tk.Button(login_frame, text="Seller login →",
          fg="darkred", bg="white", bd=0, cursor="hand2",
          command=show_seller_login).pack(pady=(5, 10))


# ============================================================
# USER REGISTER
# ============================================================
register_frame = tk.Frame(app, bg="white", bd=2, relief="groove")

tk.Label(register_frame, text="Register", font=("Garamond", 22, "bold"),
         bg="white").pack(pady=10)

tk.Label(register_frame, text="Username:", font=("Arial", 10), bg="white", anchor="w").pack(fill="x", padx=30)
reg_user = tk.Entry(register_frame, width=30, font=("Arial", 12))
reg_user.pack(pady=2, ipady=3)

tk.Label(register_frame, text="Email:", font=("Arial", 10), bg="white", anchor="w").pack(fill="x", padx=30)
reg_email = tk.Entry(register_frame, width=30, font=("Arial", 12))
reg_email.pack(pady=2, ipady=3)

tk.Label(register_frame, text="Password:", font=("Arial", 10), bg="white", anchor="w").pack(fill="x", padx=30)
reg_pass = tk.Entry(register_frame, width=30, font=("Arial", 12), show="*")
reg_pass.pack(pady=2, ipady=3)

tk.Label(register_frame, text="Confirm Password:", font=("Arial", 10), bg="white", anchor="w").pack(fill="x", padx=30)
reg_pass2 = tk.Entry(register_frame, width=30, font=("Arial", 12), show="*")
reg_pass2.pack(pady=2, ipady=3)

# Show Password Checkbox for User Register
reg_show_var = tk.BooleanVar()
tk.Checkbutton(register_frame, text="Show Passwords", variable=reg_show_var, bg="white",
               command=lambda: toggle_passwords(reg_show_var, reg_pass, reg_pass2)).pack(anchor="w", padx=30, pady=2)

# Helpful prompt indicating password rules
tk.Label(register_frame, text="Req: 6+ chars, 1 uppercase, 1 lowercase,\n1 number, 1 special symbol", 
         font=("Arial", 8), bg="white", fg="grey", justify="center").pack(pady=2)


def do_register():
    username = reg_user.get().strip()
    email    = reg_email.get().strip()
    password = reg_pass.get()
    confirm  = reg_pass2.get()

    if not username or not email or not password or not confirm:
        mb.showerror("Error", "All fields are required.")
        return
    if len(username) < 3:
        mb.showerror("Error", "Username must be at least 3 characters.")
        return
    if "@" not in email or "." not in email:
        mb.showerror("Error", "Please enter a valid email.")
        return

    pass_error = validate_password(password)
    if pass_error:
        mb.showerror("Password Error", pass_error)
        return

    if password != confirm:
        mb.showerror("Error", "Passwords do not match.")
        return

    try:
        cursor.execute(
            "INSERT INTO users (username, email, password) VALUES (?, ?, ?)",
            (username, email, hash_password(password)),
        )
        conn.commit()
    except sqlite3.IntegrityError:
        mb.showerror("Error", "Username already taken.")
        return

    mb.showinfo("Success", "Account created! Please log in.")
    for e in (reg_user, reg_email, reg_pass, reg_pass2):
        e.delete(0, "end")
    reg_show_var.set(False)
    toggle_passwords(reg_show_var, reg_pass, reg_pass2)
    show_login()


tk.Button(register_frame, text="Register", width=25,
          command=do_register).pack(pady=5)
tk.Button(register_frame, text="Already have an account? Login",
          fg="blue", bg="white", bd=0, cursor="hand2",
          command=show_login).pack(pady=(0, 10))


# ============================================================
# SELLER LOGIN
# ============================================================
seller_login_frame = tk.Frame(app, bg="white", bd=2, relief="groove")

tk.Label(seller_login_frame, text="Seller Login",
         font=("Garamond", 22, "bold"), bg="white",
         fg="darkred").pack(pady=10)

tk.Label(seller_login_frame, text="Seller Username:", font=("Arial", 10), bg="white", anchor="w").pack(fill="x", padx=30)
seller_user = tk.Entry(seller_login_frame, width=30, font=("Arial", 12))
seller_user.pack(pady=2, ipady=3)

tk.Label(seller_login_frame, text="Password:", font=("Arial", 10), bg="white", anchor="w").pack(fill="x", padx=30)
seller_pass = tk.Entry(seller_login_frame, width=30, font=("Arial", 12), show="*")
seller_pass.pack(pady=2, ipady=3)

# Show Password Checkbox for Seller Login
seller_login_show_var = tk.BooleanVar()
tk.Checkbutton(seller_login_frame, text="Show Password", variable=seller_login_show_var, bg="white",
               command=lambda: toggle_passwords(seller_login_show_var, seller_pass)).pack(anchor="w", padx=30, pady=2)


def do_seller_login():
    username = seller_user.get().strip()
    password = seller_pass.get()

    if not username or not password:
        mb.showerror("Error", "Please enter seller username and password.")
        return

    cursor.execute("SELECT password FROM seller WHERE username = ?", (username,))
    row = cursor.fetchone()

    if row is None:
        mb.showerror("Error", "Seller account not found.")
        return
    if row[0] != hash_password(password):
        mb.showerror("Error", "Wrong password.")
        return

    mb.showinfo("Success", f"Welcome, Seller {username}!")
    seller_user.delete(0, "end")
    seller_pass.delete(0, "end")
    seller_login_show_var.set(False)
    toggle_passwords(seller_login_show_var, seller_pass)
    show_seller_panel()


tk.Button(seller_login_frame, text="Login as Seller", width=25,
          fg="white", bg="darkred",
          command=do_seller_login).pack(pady=10)
tk.Button(seller_login_frame, text="Create seller account",
          fg="darkred", bg="white", bd=0, cursor="hand2",
          command=show_seller_register).pack()
tk.Button(seller_login_frame, text="← Back to user login",
          fg="blue", bg="white", bd=0, cursor="hand2",
          command=show_login).pack(pady=(5, 10))


# ============================================================
# SELLER REGISTER (requires secret key)
# ============================================================
seller_register_frame = tk.Frame(app, bg="white", bd=2, relief="groove")

tk.Label(seller_register_frame, text="Seller Registration",
         font=("Garamond", 22, "bold"), bg="white",
         fg="darkred").pack(pady=10)

tk.Label(seller_register_frame, text="Username:", font=("Arial", 10), bg="white", anchor="w").pack(fill="x", padx=30)
sr_user  = tk.Entry(seller_register_frame, width=30, font=("Arial", 12))
sr_user.pack(pady=2, ipady=3)

tk.Label(seller_register_frame, text="Email:", font=("Arial", 10), bg="white", anchor="w").pack(fill="x", padx=30)
sr_email = tk.Entry(seller_register_frame, width=30, font=("Arial", 12))
sr_email.pack(pady=2, ipady=3)

tk.Label(seller_register_frame, text="Password:", font=("Arial", 10), bg="white", anchor="w").pack(fill="x", padx=30)
sr_pass  = tk.Entry(seller_register_frame, width=30, font=("Arial", 12), show="*")
sr_pass.pack(pady=2, ipady=3)

tk.Label(seller_register_frame, text="Confirm Password:", font=("Arial", 10), bg="white", anchor="w").pack(fill="x", padx=30)
sr_pass2 = tk.Entry(seller_register_frame, width=30, font=("Arial", 12), show="*")
sr_pass2.pack(pady=2, ipady=3)

tk.Label(seller_register_frame, text="Seller Secret Key:", font=("Arial", 10), bg="white", anchor="w").pack(fill="x", padx=30)
sr_key   = tk.Entry(seller_register_frame, width=30, font=("Arial", 12), show="*")
sr_key.pack(pady=2, ipady=3)

# Show Password Checkbox for Seller Register
sr_show_var = tk.BooleanVar()
tk.Checkbutton(seller_register_frame, text="Show Passwords", variable=sr_show_var, bg="white",
               command=lambda: toggle_passwords(sr_show_var, sr_pass, sr_pass2)).pack(anchor="w", padx=30, pady=2)

tk.Label(seller_register_frame, text="Req: 6+ chars, 1 uppercase, 1 lowercase,\n1 number, 1 special symbol", 
         font=("Arial", 8), bg="white", fg="grey", justify="center").pack(pady=2)


def do_seller_register():
    username = sr_user.get().strip()
    email    = sr_email.get().strip()
    password = sr_pass.get()
    confirm  = sr_pass2.get()
    key      = sr_key.get()

    if not all([username, email, password, confirm, key]):
        mb.showerror("Error", "All fields are required.")
        return
    if key != SELLER_SECRET_KEY:
        mb.showerror("Error", "Invalid seller secret key.")
        return
    if len(username) < 3:
        mb.showerror("Error", "Username must be at least 3 characters.")
        return
    if "@" not in email or "." not in email:
        mb.showerror("Error", "Please enter a valid email.")
        return

    pass_error = validate_password(password)
    if pass_error:
        mb.showerror("Password Error", pass_error)
        return

    if password != confirm:
        mb.showerror("Error", "Passwords do not match.")
        return

    try:
        cursor.execute(
            "INSERT INTO seller (username, email, password) VALUES (?, ?, ?)",
            (username, email, hash_password(password)),
        )
        conn.commit()
    except sqlite3.IntegrityError:
        mb.showerror("Error", "Seller username already taken.")
        return

    mb.showinfo("Success", "Seller account created! Please log in.")
    for e in (sr_user, sr_email, sr_pass, sr_pass2, sr_key):
        e.delete(0, "end")
    sr_show_var.set(False)
    toggle_passwords(sr_show_var, sr_pass, sr_pass2)
    show_seller_login()


tk.Button(seller_register_frame, text="Create Seller", width=25,
          fg="white", bg="darkred",
          command=do_seller_register).pack(pady=5)
tk.Button(seller_register_frame, text="← Back to seller login",
          fg="darkred", bg="white", bd=0, cursor="hand2",
          command=show_seller_login).pack(pady=(0, 10))


# ============================================================
# USER MAIN PAGE
# ============================================================
main_frame = tk.Frame(app, bg="white", bd=2, relief="groove")
tk.Label(main_frame, text="You are logged in!",
         font=("Garamond", 24, "bold"), bg="white").pack(pady=20)
tk.Button(main_frame, text="Log out", width=25,
          command=lambda: (login_user.delete(0, "end"),
                           login_pass.delete(0, "end"),
                           login_show_var.set(False),
                           toggle_passwords(login_show_var, login_pass),
                           show_login())).pack(pady=10)


# ============================================================
# SELLER PANEL
# ============================================================
seller_frame = tk.Frame(app, bg="white", bd=2, relief="groove")
tk.Label(seller_frame, text="Seller Panel",
         font=("Garamond", 24, "bold"), bg="white",
         fg="darkred").pack(pady=20)

# Show all registered users
users_list = tk.Listbox(seller_frame, width=40, height=8, font=("Arial", 11))
users_list.pack(pady=10)

cursor.execute("SELECT id, username, email FROM users")
for row in cursor.fetchall():
    users_list.insert("end", f"#{row[0]}  {row[1]}  ({row[2]})")

tk.Label(seller_frame, text="All registered users",
         bg="white", fg="grey", font=("Arial", 9)).pack()

tk.Button(seller_frame, text="Refresh list", width=25,
          command=lambda: refresh_users(users_list)).pack(pady=5)
tk.Button(seller_frame, text="Log out", width=25,
          command=lambda: (seller_user.delete(0, "end"),
                           seller_pass.delete(0, "end"),
                           seller_login_show_var.set(False),
                           toggle_passwords(seller_login_show_var, seller_pass),
                           show_login())).pack(pady=(5, 10))


def refresh_users(listbox):
    listbox.delete(0, "end")
    cursor.execute("SELECT id, username, email FROM users")
    for row in cursor.fetchall():
        listbox.insert("end", f"#{row[0]}  {row[1]}  ({row[2]})")


# ---------------- Start ----------------
show_login()
app.mainloop()
