# Boutique-Management-2



import mysql.connector
# Making MySQL connection object
mycon = mysql.connector.connect(
    host='localhost', user='root',
    password='Karunsai@1234*******',
    database='boutique')
# Making MySQL cursor object
mycur = mycon.cursor()

def check():
    # Query to select all customer IDs
    # from the table
    qry = 'select cust_id from customer;'
     
    mycur.execute(qry)
    ''' Return a list where each element
    in the list is a tuple
    fetched from each record in table
    Each tuple contains a single element
    as only customer IDs are fetched
    from cust_id column of each record'''
   
    d = mycur.fetchall()
    # To create a list of all customer IDs in the table
    list_of_ids = []
    for ids in d:
        # A list of all customer IDs in table
        list_of_ids.append(ids[0])
         
    return list_of_ids
# To create a new account for the customer
def cust_ac():
	ask = 'Y'
	list_of_ids = check()
	while ask in 'yY':
		custid = int(input('Enter your customer id... '))
		# to check if a customer already exists with this ID
		if custid in list_of_ids:
			print('This Customer Id already exists....\
Try creating a new one')
		else:
		# Tuple to contain details of the customer
			c_det = ()
			cnam = input('First Name : ')
			clnam = input('Last Name : ')
			cphno = input('Phone Number : ')
			cadrs = input('Your Address : ')
			c_det = (custid, cnam, clnam, cphno, cadrs)
			
			''' Values inserted in the table and default NULL value are 
				provided for booked product at the time of creation 
				of customer account '''

			qry = 'insert into customer values(%s,%s,%s,%s,%s,NULL);'
			
			# value of the fields to be entered with the query
			val = c_det
			
			mycur.execute(qry, val)
			mycon.commit()
			print('Customer details entered')
			ask = input('Do you want to continue (Y/N) ')
			if ask not in ('Yy'):
				space()
				break
# To select all booked products of
# a given customer from the table
def get_bkd_pro(cust_id):
	qry = 'select bkd_pro from customer\
where cust_id=%s;'
	mycur.execute(qry, (cust_id,))
	bp = mycur.fetchone()
	bkd_pro = bp[0]
	return bkd_pro



def sign_in():
    try:
        ask = int(input('Enter customer ID to sign in: '))
        # Using check function to check whether this account exists or not
        list_of_ids = check()
        if ask in list_of_ids:
            while True:
                print('''Do you want to:
                    1) View Bookings
                    2) Book a product
                    3) Update Self Details
                    4) Cancel booked products
                    Enter 'back' to exit''')
                # Take choice of the customer
                ccc = input('Enter choice - ')
                if ccc == '1':
                    # Get booked product function is used where customer ID
                    # is passed as an argument
                    s = get_bkd_pro(ask)
                    # To check if the column has any value
                    if s is None or s == ' ':
                        print('You have not booked products yet')
                    else:
                        # If more than one products are booked,
                        # their IDs are stored as a single value
                        # separated by '_'; so we have to split the
                        # string to print each product ID.
                        d = s.split('_')
                        print('Booked products:')
                        for bkditems in d:
                            print(bkditems)
                elif ccc == '2':
                    # Check if the product to be booked exists or not
                    qry = 'SELECT pro_id FROM products;'
                    mycur.execute(qry)
                    pro_list = [item[0] for item in mycur.fetchall()]
                    # Take ID and quantity of product to be booked
                    pro_id = input('Enter the product id to book products: ')
                    if pro_id in pro_list:
                        # Customer ID is given as value along with
                        # query to fetch booked product for the given ID
                        qry = 'SELECT bkd_pro FROM customer WHERE cust_id = %s;'
                        mycur.execute(qry, (ask,))
                        pr = mycur.fetchone()
                        prl = pr[0]
                        # When the column is empty, the new product is to be stored
                        if prl is None or prl == ' ':
                            qry = 'UPDATE customer SET bkd_pro = %s WHERE cust_id = %s;'
                            val = (pro_id + '_', ask)
                            mycur.execute(qry, val)
                            mycon.commit()
                            print('Your Product is booked !!')
                        else:
                            prl1 = prl + pro_id
                            qry2 = 'UPDATE customer SET bkd_pro = %s WHERE cust_id = %s;'
                            val2 = (prl1 + '_', ask)
                            mycur.execute(qry2, val2)
                            mycon.commit()
                            print('Your Product is booked !!')
                    else:
                        print('This product does not exist. Please write the correct product id!')
                elif ccc == '3':
                    qry = 'SELECT cust_id,c_nam,c_lnam,c_phno,c_adrs FROM customer WHERE cust_id = %s'
                    mycur.execute(qry, (ask,))
                    clist = mycur.fetchone()
                    flds = ['Name', 'Last Name', 'Ph.No', 'Address']
                    dic = {}
                    print("Your existing record is:")
                    for i in range(4):
                        dic[flds[i]] = clist[i + 1]
                        print(i + 1, ' ', flds[i], ' : ', clist[i + 1])
                    for i in range(len(clist)):
                        updtc = int(input('Enter choice to update: '))
                        upval = input('Enter ' + flds[updtc - 1] + ': ')
                        dic[flds[updtc - 1]] = upval
                        yn = input('Do you want to update other details? (y/n) ')
                        if yn.lower() in 'n':
                            break
                    qry = 'UPDATE customer SET c_nam=%s, c_lnam=%s, c_phno=%s, c_adrs=%s WHERE cust_id = %s;'
                    updtl = tuple(dic.values()) + (ask,)
                    val = updtl
                    mycur.execute(qry, val)
                    mycon.commit()
                    print('Your details are updated')
                elif ccc == '4':
                    try:
                        # To get the existing bookings
                        # Booked products in the table
                        bkd_pro = get_bkd_pro(ask)
                        print('Your Booking(s): \n', bkd_pro)
                        if bkd_pro is None or bkd_pro == ' ':
                            print('You have no bookings to cancel')
                        else:
                            cw = input("To cancel all products, enter 'A'\nOR enter the product code to cancel: ")
                            if cw.lower() == 'a':
                                qry = 'UPDATE customer SET bkd_pro = NULL WHERE cust_id = %s;'
                                mycur.execute(qry, (ask,))
                                mycon.commit()
                                print('All bookings deleted')
                            elif cw in bkd_pro:
                                x = (bkd_pro[0:-1]).split('_')
                                x.remove(cw)
                                updt_pro = ''
                                for item in x:
                                    updt_pro = updt_pro + item + '_'
                                qry = 'UPDATE customer SET bkd_pro = %s WHERE cust_id = %s;'
                                val = (updt_pro, ask)
                                mycur.execute(qry, val)
                                mycon.commit()
                                print('Booking Cancelled !')
                    except Exception as e:
                        print('Some problem in updating details. Try again')
                elif ccc.lower() == 'back':
                    print("Successfully logged out")
                    break
        else:
            print('This Account does not exist.')
    except Exception as e:
        print('Some error occurred. Try Again')


# To fetch values from all columns of 
# product table to get product details
def view_pro():
	qry = 'select * from products;'
	mycur.execute(qry)
	d = mycur.fetchall()
	# contains list of all records
	dic = {}
	# Each record fetched is separated into a key value pair
	# and stored in the dictionary where product ID is the key
	for i in d:
		dic[i[0]] = i[1:]
	print('_'*80)
# Printing the dictionary in the form of a table
	print("{:<17} {:<22} {:<23} {:<19}".format(
		'Product id', 'Product name', 'Price', 'Stock'))
	print('_'*80)
	for k, v in dic.items():
		a, b, c = v
		print("{:<17} {:<22} {:<23} {:<19}".format(k, a, b, c))
	print('_'*80)
# To add a new product in Products table
def addpro():
	# Display list of products
	view_pro() 
	n = int(input('Enter no of items to insert '))
	for j in range(n):
		# Initialize tuple to store
		# product details.
		t = ()
		pronum = input("Product No. ")
		proid = input('Product ID : ')
		pprice = int(input('Price : '))
		pstk = int(input('Stock : '))
		t = (pronum, proid, pprice, pstk)
		# Using MySql query
		qry = 'insert into products values(%s,%s,%s,%s);'
		val = t
		mycur.execute(qry, val)
		mycon.commit()
		print("Product Added")
# To delete a product from the table
def delpro():
	view_pro()
	delt = input("Enter ID of product to be deleted")
	qry = 'delete from products where pro_id=%s;'
	mycur.execute(qry, (delt,))
	mycon.commit()
	print("Product is deleted")
# Assuming you have already established a connection and created a cursor named mycur and mycon
# Make sure to add the necessary imports and connection setup before using this code.

# For Employee Login
def emp_sign_in():
    try:
        ask = input('Enter id to sign in to the account: ')
        # To check if the employee with this ID exists or not.
        qry = 'select emp_id from employee;'
        mycur.execute(qry)
        d = mycur.fetchall()
        lis = [i[0] for i in d]  # Simplify list creation

        if ask not in lis:
            print('Enter the correct id')
        else:
            while True:
                space()
                ccc = input("1. Update delivered records\n"
                            "2. Add a New Product\n"
                            "3. Delete a product\n"
                            "Enter 'Back' to logout: ")

                if ccc == '1':
                    cust_id = input('Enter customer id: ')
                    # Check if the customer has bookings or not
                    bkd_pro = get_bkd_pro(cust_id)
                    if bkd_pro is None or bkd_pro == ' ':
                        print('This customer has no bookings')
                    else:
                        print('All booking(s):', bkd_pro)
                        pro_id = input('Enter product code to remove the delivered product: ')
                        # The product IDs are stored in the form of a single value separated by '_'.
                        if pro_id in bkd_pro:
                            x = (bkd_pro[0:-1]).split('_')
                            # Returns a list of all booked products,
                            # then remove the delivered product from the list
                            x.remove(pro_id)
                            # Concatenate the existing products using '_'
                            updt_pro = '_'.join(x)
                            qry = 'update customer set bkd_pro=%s where cust_id=%s;'
                            val = (updt_pro, cust_id)
                            mycur.execute(qry, val)
                            mycon.commit()
                            print('Delivered product is removed from the database.')
                        else:
                            print('Enter the correct code')
                elif ccc == '2':
                    addpro()
                elif ccc == '3':
                    delpro()
                elif ccc.lower() == 'back':
                    print("Successfully logged out")
                    break
    except Exception as e:
        print('An error occurred:', e)

# Define the functions get_bkd_pro(), addpro(), and delpro() if they are not already defined.

# Call emp_sign_in() to start the employee sign-in process.

# Close the cursor and connection when you are done.
# To add employee details
def addemp(): 
	qry = "select * from employee;"
	mycur.execute(qry)
	emp_list = mycur.fetchall()
	print("List of Employees ")
	for emp in emp_list:
		print("Emp Id : ", emp[0], " Name : ", emp[1],
			" Last Name : ", emp[2], " Phone No : ", emp[3])
	ne = []
	n = int(input('enter the no. of employees to add '))
	for i in range(1, n+1):
		t = ()
		print('enter employee id ')
		idd = int(input(str(i)+') '))
		print('Name ')
		nam = input(str(i)+') ')
		print('Last name ')
		lnam = input(str(i)+') ')
		print('Contact no. ')
		conno = int(input(str(i)+') '))
		print('Address ')
		adrs = input(str(i)+') ')
		# A tuple containing details of an employee
		t = (idd, nam, lnam, conno, adrs)
		# List containing details of n number
		# of employees to be added
		ne = ne+[t, ]
	qry = 'insert into employee values(%s,%s,%s,%s,%s);'
	# A list containing details of each employee
	# in the form of a tuple is to be passed along with the query
	for i in range(len(ne)):
		val = ne[i]
		mycur.execute(qry, val)
		mycon.commit()
	print('All Employee details added. ')
	space()
# For employer login
def employer(): 
	while True:
		print()
		print('''Enter Your Choice:													 
					1)View Product Details
					2)Add a New Employee
					enter back to exit''')
		ccc = input('Enter _____ ')
		if ccc == '1':
			view_pro()
		if ccc == '2':
			addemp()
		if ccc.lower() == "back":
			break
print('WELCOME !')
# Running a infinite loop
while True:
	print('''Are you a :												 
(A). Customer
(B). Employee
(C). Employer
enter e to exit ''')
	ch = input('Enter - ')
	try:
		if ch in 'aA':
			print(" 1. Create Account\n 2.Sign In into existing account")
			choice = input('enter- ')
			if choice == '1':
				cust_ac()
			elif choice == '2':
				sign_in()
			else:
				print('Enter correct choice')
		if ch in 'bB':
			emp_sign_in()
		if ch in 'cC':
			employer()
		elif ch.lower() == "e":
			print("Thankyou for visiting !")
			break
	except Exception:
		print('Give the right input')
	space()
# Import MySql Connector
import mysql.connector

# making MySQL connection object
mycon = mysql.connector.connect(
    host='localhost', user='root',
    password='Karunsai@123456788', database='boutique')

# making MySQL cursor object
mycur = mycon.cursor()

# To provide blank spaces in the output
def space():
    for i in range(1):
        print()

# To check if a customer of a given ID exists or not
def check():
    # query to select all customer IDs from the table
    qry = 'select cust_id from customer;'
    mycur.execute(qry)
    
    ''' a list where each element in the list is a tuple
    fetched from each record in the table
    Each tuple contains a single element as only customer IDs are fetched
    from cust_id column of each record '''
    
    d = mycur.fetchall()
    
    # to create a list of all customer IDs in the table
    list_of_ids = []
    for ids in d:
        # a list of all customer IDs in the table
        list_of_ids.append(ids[0])
    return list_of_ids

# To create a new account for the customer
def cust_ac():
    ask = 'Y'
    list_of_ids = check()
    while ask in 'yY':
        custid = int(input('Enter your customer id... '))
        # to check if a customer already exists with this ID
        if custid in list_of_ids:
            print('This Customer Id already exists.... Try creating a new one')
        else:
            # Tuple to contain details of the customer
            c_det = ()
            cnam = input('First Name : ')
            clnam = input('Last Name : ')
            cphno = input('Phone Number : ')
            cadrs = input('Your Address : ')
            c_det = (custid, cnam, clnam, cphno, cadrs)
            
            ''' Values inserted in the table and default NULL value are 
                provided for booked product at the time of creation 
                of customer account '''
    
            qry = 'insert into customer values(%s,%s,%s,%s,%s,NULL);'
            
            # value of the fields to be entered with the query
            val = c_det
            
            mycur.execute(qry, val)
            mycon.commit()
            print('Customer details entered')
            ask = input('Do you want to continue (Y/N) ')
            if ask not in ('Yy'):
                space()
                break

# To select all booked products of a given customer from the table
def get_bkd_pro(cust_id):
    qry = 'select bkd_pro from customer where cust_id=%s;'
    mycur.execute(qry, (cust_id,))
    bp = mycur.fetchone()
    bkd_pro = bp[0]
    return bkd_pro

# ... (continue with the rest of your functions)

print('WELCOME !')
# Running an infinite loop
while True:
    print('''Are you a :												 
    (A). Customer
    (B). Employee
    (C). Employer
    enter e to exit ''')
    ch = input('Enter - ')
    try:
        if ch in 'aA':
            print(" 1. Create Account\n 2.Sign In into an existing account")
            choice = input('enter- ')
            if choice == '1':
                cust_ac()
            elif choice == '2':
                sign_in()
            else:
                print('Enter the correct choice')
        if ch in 'bB':
            emp_sign_in()
        if ch in 'cC':
            employer()
        elif ch.lower() == "e":
            print("Thank you for visiting!")
            break
    except Exception as e:
        print('An error occurred:', str(e))
    space()
