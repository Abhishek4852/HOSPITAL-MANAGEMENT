# HOSPITAL-MANAGEMENT
<br>
# bill generation
<br>
from tkinter import *
import tkinter as tk
from tkinter import messagebox, ttk
import mysql.connector
from datetime import datetime

class Hospital:
    def __init__(self, root):
        self.root = root
        self.root.title("Hospital Management System")
        self.root.geometry("500x600+500+300")

        self.setup_database()
        self.setup_frames()
        self.setup_widgets()
