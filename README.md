# BANK-MANAGEMENT-SYSTEM
This project helps to manage the bank account and related work smoothly.Here we are given with different kinds of options like-
1:To Open A New Account!
2:To Deposite amount!
3: To Withdraw amount!
4: Bank Enquiry!
5: Customer Details!
6: To Update Personal Information!
7: To Close account!
8: To Show Infromation/Data of user!

# To Open A New Account!
def open_account():
    name= input("ENTER FULL NAME...")
    # Generate a new account number
    account_number = generate_account_number()

    # Use the account number in your code
    address = input("ENTER YOUR CURRENT PARMANENT ADDRESS!....")
    contact_no= int(input("ENTER YOUR MOBILE NUMBER!"))
    total_balance = int(input("DEPOSITE SOME AMOUNT TO OPEN A ACCOUNT!!"))
    account_pin = int(input("ENTER A 4 DIGIT PIN!"))

    #data1 is for a table named as account
    data1 = (name,account_number,address,contact_no,total_balance,account_pin)
    # data2 is for amount table
    data2 = (name,account_number,total_balance,account_pin)

    #entering queries in sql
    mycursor.execute("create table if not exists account(name varchar(40),account_number int(40),address varchar(50),contact_no BIGINT ,total_balance int,account_pin int(20))")
    mycursor.execute("create table if not exists amount(name varchar(40),account_number int(40),total_balance int,account_pin int(20))")

    account_table = "insert into account values(%s,%s,%s,%s,%s,%s)"
    amount_table = "insert into amount values(%s,%s,%s,%s)"

    c = con1.cursor()
    #to execute the above code and insert data into mysql
    mycursor.execute(account_table,data1)
    mycursor.execute(amount_table,data2)
    con1.commit()

    print(" ")
    print("\t\t\t _________********>>>>BANK ACCOUNT CREATED SUCESSFULLY!!<<<<********_________")
    print(" ")
    print("YOUR NEW BANK ACCOUNT NUMBER IS:",account_number,"\nNOTE:","\n Keep your account number safely and do not share with anyone...")

# To Deposite amount!
def deposite_amount():
    global total_balance
    name = input("ENTER FULL NAME...")
    account_number = input("ENTER YOUR ACCOUNT NUMBER")
    pin= int(input("enter your 4 digit pin"))
    deposite_amnt = int(input(("ENTER THE AMOUNT U WANT TO DEPOSITE")))

    # Query the amount_table to get the current balance of the account
    mycursor.execute("SELECT total_balance FROM amount WHERE account_number = %s", (account_number,))
    result = mycursor.fetchone()
    total_balance = result[0] if result else 0  # Set total_balance to the current balance of the account, or 0 if no result was returned

    # Update the total_balance in the amount_table
    mycursor.execute("UPDATE amount SET total_balance = %s WHERE account_number = %s", (total_balance + deposite_amnt, account_number))
    con1.commit()

    # Query the amount_table to get the updated balance of the account
    mycursor.execute("SELECT total_balance FROM amount WHERE account_number = %s", (account_number,))
    result = mycursor.fetchone()
    total_balance = result[0] if result else 0  # Set total_balance to the updated balance of the account, or 0 if no result was returned

    pt = PrettyTable(["total_balance"])
    pt.add_row([total_balance])
    print("\t\t\t _________********>>>>TOTAL BALANCE<<<<********_________")
    print(pt)
    print("___________________________________________________________________________________________________________")
# To Withdraw amount!
def withdraw_amount():
    name = input("ENTER FULL NAME...")
    account_number = input("ENTER YOUR ACCOUNT NUMBER")
    pin = int(input("enter your 4 digit pin"))
    withdraw_amnt = int(input(("ENTER THE AMOUNT U WANT TO WITHDRAW")))

    # Check if the entered pin is correct
    mycursor.execute("SELECT account_pin FROM amount WHERE account_number = %s", (account_number,))
    result = mycursor.fetchone()
    if result:
        # If the pin is correct, proceed with the withdraw
        account_pin = result[0]
        if account_pin == pin:
            # Check if the account has sufficient balance to perform the withdraw
            mycursor.execute("SELECT total_balance FROM amount WHERE account_number = %s", (account_number,))
            result = mycursor.fetchone()
            total_balance = result[0]
            if total_balance >= withdraw_amnt:
                # Update the total_balance in the amount table
                mycursor.execute("UPDATE amount SET total_balance = %s WHERE account_number = %s", (total_balance - withdraw_amnt, account_number))
                con1.commit()

                # Query the amount table to get the updated balance of the account
                mycursor.execute("SELECT total_balance FROM amount WHERE account_number = %s", (account_number,))
                result = mycursor.fetchone()
                total_balance = result[0]

                pt = PrettyTable(["total_balance"])
                pt.add_row([total_balance])
                print("\t\t\t _________********>>>>TOTAL BALANCE after WITHDRAW<<<<********_________")
                print(pt)
                print("___________________________________________________________________________________________________________")
            else:
                print("Insufficient balance!")
        else:
            print("Incorrect pin!")
    else:
        print("Account not found!")
# Bank Enquiry!
def check_balance():
    account_number = input("ENTER YOUR ACCOUNT NUMBER")
    pin = int(input("enter your 4 digit pin"))

    c = con1.cursor()
    mycursor.execute("select total_balance from amount where account_number=" + str(account_number))
    myresult = mycursor.fetchall()
    pt = PrettyTable(["total_balance"])
    for total_balance in myresult:
        pt.add_row([total_balance])
    print("\t\t\t _________********>>>>TOTAL BALANCE ENQUIRY SUCESSFULLY PRINTED!!<<<<********_________")
    print(pt)
    print("___________________________________________________________________________________________________________")

# Customer Details!
def customer_details():
    account_number = input("ENTER YOUR ACCOUNT NUMBER")
    pin = int(input("enter your 4 digit pin"))

    c = con1.cursor()
    c.execute("SELECT account.name, account.account_number, account.contact_no, account.address, amount.total_balance FROM account JOIN amount ON account.account_number = amount.account_number WHERE account.account_number=%s AND account.account_pin=%s", (account_number, pin))
    myresult = c.fetchall()

    pt = PrettyTable(["name","account_number","contact_no","address","total_balance"])
    for name,account_number,contact_no,address,total_balance in myresult:
        pt.add_row([name,account_number,contact_no,address,total_balance])
    print("\t\t\t _________********>>>>CuSTOMER DETAILS!!<<<<********_________")
    print(pt)
    print("___________________________________________________________________________________________________________")

# To Update Personal Information!
def update_info():
    # Check if the contact_no column exists in the account_table
    mycursor.execute("SHOW COLUMNS FROM account LIKE 'contact_no'")
    result = mycursor.fetchone()
    if not result:
        # If the contact_no column does not exist, add it
        mycursor.execute("ALTER TABLE account ADD contact_no varchar(40)")
        con1.commit()

    account_number = input("ENTER YOUR ACCOUNT NUMBER")
    pin = int(input("enter your 4 digit pin"))
    
    # Ask the user whether they want to update their contact number or address
    update_type = input("Enter '1' to update contact number or '2' to update address: ")
    
    if update_type == '1':
        new_contact_no = input("ENTER NEW CONTACT NUMBER!!")
        # Update the contact_no column in the account_table
        mycursor.execute("update account set contact_no = %s where account_number = %s", (new_contact_no, account_number))
    elif update_type == '2':
        new_address = input("ENTER NEW ADDRESS!!")
        # Update the address column in the account_table
        mycursor.execute("update account set address = %s where account_number = %s", (new_address, account_number))
    else:
        # If the user enters an invalid option, display an error message
        print("Invalid option. Please try again.")
        return

    con1.commit()

    # Get the updated information from the account_table
    mycursor.execute("SELECT * FROM account WHERE account_number = %s", (account_number,))
    updated_info = mycursor.fetchone()
    
    # Create a new PrettyTable object
    table = PrettyTable()
    
    # Add the column names to the table
    table.field_names = ["Column Name", "Value"]
    
    # Add the updated information to the table as rows
    for i in range(len(updated_info)):
        table.add_row([mycursor.description[i][0], updated_info[i]])
    
    # Print the table to the console
    print(table)
#  To Close account!
def close():
    name = input("ENTER FULL NAME...")
    account_number = input("ENTER YOUR ACCOUNT NUMBER")
    pin = int(input("enter your 4 digit pin"))

    c = con1.cursor()
    try:
        mycursor.execute("delete from amount where account_number=" + str(account_number))
        mycursor.execute("delete from account where account_number=" + str(account_number))
        con1.commit()
        print("\t\t\t _________********>>>>ACCOUNT DELETED/CLOSED SUCCESSFULLY!!<<<<********_________")
    except mysql.connector.errors.ProgrammingError as err:
        print("There was an error deleting the record from the amount table: ", err)
    print('__________________________________________________________________________________________________________')
# To Show Infromation/Data of user!
def show_info():
    mycursor = con1.cursor()
    mycursor.execute("SELECT account.name, account.account_number, account.contact_no, account.address, amount.total_balance FROM account JOIN amount ON account.account_number = amount.account_number ")
    myresult = mycursor.fetchall()
    pt = PrettyTable(["name", "account_number", "address", "contact_no", "total_balance"])
    for name, account_number, address, contact_no, total_balance in myresult:
        pt.add_row([name, account_number, address, contact_no, total_balance])
    print("\t\t\t _________********>>>>ALL INFORMATION ABOUT THIS ACCOUNT!!<<<<********_________")
    print(pt)
    print("___________________________________________________________________________________________________________")
