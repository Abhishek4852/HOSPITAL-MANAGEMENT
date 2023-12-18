# HOSPITAL-MANAGEMENT
<br>
# bill generation
<br>




     from tkinter import *
     from tkinter import messagebox, ttk
     import mysql.connector
     from datetime import datetime
     from reportlab.pdfgen import canvas

class Hospital:
    def __init__(self, root):
        self.root = root
        self.root.title("Hospital Management System")
        self.root.geometry("500x600+500+300")

        self.setup_database()
        self.setup_frames()
        self.setup_widgets()

    def setup_database(self):
        # Connect to MySQL database
        self.db = mysql.connector.connect(
            host="localhost",
            user="root",
            password="Abhishek4852@",
            database="hospital_management"
        )
        self.cursor = self.db.cursor()

    def setup_frames(self):
        # Frame for patient details
        self.frame_patient = Frame(self.root, bd=10, relief=RIDGE)
        self.frame_patient.pack(side=TOP, pady=20)

    def setup_widgets(self):
        # Entry for Patient ID
        self.lblPatientID = Label(self.frame_patient, text="Patient ID:", font=("arial", 12, "bold"), padx=10, pady=4)
        self.lblPatientID.grid(row=0, column=0, sticky=W)
        self.entPatientID = Entry(self.frame_patient, font=("arial", 12), bd=5)
        self.entPatientID.grid(row=0, column=1, columnspan=2)

        # Button to generate total bill
        self.btnGenerateBill = Button(self.frame_patient, text="Generate Total Bill", font=("arial", 12, "bold"), bd=4, padx=10, pady=4, command=self.generate_total_bill)
        self.btnGenerateBill.grid(row=2, column=0, columnspan=3, pady=10)

        # Button to print bill
        self.btnPrintBill = Button(self.frame_patient, text="Print Bill", font=("arial", 12, "bold"), bd=4, padx=10, pady=4, command=self.print_bill)
        self.btnPrintBill.grid(row=3, column=0, columnspan=3, pady=10)

        # Entry for Total Bill (output)
        self.lblTotalBill = Label(self.frame_patient, text="Total Bill:", font=("arial", 12, "bold"), padx=10, pady=4)
        self.lblTotalBill.grid(row=4, column=0, sticky=W)
        self.entTotalBill = Entry(self.frame_patient, font=("arial", 12), bd=5, state='readonly')
        self.entTotalBill.grid(row=4, column=1, columnspan=2)

    def generate_total_bill(self):
        # Function to generate total bill
        try:
            # Fetch Patient ID from the entry widget
            patient_id_str = self.entPatientID.get()

            # Validate Patient ID (non-empty and integer)
            if not patient_id_str or not patient_id_str.isdigit():
                messagebox.showwarning("Warning", "Please enter a valid Patient ID.")
                return

            patient_id = int(patient_id_str)

            # Execute SQL query to get total bill
            query = f"SELECT adimission_fees, consult_charge, bed_charge, lab_charge, surgery_charge, pharmacy_charge, no_of_days " \
                    f"FROM bill WHERE pid = {patient_id}"
            self.cursor.execute(query)
            bill_details = self.cursor.fetchone()

            # Display the total bill in the entry widget
            if bill_details:
                adimission_fees, consult_charge, bed_charge, lab_charge, surgery_charge, pharmacy_charge, no_of_days = bill_details

                # Fetch patient name for displaying in patient details
                patient_name_query = f"SELECT Patient_Name FROM bill WHERE PID = {patient_id}"
                self.cursor.execute(patient_name_query)
                patient_name = self.cursor.fetchone()[0]

                self.entTotalBill.config(state='normal')
                self.entTotalBill.delete(0, END)
                self.entTotalBill.insert(0, adimission_fees + consult_charge + (bed_charge * no_of_days) + lab_charge + surgery_charge + pharmacy_charge)
                self.entTotalBill.config(state='readonly')

                # Show a success message
                messagebox.showinfo("Success", f"Total bill for {patient_name}'s Patient ID: {patient_id} is {self.entTotalBill.get()}")
            else:
                messagebox.showwarning("Warning", "Patient details not found.")

        except mysql.connector.Error as e:
            # Handle MySQL database errors
            messagebox.showerror("Database Error", f"MySQL Error: {e}")
        except Exception as e:
            # Handle other exceptions (e.g., invalid input)
            messagebox.showerror("Error", str(e))

    def print_bill(self):
        # Function to print bill
        try:
            # Fetch Patient ID from the entry widget
            patient_id_str = self.entPatientID.get()

            # Validate Patient ID (non-empty and integer)
            if not patient_id_str or not patient_id_str.isdigit():
                messagebox.showwarning("Warning", "Please enter a valid Patient ID.")
                return

            patient_id = int(patient_id_str)

            # Execute SQL query to get patient details and bill
            query = f"SELECT b.Patient_Name, p.DOB, p.Gender, p.Address, b.adimission_date, b.Discharge_date, b.did_alloted, " \
                    f"b.consult_charge, b.bed_charge, b.lab_charge, b.surgery_charge, b.pharmacy_charge, b.no_of_days, b.adimission_fees " \
                    f"FROM patient p JOIN bill b ON p.PID = b.pid WHERE p.PID = {patient_id}"
            self.cursor.execute(query)
            patient_details = self.cursor.fetchone()

            if not patient_details:
                messagebox.showwarning("Warning", "Patient details not found.")
                return

            # Open a new window for printing the bill
            bill_window = Toplevel(self.root)
            bill_window.title("Bill")
            bill_window.geometry("500x600")

            # Create and display the bill format
            self.display_bill_format(bill_window, patient_details)

        except mysql.connector.Error as e:
            # Handle MySQL database errors
            messagebox.showerror("Database Error", f"MySQL Error: {e}")
        except Exception as e:
            # Handle other exceptions
            messagebox.showerror("Error", str(e))

    def display_bill_format(self, window, patient_details):
        # Function to display bill format in a new window
        # Extract patient details from the query result
        patient_name, dob, gender, address, admission_date, discharge_date, doctor_id, consult_charge, bed_charge, lab_charge, surgery_charge, pharmacy_charge, no_of_days, admission_fees = patient_details

        # Create a bill format using labels
        lbl_hospital = Label(window, text="Abhishek Hospital", font=("arial", 16, "bold"), pady=10)
        lbl_hospital.pack()

        lbl_bill_id = Label(window, text=f"Bill ID: {self.generate_bill_id()}", font=("arial", 12, "bold"), anchor='w')
        lbl_bill_id.place(x=10, y=50)

        lbl_bill_date = Label(window, text=f"Bill Date: {datetime.now().strftime('%Y-%m-%d')}", font=("arial", 12, "bold"), anchor='e')
        lbl_bill_date.place(x=350, y=50)

        lbl_patient_details = Label(window, text=f"Patient Details\n\n NAME OF PATIENT: {patient_name}\tDOB: {dob}\tGender: {gender}\nAddress: {address}\tAdmission Date: {admission_date}\tDischarge Date: {discharge_date}\nDoctor ID: {doctor_id}\tAdmission Fees: {admission_fees}", font=("arial", 13))
        lbl_patient_details.place(x=10, y=80)

        lbl_bill_heading = Label(window, text="Bill Details", font=("arial", 14, "bold"), pady=10)
        lbl_bill_heading.place(x=200, y=240)

        # Create a table for bill details
        columns = ["S.No", "Billing Head", "Rate", "Quantity", "Amount"]
        bill_table = ttk.Treeview(window, columns=columns, show="headings")

        for col in columns:
            bill_table.heading(col, text=col)
            bill_table.column(col, width=100)

        # Add bill details to the table
        billing_data = [
            ("1", "Admission Fees", admission_fees, "1", admission_fees),
            ("2", "Consultation Charge", consult_charge, "1", consult_charge),
            ("3", "Bed Charge", bed_charge, no_of_days, bed_charge * no_of_days),
            ("4", "Lab Charge", lab_charge, "1", lab_charge),
            ("5", "Surgery Charge", surgery_charge, "1", surgery_charge),
            ("6", "Pharmacy Charge", pharmacy_charge, "1", pharmacy_charge),
        ]

        for data in billing_data:
            bill_table.insert("", "end", values=data)

        bill_table.place(x=10, y=280)

        # Calculate and display the total amount
        total_amount = sum([float(item[4]) for item in billing_data])
        lbl_total = Label(window, text=f"Total Amount: {total_amount}", font=("arial", 12, "bold"), pady=10)
        lbl_total.place(x=200, y=500)

        # Button to download bill as PDF
        btn_download_bill = Button(window, text="Download Bill", font=("arial", 12, "bold"), bd=4, padx=10, pady=4, command=lambda: self.download_bill(patient_details))
        btn_download_bill.place(x=200, y=550)

    def generate_bill_id(self):
        # Function to generate a unique bill ID (you can customize this based on your needs)
        return f"BILL{datetime.now().strftime('%Y%m%d%H%M%S')}"

    def download_bill(self, patient_details):
        # Function to download bill as PDF
        try:
            # Extract necessary details from patient_details
            patient_name, dob, gender, address, admission_date, discharge_date, doctor_id, consult_charge, bed_charge, lab_charge, surgery_charge, pharmacy_charge, no_of_days, admission_fees = patient_details

            # Generate a bill ID
            bill_id = self.generate_bill_id()

            # Create a PDF document
            pdf_filename = f"{bill_id}.pdf"
            pdf = canvas.Canvas(pdf_filename)
            
            
            
            pdf.drawString(250, 780, f" Abhishek Hospital")
            pdf.drawString(70,  730, f"Bill ID: {bill_id}")
            pdf.drawString(350, 730, f"Bill Date: {datetime.now().strftime('%Y-%m-%d')}")
            pdf.drawString(200, 710, f"Patient Details:")
            pdf.drawString(100, 690, f"Name: {patient_name},  DOB: {dob},   Gender: {gender}")
            pdf.drawString(100, 670, f"Address: {address},   Admission Date: {admission_date}")
            pdf.drawString(100, 650, f" Discharge Date: {discharge_date}, Doctor ID: {doctor_id}")
            pdf.drawString(100, 630, f" Admission Fees: {admission_fees}")

            
            


        # Add content to the PDF
            pdf.drawString(200, 580, f" Bill discription")
            pdf.drawString(100, 540, f"S. no.     Billing head                  Rate           Quentity             Amount ")
            pdf.drawString(100, 520, f" 1         Admission Fees:                               1                      {admission_fees} ")
            pdf.drawString(100, 500, f" 2         Consultation Charge:                        1                     {consult_charge} ")
            
            pdf.drawString(100, 480, f" 3         Bed Charge                   300            {no_of_days} days              {bed_charge * no_of_days} ")
            pdf.drawString(100, 460, f" 4         Lab Charge                                                              {lab_charge} ")
            pdf.drawString(100, 440, f" 5         Surgery Charge                                                      {surgery_charge} ")
            pdf.drawString(100, 420, f" 6         Pharmacy Charge                                                   {pharmacy_charge} ")
            
            
           
            pdf.drawString(100, 380, f"Total Amount                                                     {admission_fees + consult_charge + (bed_charge * no_of_days) + lab_charge + surgery_charge + pharmacy_charge}")

            # Save the PDF
            pdf.save()

            # Show a success message
            messagebox.showinfo("Success", f"Bill downloaded successfully as {pdf_filename}")

        except Exception as e:
            # Handle exceptions
            messagebox.showerror("Error", str(e))

if __name__ == "__main__":
    root = Tk()
    ob = Hospital(root)
    root.mainloop()
