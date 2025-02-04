import tkinter as tk
from tkinter import messagebox
import random
import sqlite3




def conversion(password):
    hashtable = []
    for letters in password:
        hashtable.append(ord(letters))
    hashed = ''.join(map(str, hashtable))  # This creates a string of concatenated ASCII codes
    return hashed  # Return as stringhash value instead of the function itself

# Create or connect to the database
def logindata_db():
    conn = sqlite3.connect('user_login.db')
    cursor = conn.cursor()

    # Create users table
    cursor.execute('''
    CREATE TABLE IF NOT EXISTS users (
        user_id INTEGER PRIMARY KEY AUTOINCREMENT,
        username TEXT NOT NULL UNIQUE,
        password TEXT NOT NULL,
        chero TEXT,
        celement TEXT,
        hrarity TEXT,
        cweapon TEXT,
        cweapontype TEXT,
        wrarity TEXT
    )
    ''')

    conn.commit()
    conn.close()




    

def set_chero(hero_name):
    conn = sqlite3.connect('user_login.db')
    cursor = conn.cursor()
    cursor.execute('UPDATE users SET chero = ? WHERE user_id = ?', (hero_name,))
    conn.commit()
    conn.close()

def set_celement(element_name):
    conn = sqlite3.connect('user_login.db')
    cursor = conn.cursor()
    cursor.execute('UPDATE users SET celement = ? WHERE user_id = ?', (element_name,))
    conn.commit()
    conn.close()

def set_hrarity(hero_rarity):
    conn = sqlite3.connect('user_login.db')
    cursor = conn.cursor()
    cursor.execute('UPDATE users SET hrarity = ? WHERE user_id = ?', (hero_rarity,))
    conn.commit()
    conn.close()

def set_cweapon(weapon_name):
    conn = sqlite3.connect('user_login.db')
    cursor = conn.cursor()
    cursor.execute('UPDATE users SET cweapon = ? WHERE user_id = ?', (weapon_name,))
    conn.commit()
    conn.close()

def set_cweapontype(weapon_type):
    conn = sqlite3.connect('user_login.db')
    cursor = conn.cursor()
    cursor.execute('UPDATE users SET cweapontype = ? WHERE user_id = ?', (weapon_type,))
    conn.commit()
    conn.close()

def set_wrarity(weapon_rarity):
    conn = sqlite3.connect('user_login.db')
    cursor = conn.cursor()
    cursor.execute('UPDATE users SET wrarity = ? WHERE user_id = ?', (weapon_rarity,))
    conn.commit()
    conn.close()


def get_csummon(self, user_id):
    try:
        # Connect to the database
        conn = sqlite3.connect('user_login.db')

        # Allow for dictionary-like row access
        conn.row_factory = sqlite3.Row

        # Create a cursor and execute the query
        cursor = conn.cursor()
        cursor.execute('SELECT chero, celement, hrarity, cweapon, cweapontype, wrarity FROM users WHERE user_id = ?', (user_id,))

        # Fetch the result
        result = cursor.fetchone()

        # Close the connection
        conn.close()

        # Return the result or None if not found
        return dict(result) if result else None

    except sqlite3.Error as e:
        # Handle database connection or query errors
        print(f"Database error: {e}")
        return None

def check_summons():
    conn = sqlite3.connect('user_login.db')
    cursor = conn.cursor()
    cursor.execute('SELECT * FROM users')
    data = cursor.fetchall()
    print("Current data in database:")
    for row in data:
        print(row)
    conn.close()





class Entrance:
    def __init__(self, main,user_id):
       self.entrance = (main)
       self.entrance.title("Entrance")
       self.entrance.geometry("400x600")
       self.entrance.configure(bg="#96c4df")
       self.user_id = user_id


       self.label = tk.Label(self.entrance, text="Entrance", bg="#96c4df", font=("Helvetica", 20))
       self.label.pack(pady=20)

       self.openloginb = tk.Button(self.entrance, text="Login", bg="white", activebackground="#a5cd9d",font=("Arial", 15), command=lambda :self.openloginmenu)
       self.openloginb.pack(pady=20)

       self.openentrance = tk.Button(self.entrance, text="Register", bg="white",activebackground="#a5cd9d", font=("Arial", 15),command=lambda :self.openregistermenu)
       self.openentrance.pack(pady=20)

    def openloginmenu(self,user_id):
       self.login = LoginMenu(main,user_id)
       self.entrance.withdraw()

    def openregistermenu(self,user_id):
       self.registermenu = RegisterMenu(main,user_id)
       self.entrance.withdraw()

class LoginMenu:
    def __init__(self, main,user_id):
       self.login = tk.Toplevel(main)
       self.login.title("Login")
       self.login.configure(bg="#96c4df")
       self.login.geometry("800x400")
       self.user_id = user_id

       self.label = tk.Label(self.login, text="Login", bg="#96c4df", font=("Helvetica", 20))
       self.label.pack(pady=20)

       self.userl = tk.Label(self.login, text="Username", bg="#96c4df", font=("Arial", 15))
       self.userl.place(x=100, y=100)

       self.passl = tk.Label(self.login, text="Password", bg="#96c4df", font=("Arial", 15))
       self.passl.place(x=100, y=150)

       self.usere = tk.Entry(self.login, font=("Arial", 15))
       self.usere.place(x=240, y=100)

       self.passe = tk.Entry(self.login, font=("Arial", 15), show="*")
       self.passe.place(x=240, y=150)

       self.loginb = tk.Button(self.login, text="Login", bg="white", font=("Arial", 15),command=lambda :self.checklogin)
       self.loginb.place(x=240, y=200)

       self.back = tk.Button(self.login, text="Return to entrance page", bg="white", activebackground="#a5cd9d",font=("Arial", 15), command=lambda :self.backtoentrance)
       self.back.pack(pady=20, padx=20)
       self.back.place(x=265, y=350)

    def checklogin(self):
        username = self.usere.get()
        password = self.passe.get()

        # Connect to the database to verify the login credentials
        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()
        cursor.execute('SELECT user_id, password FROM users WHERE username = ?', (username,))
        row = cursor.fetchone()

        if row is not None:
            user_id,stored_hashed_value = row  # Get user user_id and stored hashed password from the database
            hashed_password = conversion(password)  # Convert entered password into a hashed format

            # Debugging outputs to help trace the issue

            print(f"Username: {username}")
            print(f"Hashed Password: '{hashed_password}'")

            # Check if the entered password matches the stored password
            if stored_hashed_value == hashed_password:
                self.openmainmenu(user_id)  # Proceed to main menu if the password matches
                print("Login successful")
            else:
                messagebox.showerror("Incorrect password. Please try again.")
                print("Password incorrect")
        else:
            messagebox.showerror("Username not found.")
            print("Username not found")

        conn.close()


    def openmainmenu(self,user_id):
            self.mainmenu = MainMenu(main,user_id)
            self.login.withdraw() 


    def backtoentrance(self):
       self.login.destroy()
       main.deiconify()

class RegisterMenu:
    def __init__(self, main,user_id):
       self.register = tk.Toplevel(main)
       self.register.title("Register")
       self.register.configure(bg="#96c4df")
       self.register.geometry("800x400")
       self.user_id = user_id


       self.label = tk.Label(self.register, text="Register Account", bg="#96c4df", font=("Helvetica", 20))
       self.label.pack(pady=20)

       self.userl = tk.Label(self.register, text="Username", bg="#96c4df", font=("Arial", 15))
       self.userl.place(x=100, y=100)

       self.passl = tk.Label(self.register, text="Password", bg="#96c4df", font=("Arial", 15))
       self.passl.place(x=100, y=150)

       self.usere = tk.Entry(self.register, font=("Arial", 15))
       self.usere.place(x=240, y=100)

       self.passe = tk.Entry(self.register, font=("Arial", 15), show="*")
       self.passe.place(x=240, y=150)

       self.registerb = tk.Button(self.register, text="Create account", bg="white", font=("Arial", 15),command=lambda :self.createaccount)
       self.registerb.place(x=240, y=200)

       self.back = tk.Button(self.register, text="Return to entrance page", bg="white",activebackground="#a5cd9d", font=("Arial", 15), command=lambda :self.backtoentrance)
       self.back.pack(pady=20, padx=20)
       self.back.place(x=265, y=350)

    def openmainmenu(self,user_id):
       self.mainmenu = MainMenu(main,user_id)
       self.register.withdraw()

    def backtoentrance(self):
       self.register.destroy()
       main.deiconify()

    def createaccount(self,user_id):
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
                hashed_password = conversion(password)  # Hash the password before storing
                cursor.execute('INSERT INTO users (username, password) VALUES (?, ?)', (username, hashed_password))
                conn.commit()
                messagebox.showinfo("Account created successfully!")
                print(f"New account created: Username is '{username}' and Hashed password is '{hashed_password}'") 

                self.openmainmenu(user_id)
        except sqlite3.Error as e:
            messagebox.showerror("Database Error", f"An error occurred: {str(e)}")
        finally:
            conn.close()

class MainMenu:
    def __init__(self, main,user_id):
       self.main = tk.Toplevel(main)
       self.main.title("Main Menu")
       self.main.geometry("400x200")
       self.main.configure(bg="#96c4df")
       self.user_id = user_id

        
       self.label = tk.Label(self.main, text="Main Menu", bg="#96c4df", font=("Arial", 20))
       self.label.pack(pady=10)

       self.quest = tk.Button(self.main, text="Quest", bg="white", activebackground="#a5cd9d", font=("Arial", 15),command=lambda :self.openquest)
       self.quest.pack(pady=20)
       self.quest.place(x=50, y=50)

       self.summon = tk.Button(self.main, text="Summon", bg="white", activebackground="#a5cd9d", font=("Arial", 15),command=lambda :self.opensummon)
       self.summon.pack(pady=20)
       self.summon.place(x=220, y=50)

       self.settings = tk.Button(self.main, text="Settings", bg="white", activebackground="#a5cd9d",font=("Arial", 15), command=lambda :self.opensettings)
       self.settings.pack(pady=20)
       self.settings.place(x=50, y=140)

       self.user = tk.Button(self.main, text="User", bg="white", activebackground="#a5cd9d", font=("Arial", 15),command=lambda :self.openusers)
       self.user.pack(pady=20)
       self.user.place(x=220, y=140)

    def openquest(self,user_id):
       self.main.withdraw()
       self.quest = Quest(self.main,user_id)

    def opensummon(self,user_id):
        self.main.withdraw()  
        self.summon = Summoning(self.main,user_id)


    def opensettings(self,user_id):
       self.main.withdraw()
       self.settings = Settings(self.main,user_id)

    def openusers(self,user_id):
       self.main.withdraw()
       self.users = Users(self.main,user_id)

class Quest:
    def __init__(self, main,user_id):
       self.quest = tk.Toplevel(main)
       self.quest.title("Quests")
       self.quest.geometry("400x200")
       self.quest.configure(bg="#96c4df")
       self.user_id = user_id   


       self.label = tk.Label(self.quest, text="Quests", bg="#96c4df", font=("Helvetica", 20))
       self.label.pack(pady=20)

       self.back = tk.Button(self.quest, text="Return to main menu", bg="white", activebackground="#a5cd9d",font=("Arial", 15), command=lambda :lambda: self.backtomainmenu(user_id))
       self.back.pack(pady=20)

    def backtomainmenu(self,user_id):
       self.quest.destroy()
       self.mainmenu = MainMenu(main,user_id)
       self.mainmenu.main.deiconify()

class Summoning:

    hero = [
        "Blaze", "Infernia", "Ember", "Ignis", "Vulcan",  # Fire
        "Marina", "Tuser_idal", "Cascade", "Aquaria", "Sirena",  # Water
        "Terra", "Boulder", "Gaia", "Quarrix", "Petrus",  # Earth
        "Zephyr", "Cyclone", "Aeris", "Ventra", "Skylar",  # Wind
        "Volt", "Spark", "Thunderra", "Zappia", "Storme",  # Electric
        "Frost", "Glaciel", "Chilla", "Cryonix", "Shivera",  # Ice
        "Lumina", "Radiant", "Solara", "Aethera", "Halo",  # Light
        "Umbra", "Nyx", "Shadowe", "Noctis", "Eclipse"  # Dark
    ]
    heroprob = [1, 1, 20, 10, 5,  # Fire
                20, 20, 5, 10, 1,  # Water
                20, 20, 1, 10, 5,  # Earth
                1, 20, 5, 20, 10,  # Wind
                5, 10, 1, 20, 20,  # Electric
                20, 10, 20, 1, 5,  # Ice
                10, 20, 5, 20, 1,  # Light
                20, 5, 20, 10, 1]  # Dark
    heroelements = {
    "Blaze": "Fire", "Infernia": "Fire", "Ember": "Fire", "Ignis": "Fire", "Vulcan": "Fire",
    "Marina": "Water", "Tuser_idal": "Water", "Cascade": "Water", "Aquaria": "Water", "Sirena": "Water",
    "Terra": "Earth", "Boulder": "Earth", "Gaia": "Earth", "Quarrix": "Earth", "Petrus": "Earth",
    "Zephyr": "Wind", "Cyclone": "Wind", "Aeris": "Wind", "Ventra": "Wind", "Skylar": "Wind",
    "Volt": "Electric", "Spark": "Electric", "Thunderra": "Electric", "Zappia": "Electric","Storme": "Electric",
    "Frost": "Ice", "Glaciel": "Ice", "Chilla": "Ice", "Cryonix": "Ice", "Shivera": "Ice",
    "Lumina": "Light", "Radiant": "Light", "Solara": "Light", "Aethera": "Light", "Halo": "Light",
    "Umbra": "Dark", "Nyx": "Dark", "Shadowe": "Dark", "Noctis": "Dark", "Eclipse": "Dark"
}
    weapons = [
           "Excalibur", "Flametongue", "Blade of Inferno", "Molten Edge", "Lava Cleaver",  # Sword
           "Aqua Harpoon", "Poseuser_idon’s Spear", "Coral Lance", "Tuser_idal Pike", "Marine Javelin",  # Spear
           "Rock Crusher", "Earthsplitter", "Terra Mace", "Stone Maul", "Boulder Hammer",  # Axe
           "Zephyr Bow", "Storm Bow", "Gale Shooter", "Tempest Quiver", "Windrunner Bow",  # Bow
           "Lightning Rod", "Thunder Staff", "Volt Wand", "Storm Orb", "Electro Scepter",  # Staff
           "Frost Fang", "Ice Fang", "Glacier Blade", "Cryo Dagger", "Shiver Knife",  # Dagger
           "Radiant Shield", "Halo Guard", "Solar    Defender", "Light Barrier", "Aether Buckler",  # Shield
           "Umbra Scythe", "Nyx Reaper", "Shadow Blade", "Noctis Saber", "Eclipse severer"  # Scythe
       ]

    weaponprob = [1, 20, 5, 10, 20,  # Sword
                  20, 1, 10, 5, 20,  # Spear
                  20, 1, 5, 20, 10,  # Axe
                  1, 5, 20, 10, 20,  # Bow
                  20, 20, 10, 5, 1,  # Staff
                  20, 20, 10, 1, 5,  # Dagger
                  20, 1, 5, 10, 20,  # Shield
                  20, 5, 10, 20, 1]  # Scythe
    weapontypes = {
    "Excalibur": "Sword", "Flametongue": "Sword", "Blade of Inferno": "Sword", "Molten Edge": "Sword","Lava Cleaver": "Sword",
    "Aqua Harpoon": "Spear", "Poseuser_idon’s Spear": "Spear", "Coral Lance": "Spear", "Tuser_idal Pike": "Spear","Marine Javelin": "Spear",
    "Rock Crusher": "Axe", "Earthsplitter": "Axe", "Terra Mace": "Axe", "Stone Maul": "Axe","Boulder Hammer": "Axe",
    "Zephyr Bow": "Bow", "Storm Bow": "Bow", "Gale Shooter": "Bow", "Tempest Quiver": "Bow","Windrunner Bow": "Bow",
    "Lightning Rod": "Staff", "Thunder Staff": "Staff", "Volt Wand": "Staff", "Storm Orb": "Staff","Electro Scepter": "Staff",
    "Frost Fang": "Dagger", "Ice Fang": "Dagger", "Glacier Blade": "Dagger", "Cryo Dagger": "Dagger","Shiver Knife": "Dagger",
    "Radiant Shield": "Shield", "Halo Guard": "Shield", "Solar    Defender": "Shield", "Light Barrier": "Shield","Aether Buckler": "Shield",
    "Umbra Scythe": "Scythe", "Nyx Reaper": "Scythe", "Shadow Blade": "Scythe", "Noctis Saber": "Scythe","Eclipse Cutter":"Scythe"
}
    def __init__(self, main,user_id):
       self.summon = tk.Toplevel(main)
       self.summon.title("Summon")
       self.summon.geometry("400x400")
       self.summon.configure(bg="#96c4df")
       self.user_id = user_id
       self.update_display(user_id)
 


       self.label = tk.Label(self.summon, text="Summoning Altar", bg="#96c4df", font=("Helvetica", 20))
       self.label.pack(pady=20)

       self.herob = tk.Button(self.summon, text="Summon Hero", bg="white", activebackground="#a5cd9d",font=("Arial", 15), command=lambda :self.summonhero)
       self.herob.pack(pady=10)

       self.weaponb = tk.Button(self.summon, text="Summon Weapon", bg="white", activebackground="#a5cd9d",font=("Arial", 15), command=lambda :self.summonweapon)
       self.weaponb.pack(pady=10)

       self.heroresult = tk.Label(self.summon, text="", bg="#96c4df", font=("Arial", 15))
       self.heroresult.pack(pady=10)

       self.weaponresult = tk.Label(self.summon, text="", bg="#96c4df", font=("Arial", 15))
       self.weaponresult.pack(pady=10)

       self.back = tk.Button(self.summon, text="Return to main menu", bg="white", activebackground="#a5cd9d",font=("Arial", 15), command=lambda :self.backtomainmenu)
       self.back.pack(pady=20)

       self.current_hero_label = tk.Label(self.summon, text='', bg="#96c4df", font=("Arial", 15))
       self.current_hero_label.pack(pady=5)

       self.current_weapon_label = tk.Label(self.summon, text='', bg="#96c4df", font=("Arial", 15))
       self.current_weapon_label.pack(pady=5)
       self.update_display(user_id)

       self.hero = [
           "Blaze", "Infernia", "Ember", "Ignis", "Vulcan",  # Fire
           "Marina", "Tuser_idal", "Cascade", "Aquaria", "Sirena",  # Water
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
           "Marina": "Water", "Tuser_idal": "Water", "Cascade": "Water", "Aquaria": "Water", "Sirena": "Water",
           "Terra": "Earth", "Boulder": "Earth", "Gaia": "Earth", "Quarrix": "Earth", "Petrus": "Earth",
           "Zephyr": "Wind", "Cyclone": "Wind", "Aeris": "Wind", "Ventra": "Wind", "Skylar": "Wind",
           "Volt": "Electric", "Spark": "Electric", "Thunderra": "Electric", "Zappia": "Electric","Storme": "Electric",
           "Frost": "Ice", "Glaciel": "Ice", "Chilla": "Ice", "Cryonix": "Ice", "Shivera": "Ice",
           "Lumina": "Light", "Radiant": "Light", "Solara": "Light", "Aethera": "Light", "Halo": "Light",
           "Umbra": "Dark", "Nyx": "Dark", "Shadowe": "Dark", "Noctis": "Dark", "Eclipse": "Dark"
       }

       self.weapons = [
           "Excalibur", "Flametongue", "Blade of Inferno", "Molten Edge", "Lava Cleaver",  # Sword
           "Aqua Harpoon", "Poseuser_idon’s Spear", "Coral Lance", "Tuser_idal Pike", "Marine Javelin",  # Spear
           "Rock Crusher", "Earthsplitter", "Terra Mace", "Stone Maul", "Boulder Hammer",  # Axe
           "Zephyr Bow", "Storm Bow", "Gale Shooter", "Tempest Quiver", "Windrunner Bow",  # Bow
           "Lightning Rod", "Thunder Staff", "Volt Wand", "Storm Orb", "Electro Scepter",  # Staff
           "Frost Fang", "Ice Fang", "Glacier Blade", "Cryo Dagger", "Shiver Knife",  # Dagger
           "Radiant Shield", "Halo Guard", "Solar    Defender", "Light Barrier", "Aether Buckler",  # Shield
           "Umbra Scythe", "Nyx Reaper", "Shadow Blade", "Noctis Saber", "Eclipse severer"  # Scythe
       ]

       self.weaponprob = [1, 20, 5, 10, 20,  # Sword
                           20, 1, 10, 5, 20,  # Spear
                           20, 1, 5, 20, 10,  # Axe
                           1, 5, 20, 10, 20,  # Bow
                           20, 20, 10, 5, 1,  # Staff
                           20, 20, 10, 1, 5,  # Dagger
                           20, 1, 5, 10, 20,  # Shield
                           20, 5, 10, 20, 1]  # Scythe

       # weapon info order is name,rarity,weapond type, level,dmg,ability(uniques only)
       self.weaponinfo = {
           # Swords
           "Flametongue": ["Flametongue", "Common", "Sword", "0", "25", "None"],
           "Lava Cleaver": ["Lava Cleaver", "Common", "Sword", "0", "25", "None"],
           "Molten Edge": ["Molten Edge", "Rare", "Sword", "0", "50", "None"],
           "Blade of Inferno": ["Blade of Inferno", "Epic", "Sword", "0", "75", "None"],
           "Excalibur": ["Excalibur", "Unique", "Sword", "0", "100", "Flame Sever"],

           # Spears
           "Aqua Harpoon": ["Aqua Harpoon", "Common", "Spear", "0", "25", "None"],
           "Coral Lance": ["Coral Lance", "Common", "Spear", "0", "25", "None"],
           "Tuser_idal Pike": ["Tuser_idal Pike", "Rare", "Spear", "0", "50", "None"],
           "Poseuser_idon's Spear": ["Poseuser_idon's Spear", "Epic", "Spear", "0", "75", "None"],
           "Marine Javelin": ["Marine Javelin", "Unique", "Spear", "0", "100", "Tuser_idal Fury"],

           # Axes
           "Rock Crusher": ["Rock Crusher", "Common", "Axe", "0", "25", "None"],
           "Stone Maul": ["Stone Maul", "Common", "Axe", "0", "25", "None"],
           "Terra Mace": ["Terra Mace", "Rare", "Axe", "0", "50", "None"],
           "Earthsplitter": ["Earthsplitter", "Epic", "Axe", "0", "75", "None"],
           "Boulder Hammer": ["Boulder Hammer", "Unique", "Axe", "0", "100", "Rockfall"],

           # Bows
           "Gale Shooter": ["Gale Shooter", "Common", "Bow", "0", "25", "None"],
           "Windrunner Bow": ["Windrunner Bow", "Unique", "Bow", "0", "100", "Windstrike"],
           "Storm Bow": ["Storm Bow", "Rare", "Bow", "0", "50", "None"],
           "Tempest Quiver": ["Tempest Quiver", "Rare", "Bow", "0", "50", "None"],
           "Zephyr Bow": ["Zephyr Bow", "Epic", "Bow", "0", "75", "None"],

           # Staffs
           "Volt Wand": ["Volt Wand", "Common", "Staff", "0", "25", "None"],
           "Storm Orb": ["Storm Orb", "Rare", "Staff", "0", "50", "None"],
           "Lightning Rod": ["Lightning Rod", "Rare", "Staff", "0", "50", "None"],
           "Thunder Staff": ["Thunder Staff", "Epic", "Staff", "0", "75", "None"],
           "Electro Scepter": ["Electro Scepter", "Unique", "Staff", "0", "100", "Thunderstorm"],

           # Daggers
           "Glacier Blade": ["Glacier Blade", "Common", "Dagger", "0", "25", "None"],
           "Shiver Knife": ["Shiver Knife", "Unique", "Dagger", "0", "100", "Ice Storm"],
           "Cryo Dagger": ["Cryo Dagger", "Rare", "Dagger", "0", "50", "None"],
           "Ice Fang": ["Ice Fang", "Rare", "Dagger", "0", "50", "None"],
           "Frost Fang": ["Frost Fang", "Epic", "Dagger", "0", "75", "None"],

           # Shields
           "Solar    Defender": ["Solar    Defender", "Common", "Shield", "0", "25", "None"],
           "Light Barrier": ["Light Barrier", "Rare", "Shield", "0", "50", "None"],
           "Halo Guard": ["Halo Guard", "Rare", "Shield", "0", "50", "None"],
           "Radiant Shield": ["Radiant Shield", "Epic", "Shield", "0", "75", "None"],
           "Aether Buckler": ["Aether Buckler", "Unique", "Shield", "0", "100", "Aether Guard"],

           # Scythes
           "Shadow Blade": ["Shadow Blade", "Common", "Scythe", "0", "25", "None"],
           "Noctis Saber": ["Noctis Saber", "Rare", "Scythe", "0", "50", "None"],
           "Nyx Reaper": ["Nyx Reaper", "Rare", "Scythe", "0", "50", "None"],
           "Umbra Scythe": ["Umbra Scythe", "Epic", "Scythe", "0", "75", "None"],
           "Eclipse Severer": ["Eclipse Severer", "Unique", "Scythe", "0", "100", "Shadow Cleaver"]
       }

       self.weapontypes = {
           "Excalibur": "Sword", "Flametongue": "Sword", "Blade of Inferno": "Sword", "Molten Edge": "Sword","Lava Cleaver": "Sword",
           "Aqua Harpoon": "Spear", "Poseuser_idon’s Spear": "Spear", "Coral Lance": "Spear", "Tuser_idal Pike": "Spear","Marine Javelin": "Spear",
           "Rock Crusher": "Axe", "Earthsplitter": "Axe", "Terra Mace": "Axe", "Stone Maul": "Axe","Boulder Hammer": "Axe",
           "Zephyr Bow": "Bow", "Storm Bow": "Bow", "Gale Shooter": "Bow", "Tempest Quiver": "Bow","Windrunner Bow": "Bow",
           "Lightning Rod": "Staff", "Thunder Staff": "Staff", "Volt Wand": "Staff", "Storm Orb": "Staff","Electro Scepter": "Staff",
           "Frost Fang": "Dagger", "Ice Fang": "Dagger", "Glacier Blade": "Dagger", "Cryo Dagger": "Dagger","Shiver Knife": "Dagger",
           "Radiant Shield": "Shield", "Halo Guard": "Shield", "Solar    Defender": "Shield", "Light Barrier": "Shield","Aether Buckler": "Shield",
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


    def update_display(self,user_id):
        result = get_csummon(self,user_id)
        if result:
            print("Summon results found")
            chero, celement, hrarity, cweapon, cweapontype, wrarity = result
            self.heroresult.config(text=f"{chero if chero else 'None'}({celement if celement else ''})", fg=hrarity)
            self.weaponresult.config(text=f"{cweapon if cweapon else 'None'}({cweapontype if cweapontype else ''})", fg=wrarity)
        else:
            # Handle the case where get_csummon() returns None
            print("No summon results found")
            self.heroresult.config(text="None", fg="gray")
            self.weaponresult.config(text="None", fg="gray")

    def summonhero(self,user_id,update_display):
       result = random.choices(self.hero, weights=self.heroprob, k=1)[0]
       element = self.heroelements[result]
       prob = self.heroprob[self.hero.index(result)]
       hrarity = self.rarity(prob)
       self.heroresult.config(text=f" {result} ({element})", fg=hrarity)
       set_chero(result)
       set_celement(element)
       set_hrarity(hrarity)
       self.update_display(user_id)

       if prob == 1:
           print(f"UNIQUE SUMMON: {result}")
       else:
           print(f"Summoned Hero: {result} with probability {prob}")

       if prob == 1:
           self.spinconformation(self.summonhero)

       if isinstance(self.summon, MainMenu,user_id,update_display):
           self.summon.update_display(user_id)

    def summonweapon(self):
       result = random.choices(self.weapons, weights=self.weaponprob, k=1)[0]
       weapontype = self.weapontypes[result]
       prob = self.weaponprob[self.weapons.index(result)]
       wrarity = self.rarity(prob)
       self.weaponresult.config(text=f" {result} ({weapontype})", fg=wrarity)
       set_cweapon(result)
       set_cweapontype(weapontype)
       set_wrarity(wrarity)

       self.update_display()

       if prob == 1:
           print(f"UNIQUE SUMMON: {result}")
       else:
           print(f"Summoned Weapon: {result} with probability {prob}")

       if prob == 1:
           self.spinconformation(self.summonweapon)  

       if isinstance(self.summon, MainMenu):
        self.summon.update_display()



    def spinconformation(self, summonconformation):
       response = messagebox.askyesno("Lucky summon!", "You got a unique summon are you sure you want to spin again")
       if response:
           summonconformation()

    def backtomainmenu(self,user_id):
       self.summon.destroy()
       self.mainmenu = MainMenu(main,user_id)
       self.mainmenu.main.deiconify()

class Settings:
    def __init__(self, main,user_id):
       self.settings = tk.Toplevel(main)
       self.settings.title("Settings")
       self.settings.geometry("400x200")
       self.settings.configure(bg="#96c4df")
       self.user_id = user_id


       self.label = tk.Label(self.settings, text="Settings", bg="#96c4df", font=("Helvetica", 20))
       self.label.pack(pady=20)

       self.back = tk.Button(self.settings, text="Return to main menu", bg="white", activebackground="#a5cd9d",font=("Arial", 15), command=lambda :self.backtomainmenu)
       self.back.pack(pady=20)

    def backtomainmenu(self,user_id):
       self.settings.destroy()
       self.mainmenu = MainMenu(main,user_id)
       self.mainmenu.main.deiconify()

class Users:
    def __init__(self, main,user_id):
       self.users = tk.Toplevel(main)
       self.users.title("User")
       self.users.geometry("400x200")
       self.users.configure(bg="#96c4df")
       self.user_id = user_id
 

       self.label = tk.Label(self.users, text="User Information", bg="#96c4df", font=("Helvetica", 20))
       self.label.pack(pady=20)

       self.back = tk.Button(self.users, text="Return to main menu", bg="white", activebackground="#a5cd9d",font=("Arial", 15), command=lambda :self.backtomainmenu)
       self.back.pack(pady=20)

       self.logout = tk.Button(self.users, text="Log out", bg="white", activebackground="#a5cd9d",font=("Arial", 15), command=lambda :self.backtoentrance)
       self.back.pack(pady=20)
       self.logout.place(x=145, y=150)

    def backtomainmenu(self,user_id):
       self.users.destroy()
       self.mainmenu = MainMenu(main,user_id)
       self.mainmenu.main.deiconify()

    def backtoentrance(self):
       self.users.destroy()
       main.deiconify()

# Create the main main window
main = tk.Tk()
# Opens the login menu
entrance = Entrance(main,user_id)



logindata_db()
check_summons()

main.mainloop()

