import streamlit as st
import sqlite3

conn = sqlite3.connect("hostel.db", check_same_thread=False)
c = conn.cursor()

# Create Tables
c.execute('''CREATE TABLE IF NOT EXISTS students(
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT,
    age INTEGER,
    gender TEXT,
    course TEXT
)''')

c.execute('''CREATE TABLE IF NOT EXISTS rooms(
    room_id INTEGER PRIMARY KEY,
    type TEXT,
    capacity INTEGER,
    occupied INTEGER
)''')

c.execute('''CREATE TABLE IF NOT EXISTS allocation(
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    student_id INTEGER,
    room_id INTEGER
)''')

c.execute('''CREATE TABLE IF NOT EXISTS fees(
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    student_id INTEGER,
    amount REAL,
    status TEXT
)''')

conn.commit()

st.title("🏨 Hostel Management System")

menu = ["Add Student", "Add Room", "Allocate Room", "Fees", "View Data"]
choice = st.sidebar.selectbox("Menu", menu)

# Add Student
if choice == "Add Student":
    st.subheader("Student Registration")

    name = st.text_input("Name")
    age = st.number_input("Age", 15, 40)
    gender = st.selectbox("Gender", ["Male", "Female"])
    course = st.text_input("Course")

    if st.button("Add"):
        c.execute("INSERT INTO students(name, age, gender, course) VALUES (?,?,?,?)",
                  (name, age, gender, course))
        conn.commit()
        st.success("Student Added")

# Add Room
elif choice == "Add Room":
    st.subheader("Add Room")

    room_id = st.number_input("Room ID")
    room_type = st.selectbox("Type", ["2 Sharing", "3 Sharing"])
    capacity = st.number_input("Capacity", 1, 5)

    if st.button("Add Room"):
        c.execute("INSERT INTO rooms(room_id, type, capacity, occupied) VALUES (?,?,?,0)",
                  (room_id, room_type, capacity))
        conn.commit()
        st.success("Room Added")

# Allocate Room
elif choice == "Allocate Room":
    st.subheader("Room Allocation")

    student_id = st.number_input("Student ID")
    room_id = st.number_input("Room ID")

    if st.button("Allocate"):
        c.execute("INSERT INTO allocation(student_id, room_id) VALUES (?,?)",
                  (student_id, room_id))
        c.execute("UPDATE rooms SET occupied = occupied + 1 WHERE room_id=?",
                  (room_id,))
        conn.commit()
        st.success("Room Allocated")

# Fees
elif choice == "Fees":
    st.subheader("Fees Management")

    student_id = st.number_input("Student ID")
    amount = st.number_input("Amount")
    status = st.selectbox("Status", ["Paid", "Pending"])

    if st.button("Submit"):
        c.execute("INSERT INTO fees(student_id, amount, status) VALUES (?,?,?)",
                  (student_id, amount, status))
        conn.commit()
        st.success("Fees Updated")

# View Data
elif choice == "View Data":
    st.subheader("All Students")

    data = c.execute("SELECT * FROM students").fetchall()
    st.write(data)
