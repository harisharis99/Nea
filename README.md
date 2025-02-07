import customtkinter as ctk
from tkinter import messagebox
import math
import random
import sqlite3


def conversion(password):
    hashvalue = 0
    prime = 31
    salt = 123456789  # Using a static salt to enhance the uniqueness

    for i, char in enumerate(password):
        # XOR each character's ASCII with the salt and shift by the index
        hashvalue = (hashvalue ^ (ord(char) * prime + salt)) << (i % 5)  # Bitwise shift to make it more complex
        hashvalue = hashvalue & 0xFFFFFFFFFFFFFFFF  # Keep hash within 64-bit range

    # Add some letters for even more complexity
    alphabet = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
    hashstr = str(hashvalue)
    for i in range(5):  # Add random letters to the end
        hashstr += alphabet[(int(hashvalue) + i) % len(alphabet)]

    return hashstr# Return as stringhash value instead of the function itself

# Create or connect to the database
def logindata_db():
    conn = sqlite3.connect('user_login.db')
    cursor = conn.cursor()

    # Create users table
    cursor.execute('''
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        username TEXT NOT NULL UNIQUE,
        password TEXT NOT NULL,
        chero TEXT,
        celement TEXT,
        chealth TEXT, 
        cstrenght TEXT,
        hrarity TEXT,
        cweapon TEXT,
        cweapontype TEXT,
        cweapondamage TEXT,
        cweaponabilitiy TEXT,
        wrarity TEXT,
        level INTEGER,
        currenthealth TEXT,
        currentdamage TEXT
    )
    ''')

    conn.commit()
    conn.close()

def setcurrenthealth(user_id, healthvalue):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE users SET currenthealth = ? WHERE id = ?', (healthvalue, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")

def setcurrentdamage(user_id, damagevalue):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE users SET currentdamage = ? WHERE id = ?', (damagevalue, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")

def setchero(heroname, user_id):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE users SET chero = ? WHERE id = ?', (heroname, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")

def setcelement(elementname, user_id):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE users SET celement = ? WHERE id = ?', (elementname, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")

def setchealth(healthvalue, user_id):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE users SET chealth = ? WHERE id = ?', (healthvalue, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")

def setcstrenght(strengthvalue, user_id):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE users SET cstrenght = ? WHERE id = ?', (strengthvalue, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")

def sethrarity(herorarity, user_id):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE users SET hrarity = ? WHERE id = ?', (herorarity, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")

def setcweapon(weaponname, user_id):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE users SET cweapon = ? WHERE id = ?', (weaponname, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")

def setcweapontype(weapontype, user_id):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE users SET cweapontype = ? WHERE id = ?', (weapontype, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")

def setcweapondamage(weapondamage, user_id):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE users SET cweapondamage = ? WHERE id = ?', (weapondamage, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")

def setcweaponability(weaponability, user_id):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE users SET cweaponabilitiy = ? WHERE id = ?', (weaponability, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")

def setwrarity(weaponrarity, user_id):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE users SET wrarity = ? WHERE id = ?', (weaponrarity, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")

def getcsummon(user_id):
    conn = sqlite3.connect('user_login.db')
    cursor = conn.cursor()
    cursor.execute('SELECT chero, celement, chealth, cstrenght, hrarity, cweapon, cweapontype, cweapondamage, cweaponabilitiy, wrarity FROM users WHERE id = ?', (user_id,))
    result = cursor.fetchone()
    conn.close()
    if result:
        return result  # Returns a tuple with hero, element, hero rarity, weapon, weapon type, weapon rarity
    return None

def calculatehealth(level, chealth, element):
    healthincrementperlevel = 20
    element_multipliers = {
        "Fire": 1.1,
        "Water": 1.2,
        "Earth": 1.3,
        "Wind": 1.0,
        "Electric": 1.15,
        "Ice": 1.25,
        "Light": 1.2,
        "Dark": 1.1
    }
    multiplier = element_multipliers.get(element, 1.0)
    return int(chealth + (healthincrementperlevel * level) * multiplier)

def check_db_data():
     conn = sqlite3.connect('user_login.db')
     cursor = conn.cursor()
     cursor.execute('SELECT * FROM users')
     data = cursor.fetchall()
     print("Current users in database:", data)
     conn.close()

def getuser_id(username):
    conn = sqlite3.connect('user_login.db')
    cursor = conn.cursor()
    cursor.execute("SELECT id FROM users WHERE username = ?", (username,))
    result = cursor.fetchone()
    conn.close()
    if result:
        return result[0]
    return None

def deleter():
    conn = sqlite3.connect('user_login.db')
    cursor = conn.cursor()
    cursor.execute('DROP TABLE IF EXISTS users')
    conn.commit()
    conn.close()

def setlevel(user_id, level):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE users SET level = ? WHERE id = ?', (level, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")

def calculateabilitydamage(basedamage):
    return basedamage * 10 

def getelementbonus(attacker_element, defender_element):
    elementbonus = {
        "Fire": "Ice",
        "Water": "Fire",
        "Earth": "Electric",
        "Wind": "Earth",
        "Electric": "Water",
        "Ice": "Wind",
        "Light": "Dark",
        "Dark": "Light"
    }
    if elementbonus.get(attacker_element) == defender_element:
        return 1.2  # 20% more damage
    return 1.0

class Entrance:
    def __init__(self, main,user_id):
       self.entrance = (main)
       self.entrance.title("Entrance")
       self.entrance.geometry("400x600")
       self.entrance.configure(fg_color="#96c4df")
       self.userid = user_id

       self.label = ctk.CTkLabel(self.entrance, text="Entrance", fg_color="#96c4df",text_color="#333333", font=("Helvetica", 20))
       self.label.pack(pady=20)

       self.openloginb =  ctk.CTkButton(self.entrance, text="Login", fg_color="white", hover_color="#a5cd9d",text_color="#333333",font=("Arial", 15), command=self.openloginmenu)
       self.openloginb.pack(pady=20)

       self.openentrance =  ctk.CTkButton(self.entrance, text="Register", fg_color="white",hover_color="#a5cd9d",text_color="#333333", font=("Arial", 15),command=self.openregistermenu)
       self.openentrance.pack(pady=20)

    def openloginmenu(self):
       self.login = LoginMenu(main)
       self.entrance.withdraw()

    def openregistermenu(self):
       self.registermenu = RegisterMenu(main,self.userid)
       self.entrance.withdraw()

class LoginMenu:
    def __init__(self, main):
       self.login = ctk.CTkToplevel(main)
       self.login.title("Login")
       self.login.configure(fg_color="#96c4df")
       self.login.geometry("800x400")

       self.label = ctk.CTkLabel(self.login, text="Login", fg_color="#96c4df",text_color="#333333", font=("Helvetica", 20))
       self.label.pack(pady=20)

       self.userl = ctk.CTkLabel(self.login, text="Username", fg_color="#96c4df",text_color="#333333" ,font=("Arial", 15))
       self.userl.place(x=100, y=100)

       self.passl = ctk.CTkLabel(self.login, text="Password", fg_color="#96c4df",text_color="#333333", font=("Arial", 15))
       self.passl.place(x=100, y=150)

       self.usere = ctk.CTkEntry(self.login, font=("Arial", 15))
       self.usere.place(x=240, y=100)

       self.passe = ctk.CTkEntry(self.login, font=("Arial", 15))
       self.passe.place(x=240, y=150)

       self.loginb =  ctk.CTkButton(self.login, text="Login", fg_color="white", hover_color="#a5cd9d",text_color="#333333", font=("Arial", 15),command=self.checklogin)
       self.loginb.place(x=240, y=200)

       self.back =  ctk.CTkButton(self.login, text="Return to entrance page", fg_color="white", hover_color="#a5cd9d",text_color="#333333",font=("Arial", 15), command=self.backtoentrance)
       self.back.pack(pady=20, padx=20)
       self.back.place(x=265, y=350)

    def checklogin(self):
        username = self.usere.get()
        password = self.passe.get()

        # Connect to the database to verify the login credentials
        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()
        cursor.execute('SELECT id, password FROM users WHERE username = ?', (username,))
        row = cursor.fetchone()

        if row is not None:
            self.userid, storedhashedvalue = row  # Get user ID and stored hashed password from the database
            hashedpassword = conversion(password)  # Convert entered password into a hashed format

            # Debugging outputs to help trace the issue
            print(f"User ID: {self.userid}")
            print(f"Username: {username}")
            print(f"Hashed Password: '{hashedpassword}'")

            # Check if the entered password matches the stored password
            if storedhashedvalue == hashedpassword:
                self.username = username  # Store username for later use
                self.openmainmenu()  # Proceed to main menu if the password matches
            else:
                messagebox.showerror("Incorrect password. Please try again.")
        else:
            messagebox.showerror("Username not found.")

        conn.close()

    def getusername(self):
        return self.username

    def getuser_id(self):
        return self.userid

    def openmainmenu(self):
        if hasattr(self, 'userid') and self.userid is not None:
            self.mainmenu = MainMenu(main, self.userid)  # Pass both 'main' and 'user_id' to MainMenu
            self.login.withdraw()  # Hide the login window
        else:
            messagebox.showerror("Error", "User ID not available.")

    def backtoentrance(self):
       self.login.destroy()
       main.deiconify()


class RegisterMenu:
    def __init__(self, main,user_id):
       self.register = ctk.CTkToplevel(main)
       self.register.title("Register")
       self.register.configure(fg_color="#96c4df")
       self.register.geometry("800x400")
       self.userid = user_id


       self.label = ctk.CTkLabel(self.register, text="Register", fg_color="#96c4df",text_color="#333333", font=("Helvetica", 20))
       self.label.pack(pady=20)

       self.userl = ctk.CTkLabel(self.register, text="Username", fg_color="#96c4df", font=("Arial", 15))
       self.userl.place(x=100, y=100)

       self.passl = ctk.CTkLabel(self.register, text="Password", fg_color="#96c4df", font=("Arial", 15))
       self.passl.place(x=100, y=150)

       self.usere = ctk.CTkEntry(self.register, font=("Arial", 15))
       self.usere.place(x=240, y=100)

       self.passe = ctk.CTkEntry(self.register, font=("Arial", 15))
       self.passe.place(x=240, y=150)

       self.registerb =  ctk.CTkButton(self.register, text="Create account", fg_color="white",hover_color="#a5cd9d",text_color="#333333", font=("Arial", 15),command=self.createaccount)
       self.registerb.place(x=240, y=200)

       self.back =  ctk.CTkButton(self.register, text="Return to entrance page", fg_color="white",hover_color="#a5cd9d",text_color="#333333", font=("Arial", 15), command=self.backtoentrance)
       self.back.pack(pady=20, padx=20)
       self.back.place(x=265, y=350)

    def openmainmenu(self):
       self.mainmenu = MainMenu(main,self.userid)
       self.register.withdraw()

    def backtoentrance(self):
       self.register.destroy()
       main.deiconify()

    def createaccount(self):
        username = self.usere.get()
        password = self.passe.get()

        if not username or not password:
            messagebox.showerror("Input Error", "Username and Password cannot be empty!")
            return

        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()
        try:
            cursor.execute('SELECT * FROM users WHERE username = ?', (username,))
            if cursor.fetchone():
                messagebox.showerror("Username already exists!")
            else:
                hashedpassword = conversion(password)  # Hash the password before storing
                cursor.execute('INSERT INTO users (username, password, currenthealth, currentdamage) VALUES (?, ?, ?, ?)', 
                               (username, hashedpassword, '100', '0'))
                conn.commit()
                messagebox.showinfo("Account created successfully!")
                print(f"New account created: Username is '{username}' and Hashed password is '{hashedpassword}'") 

                self.openmainmenu()
        except sqlite3.Error as e:
            messagebox.showerror("Database Error", f"An error occurred: {str(e)}")
        finally:
            conn.close()

class MainMenu:
    def __init__(self, main, user_id):
       self.main = ctk.CTkToplevel(main)
       self.main.title("Main Menu")
       self.main.geometry("400x200")
       self.main.configure(fg_color="#96c4df")
       self.userid = user_id

       self.label = ctk.CTkLabel(self.main, text="Main Menu", fg_color="#96c4df",text_color="#333333", font=("Arial", 20))
       self.label.pack(pady=10)

       self.battle =  ctk.CTkButton(self.main, text="Battle", fg_color="white", hover_color="#a5cd9d",text_color="#333333", font=("Arial", 15),command=self.openbattle)
       self.battle.pack(pady=20)
       self.battle.place(x=50, y=50)

       self.summon =  ctk.CTkButton(self.main, text="Summon", fg_color="white", hover_color="#a5cd9d",text_color="#333333", font=("Arial", 15),command=self.opensummon)
       self.summon.pack(pady=20)
       self.summon.place(x=220, y=50)

       self.settings =  ctk.CTkButton(self.main, text="Settings", fg_color="white", hover_color="#a5cd9d",text_color="#333333",font=("Arial", 15), command=self.opensettings)
       self.settings.pack(pady=20)
       self.settings.place(x=50, y=140)

       self.user =  ctk.CTkButton(self.main, text="User", fg_color="white", hover_color="#a5cd9d",text_color="#333333", font=("Arial", 15),command=self.openusers)
       self.user.pack(pady=20)
       self.user.place(x=220, y=140)

    def openbattle(self):
       self.main.withdraw()
       self.battle = Battle(self.main, self.userid)

    def opensummon(self):
        self.main.withdraw()  
        self.summon = Summoning(self.main, self.userid)


    def opensettings(self):
       self.main.withdraw()
       self.settings = Settings(self.main,self.userid)

    def openusers(self):
       self.main.withdraw()
       self.users = Users(self.main,self.userid)


class Modmenu:
    def __init__(self, main, user_id, heroes, weapons, heroprob, weaponprob, heroelements, herohealth, herostrenth, weapontypes, weapondmg, weaponabilities, userswindow):
        self.modmenu = ctk.CTkToplevel(main)
        self.modmenu.title("Mod Menu")
        self.modmenu.geometry("600x400")
        self.modmenu.configure(fg_color="#96c4df")
        self.userid = user_id
        self.heroes = heroes
        self.weapons = weapons
        self.heroprob = heroprob
        self.weaponprob = weaponprob
        self.heroelements = heroelements
        self.herohealth = herohealth
        self.herostrenth = herostrenth
        self.weapontypes = weapontypes
        self.weapondmg = weapondmg
        self.weaponabilities = weaponabilities
        self.userswindow = userswindow

        self.label = ctk.CTkLabel(self.modmenu, text="Mod Menu", fg_color="#96c4df", text_color="#333333", font=("Helvetica", 20))
        self.label.pack(pady=20)

        # Add level setting frame
        self.levelframe = ctk.CTkFrame(self.modmenu, fg_color="#96c4df")
        self.levelframe.pack(pady=10)

        self.levellabel = ctk.CTkLabel(self.levelframe, text="Set Level:", fg_color="#96c4df", text_color="#333333", font=("Arial", 15))
        self.levellabel.pack(side="left", padx=5)

        self.levelentry = ctk.CTkEntry(self.levelframe, font=("Arial", 15), width=100)
        self.levelentry.pack(side="left", padx=5)

        self.setlevelbutton = ctk.CTkButton(self.levelframe, text="Set Level", fg_color="white", hover_color="#a5cd9d", text_color="#333333", font=("Arial", 15), command=self.updatelevel)
        self.setlevelbutton.pack(side="left", padx=5)

        self.herolistbox = ctk.CTkScrollableFrame(self.modmenu, width=250, height=300)
        self.herolistbox.pack(side="left", padx=10, pady=10)

        self.weaponlistbox = ctk.CTkScrollableFrame(self.modmenu, width=250, height=300)
        self.weaponlistbox.pack(side="right", padx=10, pady=10)

        self.populatelistbox(self.herolistbox, heroes, heroprob, self.selecthero)
        self.populatelistbox(self.weaponlistbox, weapons, weaponprob, self.selectweapon)

        self.returnbutton = ctk.CTkButton(self.modmenu, text="Return to User Menu", fg_color="white", hover_color="#a5cd9d", text_color="#333333", font=("Arial", 15), command=self.returntousers)
        self.returnbutton.pack(pady=10)

    def populatelistbox(self, listbox, items, probs, command):
        for item, prob in zip(items, probs):
            color = self.getraritycolor(prob)
            button = ctk.CTkButton(listbox, text=item, text_color=color, font=("Arial", 15), command=lambda i=item: command(i))
            button.pack(pady=5)

    def getraritycolor(self, prob):
        if prob == 20:
            return "green"
        elif prob == 10:
            return "#2542ce"
        elif prob == 5:
            return "purple"
        elif prob == 1:
            return "#000000"
        return "gray"

    def selecthero(self, heroname):
        element = self.heroelements[heroname]
        health = self.herohealth[heroname]
        strength = self.herostrenth[heroname]
        prob = self.heroprob[self.heroes.index(heroname)]
        rarity = self.getraritycolor(prob)

        setchero(heroname, self.userid)
        setcelement(element, self.userid)
        sethrarity(rarity, self.userid)
        setchealth(health, self.userid)
        setcstrenght(strength, self.userid)

        messagebox.showinfo("Hero Selected", f"Hero: {heroname}\nElement: {element}\nHealth: {health}\nStrength: {strength}")

    def selectweapon(self, weaponname):
        weapontype = self.weapontypes[weaponname]
        weapondamage = self.weapondmg[weaponname]
        weaponability = self.weaponabilities[weaponname]
        prob = self.weaponprob[self.weapons.index(weaponname)]
        rarity = self.getraritycolor(prob)

        setcweapon(weaponname, self.userid)
        setcweapontype(weapontype, self.userid)
        setwrarity(rarity, self.userid)
        setcweapondamage(weapondamage, self.userid)
        setcweaponability(weaponability, self.userid)

        messagebox.showinfo("Weapon Selected", f"Weapon: {weaponname}\nType: {weapontype}\nDamage: {weapondamage}\nAbility: {weaponability}")

    def returntousers(self):
        self.modmenu.destroy()
        self.userswindow.users.deiconify()

    def updatelevel(self):
        try:
            newlevel = int(self.levelentry.get())
            if newlevel < 0:
                messagebox.showerror("Error", "Level cannot be negative!")
                return
            setlevel(self.userid, newlevel)
            messagebox.showinfo("Success", f"Level updated to {newlevel}")
        except ValueError:
            messagebox.showerror("Error", "Please enter a valid number!")

class Summoning:
    def __init__(self, main, user_id):
        self.summon = ctk.CTkToplevel(main)
        self.summon.title("Summon")
        self.summon.geometry("400x1100")  # Increased height by 100 pixels
        self.summon.configure(fg_color="#96c4df")
        self.userid = user_id

        self.label = ctk.CTkLabel(self.summon, text="Summoning Altar", fg_color="#96c4df", text_color="#333333", font=("Helvetica", 20))
        self.label.grid(row=0, column=1, columnspan=2, pady=20)

        self.herob = ctk.CTkButton(self.summon, text="Summon Hero", fg_color="white", hover_color="#a5cd9d", text_color="#333333", font=("Arial", 15), command=self.summonhero)
        self.herob.grid(row=1, column=0, pady=10)

        self.weaponb = ctk.CTkButton(self.summon, text="Summon Weapon", fg_color="white", hover_color="#a5cd9d", text_color="#333333", font=("Arial", 15), command=self.summonweapon)
        self.weaponb.grid(row=1, column=2, pady=10)

        self.heroinfo = ctk.CTkLabel(self.summon, text="HERO INFO", fg_color="#96c4df", text_color="#333333", font=("Arial", 15))
        self.heroinfo.grid(row=2, column=0, pady=10)

        self.heroresult = ctk.CTkLabel(self.summon, text="", fg_color="#96c4df", font=("Arial", 15))
        self.heroresult.grid(row=3, column=0, pady=10)

        self.elementresult = ctk.CTkLabel(self.summon, text="", fg_color="#96c4df", font=("Arial", 15))
        self.elementresult.grid(row=4, column=0, pady=10)

        self.healthresult = ctk.CTkLabel(self.summon, text="", fg_color="#96c4df", font=("Arial", 15))
        self.healthresult.grid(row=5, column=0, pady=10)

        self.strenthgresult = ctk.CTkLabel(self.summon, text="", fg_color="#96c4df", font=("Arial", 15))
        self.strenthgresult.grid(row=6, column=0, pady=10)

        self.weaponinfo = ctk.CTkLabel(self.summon, text="WEAPON INFO", fg_color="#96c4df", text_color="#333333", font=("Arial", 15))
        self.weaponinfo.grid(row=2, column=2, pady=10)

        self.weaponresult = ctk.CTkLabel(self.summon, text="", fg_color="#96c4df", font=("Arial", 15))
        self.weaponresult.grid(row=3, column=2, pady=10)

        self.typeresult = ctk.CTkLabel(self.summon, text="", fg_color="#96c4df", font=("Arial", 15))
        self.typeresult.grid(row=4, column=2, pady=10)

        self.damageresult = ctk.CTkLabel(self.summon, text="", fg_color="#96c4df", font=("Arial", 15))
        self.damageresult.grid(row=5, column=2, pady=10)

        self.abilityresult = ctk.CTkLabel(self.summon, text="", fg_color="#96c4df", font=("Arial", 15))
        self.abilityresult.grid(row=6, column=2, pady=10)

        self.back = ctk.CTkButton(self.summon, text="Return to main menu", fg_color="white", hover_color="#a5cd9d", text_color="#333333", font=("Arial", 15), command=self.backtomainmenu)
        self.back.grid(row=8, column=1, columnspan=2, pady=20)

        self.currentherolabel = ctk.CTkLabel(self.summon, text='', fg_color="#96c4df", font=("Arial", 15))
        self.currentherolabel.grid(row=9, column=0, columnspan=2, pady=5)

        self.currentweaponlabel = ctk.CTkLabel(self.summon, text='', fg_color="#96c4df", font=("Arial", 15))
        self.currentweaponlabel.grid(row=10, column=0, columnspan=2, pady=5)

        self.updatedisplay()

        self.hero = [
            "Blaze", "Infernia", "Ember", "Ignis", "Vulcan",  # Fire
            "Marina", "Tidal", "Cascade", "Aquaria", "Sirena",  # Water
            "Terra", "Boulder", "Gaia", "Quarrix", "Petrus",  # Earth
            "Zephyr", "Cyclone", "Aeris", "Ventra", "Skylar",  # Wind
            "Volt", "Spark", "Thunderra", "Zappia", "Storme",  # Electric
            "Frost", "Glaciel", "Chilla", "Cryonix", "Shivera",  # Ice
            "Lumina", "Radiant", "Solara", "Aethera", "Halo",  # Light
            "Umbra", "Nyx", "Shadowe", "Noctis", "Eclipse"  # Dark
        ]

        self.heroprob = [1, 1, 20, 10, 5,  # Fire
                         20, 20, 5, 10, 1,  # Water
                         20, 20, 1, 10, 5,  # Earth
                         1, 20, 5, 20, 10,  # Wind
                         5, 10, 1, 20, 20,  # Electric
                         20, 10, 20, 1, 5,  # Ice
                         10, 20, 5, 20, 1,  # Light
                         20, 5, 20, 10, 1]  # Dark

        self.heroelements = {
            "Blaze": "Fire", "Infernia": "Fire", "Ember": "Fire", "Ignis": "Fire", "Vulcan": "Fire",
            "Marina": "Water", "Tidal": "Water", "Cascade": "Water", "Aquaria": "Water", "Sirena": "Water",
            "Terra": "Earth", "Boulder": "Earth", "Gaia": "Earth", "Quarrix": "Earth", "Petrus": "Earth",
            "Zephyr": "Wind", "Cyclone": "Wind", "Aeris": "Wind", "Ventra": "Wind", "Skylar": "Wind",
            "Volt": "Electric", "Spark": "Electric", "Thunderra": "Electric", "Zappia": "Electric", "Storme": "Electric",
            "Frost": "Ice", "Glaciel": "Ice", "Chilla": "Ice", "Cryonix": "Ice", "Shivera": "Ice",
            "Lumina": "Light", "Radiant": "Light", "Solara": "Light", "Aethera": "Light", "Halo": "Light",
            "Umbra": "Dark", "Nyx": "Dark", "Shadowe": "Dark", "Noctis": "Dark", "Eclipse": "Dark"
        }

        self.herohealth = {
            "Blaze": "200", "Infernia": "200", "Ember": "100", "Ignis": "150", "Vulcan": "175",  # Fire
            "Marina": "100", "Tidal": "100", "Cascade": "175", "Aquaria": "150", "Sirena": "200",  # Water
            "Terra": "100", "Boulder": "100", "Gaia": "200", "Quarrix": "150", "Petrus": "175",  # Earth
            "Zephyr": "200", "Cyclone": "100", "Aeris": "175", "Ventra": "100", "Skylar": "150",  # Wind
            "Volt": "175", "Spark": "150", "Thunderra": "200", "Zappia": "100", "Storme": "100",  # Electric
            "Frost": "100", "Glaciel": "150", "Chilla": "100", "Cryonix": "200", "Shivera": "175",  # Ice
            "Lumina": "150", "Radiant": "100", "Solara": "175", "Aethera": "100", "Halo": "200",  # Light
            "Umbra": "100", "Nyx": "175", "Shadowe": "100", "Noctis": "150", "Eclipse": "200"   # Dark
        }

        self.herostrenth = {
            "Blaze": "100", "Infernia": "100", "Ember": "25", "Ignis": "50", "Vulcan": "75",  # Fire
            "Marina": "25", "Tidal": "25", "Cascade": "75", "Aquaria": "50", "Sirena": "100",  # Water
            "Terra": "25", "Boulder": "25", "Gaia": "100", "Quarrix": "50", "Petrus": "75",  # Earth
            "Zephyr": "100", "Cyclone": "25", "Aeris": "75", "Ventra": "25", "Skylar": "50",  # Wind
            "Volt": "75", "Spark": "50", "Thunderra": "100", "Zappia": "25", "Storme": "25",  # Electric
            "Frost": "25", "Glaciel": "50", "Chilla": "25", "Cryonix": "100", "Shivera": "75",  # Ice
            "Lumina": "50", "Radiant": "25", "Solara": "75", "Aethera": "25", "Halo": "100",  # Light
            "Umbra": "25", "Nyx": "75", "Shadowe": "25", "Noctis": "50", "Eclipse": "100"   # Dark
        }

        self.weapons = [
            "Excalibur", "Flametongue", "Blade of Inferno", "Molten Edge", "Lava Cleaver",  # Sword
            "Aqua Harpoon", "Poseidon’s Spear", "Coral Lance", "Tidal Pike", "Marine Javelin",  # Spear
            "Rock Crusher", "Earthsplitter", "Terra Mace", "Stone Maul", "Boulder Hammer",  # Axe
            "Zephyr Bow", "Storm Bow", "Gale Shooter", "Tempest Quiver", "Windrunner Bow",  # Bow
            "Lightning Rod", "Thunder Staff", "Volt Wand", "Storm Orb", "Electro Scepter",  # Staff
            "Frost Fang", "Ice Fang", "Glacier Blade", "Cryo Dagger", "Shiver Knife",  # Dagger
            "Radiant Shield", "Halo Guard", "Solar Defender", "Light Barrier", "Aether Buckler",  # Shield
            "Umbra Scythe", "Nyx Reaper", "Shadow Blade", "Noctis Saber", "Eclipse Cutter"  # Scythe
        ]

        self.weaponprob = [1, 20, 5, 10, 20,  # Sword
                           20, 1, 10, 5, 20,  # Spear
                           20, 1, 5, 20, 10,  # Axe
                           1, 5, 20, 10, 20,  # Bow
                           20, 20, 10, 5, 1,  # Staff
                           5, 20, 10, 10, 1,  # Dagger
                           20, 1, 5, 10, 20,  # Shield
                           20, 5, 10, 20, 1]  # Scythe

        self.weaponabilities = {
            "Excalibur": "Flame Sever", "Flametongue": "None", "Blade of Inferno": "None", "Molten Edge": "None", "Lava Cleaver": "None",
            "Aqua Harpoon": "None", "Poseidon’s Spear": "None", "Coral Lance": "None", "Tidal Pike": "None", "Marine Javelin": "Tidal Fury",
            "Rock Crusher": "None", "Earthsplitter": "None", "Terra Mace": "None", "Stone Maul": "None", "Boulder Hammer": "Rockfall",
            "Zephyr Bow": "None", "Storm Bow": "None", "Gale Shooter": "None", "Tempest Quiver": "None", "Windrunner Bow": "Windstrike",
            "Lightning Rod": "None", "Thunder Staff": "None", "Volt Wand": "None", "Storm Orb": "None", "Electro Scepter": "Thunderstorm",
            "Frost Fang": "None", "Ice Fang": "None", "Glacier Blade": "None", "Cryo Dagger": "None", "Shiver Knife": "Ice Storm",
            "Radiant Shield": "None", "Halo Guard": "None", "Solar Defender": "None", "Light Barrier": "None", "Aether Buckler": "Aether Guard",
            "Umbra Scythe": "None", "Nyx Reaper": "None", "Shadow Blade": "None", "Noctis Saber": "None", "Eclipse Cutter": "Shadow Cleaver"
        }

        self.weapondmg = {
            "Excalibur": "100", "Flametongue": "25", "Blade of Inferno": "75", "Molten Edge": "50", "Lava Cleaver": "25",
            "Aqua Harpoon": "25", "Poseidon’s Spear": "75", "Coral Lance": "25", "Tidal Pike": "50", "Marine Javelin": "100",
            "Rock Crusher": "25", "Earthsplitter": "75", "Terra Mace": "50", "Stone Maul": "25", "Boulder Hammer": "100",
            "Zephyr Bow": "75", "Storm Bow": "50", "Gale Shooter": "25", "Tempest Quiver": "50", "Windrunner Bow": "100",
            "Lightning Rod": "50", "Thunder Staff": "75", "Volt Wand": "25", "Storm Orb": "50", "Electro Scepter": "100",
            "Frost Fang": "75", "Ice Fang": "50", "Glacier Blade": "25", "Cryo Dagger": "50", "Shiver Knife": "100",
            "Radiant Shield": "75", "Halo Guard": "50", "Solar Defender": "25", "Light Barrier": "50", "Aether Buckler": "100",
            "Umbra Scythe": "75", "Nyx Reaper": "50", "Shadow Blade": "25", "Noctis Saber": "50", "Eclipse Cutter": "100"
        }

        self.weapontypes = {
            "Excalibur": "Sword", "Flametongue": "Sword", "Blade of Inferno": "Sword", "Molten Edge": "Sword","Lava Cleaver": "Sword",
            "Aqua Harpoon": "Spear", "Poseidon’s Spear": "Spear", "Coral Lance": "Spear", "Tidal Pike": "Spear","Marine Javelin": "Spear",
            "Rock Crusher": "Axe", "Earthsplitter": "Axe", "Terra Mace": "Axe", "Stone Maul": "Axe","Boulder Hammer": "Axe",
            "Zephyr Bow": "Bow", "Storm Bow": "Bow", "Gale Shooter": "Bow", "Tempest Quiver": "Bow","Windrunner Bow": "Bow",
            "Lightning Rod": "Staff", "Thunder Staff": "Staff", "Volt Wand": "Staff", "Storm Orb": "Staff","Electro Scepter": "Staff",
            "Frost Fang": "Dagger", "Ice Fang": "Dagger", "Glacier Blade": "Dagger", "Cryo Dagger": "Dagger","Shiver Knife": "Dagger",
            "Radiant Shield": "Shield", "Halo Guard": "Shield", "Solar Defender": "Shield", "Light Barrier": "Shield","Aether Buckler": "Shield",
            "Umbra Scythe": "Scythe", "Nyx Reaper": "Scythe", "Shadow Blade": "Scythe", "Noctis Saber": "Scythe","Eclipse Cutter":"Scythe"
        }

    def rarity(self, prob):
       if prob == 20:
           return "green"
       elif prob == 10:
           return "#2542ce"
       elif prob == 5:
           return "purple"
       elif prob == 1:
           return "#000000"

    def updatedisplay(self):
        result = getcsummon(self.userid)
        if result:
            chero, celement, chealth, cstrenght, hrarity, cweapon, cweapontype, cweapondamage, cweaponability, wrarity = result
            hrarity = hrarity if hrarity else "gray"  # Set default color if None
            wrarity = wrarity if wrarity else "gray"  # Set default color if None
            self.heroresult.configure(text=f"{chero if chero else 'None'}", text_color=hrarity)
            self.elementresult.configure(text=f"Element: {celement if celement else 'None'}", text_color=hrarity)
            self.healthresult.configure(text=f"Health: {chealth if chealth else 'None'}", text_color=hrarity)
            self.strenthgresult.configure(text=f"Strength: {cstrenght if cstrenght else 'None'}", text_color=hrarity)

            self.weaponresult.configure(text=f"{cweapon if cweapon else 'None'}", text_color=wrarity)
            self.typeresult.configure(text=f"Type: {cweapontype if cweapontype else 'None'}", text_color=wrarity)
            self.damageresult.configure(text=f"Damage: {cweapondamage if cweapondamage else 'None'}", text_color=wrarity)
            self.abilityresult.configure(text=f"Ability: {cweaponability if cweaponability else 'None'}", text_color=wrarity)
        else:
            # Handle the case where getcsummon() returns None
            self.heroresult.configure(text="None", text_color="gray")
            self.elementresult.configure(text="None", text_color="gray")
            self.healthresult.configure(text="None", text_color="gray")
            self.strenthgresult.configure(text="None", text_color="gray")

            self.weaponresult.configure(text="None", text_color="gray")
            self.typeresult.configure(text="None", text_color="gray")
            self.damageresult.configure(text="None", text_color="gray")
            self.abilityresult.configure(text="None", text_color="gray")


    def summonhero(self):
        result = random.choices(self.hero, weights=self.heroprob, k=1)[0]
        element = self.heroelements[result]
        health = self.herohealth[result]
        strenght = self.herostrenth[result]
        prob = self.heroprob[self.hero.index(result)]
        hrarity = self.rarity(prob)
        self.heroresult.configure(text=f" {result} ", text_color=hrarity)
        self.elementresult.configure(text=f"Element: {element}",text_color= hrarity)
        self.healthresult.configure(text=f"Health: {health}",text_color= hrarity)
        self.strenthgresult.configure(text=f"Strength: {strenght}",text_color=hrarity)
        setchero(result, self.userid)
        setcelement(element, self.userid)
        sethrarity(hrarity, self.userid)
        setchealth(health, self.userid)
        setcstrenght(strenght, self.userid)
        self.save_summon(result, element, hrarity, is_hero=True)
        self.updatedisplay()

        if prob == 1:
           print(f"UNIQUE SUMMON: {result}")
        else:
           print(f"Summoned Hero: {result} with probability {prob}")

        if prob == 1:
           self.spinconformation(self.summonhero)



    def summonweapon(self):
        result = random.choices(self.weapons, weights=self.weaponprob, k=1)[0]
        weapontype = self.weapontypes[result]
        weapondamage = self.weapondmg[result]
        weaponability = self.weaponabilities[result]
        prob = self.weaponprob[self.weapons.index(result)]
        wrarity = self.rarity(prob)
        self.weaponresult.configure(text=f" {result}", text_color=wrarity)
        self.typeresult.configure(text=f"Type: {weapontype}",text_color=wrarity)
        self.damageresult.configure(text=f"Damage: {weapondamage}",text_color=wrarity)
        self.abilityresult.configure(text=f"Ability: {weaponability}",text_color=wrarity)
        setcweapon(result, self.userid)
        setcweapontype(weapontype, self.userid)
        setwrarity(wrarity, self.userid)
        setcweapondamage(weapondamage,self.userid)
        setcweaponability(weaponability, self.userid)
        self.save_summon(result, weapontype, wrarity, is_hero=False)
        self.updatedisplay()

        if prob == 1:
           print(f"UNIQUE SUMMON: {result}")
        else:
           print(f"Summoned Weapon: {result} with probability {prob}")

        if prob == 1:
           self.spinconformation(self.summonweapon)  



    def save_summon(self, name, element_or_type, rarity, is_hero=True):
        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()

        if is_hero:
            cursor.execute('UPDATE users SET chero = ?, celement = ?, chealth = ?, cstrenght = ?, hrarity = ? WHERE id = ?', 
                           (name, element_or_type, self.herohealth[name], self.herostrenth[name], rarity, self.userid))
        else:
            cursor.execute('UPDATE users SET cweapon = ?, cweapontype = ?, cweapondamage = ?, cweaponabilitiy = ?, wrarity = ? WHERE id = ?', 
                           (name, element_or_type, self.weapondmg[name], self.weaponabilities[name], rarity, self.userid))

        conn.commit()
        conn.close()

    def spinconformation(self, summonconformation):
       response = messagebox.askyesno("Lucky summon!", "You got a unique summon are you sure you want to spin again")
       if response:
           summonconformation()

    def backtomainmenu(self):
       self.summon.destroy()
       self.mainmenu = MainMenu(main,self.userid)
       self.mainmenu.main.deiconify()

class Settings:
    def __init__(self, main,user_id):
       self.settings = ctk.CTkToplevel(main)
       self.settings.title("Settings")
       self.settings.geometry("400x200")
       self.settings.configure(fg_color="#96c4df")
       self.userid = user_id

       self.label = ctk.CTkLabel(self.settings, text="Settings", fg_color="#96c4df",text_color="#333333", font=("Helvetica", 20))
       self.label.pack(pady=20)

       self.back =  ctk.CTkButton(self.settings, text="Return to main menu", fg_color="white", hover_color="#a5cd9d",text_color="#333333",font=("Arial", 15), command=self.backtomainmenu)
       self.back.pack(pady=20)

    def backtomainmenu(self):
       self.settings.destroy()
       self.mainmenu = MainMenu(main,self.userid)
       self.mainmenu.main.deiconify()

class Users:
    def __init__(self, main,user_id):
       self.users = ctk.CTkToplevel(main)
       self.users.title("User")
       self.users.geometry("400x400")  
       self.users.configure(fg_color="#96c4df")
       self.userid = user_id 

       self.label = ctk.CTkLabel(self.users, text="User Information", fg_color="#96c4df",text_color="#333333", font=("Helvetica", 20))
       self.label.pack(pady=20)

       if self.userid == 1:
           self.modbutton = ctk.CTkButton(self.users, text="Mod Menu", fg_color="white", hover_color="#a5cd9d", text_color="#333333", font=("Arial", 15), command=self.openmodmenu)
           self.modbutton.pack(pady=20)

       self.back = ctk.CTkButton(self.users, text="Return to main menu", fg_color="white", hover_color="#a5cd9d",text_color="#333333",font=("Arial", 15), command=lambda: self.backtomainmenu(user_id))
       self.back.pack(pady=20)

       self.logout = ctk.CTkButton(self.users, text="Log out", fg_color="white", hover_color="#a5cd9d",text_color="#333333",font=("Arial", 15), command=self.backtoentrance)
       self.logout.pack(pady=20)

    def openmodmenu(self):
        if self.userid == 1:
            # Get all the necessary data that was previously in Summoning class
            self.hero = [
                "Blaze", "Infernia", "Ember", "Ignis", "Vulcan",  # Fire
                "Marina", "Tidal", "Cascade", "Aquaria", "Sirena",  # Water
                "Terra", "Boulder", "Gaia", "Quarrix", "Petrus",  # Earth
                "Zephyr", "Cyclone", "Aeris", "Ventra", "Skylar",  # Wind
                "Volt", "Spark", "Thunderra", "Zappia", "Storme",  # Electric
                "Frost", "Glaciel", "Chilla", "Cryonix", "Shivera",  # Ice
                "Lumina", "Radiant", "Solara", "Aethera", "Halo",  # Light
                "Umbra", "Nyx", "Shadowe", "Noctis", "Eclipse"  # Dark
            ]
            self.heroprob = [1, 1, 20, 10, 5,  # Fire
                             20, 20, 5, 10, 1,  # Water
                             20, 20, 1, 10, 5,  # Earth
                             1, 20, 5, 20, 10,  # Wind
                             5, 10, 1, 20, 20,  # Electric
                             20, 10, 20, 1, 5,  # Ice
                             10, 20, 5, 20, 1,  # Light
                             20, 5, 20, 10, 1]  # Dark
            self.heroelements = {
                "Blaze": "Fire", "Infernia": "Fire", "Ember": "Fire", "Ignis": "Fire", "Vulcan": "Fire",
                "Marina": "Water", "Tidal": "Water", "Cascade": "Water", "Aquaria": "Water", "Sirena": "Water",
                "Terra": "Earth", "Boulder": "Earth", "Gaia": "Earth", "Quarrix": "Earth", "Petrus": "Earth",
                "Zephyr": "Wind", "Cyclone": "Wind", "Aeris": "Wind", "Ventra": "Wind", "Skylar": "Wind",
                "Volt": "Electric", "Spark": "Electric", "Thunderra": "Electric", "Zappia": "Electric", "Storme": "Electric",
                "Frost": "Ice", "Glaciel": "Ice", "Chilla": "Ice", "Cryonix": "Ice", "Shivera": "Ice",
                "Lumina": "Light", "Radiant": "Light", "Solara": "Light", "Aethera": "Light", "Halo": "Light",
                "Umbra": "Dark", "Nyx": "Dark", "Shadowe": "Dark", "Noctis": "Dark", "Eclipse": "Dark"
            }
            self.herohealth = {
                "Blaze": "200", "Infernia": "200", "Ember": "100", "Ignis": "150", "Vulcan": "175",  # Fire
                "Marina": "100", "Tidal": "100", "Cascade": "175", "Aquaria": "150", "Sirena": "200",  # Water
                "Terra": "100", "Boulder": "100", "Gaia": "200", "Quarrix": "150", "Petrus": "175",  # Earth
                "Zephyr": "200", "Cyclone": "100", "Aeris": "175", "Ventra": "100", "Skylar": "150",  # Wind
                "Volt": "175", "Spark": "150", "Thunderra": "200", "Zappia": "100", "Storme": "100",  # Electric
                "Frost": "100", "Glaciel": "150", "Chilla": "100", "Cryonix": "200", "Shivera": "175",  # Ice
                "Lumina": "150", "Radiant": "100", "Solara": "175", "Aethera": "100", "Halo": "200",  # Light
                "Umbra": "100", "Nyx": "175", "Shadowe": "100", "Noctis": "150", "Eclipse": "200"   # Dark
            }
            self.herostrenth = {
                "Blaze": "100", "Infernia": "100", "Ember": "25", "Ignis": "50", "Vulcan": "75",  # Fire
                "Marina": "25", "Tidal": "25", "Cascade": "75", "Aquaria": "50", "Sirena": "100",  # Water
                "Terra": "25", "Boulder": "25", "Gaia": "100", "Quarrix": "50", "Petrus": "75",  # Earth
                "Zephyr": "100", "Cyclone": "25", "Aeris": "75", "Ventra": "25", "Skylar": "50",  # Wind
                "Volt": "75", "Spark": "50", "Thunderra": "100", "Zappia": "25", "Storme": "25",  # Electric
                "Frost": "25", "Glaciel": "50", "Chilla": "25", "Cryonix": "100", "Shivera": "75",  # Ice
                "Lumina": "50", "Radiant": "25", "Solara": "75", "Aethera": "25", "Halo": "100",  # Light
                "Umbra": "25", "Nyx": "75", "Shadowe": "25", "Noctis": "50", "Eclipse": "100"   # Dark
            }
            self.weapons = [
                "Excalibur", "Flametongue", "Blade of Inferno", "Molten Edge", "Lava Cleaver",  # Sword
                "Aqua Harpoon", "Poseidon’s Spear", "Coral Lance", "Tidal Pike", "Marine Javelin",  # Spear
                "Rock Crusher", "Earthsplitter", "Terra Mace", "Stone Maul", "Boulder Hammer",  # Axe
                "Zephyr Bow", "Storm Bow", "Gale Shooter", "Tempest Quiver", "Windrunner Bow",  # Bow
                "Lightning Rod", "Thunder Staff", "Volt Wand", "Storm Orb", "Electro Scepter",  # Staff
                "Frost Fang", "Ice Fang", "Glacier Blade", "Cryo Dagger", "Shiver Knife",  # Dagger
                "Radiant Shield", "Halo Guard", "Solar Defender", "Light Barrier", "Aether Buckler",  # Shield
                "Umbra Scythe", "Nyx Reaper", "Shadow Blade", "Noctis Saber", "Eclipse Cutter"  # Scythe
            ]
            self.weaponprob = [1, 20, 5, 10, 20,  # Sword
                               20, 1, 10, 5, 20,  # Spear
                               20, 1, 5, 20, 10,  # Axe
                               1, 5, 20, 10, 20,  # Bow
                               20, 20, 10, 5, 1,  # Staff
                               5, 20, 10, 10, 1,  # Dagger
                               20, 1, 5, 10, 20,  # Shield
                               20, 5, 10, 20, 1]  # Scythe
            self.weaponabilities = {
                "Excalibur": "Flame Sever", "Flametongue": "None", "Blade of Inferno": "None", "Molten Edge": "None", "Lava Cleaver": "None",
                "Aqua Harpoon": "None", "Poseidon’s Spear": "None", "Coral Lance": "None", "Tidal Pike": "None", "Marine Javelin": "Tidal Fury",
                "Rock Crusher": "None", "Earthsplitter": "None", "Terra Mace": "None", "Stone Maul": "None", "Boulder Hammer": "Rockfall",
                "Zephyr Bow": "None", "Storm Bow": "None", "Gale Shooter": "None", "Tempest Quiver": "None", "Windrunner Bow": "Windstrike",
                "Lightning Rod": "None", "Thunder Staff": "None", "Volt Wand": "None", "Storm Orb": "None", "Electro Scepter": "Thunderstorm",
                "Frost Fang": "None", "Ice Fang": "None", "Glacier Blade": "None", "Cryo Dagger": "None", "Shiver Knife": "Ice Storm",
                "Radiant Shield": "None", "Halo Guard": "None", "Solar Defender": "None", "Light Barrier": "None", "Aether Buckler": "Aether Guard",
                "Umbra Scythe": "None", "Nyx Reaper": "None", "Shadow Blade": "None", "Noctis Saber": "None", "Eclipse Cutter": "Shadow Cleaver"
            }
            self.weapondmg = {
                "Excalibur": "100", "Flametongue": "25", "Blade of Inferno": "75", "Molten Edge": "50", "Lava Cleaver": "25",
                "Aqua Harpoon": "25", "Poseidon’s Spear": "75", "Coral Lance": "25", "Tidal Pike": "50", "Marine Javelin": "100",
                "Rock Crusher": "25", "Earthsplitter": "75", "Terra Mace": "50", "Stone Maul": "25", "Boulder Hammer": "100",
                "Zephyr Bow": "75", "Storm Bow": "50", "Gale Shooter": "25", "Tempest Quiver": "50", "Windrunner Bow": "100",
                "Lightning Rod": "50", "Thunder Staff": "75", "Volt Wand": "25", "Storm Orb": "50", "Electro Scepter": "100",
                "Frost Fang": "75", "Ice Fang": "50", "Glacier Blade": "25", "Cryo Dagger": "50", "Shiver Knife": "100",
                "Radiant Shield": "75", "Halo Guard": "50", "Solar Defender": "25", "Light Barrier": "50", "Aether Buckler": "100",
                "Umbra Scythe": "75", "Nyx Reaper": "50", "Shadow Blade": "25", "Noctis Saber": "50", "Eclipse Cutter": "100"
            }
            self.weapontypes = {
                "Excalibur": "Sword", "Flametongue": "Sword", "Blade of Inferno": "Sword", "Molten Edge": "Sword","Lava Cleaver": "Sword",
                "Aqua Harpoon": "Spear", "Poseidon’s Spear": "Spear", "Coral Lance": "Spear", "Tidal Pike": "Spear","Marine Javelin": "Spear",
                "Rock Crusher": "Axe", "Earthsplitter": "Axe", "Terra Mace": "Axe", "Stone Maul": "Axe","Boulder Hammer": "Axe",
                "Zephyr Bow": "Bow", "Storm Bow": "Bow", "Gale Shooter": "Bow", "Tempest Quiver": "Bow","Windrunner Bow": "Bow",
                "Lightning Rod": "Staff", "Thunder Staff": "Staff", "Volt Wand": "Staff", "Storm Orb": "Staff","Electro Scepter": "Staff",
                "Frost Fang": "Dagger", "Ice Fang": "Dagger", "Glacier Blade": "Dagger", "Cryo Dagger": "Dagger","Shiver Knife": "Dagger",
                "Radiant Shield": "Shield", "Halo Guard": "Shield", "Solar Defender": "Shield", "Light Barrier": "Shield","Aether Buckler": "Shield",
                "Umbra Scythe": "Scythe", "Nyx Reaper": "Scythe", "Shadow Blade": "Scythe", "Noctis Saber": "Scythe","Eclipse Cutter":"Scythe"
            }
            self.users.withdraw()
            self.modmenu = Modmenu(self.users, self.userid, self.hero, self.weapons, 
                                  self.heroprob, self.weaponprob, self.heroelements, 
                                  self.herohealth, self.herostrenth, self.weapontypes, 
                                  self.weapondmg, self.weaponabilities, self)
        else:
            messagebox.showerror("Access Denied", "You do not have permission to access the Mod Menu.")

    def backtomainmenu(self,user_id):
       self.users.destroy()
       self.mainmenu = MainMenu(main,user_id)
       self.mainmenu.main.deiconify()

    def backtoentrance(self):
       self.users.destroy()
       main.deiconify()

class SiegeMenu:
    def __init__(self, main, user_id):
        self.siege = ctk.CTkToplevel(main)
        self.siege.title("Siege")
        self.siege.geometry("400x450")
        self.siege.configure(fg_color="#96c4df")
        self.userid = user_id

        self.label = ctk.CTkLabel(self.siege, text="Hall of Blades", fg_color="#96c4df", text_color="#333333", font=("Helvetica", 20))
        self.label.pack(pady=20)

        self.startbattlebutton = ctk.CTkButton(self.siege, text="Start battle", fg_color="white", hover_color="#a5cd9d", text_color="#333333", font=("Helvetica", 20), command=lambda: self.battlebutton(self.levelup, self.playscene))
        self.startbattlebutton.pack(pady=20)

        self.damagetest = ctk.CTkButton(self.siege, text="Damage test", fg_color="white", hover_color="#a5cd9d", text_color="#333333", font=("Helvetica", 20), command=self.calculatedamage)
        self.damagetest.pack(pady=20)

        self.damageresult = ctk.CTkLabel(self.siege, text="Not Measured", fg_color="#96c4df", text_color="#333333", font=("Helvetica", 20))
        self.damageresult.pack(pady=20)


        self.returnbutton = ctk.CTkButton(self.siege, text="Return to Battle Menu", fg_color="white", hover_color="#a5cd9d", text_color="#333333", font=("Helvetica", 20), command=self.backtobattlemenu)
        self.returnbutton.pack(pady=20)

    def calculatedamage(self):
        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()

        # Fetch cstrenght, cweapondamage, and level for the given user_id
        cursor.execute('''
        SELECT cstrenght, cweapondamage, level FROM users WHERE id = ?
        ''', (self.userid,))
        result = cursor.fetchone()
        conn.close()

        if result:
            cstrenght, cweapondamage, level = result
            # Convert values to integers
            cstrenght = int(cstrenght) if cstrenght else 0
            cweapondamage = int(cweapondamage) if cweapondamage else 0
            level = int(level) if level else 0

            # Define the formula to calculate damage
            basedamage = cstrenght * cweapondamage
            levelscaling = (1 + (level ** 1.2) / 10)  # Exponential scaling with level
            randomfactor = random.uniform(0.9, 1.2)  # Adds randomness (90% - 120% of calculated value)
            criticalhit = 2 if random.random() < 0.1 else 1  # 10% chance to double damage

            # Final damage calculation
            damage = math.floor(basedamage * levelscaling * randomfactor * criticalhit)

            # Update the damageresult label with the calculated damage
            self.damageresult.configure(text=f"Damage: {damage}")

            # Update currentdamage in the database
            setcurrentdamage(self.userid, str(damage))
        else:
            self.damageresult.configure(text="Damage: Not Measured")

    def levelup(self, user_id):
        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()

        # Fetch the current level
        cursor.execute('SELECT level FROM users WHERE id = ?', (user_id,))
        level = cursor.fetchone()

        if level is None:  # User not found
            conn.close()
            print("Level not found for user:", user_id)
            return None  

        clevel = level[0] if level[0] is not None else 0  # Default to level 0
        nlevel = clevel + 1

        # Update the new level in the database
        cursor.execute('UPDATE users SET level = ? WHERE id = ?', (nlevel, user_id))
        conn.commit()
        conn.close()

        print(f"You have leveled up and are now level {nlevel}!")
        return nlevel  # Return the new level

    def backtobattlemenu(self):
        self.siege.destroy()
        Battle(self.siege.master, self.userid)

    def playscene(self):
        shades = [
            "#000000", "#020202", "#040404", "#060606", "#080808", "#0A0A0A", "#0C0C0C", "#0E0E0E",
            "#101010", "#121212", "#141414", "#161616", "#181818", "#1A1A1A", "#1C1C1C", "#1E1E1E",
            "#202020", "#222222", "#242424", "#262626", "#282828", "#2A2A2A", "#2C2C2C", "#2E2E2E",
            "#303030", "#323232", "#343434", "#363636", "#383838", "#3A3A3A", "#3C3C3C", "#3E3E3E",
            "#404040", "#424242", "#444444", "#464646", "#484848", "#4A4A4A", "#4C4C4C", "#4E4E4E",
            "#505050", "#525252", "#545454", "#565656", "#585858", "#5A5A5A", "#5C5C5C", "#5E5E5E",
            "#606060", "#626262", "#646464", "#666666", "#686868", "#6A6A6A", "#6C6C6C", "#6E6E6E",
            "#707070", "#727272", "#747474", "#767676", "#787878", "#7A7A7A", "#7C7C7C", "#7E7E7E",
            "#808080", "#828282", "#848484", "#868686", "#888888", "#8A8A8A", "#8C8C8C", "#8E8E8E",
            "#909090", "#929292", "#949494", "#969696", "#989898", "#9A9A9A", "#9C9C9C", "#9E9E9E",
            "#A0A0A0", "#A2A2A2", "#A4A4A4", "#A6A6A6", "#A8A8A8", "#AAAAAA", "#ACACAC", "#AEAEAE",
            "#B0B0B0", "#B2B2B2", "#B4B4B4", "#B6B6B6", "#B8B8B8", "#BABABA", "#BCBCBC", "#BEBEBE",
            "#C0C0C0", "#C2C2C2", "#C4C4C4", "#C6C6C6", "#C8C8C8", "#CACACA", "#CCCECE", "#CECECE",
            "#D0D0D0", "#D2D2D2", "#D4D4D4", "#D6D6D6", "#D8D8D8", "#DADADA", "#DCDBDB", "#DEDEDE",
            "#E0E0E0", "#E2E2E2", "#E4E4E4", "#E6E6E6", "#E8E8E8", "#EAEAEA", "#ECECEC", "#EFEFEF",
            "#F1F1F1", "#F3F3F3", "#F5F5F5", "#F7F7F7", "#F9F9F9", "#FBFBFB", "#FDFDFD", "#FFFFFF"
        ]  # Shades from black to white

        def close_cutscene(cs):
            cs.cutscene.destroy()  # This will destroy the cutscene window
        self.siege.destroy()

        for color in shades:
            cs = Cutscene(main, color)
            main.after(1, close_cutscene, cs)  # Close the cutscene window
            main.update()

        SiegeMenu(self.siege.master, self.userid)

    def battlebutton(self, levelup, playscene):
        levelup(self.userid)
        playscene()

class Battle:
    def __init__(self, main, user_id):
        self.battle = ctk.CTkToplevel(main)
        self.battle.title("Battles")
        self.battle.geometry("400x300")
        self.battle.configure(fg_color="#96c4df")
        self.userid = user_id

        self.label = ctk.CTkLabel(self.battle, text="Battles", fg_color="#96c4df", font=("Helvetica", 20))
        self.label.pack(pady=20)

        self.siegebutton =  ctk.CTkButton(self.battle, text="Siege", fg_color="white", hover_color="#a5cd9d",
                                           text_color="#333333",font=("Arial", 15), command=self.opensiege)
        self.siegebutton.pack(pady=20)

        self.questbutton =  ctk.CTkButton(self.battle, text="Quest", fg_color="white", hover_color="#a5cd9d",
                                           text_color="#333333",font=("Arial", 15), command=self.openquest)
        self.questbutton.pack(pady=20)

        self.back_button =  ctk.CTkButton(self.battle, text="Return to Main Menu", fg_color="white", hover_color="#a5cd9d",text_color="#333333",font=("Arial", 15), command=self.backtomainmenu)
        self.back_button.pack(pady=20)

    def opensiege(self):
        self.battle.destroy()
        SiegeMenu(self.battle.master, self.userid) 

    def openquest(self):
        self.battle.destroy()
        Quest(self.battle.master, self.userid)

    def backtomainmenu(self):
        self.battle.destroy()
        self.mainmenu = MainMenu(self.battle.master, self.userid)
        self.mainmenu.main.deiconify()

class Cutscene:
    def __init__(self, main, color):
        self.cutscene = ctk.CTkToplevel(main)
        self.cutscene.title(".")

        self.cutscene.geometry("800x600")
        self.cutscene.configure(fg_color=color)



class Quest:
    def __init__(self, main, user_id):
        self.quest = ctk.CTkToplevel(main)
        self.quest.title("Quest")
        self.quest.geometry("400x300")
        self.quest.configure(fg_color="#96c4df")
        self.userid = user_id

        self.label = ctk.CTkLabel(self.quest, text="Quest", fg_color="#96c4df", text_color="#333333", font=("Helvetica", 20))
        self.label.pack(pady=20)

        self.startquestbutton = ctk.CTkButton(self.quest, text="Start Quest", fg_color="white", hover_color="#a5cd9d", text_color="#333333", font=("Helvetica", 20), command=self.startquest)
        self.startquestbutton.pack(pady=20)

        self.returnbutton = ctk.CTkButton(self.quest, text="Return to Battle Menu", fg_color="white", hover_color="#a5cd9d", text_color="#333333", font=("Helvetica", 20), command=self.backtobattlemenu)
        self.returnbutton.pack(pady=20)

    def startquest(self):
        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()

        # Fetch user level, element, and chealth
        cursor.execute('SELECT level, celement, chealth FROM users WHERE id = ?', (self.userid,))
        result = cursor.fetchone()

        if result:
            level, element, chealth = result
            level = int(level) if level else 0
            element = element if element else "None"
            chealth = int(chealth) if chealth else 100

            # Calculate new health based on level, chealth, and element
            newhealth = calculatehealth(level, chealth, element)

            # Update user's health in the database
            cursor.execute('UPDATE users SET currenthealth = ? WHERE id = ?', (newhealth, self.userid))
            conn.commit()

            # Fetch updated health
            cursor.execute('SELECT currenthealth FROM users WHERE id = ?', (self.userid,))
            userhealth = cursor.fetchone()[0]
            userhealth = int(userhealth) if userhealth else 100

            enemies = self.createenemies(level)
            self.quest.withdraw()
            Battles(self.quest, self.userid, userhealth, enemies)

        conn.close()

    def createenemies(self, level):
        elements = ["Fire", "Water", "Earth", "Wind", "Electric", "Ice", "Light", "Dark"]
        numenemies = level // 2 + 1
        enemies = []
        for i in range(numenemies):
            enemyhealth = level * 10 + random.randint(0, level * 5)
            enemydamage = level * 2 + random.randint(0, level * 2)
            enemyelement = random.choice(elements)
            enemies.append({"health": enemyhealth, "max_health": enemyhealth, "damage": enemydamage, "element": enemyelement})
        return enemies

    def backtobattlemenu(self):
        self.quest.destroy()
        Battle(self.quest.master, self.userid)


class Battles:
    def __init__(self, questwindow, user_id, userhealth, enemies):
        self.battles = ctk.CTkToplevel(questwindow)
        self.battles.title("Battle")
        self.battles.geometry("600x400")
        self.battles.configure(fg_color="#96c4df")
        self.userid = user_id
        self.userhealth = userhealth
        self.usermaxhealth = userhealth
        self.enemies = enemies
        self.currentenemy = 0
        self.questwindow = questwindow  # Store reference to quest window

        self.label = ctk.CTkLabel(self.battles, text="Battle", fg_color="#96c4df", text_color="#333333", font=("Helvetica", 20))
        self.label.pack(pady=20)

        self.userhealthlabel = ctk.CTkLabel(self.battles, text=f"Your Health: {self.userhealth}/{self.usermaxhealth}", fg_color="#96c4df", text_color="#333333", font=("Helvetica", 15))
        self.userhealthlabel.pack(pady=10)

        self.enemyhealthlabels = []
        
        for i in range(3):
            label = ctk.CTkLabel(self.battles, text="", fg_color="#96c4df", text_color="#333333", font=("Helvetica", 15))
            label.pack(pady=10)
            self.enemyhealthlabels.append(label)

        self.updatehealthlabels()

        self.attackbutton = ctk.CTkButton(self.battles, text="Attack", fg_color="white", hover_color="#a5cd9d", text_color="#333333", font=("Helvetica", 20), command=self.attack)
        self.attackbutton.pack(pady=20)

        self.returnbutton = ctk.CTkButton(self.battles, text="Return to Quest", fg_color="white", hover_color="#a5cd9d", text_color="#333333", font=("Helvetica", 20), command=self.returntoquest)
        self.returnbutton.pack(pady=20)

        self.abilitycooldown = 30  # 30 second cooldown
        self.abilityready = True
        self.currentcooldown = 0

        # Add ability button and timer label
        self.abilitybutton = ctk.CTkButton(self.battles, text="Use Ability",fg_color="purple", hover_color="#9932CC",text_color="white", font=("Helvetica", 20), command=self.useability,state="normal")
        self.abilitybutton.pack(pady=10)

        self.timerlabel = ctk.CTkLabel(self.battles, text="Ability Ready!", fg_color="#96c4df",text_color="#333333", font=("Helvetica", 15))
        self.timerlabel.pack(pady=5)

        # Start the timer update
        self.updatetimer()

    def updatehealthlabels(self):
        self.userhealthlabel.configure(text=f"Your Health: {self.userhealth}/{self.usermaxhealth}")
        for i, label in enumerate(self.enemyhealthlabels):
            if i < len(self.enemies):
                enemy = self.enemies[i]
                label.configure(text=f"Enemy {i+1} Health: {enemy['health']}/{enemy['max_health']}")
            else:
                label.configure(text="No more enemies")

    def updatetimer(self):
        if not self.abilityready:
            self.currentcooldown -= 0.5
            if self.currentcooldown <= 0:
                self.abilityready = True
                self.abilitybutton.configure(state="normal", fg_color="purple")
                self.timerlabel.configure(text="Ability Ready!")
            else:
                self.timerlabel.configure(text=f"Cooldown: {self.currentcooldown:.1f}s")

        # next update in 500ms
        self.battles.after(500, self.updatetimer)

    def useability(self):
        if not self.abilityready or self.currentenemy >= len(self.enemies):
            return

        enemy = self.enemies[self.currentenemy]
        if self.userhealth <= 0:
            messagebox.showerror("Defeat", "You lost the battle. Try again!")
            self.battles.destroy()  
            self.questwindow.deiconify()  
            return

        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()
        cursor.execute('SELECT cstrenght, cweapondamage, celement FROM users WHERE id = ?', (self.userid,))
        result = cursor.fetchone()
        conn.close()

        if result:
            userstrength, userweapondamage, userelement = map(str, result)
            userstrength = int(userstrength)
            userweapondamage = int(userweapondamage)
            basedamage = userstrength * userweapondamage

            # Apply ability multiplier
            abilitydamage = calculateabilitydamage(basedamage)

            # Apply element bonus
            elementbonus = getelementbonus(userelement, enemy["element"])
            finaldamage = int(abilitydamage * elementbonus)

            # Deal damage to enemy
            enemy["health"] -= finaldamage
            if enemy["health"] <= 0:
                enemy["health"] = 0
                self.currentenemy += 1

                if self.currentenemy >= 3:
                    self.levelupnext(self.userid)
                    return

            # Enemy's counter-attack
            if enemy["health"] > 0:
                self.userhealth -= enemy["damage"]
                if self.userhealth <= 0:
                    self.userhealth = 0
                    messagebox.showerror("Defeat", "You lost the battle!")
                    self.battles.destroy()  # Close battle screen
                    self.questwindow.deiconify()  # Show previous window
                    return

            # Update health displays
            self.updatehealthlabels()

            # Start cooldown
            self.abilityready = False
            self.currentcooldown = self.abilitycooldown
            self.abilitybutton.configure(state="disabled", fg_color="gray")
            messagebox.showinfo("Ability Used!", f"Dealt {finaldamage} damage with ability!")

    def attack(self):
        if self.currentenemy >= len(self.enemies):
            self.levelupnext(self.userid)
            return

        enemy = self.enemies[self.currentenemy]
        if self.userhealth <= 0:
            messagebox.showerror("Defeat", "You lost the battle. Try again!")
            self.battles.destroy()  # Close battle screen
            self.questwindow.deiconify()  # Show previous window
            return

        self.battle(enemy)

    def battle(self, enemy):
        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()

        cursor.execute('SELECT cstrenght, cweapondamage, celement FROM users WHERE id = ?', (self.userid,))
        result = cursor.fetchone()
        conn.close()

        if result:
            userstrength, userweapondamage, userelement = map(str, result)
            userstrength = int(userstrength)
            userweapondamage = int(userweapondamage)
            userdamage = userstrength * userweapondamage

            elementbonus = getelementbonus(userelement, enemy["element"])
            userdamage = int(userdamage * elementbonus)

            # User's turn to attack
            enemy["health"] -= userdamage
            if enemy["health"] <= 0:
                enemy["health"] = 0
                self.currentenemy += 1
                self.updatehealthlabels()

                if self.currentenemy >= 3:
                    self.levelupnext(self.userid)
                    return

            # Enemy's turn to attack only if they're alive
            if enemy["health"] > 0:
                self.userhealth -= enemy["damage"]
                if self.userhealth <= 0:
                    self.userhealth = 0
                    messagebox.showerror("Defeat", "You lost the battle!")
                    self.battles.destroy()  # Close battle screen
                    self.questwindow.deiconify()  # Show previous window
                    return

            self.updatehealthlabels()

    def levelupnext(self, user_id):
        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()

        # Get current level and increment
        cursor.execute('SELECT level, celement, chealth FROM users WHERE id = ?', (user_id,))
        result = cursor.fetchone()
        if result:
            currentlevel, element, base_health = result
            newlevel = (currentlevel or 0) + 1

            # Calculate new health with level bonus
            newhealth = calculatehealth(newlevel, int(base_health) if base_health else 100, element)

            # Update level and health
            cursor.execute('UPDATE users SET level = ?, currenthealth = ? WHERE id = ?', 
                         (newlevel, newhealth, user_id))
            conn.commit()

            # Update battle screen
            self.userhealth = newhealth
            self.usermaxhealth = newhealth

            # Generate new enemies for next round
            self.enemies = self.createenemies(newlevel)
            self.currentenemy = 0

            self.updatehealthlabels()

            messagebox.showinfo("Level Up!", 
                              f"Congratulations! You defeated all enemies!\n"
                              f"Level increased to {newlevel}\n"
                              f"Health increased to {newhealth}")

        conn.close()

    def createenemies(self, level):
        elements = ["Fire", "Water", "Earth", "Wind", "Electric", "Ice", "Light", "Dark"]
        enemies = []
        for i in range(3):  # Always create 3 enemies
            base_health = level * 10
            health_variance = random.randint(0, level * 5)
            enemyhealth = base_health + health_variance
            enemydamage = max(1, level * 2 + random.randint(0, level))
            enemyelement = random.choice(elements)
            enemies.append({
                "health": enemyhealth,
                "max_health": enemyhealth,
                "damage": enemydamage,
                "element": enemyelement
            })
        return enemies

    def returntoquest(self):
        self.battles.destroy()
        self.questwindow.deiconify()  





# Create the main main window
main = ctk.CTk()

user_id = None
entrance = Entrance(main, user_id)

logindata_db()
check_db_data()

main.mainloop()


