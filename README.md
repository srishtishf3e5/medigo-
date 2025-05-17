import mysql.connector
from datetime import datetime, date, timedelta

class MedigoProgram():
    def __init__(self) -> None:
        '''Intialises global variables of the application.'''
        self.currUser = None;
        self.curs = None;
        self.db_con = None;
        # Only True when app is connected to database
        self.intialised = False;
    
    def connect(self,host, uname , passwd,db_name, port):
        '''Connect Application to local SQL DATABASE '''

        con = mysql.connector.connect(
            user = uname,
            password = passwd,
            host = host,
            database = db_name,
            port = port
        )
        self.curs = con.cursor()
        self.db_con = con
    
    def createTables(self):
        '''
            Creates Necessary SQL Tables for the application inside the Database
        '''
        
        try :
            # Creating User Table
            self.curs.execute("Create Table IF NOT EXISTS  Users (uname varchar(255) Primary Key,password TEXT not null)")
            self.db_con.commit();

            # Creating Meds Table
            self.curs.execute("Create Table IF NOT EXISTS  Meds (mname varchar(255) PRIMARY KEY,price int not null);")
            self.db_con.commit();

            # Creating Orders Table
            self.curs.execute("Create Table IF NOT EXISTS  Orders (uname varchar(255),mname varchar(255),order_date DATE not null,del_date DATE not null,Primary Key(uname, mname))")
            self.db_con.commit();
        except mysql.connector.Error as e:
            print(e)

    def intialize(self,host, uname , passwd,db_name, port):
        '''
        Connects app to database and Creates Tables
        Important to initalize before starting the menu
        '''
        m.connect(host, user, password, database, port)
        m.createTables()
        self.intialized = True

    def createUser(self, username, password):
        '''
        Creates User instance in the User Table after supplying  username  and password.
        '''
        try:
            sql = f"INSERT INTO Users(uname, password) VALUES('{username}', '{password}')"
            self.curs.execute(sql)
            self.db_con.commit()
            return True
        except mysql.connector.Error as e:
            print(e)
            return False

    def createMedecine(self, mname, price):
        '''
        Creates Meds instance in the Med Table after supplying  medecine name and price
        '''

        try:
            sql = f"INSERT INTO Meds(mname, price) VALUES('{mname}', '{price}')"
            self.curs.execute(sql)
            self.db_con.commit()
            return True
        except mysql.connector.Error as e:
            print(e)
            return False
    def createOrder(self, uname, mname):
        '''
        Creates order instance after supplying username and medecine name ( medecine user is buying).
        '''
        try:
            # Order date
            orderDate = datetime.now().strftime("%Y%m%d")
            # 3 day delivery
            delDate = (datetime.now() + timedelta(days = 3)).strftime("%Y%m%d")
            sql = f"INSERT INTO Orders(uname, mname,order_date, del_date ) VALUES('{uname}', '{mname}' , '{orderDate}' , '{delDate}')"
            self.curs.execute(sql)
            self.db_con.commit()
            return True
        except mysql.connector.Error as e:
            print(e)
            return False

    def logUser(self, username, password):
        '''
        Authenticates user and changes apps current user to the authenticated user.
        '''
        try:
            query = f"SELECT * FROM Users WHERE uname = '{username}';"
            self.curs.execute(query)
            res = self.curs.fetchall()
            if res and res[0][1] == password:
                self.currUser = username
                return True
            else:
                return False
        except mysql.connector.Error as e:
            print("Error")
            print(e)
            return False
    def retrieveMeds(self):
        '''
        Retrieves all the medications in the database
        '''
        try:
            query = f"SELECT * FROM Meds;"
            self.curs.execute(query)
            res = self.curs.fetchall()
            return res
        except mysql.connector.Error as e:
            print("Error")
            print(e)
            return False

    def retrieveOrders(self, uname):
        '''
        Retrieves all the orders in the database.
        '''
        try:
            query = f"SELECT * FROM Orders Where uname = '{uname}'"
            self.curs.execute(query)
            res = self.curs.fetchall()
            return res
        except mysql.connector.Error as e:
            print("Error")
            print(e)
            return False
   def menu(self):
        '''
        GUI for the App
        '''
        if not self.intialized:
            print('-------FAILED--------\n')
            print("Initalise the App first.\n")
            print('-----------------------\n')
        while True:
            # Welcome the User
            print('-------MEDIGO--------\n')
            print("Welcome to Medigo\n")
            menue_op = int(input("Please input 0 for Registeration and 1 for login: "))
            print('----------------------\n')
            if menue_op == 0:
                username = input("Please input a unique username: ")
                password = input("Please input a password: ")

                if self.createUser(username, password):
                    print('-------SUCESS--------\n')
                    print("You're account was successfully Created.\n")
                    print("Please login again using username and password.\n")
                    print('----------------------\n')
                else:
                    # As username is a primary , only one name can exist
                    print('-------FAILED--------\n')
                    print("Please register using a different username , as username is already taken.")
                    print('----------------------\n')
            elif menue_op == 1:
                done = False
                print('-------MEDIGO--------\n')
                username = input("Please input you're unique username: ")
                password = input("Please input you're password: ")
                print('----------------------\n')
                if self.logUser(username, password):
                    while True:
                        log_op = int(input("Please input 0 for viewing you're orders.\n" +"Please input 1 for making an order\n" +
                                       "Please input 2 if you want to exit: "))
                        if log_op == 0:
                            meds = self.retreiveOrders(self.currUser)
                            print('--------ORDERS-------\n')
                            for med_info in meds:
                                medName = med_info[1]
                                delDate = med_info[-1].strftime("%d/%m/%Y")
                                print(f"You have ordered {medName} , will arrive on {delDate}.\n")
                            print('-----------------------\n')
                        elif log_op == 1:
                            meds = self.retreiveMeds()
                            print('--------AVAILABLE--------\n')
                            for med in meds:
                                print(f"Name : {med[0]} , Price: {med[1]}\n")
                            print('-----------------------\n')
                            print('--------ORDERING-------\n')
                            medName = input("Input the name of the medecine you want to order: ")
                            print('-----------------------\n')
                            succOrder = self.createOrder(self.currUser, medName)
                            if succOrder:
                                print('-------SUCCESS--------\n')
                                print(f"You're Order was Successful,  You can  view the order in menue.")
                                print('-----------------------\n')
                            else:
                                print('-------FAILED--------\n')
                                print(f" Unsucessful Order")
                                print('-----------------------\n')
                        elif log_op == 2:
                            # EXIT
                            done = True
                            break
                        else:
                            print('-------FAILED--------\n')
                            print("unknown Command\n")
                            print('-----------------------\n')
                    if done:
                        print('-------THANKS--------\n')
                        print('Thanks for Using Medigo!\n')
                        print('-----------------------\n')
                        break
                else:
                    print('-------FAILED--------\n')
                    print("Wrong Username/Password.\n")
                    print('-----------------------\n')

            else:
                print('-------FAILED--------\n')
                print("unknown Command\n")
                print('-----------------------\n')


m = MedigoProgram()

user='root'
password = 'Ash12#ar'
host ='localhost'
database = 'Medigo'
port = 3306

m.intialize(host,user,password,database,port)
m.menue();
   
